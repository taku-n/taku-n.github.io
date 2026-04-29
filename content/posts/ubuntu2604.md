---
title: "Ubuntu 26.04 LTS"
date: 2026-04-28T23:21:34+09:00
draft: true
---

# Ubuntu 26.04 LTS (Resolute Raccoon)

## Installation

[Get Ubuntu on WSL](https://ubuntu.com/download/wsl)  
Install ubuntu-26.04-wsl-amd64.wsl  

## Prevent the login message

```
touch .hushlogin
```

## Update

```
sudo apt update
sudo apt upgrade
```

## Locales

```
sudo apt install language-pack-en  # en_US.UTF-8
sudo apt install language-pack-ja  # ja_JP.UTF-8
sudo update-locale LANG=ja_JP.UTF-8
```

## To build

```
sudo apt install python-is-python3
sudo apt install build-essential
```

## SSH

Put keys in ~/.ssh  

```
chmod 700 ~/.ssh
cd ~/.ssh
chmod 600 private-key
```

## GitHub

```
sudo apt install gh
BROWSER=false gh auth login  ## FYI: false is a command /usr/bin/false
```

Remember the one-time code  
Open a web browser at https://github.com/login/device  
Input the one-time code  

## Git

```
git config --global user.name "Takuto Nanjo"
git config --global user.email "16991946+taku-n@users.noreply.github.com"
```

## Dot files

```
cd
gh repo clone taku-n/.dots
mv .bashrc .bashrc.orig
mv .profile .profile.orig
ln -s .dots/.bashrc .bashrc
ln -s .dots/.profile .profile
```

Restart my terminal  

## Neovim

```
curl --create-dirs --output-dir ~/.local/nvim -OL https://github.com/neovim/neovim/releases/download/
vx.y.z/nvim-linux-x86_64.appimage
mkdir ~/.local/bin
ln -s ~/.local/nvim/nvim-linux-x86_64.appimage ~/.local/bin/vi
gh repo clone taku-n/nvim ~/.config/nvim
sudo apt install lua5.1  # The version embedded in Neovim
```

Restart my terminal  
