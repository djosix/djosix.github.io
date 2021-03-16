---
layout: post
title: Python Tips
date: 2019-04-26 20:37 +0800
tags:
  - Development
---

Tips for Python 3.

{% include toc.html %}

### IPython tips

**Edit multi-line code in IPython**

1. Either open an editor: `%edit`
2. Or insert a new line in the interactive shell: `ctrl-o`

**Paste multi-line string to a variable on macOS**

- `lines = !pbpaste`

**Interpolation of Python expression inside a command**

- `!echo {'hello %s' % 'world'}`

### Pickle from Python 2 to Python 3

[https://rebeccabilbro.github.io/convert-py2-pickles-to-py3/](https://rebeccabilbro.github.io/convert-py2-pickles-to-py3/)

```python
# In Python 3
obj = pickle.loads(pickle_from_python2, encoding='bytes')
```

### Patching built-in types

```python
from forbiddenfruit import curse

def patch():
   def str_fuck(self):
       return 'fuck ' + self
   curse(str, 'fuck', str_fuck)

   def str_len(self):
       return len(self)
   curse(str, 'len', property(str_len))

patch()
print('you'.fuck()) # fuck you
print('penis'.len) # 5
```

### Read and write socket as file

```python
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(TARGET)
sock_file = sock.makefile('rwb')

sock_file.write(b'whatever\n')
sock_file.flush()

print(sock_file.readline())
```

### Processes and threads

TODO

### Asynchronous

TODO


