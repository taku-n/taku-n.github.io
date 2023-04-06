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

[AST Explorer](https://astexplorer.net/)  
[Crate syn](https://docs.rs/syn/latest/syn/)  

| syn::           | Explaination    | e.g.           |
| --------------- | --------------- | -------------- |
| Expr (Enum)     | Expression      | 3 * 19         |
| Ident (Struct)  | Identifier      | x              |
| LitInt (Struct) | Literal integer | 57             |
| UseTree (Enum)  | use path        | std::env::args |

### proc_macro

```
mkdir proc
cd proc
nvim Cargo.toml
```

```
[workspace]
members = [
    "app",
    "mcr",
]
```

```
cargo new app
cargo new --lib mcr
nvim app/Cargo.toml
```

```
[package]
name = "app"
version = "0.1.0"
edition = "2021"

[dependencies]
mcr = {path = "../mcr"}
```

```
nvim app/src/main.rs
```

```
use mcr::mcr;

fn main() {
    mcr!();
}
```

```
nvim mcr/Cargo.toml
```

```
[package]
name = "mcr"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
proc-macro2 = "*"
quote = "*"
syn = {version = "*", features = ["full"]}

[dev-dependencies]
pretty_assertions = "*"
```

```
nvim mcr/src/lib.rs
```

```
use proc_macro2::TokenStream;
use quote::quote;
use syn::Error;
use syn::Result;
use syn::parse::ParseStream;
use syn::parse::Parser;

#[proc_macro]
pub fn mcr(tokens: proc_macro::TokenStream) -> proc_macro::TokenStream {
    mcr_impl(tokens.into()).into()
}

fn mcr_impl(tokens: TokenStream) -> TokenStream {
    mcr_parse.parse2(tokens).unwrap_or_else(Error::into_compile_error)
}

fn mcr_parse(input: ParseStream) -> Result<TokenStream> {
    Ok(quote!())
}

#[cfg(test)]
mod tests {
    use super::*;

    use quote::quote;

    use pretty_assertions::assert_eq;

    #[test]
    fn mcr_impl_test() {
        assert_eq!(mcr_impl(quote!()).to_string(), quote!().to_string());
    }
}
```

#### Square

```
nvim app/src/main.rs
```

```
use mcr::mcr;

fn main() {
    println!("{:?}", mcr!(2));  //=> 4
    //mcr!(a);  // error: expected integer literal
}
```

```
nvim mcr/src/lib.rs
```

```
use proc_macro2::TokenStream;
use quote::quote;
use syn::Error;
use syn::LitInt;
use syn::Result;
use syn::parse::ParseStream;
use syn::parse::Parser;

#[proc_macro]
pub fn mcr(tokens: proc_macro::TokenStream) -> proc_macro::TokenStream {
    mcr_impl(tokens.into()).into()
}

fn mcr_impl(tokens: TokenStream) -> TokenStream {
    mcr_parse.parse2(tokens).unwrap_or_else(Error::into_compile_error)
}

fn mcr_parse(input: ParseStream) -> Result<TokenStream> {
    let x: LitInt = input.parse()?;
    Ok(quote!(#x * #x))
}

#[cfg(test)]
mod tests {
    use super::*;

    use quote::quote;

    use pretty_assertions::assert_eq;

    #[test]
    fn mcr_impl_test_ok() {
        assert_eq!(mcr_impl(quote!(2)).to_string(), quote!(2 * 2).to_string());
    }

    #[test]
    fn mcr_impl_test_ng() {
        assert_eq!(
                mcr_impl(quote!(a)).to_string(),
                quote!(:: core :: compile_error ! { "expected integer literal" }).to_string(),
        );
    }
}
```

#### Multiply

```
nvim app/src/main.rs
```

```
use mcr::mcr;

fn main() {
    println!("{:?}", mcr!(3 19));  //=> 57
    //mcr!(a b);  // error: expected integer literal
}
```

```
nvim mcr/src/lib.rs
```

```
use proc_macro2::TokenStream;
use quote::quote;
use syn::Error;
use syn::LitInt;
use syn::Result;
use syn::parse::ParseStream;
use syn::parse::Parser;

#[proc_macro]
pub fn mcr(tokens: proc_macro::TokenStream) -> proc_macro::TokenStream {
    mcr_impl(tokens.into()).into()
}

fn mcr_impl(tokens: TokenStream) -> TokenStream {
    mcr_parse.parse2(tokens).unwrap_or_else(Error::into_compile_error)
}

fn mcr_parse(input: ParseStream) -> Result<TokenStream> {
    let x: LitInt = input.parse()?;
    let y: LitInt = input.parse()?;
    Ok(quote!(#x * #y))
}

#[cfg(test)]
mod tests {
    use super::*;

    use quote::quote;

    use pretty_assertions::assert_eq;

    #[test]
    fn mcr_impl_test_ok() {
        assert_eq!(mcr_impl(quote!(3 19)).to_string(), quote!(3 * 19).to_string());
    }

    #[test]
    fn mcr_impl_test_ng() {
        assert_eq!(
                mcr_impl(quote!(a b)).to_string(),
                quote!(:: core :: compile_error ! { "expected integer literal" }).to_string(),
        );
    }
}
```

### proc_macro_attribute

### proc_macro_derive

[マクロクラブ Rust支部](https://keens.github.io/blog/2018/02/17/makurokurabu_rustshibu/)  
[Rustのマクロを覚える](https://qiita.com/k5n/items/758111b12740600cc58f)  
[Rustの手続きマクロに関する知見をまとめてみた](https://techracho.bpsinc.jp/yoshi/2020_12_24/102304)  
[セバスチャンマクロを作って学ぶRustの手続きマクロ](https://zenn.dev/kazatsuyu/articles/33e130563b87b1)  
[syn::parse::ParseBuffer::peek の謎に迫る](https://zenn.dev/frozenlib/articles/parse_buffer_peek)  
