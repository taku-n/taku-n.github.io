---
title: "Raw Level Rust"
date: 2020-01-03T03:02:51+09:00
draft: false
---

# Raw Level Rust

## ポインタをつくろう

はじめに i32型 のポインタをつくってみましょう。

```
let mut p: *mut i32;
```

どうということはないですね。  
64bit Linux のユーザ空間のメモリの範囲は，

```
0b00000000_00000000_00000000_00000000_00000000_00000000_00000000_00000000
-
0b00000000_00000000_01111111_11111111_11111111_11111111_11111111_11111111
```

つまりは，

```
0x00_00_00_00_00_00_00_00 - 0x00_00_7F_FF_FF_FF_FF_FF
```

らしいですから 
最後の 4バイト を指すポインタは，つぎのようにしてつくれます。

```
let mut p: *mut i32;
p = 0x00_00_7F_FF_FF_FF_FF_FCusize as *mut i32;
```

メモリのアドレスは，[usize型](https://doc.rust-lang.org/std/primitive.usize.html) を使います。  
usize型 を使えば，メモリのどこであっても指すことができるので，便利です。  
たとえば，64bit Linux であれば，usize型 は，8バイト の大きさをもちます。

さて，ユーザ空間の最後の 4バイト には，i32 で表すと，どんな値が入っているでしょうか。  
さっそく，見てみましょう。

```
let mut p: *mut i32;
p = 0x00_00_7F_FF_FF_FF_FF_FCusize as *mut i32;

let mut x: i32;
unsafe {
    x = *p;  // Segmentation Fault
}
println!("{:?}", x);
```

ポインタが指すアドレスの値を取ってくるのは，危険な行為なので，unsafe ブロックの中でやります。  
コメントが不穏ですが，とにかく実行してみましょう。

```
$ cargo run
Segmentation fault (コアダンプ)
```

[セグメンテーション違反](https://ja.wikipedia.org/wiki/%E3%82%BB%E3%82%B0%E3%83%A1%E3%83%B3%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E9%81%95%E5%8F%8D) が発生してしまいました!  
でも，どこで発生したかは，すぐにわかります。  
これが，Rust の unsafe のよさですね。

これでめげずに，今度は，書き込んでみましょう。

```
let mut p: *mut i32;
p = 0x00_00_7F_FF_FF_FF_FF_FCusize as *mut i32;

unsafe {
    *p = 42;  // Segmentation Fault
}
```

```
$ cargo run
Segmentation fault (コアダンプ)
```

やはり，だめですね。

## ポインタを使おう

今度は，もうちょっとちゃんとポインタを使ってみましょう。

```
let mut a: i32 = 42;
let mut p: *mut i32;
p = &mut a as *mut i32;

let mut x: i32;
unsafe {
    x = *p;
}
println!("{:?}", x);  //=> 42
```

今度は，ちゃんと読み出せました。

書き込みだってできます。

```
let mut x: i32 = 1;
let mut p: *mut i32;
p = &mut x as *mut i32;

unsafe {
    *p = 2;
}
println!("{:?}", x);  //=> 2
```

大丈夫そうですね。

こうなると，気になってくるのはアドレスですよね。  
確かめてみましょう。

```
let mut x: i32 = 42;
let mut p: *mut i32;
p = &mut x as *mut i32;

println!("{:p}", &mut x);  //=> 0x7fffd35bec64
println!("{:p}", p);       //=> 0x7fffd35bec64
```

借用もポインタも，同じ場所を指していることがわかります。
