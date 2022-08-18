---
title: "Raspberry Pi"
date: 2020-08-26
tags: ["Raspberry Pi", "Linux"]
---

# Raspberry Pi

## CPU の温度を見る

```
$ cat /sys/class/thermal/thermal_zone0/temp
```

単位は，ミリ度

## SPEEDTEST CLI

[インストール](https://www.speedtest.net/ja/apps/cli)

```
$ speedtest
```

## Firefox

```
$ sudo apt install firefox-esr-l10n-ja
```

## ALSA (Advanced Linux Sound Architecture)

とりあえず "PulseAudio 音量調節" をインストールすること

```
$ sudo apt install pavucontrol
```

起動は [RasPi] -> [サウンドとビデオ] -> [PulseAudio 音量調節]  
マイクの音量を 100% を超えて設定したりできる

使用したハードウェア  
スピーカ: サンワサプライ株式会社 USBスピーカー MM-SPU8BK  
マイク: USB Microphone MI-305

PCM (Pulse Code Modulation)

テスト用の音源をダウンロード  
(Thank [東京都市生活 + 真空管Audio + 視線入力 + 音楽 + 英国車](http://www.op316.com/))

```
$ curl -OL http://www.op316.com/tubes/tips/data/1khz-0db-30sec.wav
```

```
$ lsusb
```

### Play

可変なデバイス一覧

```
$ aplay -l
```

```
$ aplay --device=hw:[card number],[device number] [wave file]
$ # i.e. $ aplay --device=hw:0,0 1khz-0db-30sec.wav
```

不変なデバイス一覧

```
$ aplay -L
```

```
$ aplay --device=[device for playing] [wave file]
$ # i.e. $ aplay --device=sysdefault:CARD=MicroII 1khz-0db-30sec.wav
$ # i.e. $ aplay --device=front:CARD=MicroII,DEV=0 1khz-0db-30sec.wav
```

```
$ speaker-test -c [card name]
$ # i.e. $ speaker-test -c MicroII
$ # Ctrl-C で，何秒か後に止まる
```

設定を見る

```
$ cat /proc/asound/cards
```

```
$ amixer -c [card name]
$ # i.e. $ amixer -c MicroII
```

Playback の値が音量 (0 が無音)  
Playback の行の最後にある [on] や [off] は，ミュートの設定を表す  
[on] でミュートのチェックが外れた状態，[off] でミュートにチェックが入った状態

```
$ alsamixer
```

上下キーで音量の変更  
m キーでミュートの設定 (00 はミュートのチェックが外れた状態，MM でミュートにチェックが入った状態)

### Record

```
$ arecord -l
```

```
$ arecord -L
```

下記の PulseAudio をインストールしたらうまくいった気がする

```
$ amixer sget Capture
```

マイクのテストをする

```
$ sox -t alsa plughw:[card name] -n -S stats
$ # i.e. $ sox -t alsa plughw:Headset -n -S stats
```

### Settings

/usr/share/alsa/alsa.conf  
/etc/alsa/conf.d/*.conf
~/.asoundrc

## PulseAudio

```
$ sudo apt install pulseaudio
```

設定を見る (* が付いているものがデフォルトの再生デバイス)

```
$ pacmd list-sinks
```

[PulseAudio でデフォルトの入出力先を切り替える](https://qiita.com/propella/items/4699eda71cd742cba8d3#pulseaudio-%E3%81%A7%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%AE%E5%85%A5%E5%87%BA%E5%8A%9B%E5%85%88%E3%82%92%E5%88%87%E3%82%8A%E6%9B%BF%E3%81%88%E3%82%8B)

## sox

```
$ sudo apt install sox
```

```
$ play 1khz-0db-30sec.wav
```

```
$ rec record.wav
$ # Ctrl-C で録音の停止
```
