---
title: "Trait Dictionary"
date: 2021-10-27
tags: ["Trait", "Rust"]
---

# Trait Dictionary

Rust のトレイトは "[トレイトとしては別物](https://ja.wikipedia.org/wiki/%E3%83%88%E3%83%AC%E3%82%A4%E3%83%88)" らしい

## Trait

Trait T を定義

```
trait T {
}
```

Trait S を要求する Trait T を定義  
Trait S は Trait T の Supertrait  
Trait T は Trait S の Subtrait

```
trait S {
}

trait T: S {
}
```

Trait R と Trait S を要求する Trait T を定義

```
trait R {
}

trait S {
}

trait T: R + S {
}
```

<a id="std-clone-clone"></a>
## Clone (std::clone::Clone)

```
pub trait Clone {
    fn clone(&self) -> Self;

    fn clone_from(&mut self, source: &Self) {...}
}
```

```
#[derive(Debug)]
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };
    let t = s;

    dbg!(s);  // Error.
    dbg!(t);
}
```

こうすると Move された s を使ったところでエラーとなってしまうが  
Clone を実装してから工夫すれば実行できる

```
#[derive(Clone, Debug)]
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };
    let t = &s.clone();

    dbg!(s);  //=> s = S {
              //=>     x: 1,
              //=> }
    dbg!(t);  //=> t = S {
              //=>     x: 1,
              //=> }
}
```

ちなみに &s.clone() は s.clone() でもいい

いま Struct S のインスタンスを s とすると

Struct S のメソッド fn method(self) {} は  
s.method() でも &s.method() でも呼べる  
&s.method() とした呼んだ場合は  
s.method() として呼んだものとして処理される

Struct S のメソッド fn method(&self) {} は  
s.method() でも &s.method() でも呼べる  
s.method() とした呼んだ場合は  
&s.method() として呼んだものとして処理される

Rust がよしなにしてくれる

