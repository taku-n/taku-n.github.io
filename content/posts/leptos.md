---
title: "Leptos"
date: 2023-06-02T17:14:54+09:00
draft: true
---

# Leptos

```
cargo new hello
cd hello
touch index.html
trunk serve
```

Open http://127.0.0.1:8080/ by your browser.  
See a blank page.  
Press Ctrl+C to stop the test server.  

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
}

#[wasm_bindgen(start)]
async fn wain() {
    log("hello, world");
}

fn main() {
}
```

```
trunk serve
```

See your browser's console says, "hello, world".  

```
cargo add leptos --features="stable"
nvim src/main.rs
```

```
use wasm_bindgen::prelude::*;
use leptos::*;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

#[wasm_bindgen(start)]
async fn wain() {
    log("hello, world");
    mount_to_body(|cx| view!{cx,
<p>"hello, world"</p>
    });
}

fn main() {
}
```
