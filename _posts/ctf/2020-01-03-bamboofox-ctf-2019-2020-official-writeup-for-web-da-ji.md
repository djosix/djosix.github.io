---
layout: post
title: BambooFox CTF 2019-2020 Official Writeup for [Web] Da Ji
date: 2020-01-03 13:52 +0800
tags:
  - CTF
---


*Web & Crypto, 8 solves, 588 points.*

> Our programmer was drunk while implementing the session.
>
> http://34.82.101.212:8002/ (down)  
> http://ctf.bamboofox.cs.nctu.edu.tw:8002/  
> 
> author: djosix

There was a page where you could post your name. And it would give you a response: "Hello, {YOUR NAME}. You cannot see the flag.".

There was a cookie called `session` stored after you post your name. By the challenge description, you could guess that the session was implemented by ourselves. Moreover, if you entered a very long name, the session value got long too. That was typically encrypted data. And there is a famous [padding oracle attack](https://en.wikipedia.org/wiki/Padding_oracle_attack) you can try.

By changing the last byte of the second last block (16 bytes block) we could confirm that that was encrypted in CBC mode with 16 bytes block size.

I wrote a convenient python module to run padding oracle attack: [https://github.com/djosix/padding_oracle.py](https://github.com/djosix/padding_oracle.py). The script looked like this:

```python
import requests
from padding_oracle import *

URL = 'http://34.82.101.212:8002/'
sess = requests.Session()
session = '%2Bfs7r4VO2kxNDdi0arbP7r6bqqf993hx739dOLzBYo5HKnKHZCTLjRBlCYlSTLEszQzRJldsd9Tfv04AUNsFtA%3D%3D'
cipher = base64_decode(urldecode(session))

def oracle(cipher):
    r = sess.get(URL, cookies={'session': urlencode(base64_encode(cipher))})
    return 'error' not in r.text

plaintext = padding_oracle(cipher, 16, oracle, 64)

print(remove_padding(plaintext).decode())
```

Then I got `b'a:2:{s:4:"show";b:0;s:4:"name";s:1:"a";}\x08\x08\x08\x08\x08\x08\x08\x08'`, which is a serialized PHP array. It's obvious that you should change `show` to `TRUE`. But how to make it decrypt successfully? Since this is [CBC](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_Block_Chaining_(CBC)), we can align our plaintext with PCKS#7 padding bytes to certain blocks, then we send only those cipher blocks as our session next time. In short, we can inject `name` as `"___________a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}";}\x01"` to make the decrypted plaintext be `"a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}";}\x01"`:

```
The original plaintext decypted by padding oracle:
                a:2:{s:4:"show";b:0;s:4:"name";s:3:"asd";}666666
                                                          ^^^^^^ padding bytes \x06
[------IV------][----Block-----][----Block-----][----Block-----]


We want to let the decrypted session be this: 
a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}

So we send:
name=___________a%3A2%3A%7Bs%3A4%3A%22show%22%3Bs%3A1%3A%221%22%3Bs%3A4%3A%22name%22%3Bs%3A1%3A%22a%22%3B%7D%22%3B%7D%01

The cipher we recieved after using "___________a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}";}\x01" as name:
[------IV------][----Block-----][----Block-----][----Block-----][----Block-----][----Block-----][----Block-----][----Block-----]
                              our new cipher -> [------IV------][----Block-----][----Block-----][----Block-----]
                a:2:{s:4:"show";b:0;s:4:"name";s:59:"___________a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}";}1";}
|               |               |               |               |               |               |              ^ padding byte \x01
0               16              32              48              64              80              96              112             128
```

So, this time you just send 48 to 111 bytes of the cipher instead:

```python
import requests
from padding_oracle import *

URL = 'http://34.82.101.212:8002/'

name = '___________a:2:{s:4:"show";s:1:"1";s:4:"name";s:1:"a";}";}\x01'
session = requests.post(URL, data={'name': name}).cookies.get('session')

cipher = base64_decode(urldecode(session))
cipher = cipher[48:112]
session = urlencode(base64_encode(cipher))
print(requests.get(URL, cookies={'session': session}).text)
```

Then you got a response:

```
<title>大吉</title><h1>Hello, a</h1>This is your flag: <b>BAMBOOFOX{78c75409bab501f3973ac6dc7e309b59}</b>
```
