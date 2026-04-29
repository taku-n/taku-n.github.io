---
title: "Podman"
date: 2026-04-29T13:11:51+09:00
draft: true
---

# Podman

[Podman](https://podman.io/)  
[コンテナーの構築、実行、および管理](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html/building_running_and_managing_containers/)  

## Installation

```
sudo apt update
sudo apt install podman
```

## Print information

```
podman version
podman info
```

## Set the mount propagation of / shared

In case you get a warning below when you run "podman version"  

```
WARN[0000] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers
```

[Podman で Singularity コンテナを動かすまでの覚書](https://zenn.dev/tom_tan/articles/405ae402b43f37#%2F-%E3%81%AE-mount-propagation-%E8%A8%AD%E5%AE%9A%E3%81%AE%E7%A2%BA%E8%AA%8D)  

## Set the Linger yes

In case when you run  

```
loginctl user-status $USER | grep Linger
```

it prints "Linger: no"  

[Podman で Singularity コンテナを動かすまでの覚書](https://zenn.dev/tom_tan/articles/405ae402b43f37#systemd-%E3%81%AE-linger-%E8%A8%AD%E5%AE%9A%E3%81%AE%E7%A2%BA%E8%AA%8D)  

## hello, world

```
podman run hello
```

## Images

### Print the image I have got

```
podman images
```

### Remove the image

```
podman rmi image-name
```

## Containers

### Print the containers

```
podman ps -a
podman ps  # Running containers only
```

### Stop the container

```
podman stop container-name
```

### Remove the container having been stopped

```
podman rm container-name
```

## Network

### Print the network list

```
podman network ls
```

### Print the network detail

```
podman network inspect network-name
```

The default network is podman "subnet": "10.88.0.0/16", "gateway": "10.88.0.1"  

## Ubuntu

### Run a command in a container  

```
podman run -it --rm --name ubuntu docker.io/ubuntu echo hello, world
# -i: Enable you to input against the container
# -t: Open a terminal after starting the container
# --rm: Remove the container after the use
# --name: Set container name
```

### Run a container in the background and dive into its Bash shell

```
podman run -d -it --rm --name ubuntu docker.io/ubuntu
# -d: Run in the background
# -it: Without these flags, the container ends immediately

podman exec -it ubuntu bash
```

### Run a container with a network connecting to the default network podman by 10.88.1.1

```
podman run -d -it --rm --net podman --ip=10.88.1.1 --name ubuntu docker.io/ubuntu
# --net network-name
```

## nginx

### Host a website

```
podman run -d --rm -p 8888:80 --name nginx docker.io/nginx
# -d: Run in the background
# -p: host-side-port:container-side-port
curl 127.0.0.1:8888
# Open http://127.0.0.1:8888/ from a browser in Windows
```

To access it from another device, open the TCP port 8888 of your Windows  
See [[Windows 11] Windowsファイアウォールでポートを開く / 閉じる方法](https://solutions.vaio.com/5464)  

### Host a website for another container as well

```
podman run -d --rm --net podman --ip=10.88.1.1 -p 8888:80 --name nginx docker.io/nginx
podman run -d -it --rm --net podman --ip=10.88.1.2 --name ubuntu docker.io/ubuntu
podman exec -it ubuntu bash
```

In the ubuntu container  

```
apt update
apt upgrade
apt install curl
curl 10.88.1.1:80  # The port is not 8888 in this case
```

## Podman Compose

### Installation

```
sudo apt update
sudo apt install podman-compose
```

### Print information

```
podman compose version
```

### Error from nftables

In case you get this error

```
Error: unable to start container "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx": netavark: nftables error: "nft" did not return successfully while applying ruleset
```

```
sudo apt update
sudo apt install iptables
mkdir ~/.config/containers
vi ~/.config/containers/containers.conf
```

```
[network]
firewall_driver = "iptables"
```

```
podman system reset
```

[Podman composeを使ってAnsible管理対象ノードをRootlessコンテナとして起動する](https://note.com/fminamot/n/nf8f8e232d94b)  

### Print the status

```
podman compose ps
```

### Run

```
podman compose up -d
# -d: Run in the background
```

### Stop

```
podman compose down
```

### nginx

```
vi podman-compose.yml
```

```
services:
  web:
    image: docker.io/nginx
    ports:
      - "8888:80"
```

```
podman compose up -d
curl 127.0.0.1:8888
```
