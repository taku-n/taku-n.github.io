---
title: "Rust"
date: 2021-10-26
tags: ["Rust"]
---

# Rust

[Trait Dictionary](../trait/)  
[Type Dictionary](../type/)  
[Raw Level Rust](../raw/)  
[On Rust](../macro/)  
[wasm-bindgen](../wasm/)  
[axum](../axum/)  
[warp](../warp/)

## rustup

### バージョンの確認

```
rustup -V
```

### rustup 自身のアップデート

```
rustup self update
```

### Rust のアップデート

```
rustup update
```

### 現在の Toolchain を確認

```
rustup show
```

### 可用な Toolchain の表示

```
rustup target list
```

## Cargo

### ソースコードのチェック

```
cargo clippy
```

### ソースコードを整える

```
cargo fmt
```

### 依存している Crate の表示

```
cargo tree
```

### 依存している Crate のアップデート

```
cargo update
```

[PSA: writing "*" for crates in cargo.toml won't always mean "latest version"](https://www.reddit.com/r/rust/comments/a8kzo6/psa_writing_for_crates_in_cargotoml_wont_always/)

cargo edit の cargo upgrade を使うのがいいらしい

### Binary Crate をつくる

```
cargo new hoge
```

### Library Crate をつくる

```
cargo new --lib hoge
```

### GitHub レポジトリをつくる

#### Binary Crate

```
gh repo create hoge
cargo init hoge
```

#### Library Crate

```
gh repo create hoge
cargo init --lib hoge
```

### Binary を指定して実行

crate/src/bin ディレクトリを作成したあとに  
crate/src/bin/hoge.rs を準備して

```
cargo run --bin hoge
```

### Example を指定して実行

crate/examples ディレクトリを作成したあとに  
crate/examples/hoge.rs を準備して

```
cargo run --example hoge
```

### Test を実行

```
cargo test
```

### Release Build

```
cargo build --release
```

## cargo edit

### Install (Ubuntu 20.04 LTS)

```
sudo apt update
sudo apt full-upgrade
sudo apt install libssl-dev pkg-config
cargo install cargo-edit
```

### Add a Crate

```
cargo add rand
cargo add tokio --features full
cargo add web-sys --features "BinaryType Blob ErrorEvent FileReader"
cargo add hoge --git https://github.com/fuga/hoge
cargo add foo --git https://github.com/bar/foo --branch stable
```

### Upgrade Crates

```
# crates.io ではなく Git 上の Crate を利用している場合には, こちらも実行してから
cargo update

cargo upgrade
```

### Delete a Crate

```
cargo rm hoge
```

## cargo-update

### Install

```
cargo install cargo-update
```

### Update installed excutables via cargo install, including itself

```
cargo install-update --all
```

## Syntax

### self

&self は self: &Self の Syntax Sugar  
Self は呼び出し元の型

[Associated functions & Methods](https://doc.rust-lang.org/rust-by-example/fn/methods.html)

### デバッグとリリースでコードを変更

[How to check release / debug builds using cfg in Rust?](https://coderedirect.com/questions/264155/how-to-check-release-debug-builds-using-cfg-in-rust)

## Tokio

```
cargo new hoge
cargo add tokio --features full
```

```
#[tokio::main]
async fn main() {
}
```
