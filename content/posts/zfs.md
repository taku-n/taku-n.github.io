---
title: "Oracle に怯えながら ZFS を使う (((ﾟДﾟ;)))"
date: 2020-04-29
tags: ["ZFS", "Ubuntu", "Linux"]
---

# Oracle に怯えながら ZFS を使う (((ﾟДﾟ;)))

"Don't use ZFS." - Linus Torvalds

## パーティションの情報を集めたい

### dmesg

```
$ dmesg | grep sd
```

### /dev

```
$ la /dev
```

### gdisk

```
$ sudo gdisk -l /dev/sda
```

### df

```
$ df -hT
```

### du

```
$ sudo du -hx -d1 /
```

### mount

```
$ mount | grep -e vfat -e zfs | sort
```

### fstab

```
$ cat /etc/fstab
```

### zpool

```
$ zpool status
$ zpool status bpool
$ zpool status rpool
$ zpool list
```

### zfs

```
$ zfs list
$ zfs list -r bpool
$ zfs list -r rpool
$ zfs list -o name,used,compression -r xpool

$ zfs get all xpool
$ zfs get compression xpool
$ zfs get -p name,used xpool
```

## ZFS の設定

### ユーザディレクトリ用の ZFS パーティションを用意する

```
$ cd
$ pwd               # Check your home directory.

$ sudo -i           # To avoid creating ".sudo_as_admin_successful" when you execute zfs commands.
# shopt -s dotglob  # To move dot files too.
# mkdir /tmp/temp
# mv /home/{your-username}/* /tmp/temp

# # These properties are inherited from their pool.
# # compression, devices, xattr, dnodesize, acltype, relatime
# zfs create -o mountpoint=/home/{your-username} rpool/home/{your-username}

# mv /tmp/temp/* /home/{your-username}
# chown {your-username}:{your-username} /home/{your-username}
# exit
$ exit              # To see the files moved.
```

GUI 環境の場合は，まずログアウトしたあと，Ctrl + Alt + F2 を押して，2番目 の仮想端末にログインしてから行う (方がいい気がする)。  
Ubuntu 20.04 LTS の ZFS インストールをした場合には，つぎの手順に従う。  
(かなり無茶してる気がするので，インストール直後が無難)。  
(ユーザが複数いる場合は，やらないで)。  
(/home 用のパーティションがいらないなら，やる必要ない。もともとユーザディレクトリ用の ZFS パーティションは，用意されている)

```
$ sudo -i           # To avoid creating ".sudo_as_admin_successful" when you execute zfs commands.

# shopt -s dotglob  # To move dot files too.
# mkdir /tmp/temp
# mv /home/{your-username}/* /tmp/temp

# umount -l /home/{your-username}
# rm -r /home

# zfs list  # To check 6-characters being used in the next line.
# zfs destroy -r rpool/USERDATA/{your-username}_{6-characters}
# zfs create -o mountpoint=/home rpool/home
# zfs create -o mountpoint=/home/{your-username} rpool/home/{your-username}

# mv /tmp/temp/* /home/{your-username}
# chown {your-username}:{your-username} /home/{your-username}

# systemctl reboot
```

### ZFS ファイルシステムのクオータを表示する

```
$ zfs get quota
```

### ZFS ファイルシステムのクオータを 1GB に設定する

```
$ sudo zfs set quota=1G rpool/home
```

### ZFS ファイルシステムのクオータを解除する

```
$ sudo zfs set quota=none rpool/home
```

### ZFS ファイルシステムを作成する

```
$ sudo zfs create -o mountpoint=/usr/home/hoge zroot/usr/home/hoge
```

### ZFS ファイルシステムを削除する

```
$ sudo zfs destroy rpool/hoge
```

## スナップショット

スナップショットの作成

```
$ sudo zfs snapshot bpool@snapyyyymmdd     # スナップショットの名前 (@ よりあと) は数字ではじめない
$ sudo zfs snapshot -r rpool@snapyyyymmdd  # 子孫ファイルシステムのスナップショットも作成
```

スナップショットの確認

```
$ zfs list -t snapshot
```

スナップショットの破棄

```
$ sudo zfs destroy rpool@snapyyyymmdd
```

スナップショットのロールバック

```
$ sudo zfs rollback rpool@snapyyyymmdd
```

[ZFSのsnapshotをまとめて削除する](https://qiita.com/belgianbeer/items/822733b30200a2b6727f)

## ConoHa

### パーティション設計 (for ConoHa)

Logical Sector Size: 512 bytes  
Partitions will be aligned on 2048-sector boundaries  
1MiB =    1048576  
1GiB = 1073741824

|Sector |Beginning Byte|Ending Byte|Size      |Purpose             |Type |Code|Name                |
|------:|-------------:|----------:|---------:|:------------------:|:---:|:--:|:------------------:|
|      0|             0|        511|       512|Master Boot Record  |NA   |NA  |NA                  |
|      1|           512|       1023|       512|First GPT Header    |NA   |NA  |NA                  |
|      2|          1024|           |          |                    |     |    |                    |
|    ...|              |           |     16384|First GPT Entries   |NA   |NA  |NA                  |
|     33|              |      17407|          |                    |     |    |                    |
|     34|         17408|           |          |                    |     |    |                    |
|    ...|              |           |   1031168|BIOS Boot Partition |NA   |EF02|BIOS boot partition |
|   2047|              |    1048575|          |                    |     |    |                    |
|   2048|       1048576|           |          |                    |     |    |                    |
|    ...|              |           |1073741824|EFI System Partition|FAT32|EF00|EFI system partition|
|2099199|              | 1074790399|          |                    |     |    |                    |
|2099200|    1074790400|           |          |                    |     |    |                    |
|    ...|              |           |1073741824|Linux Swap          |Swap |8200|Linux swap          |
|4196351|              | 2148532223|          |                    |     |    |                    |
|4196352|    2148532224|           |          |                    |     |    |                    |
|    ...|              |           |1073741824|ZFS Boot Pool       |ZFS  |BF01|Mac ZFS             |
|6293503|              | 3222274047|          |                    |     |    |                    |
|6293504|    3222274048|           |          |                    |     |    |                    |
|    ...|              |           |        NA|ZFS Root Pool       |ZFS  |BF01|Mac ZFS             |

### conoha-iso

基本はこちらを読めばいい  
[conoha-iso](https://github.com/hironobu-s/conoha-iso)  
[CLIツールで簡単にISOイメージをマウントする](https://support.conoha.jp/v/clitools/)

#### ポイント

- ConoHa で API ユーザを作成しておく
- OS_USERNAME, OS_TENANT_NAME, OS_REGION_NAME を環境変数に追加しておく
- OS_REGION_NAME はエンドポイントの tyo2 など

ISO を ConoHa にダウンロードさせる

```
$ conoha-iso download -p {Your API Password} -i https://releases.ubuntu.com/20.04/ubuntu-20.04-live-server-amd64.iso
```

その確認 (ダウンロード完了まで意外と時間がかかる)

```
$ conoha-iso list -p {Your API Password}
```

ISO を入れる

```
$ conoha-iso insert -p {Your API Password}
```

ISO を出す

```
$ conoha-iso eject -p {Your API Password}
```

## Ubuntu Server 20.04 LTS + ZFS 構築用スクリプト

[lesser-zfs-installer](https://github.com/taku-n/lesser-zfs-installer)
