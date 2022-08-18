---
title: "HAProxy"
date: 2021-04-13
tags: ["HAProxy"]
---

# HAProxy

## インストール (Ubuntu 20.04 LTS)

```
$ sudo apt update
$ sudo apt full-upgrade
$ sudo apt autoremove
$ sudo apt install haproxy
```

## 設定ファイルのテスト

```
$ sudo haproxy -c -V -f /etc/haproxy/haproxy.cfg
```

## Let's Ecrypt

基本は，[こちら](https://kevinbentlage.nl/blog/lets-encrypt-with-haproxy/)に従う  
インストールと certbot コマンドは，以下の通り  
また，certbot コマンドを打つ前に，設定ファイルに追記した内，https の関係する部分は，コメントアウトしておく

```
$ sudo snap refresh core
$ sudo snap install --classic certbot
```

```
$ sudo systemctl restart haproxy
$
$ # 正常に起動したら，つぎを実行
$
$ sudo certbot certonly --standalone --preferred-challenges http --http-01-address 127.0.0.1 --http-01-port 9080 -d hoge.com -d '*.hoge.com' --email hoge@hoge.com --agree-tos --non-interactive
```
