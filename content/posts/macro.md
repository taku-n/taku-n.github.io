---
title: "On Rust"
date: 2021-10-28
tags: ["Macro", "Rust"]
---

# On Rust

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
    hello!();
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
