---
layout: post
title: 用 proxy.py 修改瀏覽器 HTTP 連線
date: 2022-12-08 19:24 +0800
---

假如我想要把 `www.google.com` 對到我自己的電腦 `127.0.0.1`，讓我瀏覽器一開啟 Google 就會訪問我開在 local 的 Web Server。該如何做到？

一般來說，有幾種方式可以達成：

1. 修改 hosts 檔
    - 這檔案在主流系統都有：
        - Windows: 修改 `C:\WINDOWS\system32\drivers\etc\hosts`
        - Unix-like: 修改 `/etc/hosts`
    - 加入一行 `127.0.0.1 www.google.com` 之後，訪問 `www.google.com` 都會連到 `127.0.0.1`
2. 修改 DNS server 設定
    - 自己架設 DNS server，並修改系統預設 DNS server
    - 連線使用 DHCP 時，DNS server 有時會被自動設定成 router，這種時候 router 可能會可以設定 mapping
3. 使用自己架設的 proxy server

### 使用自己架設的 proxy server 來修改 connection destination

這裡要舉例的是使用 [`proxy.py`](https://github.com/abhinavsingh/proxy.py)，可直接用 pip 安裝：

```shell
python3 -m pip install -U proxy.py
```

安裝完成後會多個指令

```shell
proxy
# 或 python3 -m proxy
```

此指令預設會在 `127.0.0.1:8899` 開啟一個 HTTP proxy server，只有本機能連，若要讓本機以外也能連，可以加上 `--hostname 0.0.0.0`：

```shell
proxy --hostname 0.0.0.0
```

當 proxy server 開啟之後，可以透或擴充元件將瀏覽器設定成透過我們的 server 來上網。例如我使用的 Microsoft Edge 可以安裝 [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-TW)，如果你使用其他瀏覽器，應該也能找到類似的擴充元件。

Microsoft Edge / Google Chrome 的做法：

1. 安裝 [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-TW)
2. 點 "New profile..." 新增一個 profile  
3. 選擇 HTTP proxy 並輸入 proxy server 的 host 和 port，點 "Apply changes"
4. 選擇剛剛建立的 profile，之後所有連線都會由這個 proxy server 代理
5. 隨便連個網站看一下連線是否成功

很不幸的是，`proxy` 這指令只是開個普通的 proxy server，除了代理以外沒有任何功能。想要它做別的事需要寫 plugin，也就是在 HTTP 流程中插入自己的 code 去修改 request 或 response：

```python
# proxy_plugin.py

from proxy.http.proxy import HttpProxyBasePlugin
from proxy.http.parser import HttpParser
from typing import Optional


class CustomPlugin(HttpProxyBasePlugin):
    def before_upstream_connection(self, request: HttpParser) -> Optional[HttpParser]:
        print(request.host)
        if request.host == b'www.google.com':
            request.host = b'127.0.0.1'
        return request


if __name__ == '__main__':
    from proxy.proxy import main
    main(plugins=[CustomPlugin])
```

上述 Python code 的使用方式有兩種：

- 直接執行 `python3 proxy_plugin.py`
- 作為 plugin 被使用 `proxy --plugins proxy_plugin.CustomPlugin`

此時只要訪問 `www.google.com` 都會連過去 `127.0.0.1`。

### 後記

其他 debug proxy 也可以做到類似的事情，例如 `mitmproxy`，但我沒有深入研究。`proxy.py` 是我看過相對簡單的方式。
