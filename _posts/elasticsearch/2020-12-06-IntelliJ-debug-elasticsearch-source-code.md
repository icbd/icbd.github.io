---
layout: post
title:  IntelliJ debug elasticsearch 7.10 source code
date:   2020-12-06
Author: CBD
tags: [Elasticsearch]
---

源码启动 Elasticsearch 7.10 并用 IntelliJ IDEA 断点调试.

## 下载源码

限于国内的网路, 下载 zip 要比 clone 快得多, 直接下载 7.10 源码:

[https://github.com/elastic/elasticsearch/archive/7.10.zip](https://github.com/elastic/elasticsearch/archive/7.10.zip) .

## 打开项目

IntelliJ 打开项目, 选择源码目录.

Auto import gradle project.

添加 remote:

1. 选择 listen 模式
2. 勾选 auto restart

![remote-listen.png](/images/remote-listen.png)

## 修改设置(可选)

为了方便, 在 `gradle/run.gradle` 中关闭 Auth 认证:

`setting 'xpack.security.enabled', 'false'`

或者使用其中的用户名密码:

`user username: 'elastic-admin', password: 'elastic-password', role: 'superuser'`

## 启动

先启动上面的 remote debug, 然后用 gradlew 启动项目:

```sh
./gradlew :run --debug-jvm
```

至少要经过数分钟才能启动起来, 启动之后在 IntelliJ 中打断点就可以正常调试了.

比如, 断点加在 `org/elasticsearch/rest/action/search/RestSearchAction.java` 137 行,

执行任意搜索就会进入断点.

![es7-10-debug](/images/es7-10-debug.png)