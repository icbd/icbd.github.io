---
layout: post
title:  HTTP Redirect
date:   2020-10-17
Author: CBD
tags: [HTTP]
---

[https://tools.ietf.org/html/rfc7231](https://tools.ietf.org/html/rfc7231) 描述了 HTTP 协议对重定向的要求, 我们拿具体例子看一下.

当访问 `http://zhihu.com` 时, 知乎服务器的响应是:

```text
HTTP/1.1 301 Moved Permanently
Server: CLOUD ELB 1.0.0
Date: Sat, 17 Oct 2020 11:26:11 GMT
Content-Type: text/html
Content-Length: 182
Connection: keep-alive
Location: https://www.zhihu.com/
X-Backend-Response: 0.000
Vary: Accept-Encoding
Referrer-Policy: no-referrer-when-downgrade
X-SecNG-Response: 0.0010001659393311
X-UDID: XXX
x-lb-timing: 0.002
x-idc-id: 2
Set-Cookie: KLBRSID=XXX; Path=/
```

浏览器读到 response 的 HTTP Code 为 301, 然后发起了一个新请求到 Location 指定的 URL, 即 `https://www.zhihu.com/`.
这是大部分网站的典型操作: 把顶级域名重定向到 `www` 的二级域名, 并且使用 HTTPS 访问.

当我们再次访问 `http://zhihu.com` 时, 会发现 `from disk cache` 的提示:

```text
Request URL: http://zhihu.com/
Request Method: GET
Status Code: 301 Moved Permanently (from disk cache)
Remote Address: 103.41.167.234:80
Referrer Policy: strict-origin-when-cross-origin
```

这就是 301 的效果, **永久重定向**, 浏览器会缓存 301 的响应, 再次请求时会略过源 Location (`http://zhihu.com/`) 直接请求新 Location (`https://www.zhihu.com/`) .

相对应的, 302 是 **临时重定向**, 浏览器不会缓存, 每次都走先请求源 Location, 然后根据响应继续后续的请求. 使用 302 的服务器可以随时修改新的目的 Location, 比如 OAuth2 认证过程中, 使用临时重定向就更合适.

对于一个 POST 请求, 很可能携带了请求体, 重定向之后, 浏览器会携带请求体去请求新的 Location .
不过出于历史原因, POST 的请求在重定向 301 302 时, 可能会用 GET 去请求新的 Location, 为了避免这种歧义, 302 的后继 307 明确要求: 重定向请求不得修改 HTTP Method (301 没有对应的).

## 小结

* 301 永久重定向, 缓存 Location;
* 302 / 307 临时重定向, 不缓存 Location;
* 307 不得变更 HTTP Method.
