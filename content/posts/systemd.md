---
title: "systemd"
date: 2020-10-12
tags: ["systemd", "Linux"]
---

# systemd

[systemctlとserviceの違い](http://equj65.net/tech/systemd-manage/)

プロセスの確認

```
$ ps aux
$ pstree
```

Unit の一覧と自動起動するかを見る

```
$ systemctl list-unit-files -t service
```

Unit の一覧と起動状態を見る

```
$ systemctl list-units -t service
```

Unit の状態を確認する

```
$ systemctl status [unit-name]
```

Unit の自動起動を有効にする

```
$ systemctl enable [unit-name]
```

Unit の自動起動を無効にする

```
$ systemctl disable [unit-name]
```

Unit を起動する

```
$ systemctl start [unit-name]
```

Unit を停止する

```
$ systemctl stop [unit-name]
```

Unit を再起動する

```
$ systemctl restart [unit-name]
```

```
$ systemctl daemon-reload
```

```
$ systemctl show hoge.timer
```

```
$ journalctl -f -u hoge.service
```

もっといい感じにまとめてくださっている人がいました  
[@sinsengumi](https://qiita.com/sinsengumi/items/24d726ec6c761fc75cc9)  
[SEの道標](https://milestone-of-se.nesuke.com/sv-basic/linux-basic/systemctl/)
[「Systemd」を理解する ーシステム起動編ー](http://equj65.net/tech/systemd-boot/)
[man systemd.unit の訳](https://sites.google.com/site/kandamotohiro/systemd/man-systemd-unit-no-yi)
[Systemdで起動時にスクリプトを実行する](https://students-tech.blog/post/systemd-startup.html)

## Timer

```
$ sudo nvim /etc/systemd/system/hello.service
```

```
[Unit]
Description=Say "hello, world".

[Service]
Type=oneshot
ExecStart=echo "hello, world"

[Install]
WantedBy=multi-user.target
```

```
$ sudo nvim /etc/systemd/system/hello.timer
```

```
[Unit]
Description=Timer to say "hello, world".

[Timer]
OnUnitActiveSec=20s

[Install]
WantedBy=timers.target
```

まずタイマをスタートさせて

```
$ systemctl start hello.timer
```

最後につぎを実行すると，定期実行がはじまる

```
$ systemctl start hello.service
```

再起動しても定期実行されるように登録する

```
$ systemctl enable hello.timer
$ systemctl enable hello.service
```

hello.timer や hello.service を更新したら  
つぎのコマンドで反映させる

```
$ systemctl daemon-reload
```

タイマに関する情報を得る

```
$ systemctl list-dependencies timers.target
```

```
$ systemctl list-timers
```

削除の方法は，未検証

```
$ systemctl disable hello.timer
$ systemctl disable hello.service
$ systemctl stop hello.timer
$ systemctl stop hello.service
```
