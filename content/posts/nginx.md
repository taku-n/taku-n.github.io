---
title: "nginx"
date: 2021-04-16
tags: ["nginx", "Kubernetes", "Linux"]
---

# nginx

## Install

```
$ sudo apt install nginx
```

## Check

```
$ systemctl status nginx
```

別の端末からテストする  
(WSL2 は，IPv6 に対応していないので注意)

```
$ curl -4 http://hoge.com/
$ curl -6 http://hoge.com/
```

[Website IPv6 accessibility validator](https://ipv6-test.com/validate.php)

## 起動と停止と再起動

```
$ systemctl start nginx
$ systemctl stop nginx
$ systemctl restart nginx
```

## 設定ファイル

大元の設定ファイルは

```
/etc/nginx/nginx.conf
```

だが，実際に設定するのは，こちら

```
/etc/nginx/sites-available/default
```

`server` ではじまっている部分が  
それぞれ，各 Virtual Host になっている

初期設定として

```
server_name _;
```

の部分を

```
server_name ach.moe;
```

とする  
また，TLS を有効にするため

```
# listen 443 ssl default_server;
# listen [::]:443 ssl default_server;
```

の部分を

```
listen 443 ssl default_server;
listen [::]:443 ssl default_server;
ssl_certificate /etc/letsencrypt/live/hoge.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/hoge.com/privkey.pem;
```

とする

設定ファイルのテスト

```
$ sudo nginx -t
```

設定を反映させる

```
$ systemctl reload nginx
```

## Virtual Host (TLS 用のワイルドカードの証明書が必要)

```
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

の部分を

```
server {
        listen 80;
        listen [::]:80;
        listen 443 ssl;
        listen [::]:443 ssl;
        ssl_certificate /etc/letsencrypt/live/hoge.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/hoge.com/privkey.pem;

        server_name www.hoge.com;

        root /home/hoge/www;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

とする

## Reverse Proxy

```
location / {
	# First attempt to serve request as file, then
	# as directory, then fall back to displaying a 404.
	try_files $uri $uri/ =404;
}
```

の部分を

```
location / {
	proxy_pass http://10.96.1.1;                   # 転送先
        proxy_set_header X-Forwarded-For $remote_addr;  # アクセス元の情報を転送

	# First attempt to serve request as file, then
	# as directory, then fall back to displaying a 404.
	try_files $uri $uri/ =404;
}
```

とする
