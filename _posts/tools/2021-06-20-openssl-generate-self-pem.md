---
layout: post
title:  OpenSSL 自签名
date:   2021-06-20
Author: CBD
tags:   [SSL]
---

```zsh
openssl genrsa -out key.pem 4096
openssl req -new -x509 -days 1000 -key key.pem -out cert.pem
```

* cert.pem 证书
* key.pem 私钥
