---
layout: post
title: sudo 使用 MacBook Touch ID 認證
date: 2019-12-29 07:56 +0800
---

只要修改 `/etc/pam.d/sudo`

```
# sudo: auth account password session
auth       sufficient     pam_tid.so
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```

就是加上這行：`auth       sufficient     pam_tid.so`
