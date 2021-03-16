---
layout: post
title: macOS sandbox-exec
date: 2019-08-24 23:58 +0800
  - Security
---

mac 有個指令叫做 `sandbox-exec`，可以用它來執行程式，執行時可以有許多功能限制。

## 基本用法

首先你必須有個 sandbox profile，說明這個 sandbox 能幹嘛或不能幹嘛。假設我們已經有個 profile 檔案如下：

```
; no-network.sb

(version 1)

(allow default)
(deny network*)
```

沒錯，語法長得很像 lisp。接下來打這行就可以執行了：

```shell
sandbox-exec -f no-network.sb curl djosix.com
```

因為 profile 中設定不能做任何網路相關操作，所以會被擋掉，然後噴出以下訊息：

```
curl: (6) Could not resolve host: djosix.com
```

再一個禁止寫檔的 profile 範例：

```
; no-file-write.db

(version 1)

(allow default)
(deny file-write*)
```

拿 touch 來測試

```shell
sandbox-exec -f no-file-write.sb touch your_nipples
```

當然不能 touch your_nipples

```
touch: your_nipples: Operation not permitted
```

然後這些地方可以找到現有的 profile：

- /System/Library/Sandbox/Profiles/
- /usr/share/sandbox/


## 補充資料

- [OS X: Run any command in a sandbox](https://www.davd.io/os-x-run-any-command-in-a-sandbox/)
- [Apple Sandbox Guide v1.0](https://reverse.put.as/wp-content/uploads/2011/09/Apple-Sandbox-Guide-v1.0.pdf)
