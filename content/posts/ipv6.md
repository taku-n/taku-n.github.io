---
title: "IPv6"
date: 2020-12-13
tags: ["IPv6"]
---

# IPv6

Server OS: Ubuntu 20.04
Client OS: Windows 10

## IPv6 が有効になっているか調べる

```
$ ip -6 a
```

## コマンド プロンプト から IPv6 アドレスに ping する

```
>ping {IPv6 address}
```

## SSH Server が IPv6 を受け入れるようにする

```
$ sudo nvim /etc/ssh/sshd_config
```

```
#ListenAddress ::
```

を

```
ListenAddress ::
```

に変更する

## コマンド プロンプト から IPv6 アドレスに SSH で接続する

```
>ssh {user name}@{IPv6 address}
```