Required:  
[Copy (std::marker::copy](#std-marker-copy)

[Cloning a reference and method call syntax in Rust](https://www.fpcomplete.com/blog/cloning-reference-method-calls/)  
[[Rust] メソッド呼び出し時におけるメソッド探索の仕組み: 自動参照 & 自動参照外し及び Unsized 型強制](https://qiita.com/kerupani129/items/8dba9f5bb2c009c4d08d)

<a id="std-marker-copy"></a>
## Copy (std::marker::Copy)

```
pub trait Copy: Clone {}
```

```
#[derive(Debug)]
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };
    let t = s;

    dbg!(s);  // Error.
    dbg!(t);
}
```

こうすると Move された s を使ったところでエラーとなってしまうが  
Copy を実装すれば実行できる

```
#[derive(Copy, Clone, Debug)]
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };
    let t = s;

    dbg!(s);  //=> s = S {
              //=>     x: 1,
              //=> }
    dbg!(t);  //=> t = S {
              //=>     x: 1,
              //=> }
}
```

Copy は Clone を要求していることに注意したい

Copy を実装できない例

```
#[derive(Copy, Clone)]
struct S {
    x: Box<i32>,  // Error.
}
```

Clone は実装できる

```
#[derive(Clone)]
struct S {
    x: Box<i32>,
}
```

Requires:  
[Clone (std::clone::Clone](#std-clone-clone)

[RustのCloneとCopyについての素朴な疑問](https://teratail.com/questions/253918)

<a id="std-fmt-debug"></a>
## Debug (std::fmt::Debug)

```
pub trait Debug {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), Error>;
}
```

```
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };

    dbg!(s);  // Error.
}
```

こうするとエラーになってしまうが  
Debug を実装すれば実行できる

```
#[derive(Debug)]
struct S {
    x: i32,
}

fn main() {
    let s = S {
        x: 1,
    };

    dbg!(s);  //=> s = S {
              //=>     x: 1,
              //=> }
}
```

<a id="std-ops-deref"></a>
## Deref ([std::ops::Deref](https://doc.rust-lang.org/std/ops/trait.Deref.html))

```
pub trait Deref {
    type Target: ?Sized;
    fn deref(&self) -> &Self::Target;
}
```

```
let b = Box::new(1);
dbg!(*b)  //=> *b = 1
```

これができるのは Deref のおかげ

[Derefトレイトでスマートポインタを普通の参照のように扱う](https://doc.rust-jp.rs/book-ja/ch15-02-deref.html)

<a id="std-ops-fn"></a>
## Fn (std::ops::Fn)

```
pub trait Fn<Args>: FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}
```

```
fn main() {
    let mut n = 5;
    {
        let mut add = |x| n += x;
        add(5);
        dbg!(n);  // n = 10
    }
    dbg!(n);  // n = 10
}
```

```
fn main() {
    let mut n = 5;
    {
        let mut add = move |x| n += x;  // Copy of n occured because it is a move closure.
        add(5);  // This does not change the original value of n but changes copied value of n.
        dbg!(n);  //=> n = 5  // Original value of n.
    }
    dbg!(n);  //=> n = 5  // Original value of n.
}
```

Requires:  
[FnMut (std::ops::FnMut)](#std-ops-fnmut)  

[Trait std::ops::Fn](https://doc.rust-lang.org/std/ops/trait.Fn.html)  
[Rustのクロージャtraitについて調べた(FnOnce, FnMut, Fn)](https://qiita.com/shortheron/items/c1735dc4c7c78b0b55e9)  
[Rust の Fn, FnMut, FnOnce の使い分け](https://kanejaku.org/posts/2018/12/rust-closure-types/)  
[Rustで高階関数を使う](https://bablovia.hatenablog.com/entry/2020/11/17/165344)
[Rustのクロージャ3種を作って理解する](https://keens.github.io/blog/2016/10/10/rustnokuro_ja3tanewotsukutterikaisuru/)

<a id="std-ops-fnmut"></a>
## FnMut (std::ops::FnMut)

```
pub trait FnMut<Args>: FnOnce<Args> {
    extern "rust-call" fn call_mut(
        &mut self, 
        args: Args
    ) -> Self::Output;
}
```

Requires:  
[FnOnce (std::ops::FnOnce](#std-ops-fnonce)

Required:  
[Fn (std::ops::Fn)](#std-ops-fn)

[Trait std::ops::FnMut](https://doc.rust-lang.org/std/ops/trait.FnMut.html)

<a id="std-ops-fnonce"></a>
## FnOnce (std::ops::FnOnce)

```
pub trait FnOnce<Args> {
    type Output;
    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
```

Required:  
[FnMut (std::ops::FnMut)](#std-ops-fnmut)

[Trait std::ops::FnOnce](https://doc.rust-lang.org/std/ops/trait.FnOnce.html)

<a id="std-convert-from"></a>
## From (std::convert::From)

```
pub trait From<T> {
    fn from(T) -> Self;
}
```

おなじみのやつ

```
let s = String::from("str");
```

Complementary:  
[Into (std::convert::Into](#std-convert-into)

<a id="std-future-future"></a>
## Future (std::future::Future)

```
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Enum Poll は

```
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

[Rust でお気楽非同期プログラミング](https://qiita.com/Kumassy/items/fec47952d70b5073b1b7)  
[Rustの非同期プログラミングをマスターする](https://tech-blog.optim.co.jp/entry/2019/11/08/163000)  
[非同期 Rust パターン](https://qiita.com/legokichi/items/4f2c09330f90626600a6)

<a id="std-convert-into"></a>
## Into (std::convert::Into)

```
pub trait Into<T> {
    fn into(self) -> T;
}
```

変換先の型を推論して，よしなに変換してくれる。強い

たとえば

```
web_sys::console::log_1(data_1: &JsValue)
```

に参照を渡したいとき &JsValue ってなにとなるが，とりあえず into() しておけば，勝手にうまいことやってくれるかもしれない

```
console::log_1(&hoge.into());
```

Complementary:  
[From (std::convert::From)](#std-convert-from)

[From and Into](https://doc.rust-lang.org/rust-by-example/conversion/from_into.html)
