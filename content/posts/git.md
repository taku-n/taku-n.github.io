---
title: "Git"
date: 2020-06-22
tags: ["Git", "GitHub", "GitLab"]
---

# Git

## GitHub CLI

[GitHub CLI](https://cli.github.com/)

### インストール

```
$ sudo snap install gh
```

### 補完の設定

~/.bashrc に追記

```
eval "$(gh completion -s bash)"
```

### gh auth

まずは，認証する

```
$ gh auth login
```

認証の状態を表示  
-t, --show-token でトークンも表示する

```
$ gh auth status -t
```

### gh ssh-key

GitHub に登録されている公開鍵の一覧を表示

まず権限を付与する

```
$ gh auth refresh -s read:public_key
```

表示する

```
$ gh ssh-key list
```

GitHub に公開鍵を追加する

まず権限を付与する

```
$ gh auth refresh -s write:public_key
```

追加する

```
$ gh ssh-key add ~/.ssh/id_rsa.pub
```

### gh repo

リポジトリの一覧を表示  
-L, --limit int は，表示する最大数を指定

```
$ gh repo list hoge -L 1000
```

リポジトリの概要を表示

```
$ gh repo view hoge/foo
```

リポジトリの作成

```
$ gh repo create foo
```

リポジトリのフォーク

```
$ gh repo fork cli/cli
```

リポジトリのクローン

```
$ gh repo clone hoge/foo
```

レポジトリの削除は，あえてできないようにしているらしい  
[How to delete a repo using gh cli #2461](https://github.com/cli/cli/issues/2461)

### gh gist

gist の一覧を表示する

```
$ gh gist list -L 1000
```

gist を表示する

```
$ gh gist view  # and choose a gist
$ # or
$ gh gist view 0123456789abcdef0123456789abcdef
```

gist を作成する

```
$ gh gist create --public my-file
```

gist を更新する

```
$ gh gist edit 0123456789abcdef0123456789abcdef -a my-file
```

gist を削除する

```
$ gh gist delete 0123456789abcdef0123456789abcdef
```

## GitHub

### 公開鍵認証を利用する

#### テスト

```
$ ssh -T git@github.com
```

### git remote

#### 確認

```
$ git remote -v
```

#### 詳細の確認

```
$ git remote show origin
```

#### 追加

```
$ git remote add origin git@github.com:{user-name}/{repository}.git
```

ここで

```
$ git remote add origin https://github.com/{user-name}/{repository}.git
```

としてしまうと，公開鍵認証ができない

#### 削除

```
$ git remote rm origin
```

### デフォルトブランチ名の変更

#### リモート側

レポジトリのページから  
Settings -> Branches -> Default branch -> Rename branch  
で変更

#### ローカル側

```
$ git branch -m master main
$ git fetch origin
$ git branch -u origin/main main
$ git remote set-head origin -a
```

### よくわからないこまんどたち

```
$ git branch -m main sub
$ git fetch origin
$ git branch -u origin/sub sub
$ git remote set-head origin -a  # -a とは?
```

## GitLab

### 公開鍵認証を利用する

#### 鍵を作成

```
$ ssh-keygen -t rsa -b 4096 -C {your users.noreply.gitlab.com address}
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): /home/username/.ssh/id_rsa_gitlab
Enter passphrase (empty for no passphrase):  # Leave this empty.
Enter same passphrase again:                 # Leave this empty.
```

#### 公開鍵を登録

```
$ cat ~/.ssh/id_rsa_gitlab.pub
```

を実行して，表示された内容を GitLab の自分のアカウントに登録する

#### Git の設定を変更

~/.ssh/config を開いて，つぎの内容を追加

```
Host gitlab gitlab.com
  HostName gitlab.com
  IdentityFile ~/.ssh/id_rsa_gitlab
  User git
```

#### テスト

```
$ ssh -T gitlab
```
