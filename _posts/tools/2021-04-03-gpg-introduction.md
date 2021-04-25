---
layout: post
title:  GPG Introduction
date:   2021-04-03
Author: CBD
tags:   [GPG]
---

HTTPS 在 HTTP 的基础上提供了安全支持. 一方面是证书, 利用非对称加密方式确认对方的身份(验证签名); 一方面是加 TLS, 利用对称加密方式传递加密后的密文.

GPG 提供了一套加密解密工具帮我们在更通用的环境下完成上面的步骤.

![pgp-flow.png](/images/pgp-flow.png)

## PGP & OpenPGP & GPG

PGP 全称是 `Pretty Good Privacy`, 由 [Philip Zimmermann](https://philzimmermann.com/) 开发, 最初用来加密电子邮件, 于 1991 免费发布(免费但不开源).
1996 年 PGP 归入 `PGP Inc`, 后几经转手在 2010 年被赛门铁克收购.

[OpenPGP](https://www.openpgp.org/) 是一个非专有协议(non-proprietary protocol).
1997 年 IETF 成立了 OpenPGP 工作组, 在 PGP 的基础上制定了 OpenPGP 标准, 也就是 [RFC 4880](https://tools.ietf.org/html/rfc4880).
OpenPGP 是目前电子邮件加密的事实标准, 任何组织和个人都可以实现它, 不需要授权费.

[GPG](https://www.gnupg.org/) 全称是 `GnuPG`, 也就是 `Gnu Privacy Guard`, 是 OpenPGP 完整的开源的实现.
GPG 易于集成, 为很多工具提供支持, 比如  S/MIME 和 Secure Shell (ssh).

## GPG 使用

### 生成密钥

`gpg --full-generate-key` 交互式地让你填写选项: 密钥类型, 密钥长度, 密钥过期时间, 名字, 邮箱, 备注信息, passphrase.

`gpg --list-keys` 查看刚刚生成的密钥, 我的是这样:

```text
pub   rsa3072 2021-04-01 [SC]
      DAAB4DF81D8D1AEB0F2E1BA4533E72085BB161C0
uid           [ 绝对 ] Baodong Cao (CBD) <wwwicbd@gmail.com>
sub   rsa3072 2021-04-01 [E]
```

这是我们的 "根密钥", 类似 HTTPS 的证书链一样, PGP 也可以用根密钥签发子密钥.

`DAAB4DF81D8D1AEB0F2E1BA4533E72085BB161C0` 是公钥ID, 其实可以只使用最后 16 个字符 `533E72085BB161C0`.

sub 这一行是 GPG 默认为我们创建的子密钥, `[E]` 表示仅用于加密.

我们最好使用子密钥, 一旦子密钥不安全了, 可以只单独吊销这个子密钥.

`gpg --edit-key <你的公钥ID>` 进入交互界面, 输入 `help` 查看所有选项.

```text
gpg> addkey
```

之后根据提示选择参数, 输入 passphrase.

我创建了一个仅用于签名的子密钥:

```text
ssb  dsa2048/DE119A7F72A38C8E
     创建于：2021-04-03  有效至：永不       可用于：S
```

```text
gpg> save
```

保存修改, 退出交互界面.

### 导出公钥, 私钥, 吊销证书

```zsh
gpg --armor --export -o gpg.pub <你的公钥ID>
gpg --export-secret-key -o gpg.secret <你的公钥ID>
gpg --generate-revocation -o gpg.revocation <你的公钥ID>
```

* 公钥是需要发布出去的公开信息;
* 私钥的安全是你数据安全的前提, 要妥善保存不得泄露;
* 吊销证书用来吊销不再使用的子密钥;
* passphrase 不得泄露, 跟私钥分开保存.

### 迁移

1. 旧设备导出公钥, 私钥
2. 旧设备移除私钥
3. 新设备导入公钥, 私钥

```zsh
# 旧设备
gpg --delete-keys <你的公钥ID>
gpg --delete-secret-keys <你的公钥ID>
```

```zsh
# 新设备
gpg --import gpg.pub
gpg --import gpg.secret
```

### 发布公钥

```zsh
gpg --send-keys <你的公钥ID>
```

GPG 默认的 keyserver 在大陆访问有问题, 可以自行使用备选服务器
`gpg --keyserver hkp://keyserver.ubuntu.com:80 --send-keys <你的公钥ID>`

也就是说没有唯一的 keyserver, 选择是否信任又用户自己约定, 自行决定.

### 用户导入公钥

直接导入公钥文件:

```sh
gpg --import gpg.pub
```

通过公钥 ID 导入:

```sh
# gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 8D709DB50E5B773EF98E50AB04255F266E5C3842
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 8D709DB50E5B773EF98E50AB04255F266E5C3842
```

### 加密 解密 签名 验签

非对称加密中, 大家把公钥互相公开, 各自保管自己的密钥:

* 用我自己的密钥对内容签名, 接收方拿我的公钥验证签名;
* 我拿接收方的公钥对内容加密, 接收方拿他自己的密钥解密.

`gpg --sign book.pdf` 会得到一个新文件 `book.pdf.gpg`, 这个文件包含了签名信息, 也会被自动压缩. 我们把 `book.pdf.gpg` 发给朋友, 并告诉他我的公钥或者公钥ID.

朋友需要导入公钥.

然后 `gpg book.pdf.gpg`, gpg 会先验证签名, 再把文件解压, 得到 `book.pdf`.

可能有的人并不关心签名, 也可能没有 gpg, 特别是对公开发布的文件, 最好使用 `--detach-sign` 来签名, 这种方式会把签名文件独立保存.

加密文件前, 先导入接收方的公钥, 然后 `gpg --recipient <朋友的公钥ID> --encrypt book.pdf` 得到加密并压缩的文件 `book.pdf.gpg` .

朋友收到文件后解压 `gpg book.pdf.gpg`, 因为需要用私钥解压, 这里他要输入自己的 passphrase .

加密和签名也可以在一步中组合使用.

### Sign Git commit

```zsh
git config --global user.signingkey <你的公钥ID>
git config --global commit.gpgsign true
```

```zsh
export GPG_TTY=$(tty)
```
