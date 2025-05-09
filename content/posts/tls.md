---
title: "TLS"
date: 2021-02-19
tags: ["TLS"]
---

# TLS (Transport Layer Security)

## Certbot

インストール方法は，公式サイトを参照  
[https://certbot.eff.org/](https://certbot.eff.org/)

しかし

```
$ sudo certbot --nginx
```

がエラーになってしまう  
とりあえず

```
$ sudo nginx -t
$ sudo nvim /etc/nginx/sites-enabled/default
```

managed by Certbot となっている行をすべてコメントアウト  
その後

```
$ sudo systemctl restart nginx
$ sudo certbot --nginx
```

### サブコマンドとオプション

`certonly`  
証明書の取得のみを行う

`renew`  
期限の残りが 30日未満 の証明書を更新

`--dry-run`  
実際には，証明書の更新を行わない

`--force-renewal`  
すべての証明書を期限の残りにかかわらず，強制的に更新

`--manual`  
Nginx とかを使わずに手動で認証

### DNS の場合 (ConoHa DNS)

最も重要な点として，DNS サーバが API による操作に対応していなければならない  
ここでは，ConoHa の DNS サーバを利用することにする

初回のやり方は，つぎのとおり

```
$ sudo certbot certonly --manual --preferred-challenges dns -d hoge.com -d '*.hoge.com'
```

あとは，指示どおりに TXT レコードを追加すれば OK

更新する場合は，こちらをまず確認  
[Pre and Post Validation Hooks](https://certbot.eff.org/docs/using.html#pre-and-post-validation-hooks)

フックを準備

```
sudo touch /etc/letsencrypt/authenticator.py
sudo touch /etc/letsencrypt/cleanup.py
sudo touch /etc/letsencrypt/log.txt
sudo chmod 700 /etc/letsencrypt/authenticator.py
sudo chmod 700 /etc/letsencrypt/cleanup.py
sudo chmod 600 /etc/letsencrypt/log.txt
cd
curl -OL https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
sudo /usr/local/bin/pip install -U pip
sudo /usr/local/bin/pip install git+https://github.com/taku-n/mikumo.git@stable
```

/etc/letsencrypt/authenticator.py をつぎのようにする

```
#!/bin/env python

import datetime
import os
import sys

import mikumo  # A library for ConoHa API

############################
# Set your settings below. #
############################

# Credentials for ConoHa
API_USERNAME      = 'your-username'
API_PASSWORD      = 'your-password'
TENANT_ID         = '0123456789abcdef0123456789abcdef'
IDENTITY_ENDPOINT = 'https://identity.****.conoha.io/v2.0'

############################
# Set your settings above. #
############################

def log(str):
    now = datetime.datetime.now()
    now_str = now.strftime('%Y.%m.%d %H:%M:%S')

    with open('/etc/letsencrypt/log.txt', 'a') as f:
        print(now_str, str, file = f)

log(f'CERTBOT_DOMAIN: {os.environ["CERTBOT_DOMAIN"]}')
log(f'CERTBOT_VALIDATION: {os.environ["CERTBOT_VALIDATION"]}')
log(f'CERTBOT_REMAINING_CHALLENGES: {os.environ["CERTBOT_REMAINING_CHALLENGES"]}')
log(f'CERTBOT_ALL_DOMAINS: {os.environ["CERTBOT_ALL_DOMAINS"]}')

credentials = {'api-username': API_USERNAME, 'api-password': API_PASSWORD,
        'tenant-id': TENANT_ID, 'identity-endpoint': IDENTITY_ENDPOINT}

code, access = mikumo.id.get_access(credentials)
if code != 200:
    log(f'Error: mikumo.id.get_access(): {code}')
    sys.exit(1)

domain_name = os.environ['CERTBOT_DOMAIN']
validation = os.environ['CERTBOT_VALIDATION']
code, data = mikumo.dns.create_record_by_domain_name(
        access, domain_name, '_acme-challenge', 'TXT', validation, None)
if code != 200:
    log(f'Error: mikumo.dns.create_record_by_domain_name(): {code}')
    sys.exit(1)
```

/etc/letsencrypt/cleanup.py をつぎのようにする

```
#!/bin/env python

import datetime
import os
import sys

import mikumo  # A library for ConoHa API

############################
# Set your settings below. #
############################

# Credentials for ConoHa
API_USERNAME      = 'your-username'
API_PASSWORD      = 'your-password'
TENANT_ID         = '0123456789abcdef0123456789abcdef'
IDENTITY_ENDPOINT = 'https://identity.****.conoha.io/v2.0'

############################
# Set your settings above. #
############################

def log(str):
    now = datetime.datetime.now()
    now_str = now.strftime('%Y.%m.%d %H:%M:%S')

    with open('/etc/letsencrypt/log.txt', 'a') as f:
        print(now_str, str, file = f)

log(f'CERTBOT_DOMAIN: {os.environ["CERTBOT_DOMAIN"]}')
log(f'CERTBOT_VALIDATION: {os.environ["CERTBOT_VALIDATION"]}')
log(f'CERTBOT_REMAINING_CHALLENGES: {os.environ["CERTBOT_REMAINING_CHALLENGES"]}')
log(f'CERTBOT_ALL_DOMAINS: {os.environ["CERTBOT_ALL_DOMAINS"]}')

credentials = {'api-username': API_USERNAME, 'api-password': API_PASSWORD,
        'tenant-id': TENANT_ID, 'identity-endpoint': IDENTITY_ENDPOINT}

code, access = mikumo.id.get_access(credentials)
if code != 200:
    log(f'Error: mikumo.id.get_access(): {code}')
    sys.exit(1)

domain_name = os.environ['CERTBOT_DOMAIN']
code, data = mikumo.dns.delete_record_by_domain_name_and_name(
        access, domain_name, '_acme-challenge', 'TXT')
if code != 200:
    log(f'Error: mikumo.dns.delete_record_by_domain_name_and_name(): {code}')
    sys.exit(1)
```

できたら，試す

```
sudo certbot renew --manual \
--manual-auth-hook /etc/letsencrypt/authenticator.py \
--manual-cleanup-hook /etc/letsencrypt/cleanup.py \
--force-renewal --dry-run
```

うまくいったら，自動更新できるようにする

```
sudo touch /etc/letsencrypt/renew.sh
sudo chmod 700 /etc/letsencrypt/renew.sh
```

/etc/letsencrypt/renew.sh をつぎのようにする  
openssl コマンドの `hoge.com` のところは，自分のドメインに修正する

```
#!/bin/bash

echo "Starting certbot."

certbot renew --manual \
        --manual-auth-hook /etc/letsencrypt/authenticator.py \
        --manual-cleanup-hook /etc/letsencrypt/cleanup.py

echo "certbot finished."

openssl x509 -in /etc/letsencrypt/live/hoge.com/fullchain.pem -noout -dates

# To see a log by journalctl, type "journalctl -e -u letsencrypt".
# A log file of certbot is /var/log/letsencrypt/letsencrypt.log.
# A log file of hooks is /etc/letsencrypt/log.txt.
```

/etc/systemd/system/letsencrypt.service をつぎのようにする

```
[Unit]
Description=Renew Let's Encrypt

[Service]
Type=oneshot
ExecStart=/etc/letsencrypt/renew.sh

[Install]
WantedBy=multi-user.target
```

/etc/systemd/system/letsencrypt.timer をつぎのようにする

```
[Unit]
Description=Timer to renew Let's Encrypt

[Timer]
OnUnitActiveSec=1d

[Install]
WantedBy=timers.target
```

自動実行を開始する

```
sudo systemctl start letsencrypt.timer
sudo systemctl start letsencrypt.service
```

再起動しても自動実行されるように登録する

```
sudo systemctl enable letsencrypt.timer
sudo systemctl enable letsencrypt.service
```

## OreOreCA

PKI: Public Key Infrastructure  
　CA: Certificate Authority  
　CSR: Certificate Signing Request  
　　C: Country Name  
　　ST: State or Province Name  
　　L: Locality Name  
　　O: Organization Name  
　　OU: Organizational Unit Name  
　　CN: Common Name  
　X.509  
PEM: Privacy Enhanced Mail  

[PEMとは？ わかりやすく10分で解説](https://www.netattest.com/pem-2024_mkt_tst)  

### CA をつくる

```
mkdir oreoreca
cd oreoreca
```

CA の Private Key をつくる  

```
openssl genrsa -aes256 -out ca-pri.pem 2048
```

CA の CSR をつくる (Public Key + Information)  

```
openssl req -new -key ca-pri.pem -subj "/C=JP/ST=Tokyo/L=Chiyoda/O=CRT/OU=Workshop/CN=OreOreCA/" -out ca-csr.pem
```

CA の Certificate をつくる (Private Key で CSR に署名)  

```
openssl x509 -req -in ca-csr.pem -days 397 -signkey ca-pri.pem -out ca-crt.pem
```

### Server Certificate をつくる

```
mkdir server
cd server
```

Server の Private Key をつくる  

```
openssl genrsa -aes256 -out svr-pri.pem 2048
```

Server の CSR をつくる (Public Key + Information)  

```
openssl req -new -key svr-pri.pem -subj "/C=JP/ST=Tokyo/L=Chiyoda/O=CRT/OU=Workshop/CN=hello.lan/" -out svr-csr.pem
```

Extension File をつくる  

svr-ext.cnf  

```
subjectAltName=DNS:*.hello.lan,IP:::,IP:0.0.0.0
```

Server の Certificate をつくる (CA の Private Key で Server の CSR に署名)  

```
openssl x509 -req -in svr-csr.pem -days 397 -CA ../ca-crt.pem -CAkey ../ca-pri.pem -CAcreateserial -out svr-crt.pem -extfile svr-ext.cnf
```

[信頼済み証明書に関する今後の制限について](https://support.apple.com/ja-jp/102028)  
[Why does signing a certificate require `-CAcreateserial` argument?](https://stackoverflow.com/questions/66357451/why-does-signing-a-certificate-require-cacreateserial-argument)  
[OpenSSL で”error while loading serial number”が出た](https://btatsu.hatenablog.com/entry/2019/11/28/113945)  

### 検証

```
openssl verify -CAfile ../ca-crt.pem svr-crt.pem
```

### 読む

Private Key  

```
openssl rsa -text -in pri.pem -noout
```

CSR  

```
openssl req -text -in req.csr -noout
```

Certificate  

```
openssl x509 -text -in crt.pem -noout
```

### mkcert

```
sudo aptitude install mkcert

mkcert -install
# ~/.local/share/mkcert/rootCA-key.pem is the private key of CA
# ~/.local/share/mkcert/rootCA.pem is the certificate of CA

mkcert -key-file svr-pri.pem -cert-file svr-crt.pem *.hello.lan :: 0.0.0.0
# svr-pri.pem is the private key of the Server
# svr-crt.pem is the certificate of the Server
```
