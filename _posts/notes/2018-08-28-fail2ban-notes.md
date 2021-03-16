---
layout: post
title: Fail2ban Notes
date: 2018-08-28 22:24 +0800
tags:
  - Security
---

Damn, so how do I unban my IP address?

{% include toc.html %}


### List jails

```shell
fail2ban-client status
```

### Show status of sshd jail

```shell
fail2ban-client status sshd
```

### Show status of all fail2ban jails at once

```shell
jails=`fail2ban-client status | grep "Jail list" | sed -E 's/^[^:]+:[ \t]+//' | sed 's/,//g'`
for jail in $jails
do
  fail2ban-client status $jail
done
```

### Unban an IP

```shell
fail2ban-client unban sshd 127.0.0.1
```

### Ban and unban

```
fail2ban-client ban 127.0.0.1
fail2ban-client unban 127.0.0.1

fail2ban-client set sshd banip 127.0.0.1
fail2ban-client set sshd unbanip 127.0.0.1
```

### Retries

```shell
fail2ban-client get sshd maxretry
fail2ban-client set sshd maxretry 10
```

### Ban time

```shell
fail2ban-client get sshd bantime # seconds
fail2ban-client set sshd bantime 3600
```

### Start and stop jail

```shell
fail2ban-client start sshd
fail2ban-client stop sshd
```
