---
title: "Raw Level Rust"
date: 2020-01-03T03:02:51+09:00
draft: false
---

# Raw Level Rust (1.40)

## ポインタ

### ポインタをつくろう

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

### ポインタを使おう

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

ところで，ここまで，借用からポインタをつくってきました。  
今度は逆に，ポインタから借用をつくってみましょう。

```
let mut x: i32 = 42;
let mut p: *mut i32;
p = &mut x as *mut i32;

let mut y: &mut i32;
unsafe {
    y = &mut *p;
}
println!("{:?}", y);  //=> 42
```

### *mut T型 と *const T型

これまでずっと，ポインタの型として *mut T型 を使ってきました。  
しかし実は，ポインタの型には，もうひとつあります。  
それが，*const T型 です。  
*const T型 を使うと，ポインタの指すデータを書き換えることができなくなります。  
さっそく，試してみましょう。

```
let mut x: i32 = 1;
let mut p: *const i32;
p = &x as *const i32;

unsafe {
    *p = 2;  // error[E0594]: cannot assign to `*p` which is behind a `*const` pointer
}
```

無理に書き換えようとすると，コンパイルエラーが出ました!  
*const T型 のポインタは，参照からつくっていることにも注目してください。  
そして，逆に，参照に変換することもできます。

```
let mut x: i32 = 42;
let mut p: *const i32;
p = &x as *const i32;

let mut y: &i32;
unsafe {
    y = &*p;
}
println!("{:?}", y);  //=> 42
```

### ヌルポインタをつくろう

[std::ptr](https://doc.rust-lang.org/std/ptr/) を使ってつくれます。

```
use std::ptr::*;

let mut p: *const i32 = null();
let mut q: *mut i32   = null_mut();

println!("{:p}", p);  //=> 0x0
println!("{:p}", q);  //=> 0x0
```

簡単ですね。  
ポインタが null かどうか確かめる関数も用意されています。

```
use std::ptr::*;

let p: *const i32 = null();
let q: *mut i32   = null_mut();

println!("{:?}", p.is_null());  //=> true
println!("{:?}", q.is_null());  //=> true
```

### ポインタ演算風

ポインタ演算風をする前に，型のサイズを確認しておきましょう。

```
use std::mem::*;

println!("{:?}", size_of::<i32>());  //=> 4
```

i32型 のサイズは，4バイト であることがあらためてわかりました。  
では，ポインタを 4バイト ずつずらして，i32型 の配列を確認したり，変更したりしてみましょう。

```
let mut xs: [i32; 3] = [1, 2, 3];
let mut p: *mut i32 = xs.as_mut_ptr();

println!("{:p}", p);  //=> 0x7fffeb82bbe4
unsafe {
    println!("{:?}", *p);  //=> 1

    println!("{:p}: {:?}", p.offset(0), *p.offset(0));  //=> 0x7fffeb82bbe4: 1
    println!("{:p}: {:?}", p.offset(1), *p.offset(1));  //=> 0x7fffeb82bbe8: 2
    println!("{:p}: {:?}", p.offset(2), *p.offset(2));  //=> 0x7fffeb82bbec: 3

    *p.offset(1) = 4;
}

println!("{:?}", xs);  //=> [1, 4, 3]
```

型のサイズに合わせて，ポインタが増加しているのがよくわかります。  
では，ポインタの型をキャストして，増加量が変化する様子を見てみましょう。

```
let mut x: u32 = 0x12345678;
let mut p: *mut u32 = &mut x as *mut u32;
let mut q: *mut u8  = p.cast::<u8>();

println!("{:p}", p);  //=> 0x7fffeba3cb04
unsafe {
    println!("0x{:08X}", *p);  //=> 0x12345678

    println!("{:p}: 0x{:02X}", q.offset(0), *q.offset(0));  //=> 0x7fffeba3cb04: 0x78
    println!("{:p}: 0x{:02X}", q.offset(1), *q.offset(1));  //=> 0x7fffeba3cb05: 0x56
    println!("{:p}: 0x{:02X}", q.offset(2), *q.offset(2));  //=> 0x7fffeba3cb06: 0x34
    println!("{:p}: 0x{:02X}", q.offset(3), *q.offset(3));  //=> 0x7fffeba3cb07: 0x12
}
```

今度は u8型 のポインタなので，ポインタが 1バイト ずつ増加しています。

## ビットを味わう

### transmute

[std::mem::transmute](https://doc.rust-lang.org/std/mem/fn.transmute.html) を使うと，ビットの解釈を変更することができます。  
たとえば，u8型 の 長さ4 の配列を u32型として解釈させてみましょう。

```
use std::mem::*;

let mut x: [u8; 4] = [0x78u8, 0x56u8, 0x34u8, 0x12u8];

let mut y: u32;
unsafe {
    y = transmute::<[u8; 4], u32>(x);
}
println!("0x{:08X}", y);  //=> 0x12345678
```

順番が逆転していますが，違う解釈をさせることができました。  
ポインタをキャストしたときの話と合わせると，わかりやすいと思います。

## libc

### libc を使う

[libc](https://crates.io/crates/libc) を使うには，Cargo.toml の [dependencies] のところを，つぎのようにします。

```
[dependencies]
libc = "0.2"
```
