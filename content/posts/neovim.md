---
title: "Neovim"
date: 2019-12-11T10:20:39+09:00
draft: false
---

# Neovim (0.2.2)

## ウェブサイト

[Neovim](https://neovim.io/)  
[User manual](https://neovim.io/doc/user/)

## バージョンを確認

```
$ nvim --version
```

## 設定ファイル

~/.config/nvim/init.vim

## dein.vim

### インストール

```
$ curl -OL https://raw.githubusercontent.com/Shougo/dein.vim/master/bin/installer.sh
$ sh installer.sh ~/.config/nvim/dein
```

dein.vim 用の設定をしていなければ，表示された指示に従って設定して，そのあと

```
$ nvim
```

```
:call dein#install()
```

### アップデート

```
:call dein#update()
```

## runtimepath の表示

```
:set runtimepath
```

## Neovim に読み込まれているファイルを一覧表示

```
:scriptnames
```

## Vimscript

### hello, world (hello.vim)

```
$ nvim hello.vim
```

```
echo "hello, world"
```

```
$ nvim -S hello.vim
```

### コメント (comment.vim)

```
" 行全体のコメント
echo "hello, world"  |" 行末尾のコメント
```

### 行の継続 (lines.vim)

```
echo "hello,"
    \ "world"  |"=> hello, world
```

### 数を表示 (num.vim)

```
echo 1  |"=> 1
```

### 数を計算 (calc.vim)

```
echo 1 + 2  |"=> 3
```

### 変数 (var.vim)

```
let x = 1
echo x  |"=> 1
```

### if ... endif (if1.vim)

```
let x = 1

if x == 1
  echo 1
endif
```

```
1
```

### if ... else ... endif (if2.vim)

```
let x = 2

if x == 1
  echo 1
else
  echo 2
endif
```

```
2
```

### if ... elseif ... else ... endif (if3.vim)

```
let x = 3

if x == 1
  echo 1
elseif x == 2
  echo 2
elseif x == 3
  echo 3
else
  echo 4
endif
```

```
3
```

### リスト (list.vim)

```
let xs = [1, 2, 3]
echo xs  |"=> [1, 2, 3]
```

### for x in リスト (for1.vim)

```
for i in [1, 2, 3]
  echo i
endfor
```

```
1
2
3
```

### for x in range(a) (for2.vim)

```
for i in range(3)
  echo i
endfor
```

```
0
1
2
```

### for x in range(a, b) (for3.vim)

```
for i in range(1, 3)
  echo i
endfor
```

```
1
2
3
```

### for x in range(a, b, c) (for4.vim)

```
for i in range(1, 9, 3)
  echo i
endfor
```

```
1
4
7
```

### 関数 (引数なし, 戻り値なし) (func1.vim)

```
function! Hello()
  echo "hello, world"
endfunction

call Hello()  |"=> hello, world
```

### 関数 (引数あり, 戻り値あり) (func2.vim)

```
function! Add(x, y)
  return a:x + a:y
endfunction

echo Add(1, 2)
```
