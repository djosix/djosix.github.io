---
layout: post
title: Variable Sequence Lengths for PyTorch RNNs
date: 2017-09-05 01:52 +0800
tags:
  - Machine Learning
---

In this post, I will introduce 2 ways to input sequences with different lenghs for PyTorch RNNs (though this is usually not necessary).

When you feed a mini-batch to PyTorch RNN with variable sequence lengths (let's say you want the hidden state at each sequence's end), you need to **pack** it first, RNN in PyTorch requires a `PackedSequence` object instead of a `Tensor` or `Variable`. It also requires that the `PackedSequence` is sorted, which means that you need to pad the sequences to the same size and sort them by their lengths, then use `torch.nn.utils.rnn.pack_padded_sequence()` to create a `PackedSequence` object as an input of the RNN function. After computed by RNN, it outputs a `PackedSequence` as well. To use it, convert it back to a `Variable` or `Tensor` using `torch.nn.utils.rnn.pad_packed_sequence()`.


### Steps

1. Pad the input sequences to the same length
2. Sort them by their lengths (desc order)
3. Use `torch.nn.utils.rnn.pack_padded_sequence()`
4. RNN
5. Use `torch.nn.utils.rnn.pad_packed_sequence()`
6. Unsort output sequences
7. Unpad output sequences

### Method 1: Sort in Module

- Padding in NumPy

  ```python
  # this is much faster than zero-padding
  sequence = np.random.random([seq_len, dim])
  padded = np.resize(sequence, [max_len, dim])
  ```


- Sorting and unsorting in PyTorch

  ```python
  y, i = x.sort()
  # y: sorted tensor
  # i: sorted indeces

  _, r = i.sort()
  # r: sorted indeces of sorted indeces of x (?)

  assert all(y == x[i])
  assert all(y[r] == x)
  ```


- A simple example

  ```python
  import torch

  CUDA = True

  class Encoder(nn.Module):
      def __init__(self):
          super(Encoder, self).__init__()
          self.gru = torch.nn.GRU(512, 256, 2,
                                  dropout=0.1,
                                  bidirectional=True)

      def forward(self, x, l):
          """ args
          x: padded sequences [batch, step, dim]
          l: sequence lengths [n_sequences]
          """
          x, r = self.pack(x, l)
          x, h = self.gru(x, None)
          x = self.unpack(x, r)

          x = x[:, 256:] + x[:, :256] # sum up outputs
          x = (x + 1) / 2
          return x

      def pack(self, x, l):
          l = torch.LongTensor(l)
          l = l.cuda() if CUDA else l
          l, i = l.sort(0, True)
          r = i.sort()[1]             # reverse indeces
          x = torch.nn.utils.rnn.pack_padded_sequence(
                  x[i], l.tolist(), batch_first=True)
          return x, r

      def unpack(self, x, r):
          x, l = torch.nn.utils.rnn.pad_packed_sequence(x)
          f = lambda x: (x[0], x[1] - 1)
          i, j = zip(*map(f, enumerate(l)))
          x = x[j, i][r]              # unsort sequences (and get the last output)
          return x
  ```

### Mothod 2: Sort Before Forwarding

- Let's define some modules.

  ```python
  import torch
  from torch import nn
  from torch.nn import functional as F
  from torch.autograd import Variable
  from torch.nn.utils.rnn import pack_padded_sequence, pad_packed_sequence

  class Encoder(nn.Module):
      def __init__(self, input_size, hidden_size, rnn_layers=1):
          super(Encoder, self).__init__()
          self.rnn = nn.GRU(input_size, hidden_size, rnn_layers)
      
      def forward(self, seqs, seq_lens):
          packed = pack_padded_sequence(seqs, seq_lens)
          outputs, hidden = self.rnn(packed, None)
          return hidden

  class Decoder(nn.Module):
      def __init__(self, input_size, hidden_size, output_size, rnn_layers=1):
          super(Decoder, self).__init__()
          self.rnn = nn.GRU(input_size, hidden_size, rnn_layers)
          self.out = nn.Linear(hidden_size, output_size)
      
      def forward(self, decoder_input, hidden, steps):
          # decoder_input: [1, batch_size, input_size]
          
          rnn_input = decoder_input
          rnn_outputs = []
          for _ in range(steps):
              rnn_output, hidden = self.rnn(rnn_input, hidden)
              # rnn_output: [1, batch_size, hidden_size]

              rnn_outputs.append(rnn_output)
              rnn_input = rnn_output

          rnn_outputs = torch.cat(rnn_outputs, 0)
          # rnn_outputs: [steps, batch_size, hidden_size]

          outputs = self.out(rnn_outputs)
          # outputs: [steps, batch_size, output_size]
          
          probs = F.softmax(outputs, -1)
          # probs: [steps, batch_size, output_size]

          return probs

  ```

- Sort the batch first, including inputs and targets, then forward it.

  ```python
  import helper, itertools

  CUDA = torch.cuda.is_available()
  DICT_SIZE = 2048
  HIDDEN_SIZE = 128
  MAX_STEPS = 15
  BOS, EOS, PAD = [0, 1, 2]

  # Let's say inputs are unsorted
  # inputs: a list of input word indeces
  # targets: a list of target word indeces
  inputs, targets = helper.generate_batch(64)
  max_input_steps = max(len(seq) for seq in inputs)

  # Generate a list of (input, target)
  batch = list(zip(inputs, targets))
  batch_size = len(batch)

  # Sort the batch by len(input)
  sort_key = lambda pair: len(pair[0])
  batch.sort(key=sort_key, reverse=True)

  # Now inputs and targets are sorted
  inputs, targets = zip(*batch)

  input_lens = [len(seq) for seq in inputs]
  target_lens = [len(seq) for seq in targets]

  # Let's do padding, after this the shape will be step-first
  inputs = itertools.zip_longest(*inputs, fillvalue=PAD)
  targets = itertools.zip_longest(*targets, fillvalue=PAD)

  inputs = Variable(torch.LongTensor(inputs))
  targets = Variable(torch.LongTensor(targets))

  inputs = inputs.cuda() if CUDA else inputs
  targets = targets.cuda() if CUDA else targets

  embedding = nn.Embedding(DICT_SIZE, HIDDEN_SIZE)
  encoder = Encoder(HIDDEN_SIZE, HIDDEN_SIZE)
  decoder = Decoder(HIDDEN_SIZE, HIDDEN_SIZE, DICT_SIZE)

  embedded = embedding(inputs)
  hidden = encoder(embedded, input_lens)

  decoder_inputs = [[BOS] * batch_size] # [1, batch_size]
  decoder_inputs = Variable(torch.LongTensor(decoder_inputs))
  decoder_inputs = decoder_inputs.cuda() if CUDA else decoder_inputs

  embedded_decoder_inputs = embedding(decoder_inputs)
  # embedded_decoder_inputs: [1, batch_size, hidden_size]

  decoded = decoder(embedded_decoder_inputs, hidden, MAX_STEPS)
  # decoded: [max_input_steps, batch_size, DICT_SIZE]

  ```
