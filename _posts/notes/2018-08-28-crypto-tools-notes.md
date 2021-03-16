---
layout: post
title: Crypto Tools Notes
date: 2018-08-28 22:34 +0800
tags:
  - Crypto
---

GnuPG, OpenSSL, and some others.

{% include toc.html %}

## Useful crypto tools

- [quipquip](https://quipqiup.com/) - solve simple substitution ciphers.
- [CyberChef](https://gchq.github.io/CyberChef/) - a data pipeline tool supporting lots of encodings.
- [libnum](https://github.com/hellman/libnum) - Python lib to work on numbers.
  - [Python 3.7 fork of libnum](https://github.com/sasdf/libnum)
- [factordb](http://factordb.com/) - factorize big numbers.

## `gpg`

GnuPG allows you to encrypt and sign your data and communications.

### List public keys

```shell
gpg -k
gpg --list-keys
```

### List private keys

```shell
gpg -K
gpg --list-secret-keys
```

### Encryption

```shell
gpg -c secret.txt 
gpg -o secret.txt.gpg -c secret.txt 
```

### Decryption

```shell
gpg secret.txt.gpg
gpg -o secret.txt secret.txt.gpg
```

### Generate a key

```shell
gpg --gen-key
```

### Generate a key (full)

```shell
gpg --full-generate-key
```

### Export public key

```shell
gpg --output public.key --export [USER] # binary

gpg --armor --output public.key --export [USER] # ASCII armored
gpg -a -o public.key --export [USER] # short version
```

### Export private key

```shell
gpg --output private.key --export-secret-keys [USER] # binary

gpg --armor --output private.key --export-secret-keys [USER] # ASCII armored
gpg -a -o private.key --export-secret-keys [USER] # short version
```

### Import a public key

```shell
gpg --import public.key
```

### Encryption

```shell
gpg --encrypt --recipient Djosix secret.txt
gpg -e -r Djosix secret.txt # short

gpg --encrypt --output secret.txt.gpg --recipient Djosix secret.txt
gpg -e -r Djosix -o secret.txt.gpg secret.txt # short
```

### Decryption

```shell
gpg --output secret.txt --decrypt secret.txt.gpg
gpg -o secret.txt -d secret.txt.gpg # short
```


## `openssl`


### Symmetric

Encryption

```shell
openssl enc -e -aes256 -in secret.txt -out secret.txt.enc
```

Decryption

```shell
openssl enc -d -aes256 -in secret.txt.enc -out secret.txt
```

Ignoring `-in` or `-out` will enable `stdin` or `stdout`.

### RSA

Generate private key

```shell
openssl genrsa -out private_key.pem 4096
```

Generate public key

```shell
openssl rsa -in private_key.pem -out public_key.pem -outform PEM -pubout
```

Encrypt using public key

```shell
openssl rsautl -encrypt -inkey public_key.pem -pubin -in secret.txt -out secret.txt.enc
```

Decrypt using private key

```shell
openssl rsautl -decrypt -inkey private_key.pem -in secret.txt.enc -out secret.txt
```

### RSA With Large Files

Generate a random key for symmetric encryption

```shell
openssl rand -base64 64 > key.bin
```

Encrypt using that key

```shell
openssl enc -aes-256-cbc \
    -salt -in test.txt \
    -out test.txt.enc \
    -pass file:key.bin
```

Encrypt the key using RSA

```shell
openssl rsautl -encrypt \
    -inkey public_key.pem -pubin \
    -in key.bin \
    -out key.bin.enc
```

Decrypt the key using RSA

```shell
openssl rsautl -decrypt -inkey private_key.pem -in key.bin.enc -out key.bin
```

Decrypt files with that key

```shell
openssl enc -d -aes-256-cbc -in test.txt.enc -out test.txt -pass file:key.bin
```

### SSL Connection

```shell
openssl s_client -connect github.com:443
# "GET / HTTP/1.1\r\nHost: github.com\r\n\r\n"
```

But using `ncat` is much easier

```shell
ncat --ssl github.com 443
```
