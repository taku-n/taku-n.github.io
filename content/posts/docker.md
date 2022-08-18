---
title: "Docker"
date: 2020-06-20
tags: ["Docker"]
---

# Docker

[Docker Engine](https://docs.docker.com/engine/)  
[Docker Compose](https://docs.docker.com/compose/)

## Docker

[Linux環境でdockerのコマンドをsudoなしで打てるようにする](https://hodalog.com/execute-docker-command-in-linux-without-sudo/)

### バージョン確認

```
$ docker -v
```

### Docker デーモンの状態を確認

```
$ service docker status
```

### Docker デーモンの開始

```
$ sudo service docker start
```

### Docker デーモンが自動的に起動するように設定 (WSL2)

[wsl2でDocker自動起動設定](https://qiita.com/ktaidot/items/949d358163bbbad5a91e)  
Thank @ktaidot.

### コンテナの実行のテスト

```
$ sudo docker run hello-world
$ sudo docker run docker/whalesay cowsay boo
$ sudo docker run -it ubuntu bash
```

### Docker デーモンが起動しないとき

```
$ sudo dockerd --debug
```

### イメージの一覧

```
$ sudo docker image list
```

### コンテナの一覧

```
$ sudo docker ps -a
$ sudo docker ps --no-trunc --format "table {{.Command}}"  # コマンドを見たいとき
```

[docker psでコンテナリストを表示・取得する](https://unskilled.site/docker-ps%E3%81%A7%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%83%AA%E3%82%B9%E3%83%88%E3%82%92%E8%A1%A8%E7%A4%BA%E3%83%BB%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B/)  
[【docker ps】コンテナ情報出力](https://qiita.com/chihiro/items/3162a484fd257e114c18)

### コンテナの停止

```
$ sudo docker stop {CONTAINER ID|NAMES}
```

## Dockerfile

### Dockerfile からイメージをビルド

Dockerfile をつぎの内容で作成

```
FROM ubuntu
```

ビルド  
(最後のドットは Dockerfile がカレントディレクトリにあることを示す)

```
$ sudo docker build -t dockerfile-test .
```

確認

```
$ sudo docker image list
```

## Docker Compose

docker-compose.yml をつぎの内容で作成

```
version: '3'

services:
  web:
    image: nginx
    ports:
      - 80:80
```

実行 (-d はバックグラウンド)

```
$ sudo docker-compose up -d
```

ブラウザで http://localhost/ にアクセスして確認  
つぎのコマンドで確認

```
sudo docker-compose ps
```

停止

```
$ sudo docker-compose down --remove-orphans
```
