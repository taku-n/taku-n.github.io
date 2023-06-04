---
title: "Leptos"
date: 2023-06-02T17:14:54+09:00
draft: true
---

# Leptos

[Leptos](https://leptos.dev/)  
[Leptos (GitHub)](https://github.com/leptos-rs/leptos)  
[Leptos in Five Minutes (Video)](https://www.youtube.com/watch?v=K_TmEPAD9Ig)  
[Documentation](https://leptos-rs.github.io/leptos/)  

## Trunk

```
cargo install trunk
cargo new hello
cd hello
touch index.html
trunk serve
```

Open http://127.0.0.1:8080/ by your browser.  
See a blank page.  
Press Ctrl+C to stop the test server.  

## console.log() and console.error()

```
cargo add wasm-bindgen
cargo add wasm-bindgen-futures
nvim src/main.rs
```

```
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    #[wasm_bindgen(js_namespace = console)]
    fn error(s: &str);
}

#[wasm_bindgen(start)]
async fn wain() {
    log("hello, world");
    error("Error!");
}

fn main() {
}
```

```
trunk serve
```

See your browser's console says, "hello, world".  
See an error message, "Error!", as well.  


## Panic Hook

```
nvim src/main.rs
```

```
use wasm_bindgen::prelude::*;
use std::panic;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    #[wasm_bindgen(js_namespace = console)]
    fn error(s: &str);
}

fn panic_hook(panic_info: &panic::PanicInfo) {
    error(&panic_info.to_string());
}

#[wasm_bindgen(start)]
async fn wain() {
    panic::set_hook(Box::new(panic_hook));
    log("hello, world");
    error("Error!");
    panic!("Panic!");
}

fn main() {
}
```

```
trunk serve
```

See a panic message, "Panic!".  

## Test

```
sudo snap install --classic node
cargo install wasm-bindgen-cli
cargo add --dev wasm-bindgen-test
mkdir .cargo
nvim .cargo/config.toml
```

```
[build]
target = "wasm32-unknown-unknown"

[target.wasm32-unknown-unknown]
runner = 'wasm-bindgen-test-runner'
```

```
nvim src/main.rs
```

```
use wasm_bindgen::prelude::*;
use std::panic;

#[wasm_bindgen(start)]
async fn wain() {
    panic::set_hook(Box::new(panic_hook));
}

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    #[wasm_bindgen(js_namespace = console)]
    fn error(s: &str);
}

fn panic_hook(panic_info: &panic::PanicInfo) {
    error(&panic_info.to_string());
}

fn main() {
}

#[cfg(test)]
mod tests {
    use wasm_bindgen_test::*;

    #[wasm_bindgen_test]
    fn pass() {
        assert_eq!(1, 1);
    }
}
```

```
cargo test
```

```
mkdir tests
nvim tests/tests.rs
```

```
use wasm_bindgen_test::*;

#[wasm_bindgen_test]
fn pass() {
    assert_eq!(1, 1);
}
```

```
cargo test
```

## RSX

```
cargo add leptos --features="stable"
nvim src/main.rs
```

```
use wasm_bindgen::prelude::*;
use leptos::*;
use std::panic;

#[wasm_bindgen(start)]
async fn wain() {
    panic::set_hook(Box::new(panic_hook));
    mount_to_body(|cx| view!{cx,
<p>"hello, world"</p>
    });
}

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    #[wasm_bindgen(js_namespace = console)]
    fn error(s: &str);
}

fn panic_hook(panic_info: &panic::PanicInfo) {
    error(&panic_info.to_string());
}

fn main() {
}
```

```
trunk serve
```

See a message, "hello, world", in the page.  
After this, you can keep the test server running and keep your browser opening http://127.0.0.1:8080/.  
Edit your files in another terminal.  
