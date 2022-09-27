---
layout: post
title: 在 macOS 上使用 X11 Forwarding
date: 2019-06-12 06:22 +0800
tags:
  - SysAdmin
---

聽說用 `ssh -Y` 不好。

## Server 端（以 Linux 為例）

- 編輯 `/etc/ssh/sshd_config`，確認一下有這行：
  ```
  X11Forwarding yes
  ```
- 確定有安裝 `xauth`。
- 如果有做任何修改，重開 `sshd`：
  ```shell
  sudo systemctl restart sshd
  ```

## Client 端（以 macOS 為例）

- 確認有安裝 [XQuartz](https://www.xquartz.org/)。
- 編輯 `~/.ssh/config`，加入 `xauth` 路徑：
  ```
  XAuthLocation /opt/X11/bin/xauth
  ```
- 連過去：
  ```shell
  ssh -X user@hostname
  ```
