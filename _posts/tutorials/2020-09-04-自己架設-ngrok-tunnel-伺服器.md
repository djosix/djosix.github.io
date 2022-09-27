---
layout: post
title: 自己架設 ngrok tunnel 伺服器
date: 2020-09-04 21:02 +0800
tags:
  - SysAdmin
---

ngrok 是一個可以將 local port 綁定到外部擁有公開 IP 的伺服器上的工具，好讓你存取內部服務。

> ngrok 1.x 聽說有 memory leak

### 原始碼

- 目前官方的 ngrok 版本（2.x）沒有開源
- ngrok 1.x [有開源](https://github.com/inconshreveable/ngrok)，可以自己架

### 自簽 TLS 憑證

使用 openssl 建立憑證：

```shell
mkdir cert

openssl req -new -newkey rsa:4096 -x509 -sha256 -days 1825 -nodes \
    -out cert/self.crt -keyout cert/self.key
# Common Name (e.g. server FQDN or YOUR name) []: ngrok.example.com
```

### 編譯 ngrok

- 要先裝 golang
- 要把憑證放進去 client asset 再編譯
- 編譯完成後會產生 `./ngrok/bin/ngrokd` 和 `./ngrok/bin/ngrok`

編譯 ngrok client 和 server：

```shell
git clone https://github.com/inconshreveable/ngrok.git

cp -fv cert/self.crt ngrok/assets/client/tls/ngrokroot.crt

( cd ngrok && make release-all )
```

### Server 端

Server 程式為 `ngrokd`：

```shell
./ngrok/bin/ngrokd \
    -domain=ngrok.example.com \
    -httpAddr=:30080 -httpsAddr=:30443 -tunnelAddr=:34443 \
    -tlsCrt=cert/self.crt -tlsKey=cert/self.key
```

### Client 端

Client 程式為 `ngrok`，用自己架設的 server 要先建立設定檔才能跑：

```shell
echo 'server_addr: ngrok.example.com:34443                                                                                    │~
trust_host_root_certs: false' > $HOME/.ngrok

./ngrok/bin/ngrok 80
```

如果要 debug 可以用：

```shell
./ngrok/bin/ngrok -log-level DEBUG -log client.log 80
```
