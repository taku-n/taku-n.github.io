---
title: "Kubernetes"
date: 2020-06-21
tags: ["Kubernetes"]
---

# Kubernetes (1.22, Single Node, Ubuntu 20.04 LTS, ZFS, Conoha) (Almost IPv6 Only)

## クラスタの構築 (といってもシングルノードだが)

[kubeadmのインストール](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  
[kubeadmを使用したシングルコントロールプレーンクラスターの作成](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)  
を参考にしていく

### Swap をオフにする

```
$ sudo nvim /etc/fstab
```

swap の行の先頭に # を置いて，コメントアウトすればいい

### br_netfilter モジュールを読み込ませる

```
$ sudo nvim /etc/modules
```

つぎを追記する

```
br_netfilter
```

### sysctl の設定

```
$ sudo nvim /etc/sysctl.d/k8s.conf
```

つぎのように

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

書いて

```
$ sudo sysctl --system
```

### iptables をレガシーモードにする

```
$ sudo apt install iptables arptables ebtables
$ sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
$ sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
$ sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
$ sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

### Docker をインストールする

はじめに注意だが，/var/lib/docker を ZFS で構築するのは，おすすめできない  
[docker ZFS driver creates hundreds of datasets and doesn’t clean them #41055](qhttps://github.com/moby/moby/issues/41055)

つぎのようにして回避する  
[taku-n/lesser-zfs-installer](https://github.com/taku-n/lesser-zfs-installer#for-docker)

Docker は，ちょっと前のバージョンを入れるので注意

基本的には，こちらに従う:  
[CRIのインストール](https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/)  

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)  
[Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/)  
[「コントロールプロセスがエラーコードで終了したため、docker.serviceのジョブが失敗しました」を修正する方法 ](https://www.366service.com/jp/qa/278a4d77f7955fe8a2f58bb314c71a79)
[Docker snap “failed to mount overlay” and where to report bugs?](https://forum.snapcraft.io/t/docker-snap-failed-to-mount-overlay-and-where-to-report-bugs/20010)

Docker のパッケージのバージョンを固定する

```
$ sudo apt-mark hold containerd.io docker-ce docker-ce-cli
```

/var/lib/docker に ZFS を採用している場合には，つぎのようにする  
(ただし，上記のとおりの問題があるので ext4 を勧めておく

```
$ sudo nvim /etc/docker/daemon.json
```

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "zfs"
}
```

### kubeadm, kubelet, kubectl をインストールする

```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo nvim /etc/apt/sources.list.d/kubernetes.list
```

つぎを書く

```
deb https://apt.kubernetes.io/ kubernetes-xenial main
```

```
$ sudo apt update
$ sudo apt install kubeadm kubectl kubelet
$ sudo apt-mark hold kubeadm kubectl kubelet
```

### IPv6 を有効化

```
$ sudo nvim /etc/sysctl.conf
```

つぎの行を追加

```
net.ipv6.conf.all.forwarding = 1
```

反映

```
$ sudo sysctl -p
```

### コントロールプレーンノード (マスタ) の初期化

```
sudo kubeadm init \
--apiserver-advertise-address 2400:your:ip:v6:a:dd:re:ss \
--apiserver-bind-port 6443 \
--control-plane-endpoint 2400:your:ip:v6:a:dd:re:ss \
--service-cidr fc00::/112 \
--pod-network-cidr fd00::/48
```

設定ファイルを使う場合は，以下のような感じだと思うが，よくわからなかった

```
nvim kubeadm-config.yaml
```

つぎのようにする

```
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
networking:
  serviceSubnet: 10.96.0.0/12,fd00::/108
  podSubnet: 192.168.0.0/16,fd00:1::/48
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
```

この構成ファイルについて詳しくは  
[https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta3](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta3)

つぎのようにして初期化する

```
sudo kubeadm init --config=kubeadm-config.yaml
```

やり直したいときは

```
sudo kubeadm reset
```

としてから，もう一度実行する

```
ERROR FileAvailable--etc-kubernetes-manifests-*
```

が出るときは

```
$ sudo kubeadm reset
$ sudo rm -r /etc/cni/net.d
$ rm ~/.kube/config
```

### kubectl の設定

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### CNI (Container Network Interface) を入れる

Calico を選びました

[Install Calico networking and network policy for on-premises deployments](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises)

```
cd ~/.kube
curl -OL https://docs.projectcalico.org/manifests/calico.yaml
nvim calico.yaml
```

つぎのように変更します

```
--- calico.yaml.orig    2021-09-09 17:22:09.760528348 +0900
+++ calico.yaml 2021-09-09 18:19:40.057414568 +0900
@@ -32,7 +32,9 @@
           "nodename": "__KUBERNETES_NODE_NAME__",
           "mtu": __CNI_MTU__,
           "ipam": {
-              "type": "calico-ipam"
+              "type": "calico-ipam",
+              "assign_ipv4": "true",
+              "assign_ipv6": "true"
           },
           "policy": {
               "type": "k8s"
@@ -3885,9 +3887,11 @@
               value: "ACCEPT"
             # Disable IPv6 on Kubernetes.
             - name: FELIX_IPV6SUPPORT
-              value: "false"
+              value: "true"
             - name: FELIX_HEALTHENABLED
               value: "true"
+            - name: IP6
+              value: "autodetect"
           securityContext:
             privileged: true
           resources:
```

```
$ kubectl apply -f calico.yaml
```

### コントロールプレーンノード (マスタ) に Pod を配置できるようにする

```
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

### 最終確認

```
$ kubectl get pod -A
```

運良く，つぎが表示されれば，ひとまずゴールです

```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-744cfdf676-zkwpz   1/1     Running   0          9m55s
kube-system   calico-node-9d2gw                          1/1     Running   0          10m
kube-system   coredns-74ff55c5b-pxn4n                    1/1     Running   0          38m
kube-system   coredns-74ff55c5b-q2zwx                    1/1     Running   0          38m
kube-system   etcd-www                                   1/1     Running   0          38m
kube-system   kube-apiserver-www                         1/1     Running   0          38m
kube-system   kube-controller-manager-www                1/1     Running   0          38m
kube-system   kube-proxy-j7v5d                           1/1     Running   0          38m
kube-system   kube-scheduler-www                         1/1     Running   0          38m
```

## ほかにやっておきたいこと

### calicoctl をインストール

[Install calicoctl](https://docs.projectcalico.org/getting-started/clis/calicoctl/installhttps://docs.projectcalico.org/getting-started/clis/calicoctl/install)

### ノード内に入るための Ubuntu を動かす (シングルノード仕様

マルチノード構成で使うなら，ノードセレクタを追加されたい

ZFS 環境なら /k8s にファイルシステムをつくっておくといいと思う

```
nvim ubuntu.yaml
```

```
apiVersion: apps/v1

kind: Deployment

metadata:
  name: ubuntu

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        volumeMounts:
        - name: ubuntu
          mountPath: /root
        command: ["/bin/sleep", "infinity"]
      volumes:
      - name: ubuntu
        hostPath:
          path: /k8s/ubuntu/root
```

```
kubectl apply -f ubuntu.yaml
```

```
$ kubectl get pod  # NAME を調べておく
$ kubectl exec ubuntu-123456789a-12345 -it -- bash
```

コンテナのホームディレクトリ /root には  
ノードから /k8s/ubuntu/root でアクセスできる

### Docker Registry を動かす (シングルノード仕様

マルチノード構成で使うなら，ノードセレクタを追加されたい

ZFS 環境なら /k8s にファイルシステムをつくっておくといいと思う

/etc/docker/daemon.json に，insecure-registries を下記のように追記する  
(認証を行わないようにするため)  

storage-driver の行の末尾のカンマに注意  
localhost の部分は，IP アドレスでもホスト名でもいい

`$ sudo nvim /etc/docker/daemon.json`

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "insecure-registries": ["ip6-localhost:5000"],
  "ipv6": true,
  "fixed-cidr-v6": "fd01::/64"
}
```

`$ systemctl reload docker`

`$ nvim registry.yaml`

```
apiVersion: apps/v1

kind: Deployment

metadata:
  name: registry

spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry
  template:
    metadata:
      labels:
        app: registry
    spec:
      hostNetwork: true  # ノードからアクセス可能にする  $ curl -v http://ip6-localhost:5000/  でテスト
      containers:
      - name: registry
        image: registry
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: registry
          mountPath: /var/lib/registry
      volumes:
      - name: registry
        hostPath:
          path: /k8s/registry
```

`$ kubectl apply -f registry.yaml`

動作しているかのテスト

```
$ curl -v http://ip6-localhost:5000/
< HTTP/1.1 200 OK
```

Push のテスト  
localhost の部分は，/etc/docker/daemon.json で設定したものにする  
(ホスト名にした場合は，なぜか localhost でもいけた

```
$ docker pull hello-world
$ docker images | grep hello-world  # hello-world の IMAGE ID を調べる
$ docker tag 123456789abc ip6-localhost:5000/hello-world
$ docker push ip6-localhost:5000/hello-world
$ ls /k8s/registry/docker/registry/v2/repositories
hello-world
$ curl http://ip6-localhost:5000/v2/_catalog
{"repositories":["hello-world"]}
```

http: server gave HTTP response to HTTPS client  
となってしまった場合は，/etc/docker/daemon.json の設定ミスである

Pull のテスト

```
$ kubectl run hello-world --image=ip6-localhost:5000/hello-world -it --restart=Never --rm
```

ちなみにここまでの :5000 が無いと，:80 を見に行く模様

最後に，クラスタに外部から ポート5000番 でアクセスできないようにしておく  
さもないと，知らないだれかに外から Push されてしまう

## Ingress

### MetalLB

[Site](https://metallb.universe.tf/)

```
curl -OL https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
curl -OL https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
kubectl apply -f namespace.yaml
kubectl apply -f metallb.yaml
nvim config.yaml
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 2400:your:ip:v6:a:dd:re:ss-2400:your:ip:v6:a:dd:re:ss
```

```
kubectl apply -f config.yaml
```

### NGINX Ingress Controller

[Site](https://kubernetes.github.io/ingress-nginx/)

Docker Desktop のものを使う

```
curl -OL https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f deploy.yaml
```

これで Ingress が使えるようになった (もちろん IPv6 Only)

バージョンの確認 (?)

```
$ kubectl get pod -A  # NAME の確認
$ kubectl exec -n ingress-nginx ingress-nginx-controller-123456789a-bcdef -it -- /nginx-ingress-controller --version
```

#### 検証

前提: 事前にサーバのポート 30000 - 32767 が開いているか確認しておく

```
nvim hello.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: svc-hello
spec:
  selector:
    app: hello
  ports:
  - protocol: TCP
    port: 8080
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: gcr.io/google-samples/hello-app:1.0
```

```
$ kubectl apply -f hello.yaml
$ kubectl get svc
svc-hello    NodePort    fc00::b136   <none>        8080:31234/TCP   14m
```

svc-hello の NodePort のポート番号 (30000 - 32767) を確認しておく  
外部から curl する

```
curl -6 -v http://[2400:your:ip:v6:a:dd:re:ss]:31234/
```

```
Hello, world!
Version: 1.0.0
Hostname: hello-123456789a-bcdef
```

こんな感じで返ってくれば，とりあえずサービスは動いている

```
nvim ingress.yaml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-hello
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: hoge.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-hello
            port:
              number: 8080
```

```
kubectl apply -f ingress.yaml
curl -6 -v http://hoge.com/
```

```
Hello, world!
Version: 1.0.0
Hostname: hello-123456789a-bcdef
```

さきほどと同じものが返ってくれば，Ingress 経由でアクセスできたことになる

#### TLS

```
sudo kubectl --kubeconfig /home/user/.kube/config create secret tls tls-certificate --key /etc/letsencrypt/live/hoge.com/privkey.pem --cert /etc/letsencrypt/live/hoge.com/fullchain.pem
```

```
cp ingress.yaml ingress-tls.yaml
nvim ingress-tls.yaml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-tls-hello
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - hoge.com
    secretName: tls-certificate
  rules:
  - host: hoge.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-hello
            port:
              number: 8080
```

```
kubectl delete -f ingress.yaml
kubectl apply -f ingress-tls.yaml
```

```
curl -6 -v -L http://hoge.com/
```

HTTPS にリダイレクトされて，さきほどと同じ結果が返ってくれば完了  

#### 自動更新させる (ConoHa 限定)

[こちら](../tls/) の renew.sh に追記する

```
#!/bin/bash

echo "Starting certbot."

certbot renew --manual \
        --manual-auth-hook /etc/letsencrypt/authenticator.py \
        --manual-cleanup-hook /etc/letsencrypt/cleanup.py

echo "certbot finished."

kubectl --kubeconfig /home/user/.kube/config delete secret tls-certificate
kubectl --kubeconfig /home/user/.kube/config create secret tls tls-certificate \
        --key /etc/letsencrypt/live/hoge.com/privkey.pem \
        --cert /etc/letsencrypt/live/hoge.com/fullchain.pem

# Print expiring dates
openssl x509 -in /etc/letsencrypt/live/hoge.com/fullchain.pem -noout -dates

# To see a log by journalctl, type "journalctl -e -u letsencrypt".
# A log file of certbot is /var/log/letsencrypt/letsencrypt.log.
# A log file of hooks is /etc/letsencrypt/log.txt.
```

```
kubectl delete secret tls-certificate
curl -6 https://hoge.com/
```

エラーになることを確認してから

```
sudo systemctl start letsencrypt.service
curl -6 https://hoge.com/
```

正常な応答があれば完了  
こちらで動作の確認もしておく

```
systemctl status letsencrypt.service
```

## クラスタのアップグレード (あまり自信はない

公式のドキュメントはこちら [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

最新のバージョンを [https://kubernetes.io/ja/](https://kubernetes.io/ja/) の右上の `バージョン` で確認

バージョンが固定されているパッケージの確認

```
$ sudo apt-mark showhold
```

基本的なパッケージのバージョンの確認

```
$ apt list --installed | \
grep -e containerd.io -e docker-ce -e docker-ce-cli -e kubeadm -e kubectl -e kubelet
```

kubeadm のバージョンが一致していることを確認

```
$ kubeadm version
```

アップグレードできるかの確認

```
$ sudo kubeadm upgrade plan
```

kubelet のバージョンの確認 (たぶんこう

```
$ kubectl get node -o yaml | grep kubelet
```

Control Plane をアップグレードしたあとに実行する必要があるかもしれないコマンド  
(たぶんいらない

```
$ kubeadm upgrade apply
```

### Calico のアップグレード

Calico の最新バージョンの確認

[Release notes](https://docs.projectcalico.org/release-notes/)

Calico のバージョンの確認

```
$ calicoctl version
```

calicoctl のアップグレード

ここから入手して上書きする  
[Install calicoctl](https://docs.projectcalico.org/getting-started/clis/calicoctl/install)

Calico のアップグレード

[Upgrade Calico on Kubernetes](https://docs.projectcalico.org/maintenance/kubernetes-upgrade)

## クラスタの運用

### hello, world

```
$ kubectl run hello-world --image=hello-world -it --restart=Never --rm
```

### whalesay

```
$ kubectl run whalesay --image=docker/whalesay -it --restart=Never --rm -- sh -c "cowsay Hello Kubernetes"
```

### nginx

```
$ kubectl create deployment nginx --image=nginx
```

### busybox

```
$ kubectl run busybox --image=busybox -it --restart=Never --rm -- sh
```

### Ubuntu

```
$ kubectl run ubuntu --image=ubuntu -it --restart=Never --rm -- bash
```

### クラスタの状態を確認

```
$ kubectl cluster-info
```

### ノードの状態を確認

```
$ kubectl get node
```

### すべての処理の状態を確認

```
$ kubectl get all
```

### さらに多くの処理の情報を IP アドレスと共に表示

```
$ kubectl get all -A -o wide
```

### ポッドの状態を確認

```
$ kubectl get pod
```

IP アドレスも表示

```
$ kubectl get pod -o wide
```

### ポッドの起動 (自動的に終了)

```
$ kubectl run hoge --image=hoge -it --restart=Never --rm
```

### ポッドの削除

```
$ kubectl delete pod hoge
```

### デプロイメントの状態を確認

```
$ kubectl get deployment
```

### デプロイメントの起動

```
$ kubectl create deployment hoge --image=hoge
```

### デプロイメントの削除

```
$ kubectl delete deployment hoge
```

### ジョブの状態を確認

```
$ kubectl get job
```

### ジョブの起動

```
$ kubectl create job hoge --image=hoge
```

### ジョブの削除

```
$ kubectl delete job hoge
```

### マニフェストで起動, 起動中のマニフェストを更新

```
$ kubectl apply -f hoge.yaml
```

### マニフェストで削除

```
$ kubectl delete -f hoge.yaml
```

### カレントディレクトリ内のマニフェストで起動, 更新

```
$ kubectl apply -f .
```

### カレントディレクトリ内のマニフェストで削除

```
$ kubectl delete -f .
```

### Pod の標準出力と標準エラー出力をリアルタイムで見る

たとえば nginx の場合

```
$ kubectl get pod
$ kubectl logs nginx-123456789a-12345 -f
```

ちなみに nginx イメージは

```
/var/log/nginx/access.log -> /dev/stdout
/var/log/nginx/error.log -> /dev/stderr
```

となっている  
ただし，nginx において，クラスタの外部からのアクセスの場合，標準エラー出力は出ない  
理由はわからない (IPv6 でアクセスしているからみたい

### 稼働中の Pod のシェルを起動して入る

たとえば

```
$ nvim ubuntu.yaml
```

```
apiVersion: apps/v1

kind: Deployment

metadata:
  name: ubuntu

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["/bin/sleep", "infinity"]
```

```
$ kubectl apply -f ubuntu.yaml
```

に対して

```
$ kubectl get pod
$ kubectl exec ubuntu-12345678-12345 -it -- bash
```

### Ingress のサンプル

$ nvim ingress.yaml

```
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:
  name: ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  rules:
  - host: hoge.com  # Your domain name (subdomain can be added).
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

$ nvim nginx.yaml

```
apiVersion: v1

kind: Service

metadata:
  name: nginx

spec:
  selector:
    app: nginx
  externalIPs:
  - 1.0.0.0  # Your global IP address (これがないせいで外部からつながらずはまった).
  ports:
  - port: 80
    targetPort: 80

---

apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```

### クラスタの停止 (シングルノード構成, 調査中

ノード名は hoge とする

```
$ kubectl get node hoge
```

```
$ kubectl get pod -A
```

ここで表示される Pod がゼロになることを目指す (?)

Pod が生成されないようにする

```
$ kubectl cordon hoge
```

### Rust を動かす

```
cargo new hello
cd hello
cargo build --release
nvim Dockerfile
```

```
FROM gcr.io/distroless/cc-debian10
COPY target/release/hello /
CMD ["/hello"]
```

```
docker build -t ip6-localhost:5000/hello .
docker push ip6-localhost:5000/hello
kubectl run hello --image=ip6-localhost:5000/hello -it --restart=Never --rm
```

## k0s (Single Node, Ubuntu 20.04 LTS) 現状ではうまく動かない (v0.9.1)

[k0s](https://k0sproject.io/)  
[k0s GitHub](https://github.com/k0sproject/k0s)

### PATH が通っていなければ通しておく

```
$ nvim ~/.bashrc
```

つぎを追記

```
export PATH=~/.local/bin:$PATH
```

sudo 時に PATH が通らなくなるのをなんとかする

```
$ sudo visudo
```

```
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

の部分を

```
#Defaults       secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        env_keep +="PATH"
```

に変更

### ディレクトリがなければつくる

```
$ mkdir -p ~/.local/bin
$ mkdir -p ~/.local/k0s
$ mkdir -p ~/.kube
```

### ダウンロードとインストール

```
$ cd ~/.local/k0s
$ # バージョンはこちらで各自確認 https://github.com/k0sproject/k0s/releases/latest
$ curl -OL https://github.com/k0sproject/k0s/releases/download/v0.9.0/k0s-v0.9.0-amd64
$ chmod +x k0s-v0.9.0-amd64

$ cd ~/.local/bin
$ ln -s ~/.local/k0s/k0s-v0.9.0-amd64 k0s

$ sudo visudo
```

sudo 時のパスワードがだるいのでつぎを追記

```
your-username ALL=(ALL:ALL) NOPASSWD: /home/your-username/.local/bin/k0s
```

```
$ cd ~/.kube
$ k0s default-config > k0s.yaml
```

### バージョン確認

```
$ k0s version
```

### 起動

```
$ cd ~/.kube  # k0s.yaml のあるディレクトリに移動しておく
$ sudo k0s server --enable-worker
```

または

```
$ sudo k0s server -c ~/.kube/k0s.yaml --enable-worker
```

### k0s.yaml

```
apiVersion: k0s.k0sproject.io/v1beta1
images:
  konnectivity:
    image: us.gcr.io/k8s-artifacts-prod/kas-network-proxy/proxy-agent
    version: v0.0.13
  metricsserver:
    image: gcr.io/k8s-staging-metrics-server/metrics-server
    version: v0.3.7
  kubeproxy:
    image: k8s.gcr.io/kube-proxy
    version: v1.20.1
  coredns:
    image: docker.io/coredns/coredns
    version: 1.7.0
  calico:
    cni:
      image: calico/cni
      version: v3.16.2
    flexvolume:
      image: calico/pod2daemon-flexvol
      version: v3.16.2
    node:
      image: calico/node
      version: v3.16.2
    kubecontrollers:
      image: calico/kube-controllers
      version: v3.16.2
installConfig:
  users:
    etcdUser: etcd
    kineUser: kube-apiserver
    konnectivityUser: konnectivity-server
    kubeAPIserverUser: kube-apiserver
    kubeSchedulerUser: kube-scheduler
kind: Cluster
metadata:
  name: k0s
spec:
  api:
    address: 118.27.29.81
    externalAddress: ""
    sans:
    - 118.27.29.81
    - 118.27.29.81
    extraArgs: {}
  controllerManager:
    extraArgs: {}
  scheduler:
    extraArgs: {}
  storage:
    type: etcd
    kine: null
    etcd:
      peerAddress: 118.27.29.81
  network:
    podCIDR: 10.244.0.0/16
    serviceCIDR: 10.96.0.0/12
    provider: calico
    calico:
      mode: vxlan
      vxlanPort: 4789
      vxlanVNI: 4096
      mtu: 1450
      wireguard: false
      flexVolumeDriverPath: /usr/libexec/k0s/kubelet-plugins/volume/exec/nodeagent~uds
      withWindowsNodes: false
  podSecurityPolicy:
    defaultPolicy: 00-k0s-privileged
  workerProfiles: []
telemetry:
  interval: 10m0s
  enabled: true
```

### kubectl のインストール

```
$ sudo snap install --classic kubectl
```

### kubectl のバージョンの確認

```
$ kubectl version --client
```

### kubectl の設定

```
$ sudo cat /var/lib/k0s/pki/admin.conf > ~/.kube/config
```

### kubectl の現在の設定を確認

```
$ kubectl config view
```

### kubectl が正常に設定されているか確認

```
$ kubectl cluster-info
```

### クラスタの状態の確認

```
$ kubectl get pods --all-namespaces
```

または

```
$ kubectl get pods -A
```

IP アドレスも表示

```
$ kubectl get pods -A -o wide
```

### ポッドの総合的な情報を確認

```
$ kubectl get all -n NAMESPACE
```

### ポッドの状態の確認

```
$ kubectl get pod -n NAMESPACE NAME
```

### ポッドの詳細を確認

```
$ kubectl describe pod -n NAMESPACE NAME
```

## kind

[kind](https://kind.sigs.k8s.io/)

### 下準備

Go と Docker を入れておきます  
つぎのように設定しました

```
export GOPATH=~/.local/gopath
export PATH=~/.local/gopath/bin:$PATH
```

WSL2 の場合，eth0 の IP アドレスが毎回変わってしまうため，ダメみたい

## Kubernetes のインストール (Ubuntu 20.04 LTS)

[公式](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  
[CL LAB](https://www.creationline.com/lab/36769)
[VA Linux](https://valinux.hatenablog.com/entry/20200722)

### systemd をインストールする

こちらを参考にして dotnet-sdk-3.1 をインストールする  
[Snow System](https://snowsystem.net/other/windows/wsl2-ubuntu-systemctl/)

dotnet-sdk-3.1 をインストールしたあと  
~/.bashrc につぎの環境変数を追加する

```
export DOTNET_CLI_TELEMETRY_OPTOUT=true
```

そのあとファイルをつぎの内容で作成する

```
$ sudo nvim /etc/apt/sources.list.d/wsl-translinux.list
```

```
deb [trusted=yes] https://wsl-translinux.arkane-systems.net/apt/ /
```

そして

```
$ sudo apt install systemd-genie
```

そのあと  
[Snow System](https://snowsystem.net/other/windows/wsl2-ubuntu-systemctl/)  
を参考にして自動的に実行されるようにする

tmux を使っているなら日本語が文字化けするようになるので  
.bashrc につぎを追加

```
alias tmux="tmux -u"
```

pstree で systemd が実行されていることを確認

```
$ pstree
```

## SWAP をオフにする

[Snow System](https://snowsystem.net/other/windows/wsl2-swap-off/)

## Docker をインストールする

先に Docker をインストールしておきます  
ここは Docker 公式サイトの通りにやれば大丈夫です

そのあとに  
[KubernetesをDual Stackで動かす](https://imokuri123.com/blog/2020/01/kubernetes-dual-stack/)  
の JSON を設定して

```
$ sudo mkdir -p /etc/systemd/system/docker.service.d
$ systemctl daemon-reload
$ systemctl restart docker
```

### ネットワークの設定

```
$ sysctl -a | grep net.bridge
```

を実行して

```
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

となっているのを確認する  
また

```
$ sudo nvim /etc/sysctl.conf
```

で

```
#net.ipv6.conf.all.forwarding=1
```

の # を削除する

```
$ sudo sysctl -p
$ sudo sysctl -a | grep net.ipv6.conf.all.forwarding
```

を実行して

```
net.ipv6.conf.all.forwarding = 1
```

となっているのを確認する

### nftables を使わないようにする

iptables の代替として開発されている nftables は  
Kubernetes は対応していないので  
使わないように設定しておく

```
$ sudo apt update
$ sudo apt full-upgrade
$ sudo apt install iptables arptables ebtables
```

```
$ sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
$ sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
$ sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
$ sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

### kubeadm, kubectl, kubelet のインストール

```
$ sudo apt install apt-transport-https curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo apt update
$ sudo apt install kubeadm kubectl kubelet
```

## Kubernetes の初期設定

Kubernetes ではまずクラスタを構築する必要があるようだ  
構築するには公式ツールの [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) を使うことができる

```
ERROR FileAvailable--etc-kubernetes-manifests-*
```

が出るときは

```
$ sudo kubeadm reset
$ sudo rm -r /etc/cni/net.d
$ rm ~/.kube/config
```

とする

```
$ mkdir -p ~/.kube
$ sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
$ sudo chown $(id -u):$(id -g) ~/.kube/config
```

```
$ kubectl get node -o wide  # sudo をつけないように注意
```

[How to enable IPv6 on Kubernetes (aka dual-stack cluster)](https://medium.com/@elfakharany/how-to-enable-ipv6-on-kubernetes-aka-dual-stack-cluster-ac0fe294e4cf)
[Project CalicoをKubernetesで使ってみる：構築編](https://thinkit.co.jp/article/14638)

calicoctl のインストール  
バージョンはこちらで確認する  
[https://github.com/projectcalico/calicoctl/releases/](https://github.com/projectcalico/calicoctl/releases/)

```
$ cd ~/.local/bin
$ curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/vx.y.z/calicoctl
$ chmod +x calicoctl
$ sudo nvim /etc/calico/calicoctl.cfg
```

```
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "kubernetes"
  kubeconfig: "/home/your-name/.kube/config"
```

```
$ calicoctl get nodes  # Test
```

Master にも Pod がデプロイされるようにする

[kubeadm を使って Kubernetes v1.13 をインストールしてみた](https://tech-lab.sios.jp/archives/13555)
[lab2.1 kubectl untainted not working](https://forum.linuxfoundation.org/discussion/846483/lab2-1-kubectl-untainted-not-working)

```
$ kubectl get nodes
```

でマスターの名前を確認  
接続先を指定する場合は

```
$ kubectl -s localhost:8080 get nodes
```

```
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

```
$ kubectl -n kube-system get cm kubeadm-config -oyaml
```

```
$ kubectl config view  # kubectl の設定を表示 (~/.kube/config)
```

### クラスタ設定用の設定ファイルを書く

[kubeadmを使ったコントロールプレーンの設定のカスタマイズ](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/control-plane-flags/)  
[package v1beta2](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2)

```
$ mkdir ~/k8s
$ cd ~/k8s
$ nvim kubeadm-init-conf.yaml
```

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
```

```
$ chmod +x kubeadm-init-conf.yaml
$ sudo ./kubeadm-init-conf.yaml
```

```
$ sudo kubeadm init \
> --feature-gates="IPv6DualStack=true" \
> --pod-network-cidr="fd00:1::/48,10.244.0.0/16" \
> --service-cidr="fd00::/108,10.96.0.0/12" \
> --apiserver-advertise-address="2001:db8::11"
```

```
$ sudo systemctl status kubelet
$ sudo journalctl -xeu kubelet
$ sudo docker ps -a | grep kube | grep -v pause
$ sudo docker logs CONTAINERID
```

```
$ kubeadm version
```

```
$ kubectl config view
```

マニフェストの場所: /etc/kubernetes/manifests

よくわからないが Minikube を目指すのがいいらしい

環境:  
Ubuntu 20.04 LTS on WSL2  
Docker 19.03.11

[Nested Virtualization for WSL2 VM](https://github.com/microsoft/WSL/issues/4193)

## Minikube

### Minikube のインストール

#### 仮想化のサポートを確認

コマンド プロンプト でこちらを実行

```
>systeminfo
```

```
Hyper-V の要件: ハイパーバイザーが検出されました。Hyper-V に必要な機能は表示されません。
```

が表示されることを確認

```
$ sudo kvm-ok
```

#### kubectl のインストール

```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
```

```
$ kubectl version --client
```

を実行して確認

[kubectlのインストールおよびセットアップ](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)

#### KVM のインストール

```
$ sudo apt -y install qemu-kvm libvirt-daemon bridge-utils virtinst libvirt-daemon-system
$ sudo apt -y install virt-top libguestfs-tools libosinfo-bin  qemu-system virt-manager
```

[Install KVM Hypervisor on Ubuntu 20.04 (Focal Fossa)](https://computingforgeeks.com/install-kvm-hypervisor-on-ubuntu-focal-fossa/)

#### Minikube のインストール

```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
$ sudo dpkg -i minikube_latest_amd64.deb
$ sudo apt install -y conntrack
```

```
$ sudo minikube start --vm-driver=none
```

を実行して確認

[minikube start](https://minikube.sigs.k8s.io/docs/start/)

[Minikubeのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-minikube/)
