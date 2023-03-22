---
title: "On Rust"
date: 2021-10-28
tags: ["Macro", "Rust"]
---

# On Rust

## hello, world

```
$ cargo install cargo-expand
```

```
macro_rules! hello {
    () => {
        println!("hello, world");
    };
}

fn main() {
    hello!();  //=> hello, world
}
```

```
$ cargo expand main
```

```
fn main() {
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(&["hello, world\n"], &[]));
    };
}
```

## Declarative Macros

### Simplest Macro

```
macro_rules! pass {
    () => {};
}

fn main() {
    pass!();
}
```

```
$ cargo expand main
```

```
fn main() {}
```

### Square

```
macro_rules! square {
    ($x:expr) => {
        $x * $x
    };
}

fn main() {
    println!("{:?}", square!(2));  //=> 4
}
```

```
$ cargo expand main
```

```
fn main() {
    {
        ::std::io::_print(format_args!("{0:?}\n", 2 * 2));
    };
}
```

### Multiply

```
macro_rules! multiply {
    ($x:expr, $y:expr) => {
        $x * $y
    };
}

fn main() {
    println!("{:?}", multiply!(3, 19));  //=> 57
}
```

```
$ cargo expand main
```

```
fn main() {
    {
        ::std::io::_print(format_args!("{0:?}\n", 3 * 19));
    };
}
```

### Multiple Addition

```
macro_rules! ma {
    ($($x:expr),*) => {  // The , in this line is the separator. Nothing is also allowed
        0 $(+ $x)*
    };
}

fn main() {
    println!("{:?}", ma!(2, 3, 5));  //=> 10
}
```

```
$ cargo expand main
```

```
fn main() {
    {
        ::std::io::_print(format_args!("{0:?}\n", 0 + 2 + 3 + 5));
    };
}
```

### Calculation

```
macro_rules! calc {
    (* $($x:expr)*) => {
        1 $(* $x)*
    };
    (+ $($x:expr)*) => {
        0 $(+ $x)*
    };
}

fn main() {
    println!("{:?}", calc!(* 2 3 5));  //=> 30
    println!("{:?}", calc!(+ 2 3 5));  //=> 10
}
```

```
$ cargo expand main
```

```
fn main() {
    {
        ::std::io::_print(format_args!("{0:?}\n", 1 * 2 * 3 * 5));
    };
    {
        ::std::io::_print(format_args!("{0:?}\n", 0 + 2 + 3 + 5));
    };
}
```

## Procedural Macros

[マクロクラブ Rust支部](https://keens.github.io/blog/2018/02/17/makurokurabu_rustshibu/)  
[Rustのマクロを覚える](https://qiita.com/k5n/items/758111b12740600cc58f)  
[Rustの手続きマクロに関する知見をまとめてみた](https://techracho.bpsinc.jp/yoshi/2020_12_24/102304)  
[セバスチャンマクロを作って学ぶRustの手続きマクロ](https://zenn.dev/kazatsuyu/articles/33e130563b87b1)  
