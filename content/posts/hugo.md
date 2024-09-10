---
title: "Hugo"
date: 2019-11-12T15:18:41+09:00
draft: false
---

# Hugo

[Hugo](https://gohugo.io/)  
[Hugo Documentation](https://gohugo.io/documentation/)  

Hugo について，わかったことをメモしたいと思う  
正確かどうかは，わからない

コンテンツの追加

```
$ hugo new content content/posts/my-first-post.md
```

サーバの起動

```
$ hugo server -D  # includes draft content
http://127.0.0.1:1313/
```

ビルド

$ hugo -D
($ hugo となにが違うのだろう? -> ドラフトを含むか含まないか

When you cloned a hugo site you have created, you must enter the directory and type:  
$ git submodule update --init --recursive  

## Delete

Delete a md file and type:  
$ hugo -D --cleanDestinationDir  

## GitHub Actions

[GitHub Actions for Hugo](https://github.com/peaceiris/actions-hugo)  
[GitHub Pages Action](https://github.com/peaceiris/actions-gh-pages)  

Change its branch from master to main.  
Create .github/workflows/gh-pages.yml.  
Push your repository.  
Let GitHub protect main branch with no checks.  
Remove the master branch.  
Open Settings -> Pages and change Branch to main and / (root).  
Open your Settings of your repository.  
Open Environments -> github-pages.  
Change the setting of Deployment branches to All branches.  
Open Actions -> pages build and deployment -> Re-run jobs -> Re-run all jobs -> Re-run jobs.  
Struggle for a while and get the gh-pages branch.  

###### H6 Test (シャープ 6つ だとバグる?
##### H5 Test

Front Matter? => どうやら，各 md ファイルの先頭に自動的に作成される  
3つ のハイフンで区切られたブロックのことみたい  
title, date, draft, ...

たぶん，content ディレクトリ直下にファイルは置けなさそう  
section となるディレクトリを置いていく感じみたい  
_index.md は置けるみたい (Page Bundles -> Branch Bundles

index.md があるディレクトリは Leaf Bundle になり  
_index.md があるディレクトリは Branch Bundle になるみたい

section? => content ディレクトリ直下のディレクトリや content 内で _index.md を包含しているディレクトリのことをいうみたい たぶん

LiveReload が働かない? それ <body></body> がないからです (Getting Started -> Basic Usage

新規作成したページがページリストに出てこない  
config.toml に buildFuture = true を追加する

とりあえず hugo help してみると発見がある

ドキュメント https://gohugo.io/documentation/

### About Hugo  
### Getting Started  
##### Basic Usage (hugo コマンドの基本 読んだほうがいい

### Hugo Modules (モジュールシステムについて とりあえず読まなくてよさそう

### Content Management (一番重要っぽい
##### Content Management Overview (目次
##### Organization (content ディレクトリの基本 必読 index.md と _index.md の違いは Page Bundles で出てくる
##### Page Bundles (index.md と _index.md の違い 正直，よくわからない いろいろ試す必要がありそう
##### Supported Content Formats (HTML も使えるよって話と HTML でも Front Matter はつけてねって話
##### Front Matter (Front Matter は 各 md の先頭に自動的に付加される 3つ のハイフンで囲まれたブロックのこと TOML, YAML, JSON, ORG 好きな形式で書ける どんな値をセットできるかも書いてある
##### Page Resources (画像とかについて 画像とかはメタデータが必要っぽいがよくわからない どこに書くんだこれ
##### Image Processing (なんと画像を引き伸ばしたりもできるらしい
##### Shortcodes (マークダウンの独自拡張 ここはさらっと読んどいたほうがいい 互換性は損なわれるが ぱっと見 相当便利そう 特に gist と シンタックスハイライトと他ページ埋め込みと YouTube
##### Related Content (記事末尾によくある See Also こちらの記事もどうぞ のカスタマイズ とりあえずスルー
##### Sections (_index.md とは? 再び いまだによくわからない Page Bundles と合わせて読むべし
##### Types (まったくわからないが type の値は常に単数形であることはわかった でっていう Archetype とかいうなぞ概念も出てきた
##### Archetypes (hugo new したときの雛形 各セクションごとに準備できるみたい 読んどいたほうがいい
##### Taxonomies (記事をグループ分けしたりする機能 スルー
##### Summaries (要約を設定できるらしい スルー
##### Links and Cross References (リンク用のショートコード マークダウンの独自拡張 ref が絶対参照 relref が相対参照
##### URL Management (ひきつづきリンクの話 結構複雑そうなので 後回しに
##### Menus (メニューについて あとあとつけたくなったら読む
##### Static Files (スタイルシートとか JavaScript とか画像とかを置いておくところについて 知っておく必要があるだろう
##### Table of Contents (Hugo はマークダウンを読み取って勝手に目次をつくってくれる スルー
##### Comments (Disqus みたいなコメント機能について この点は静的サイトジェネレータの弱点だよね 自分で実装した Comento を使うつもり
##### Multilingual Mode (多言語対応について 日本語しか使えないのでスルー;;
##### Syntax Highlighting (シンタックスハイライトの詳細が記されている 必要になったら読みたい

### Templates (ここは読まないとダメでしょうね
##### Templates Overview (目次
##### Introduction (必読です Golang感 がすごい ところでこれらはどこに書いて実行するのだ? => Template Lookup Order に書いてある
##### Template Lookup Order (Introduction にある {{ }} ってやつは layouts/_default/single.html とか layouts/_default/list.html とかに書けばいいとわかる

### Functions  
### Variables  
### Hugo Pipes  
### CLI

### Troubleshooting  
### Tools  
### Hosting & Deployment  
### Contribute

### Maintenance

.bashrc

```
hugo-server () {
	hugo server --bind $1 --baseURL=http://$1/
}
```

### Themes

[Beautiful Hugo](https://themes.gohugo.io/themes/beautifulhugo/)  
[Ananke Gohugo Theme](https://themes.gohugo.io/themes/gohugo-theme-ananke/)  
[Paper](https://themes.gohugo.io/themes/hugo-paper/)  

[The world’s fastest framework for building websites | Hugo](https://gohugo.io/)  
[Hugo 静的サイトジェネレーターによるサイト構築と公開 | パソコン工房 NEXMAG](https://www.pc-koubou.jp/magazine/30737)  
[HUGOで作ったブログにテーマを導入してみよう!](https://zenn.dev/daira0910/articles/51fc5f33eeef05)  

<p>{{ .Title }}<p>
