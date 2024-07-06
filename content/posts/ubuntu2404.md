---
title: "Ubuntu2404"
date: 2024-05-31T14:05:50+09:00
draft: true
---

# Ubuntu 24.04 LTS (noble)

Microsoft Store で "Ubuntu 24.04 LTS" をインストール  

```
sudo apt update
sudo apt full-upgrade
sudo apt install aptitude
sudo aptitude update
sudo aptitude full-upgrade
```

[Linuxゲリラ戦記 aptitudeコマンドの使い方](https://linuxgerira.com/linux/aptitude.php)  

```
sudo aptitude install language-pack-en  # en_US.UTF-8
sudo aptitude install language-pack-ja  # ja_JP.UTF-8
sudo update-locale LANG=ja_JP.UTF-8
```

```
sudo aptitude install python-is-python3
sudo aptitude install build-essential
```

~/.ssh 700 に鍵を入れる  
秘密鍵だけが 600 で、あとは 644  

[第 6 章 Secure Shell の管理 (参照)](https://docs.oracle.com/cd/E19683-01/817-0163/6mfvv54nr/index.html)  

```
sudo aptitude install gh
gh auth login
git config --global user.name "Takuto Nanjo"
git config --global user.email "16991946+taku-n@users.noreply.github.com"
```

```
cd
gh repo clone taku-n/.dots
mv .bashrc .bashrc.orig
mv .profile .profile.orig
ln -s .dots/.bashrc .bashrc
ln -s .dots/.profile .profile
```

```
sudo aptitude install neovim
sudo aptitude install lua5.1  # Neovim 内蔵のものと同じバージョン
cd ~/.config
gh repo clone taku-n/nvim
```

Gauche  

```
sudo aptitude install libgdbm-dev zlib1g-dev texinfo
cd ~/.local
mkdir bin
cd bin
curl https://raw.githubusercontent.com/shirok/get-gauche/master/get-gauche.sh -o get-gauche.sh
chmod 755 get-gauche.sh
cd ~/.local
mkdir gauche
cd             # ホームディレクトリに戻らないと、なぜか curl がエラーを吐いたがおま環かもしれない
get-gauche.sh  # インストール先は /home/taku/.local/gauche
vi ~/.dots/.bashrc
```

~/.dots/.bashrc のどこかに書く  

```
# Gauche
export PATH="~/.local/gauche/bin":"$PATH"
```
