---
title: "Crossterm"
date: 2025-02-15T15:40:32+09:00
draft: false
---

# Crossterm

Imports  

```
use std::io::Stdout;
use std::io::stdout;

use crossterm::ExecutableCommand;
use crossterm::cursor::MoveTo;
use crossterm::style::Print;
use crossterm::terminal::Clear;
use crossterm::terminal::ClearType::All;
use crossterm::terminal::disable_raw_mode;
use crossterm::terminal::enable_raw_mode;
```

Get the standard output  

```
let mut o: Stdout = stdout();
```

Enable the raw mode  

```
enable_raw_mode()?;
```

Disable the raw mode  

```
disable_raw_mode()?;
```

Clear the screen  

```
o.execute(Clear(All))?;
```

Move the cursor  

```
o.execute(MoveTo(0, 0))?;
```

Print  

```
o.execute(Print("hello, world"))?;
```

A sample using tokio  

```
[package]
name = "stream"
version = "0.1.0"
edition = "2021"

[dependencies]
anyhow = "1"
crossterm = { version = "0.28", features = ["event-stream"] }
futures = "0.3"
tokio = { version = "1", features = ["full"] }
```

```
use std::io::stdout;
use std::io::Stdout;
use std::time::Duration;

use anyhow::*;
use futures::FutureExt; // fuse()
use futures::StreamExt; // next()
use tokio::sync::mpsc;
use tokio::time::sleep;

use crossterm::cursor::MoveTo;
use crossterm::event::Event::Key;
use crossterm::event::EventStream;
use crossterm::event::KeyCode;
use crossterm::style::Print;
use crossterm::terminal::disable_raw_mode;
use crossterm::terminal::enable_raw_mode;
use crossterm::terminal::Clear;
use crossterm::terminal::ClearType::All;
use crossterm::ExecutableCommand; // execute()

enum Evt {
    EKey(String),
    ETime(i32),
    None,
}

#[tokio::main]
async fn main() -> Result<()> {
    enable_raw_mode()?;

    let mut o: Stdout = stdout();
    let (tx, mut rx) = mpsc::channel(137);

    tokio::spawn(async move {
        let mut x = 0;
        loop {
            tx.send(x).await;
            x = x + 1;
            sleep(Duration::from_secs(3)).await;
        }
    });

    let mut reader = EventStream::new();
    loop {
        let evt = get_event(&mut reader, &mut rx).await;
        match evt {
            Evt::EKey(x) if x == String::from("a") => print(&mut o, &String::from("a")).await,
            Evt::EKey(x) if x == String::from("esc") => break,
            Evt::EKey(_) => panic!(),
            Evt::ETime(x) => print(&mut o, &x.to_string()).await,
            Evt::None => continue,
        }
    }

    disable_raw_mode()?;

    Ok(())
}

async fn get_event(reader: &mut EventStream, rx: &mut mpsc::Receiver<i32>) -> Evt {
    let event = reader.next().fuse();

    tokio::select! {
        option_evt = event => {
            let result = option_evt.unwrap();
            let evt = result.unwrap();
            if evt == Key(KeyCode::Char('a').into()) {
                return Evt::EKey("a".into());
            }
            if evt == Key(KeyCode::Esc.into()) {
                return Evt::EKey("esc".into());
            }
        }
        option_time = rx.recv() => {
            let x = option_time.unwrap();
            return Evt::ETime(x);
        }
    };

    Evt::None
}

async fn print(o: &mut Stdout, string: &String) {
    o.execute(Clear(All));
    o.execute(MoveTo(0, 0));
    o.execute(Print(string));
}
```

[https://github.com/crossterm-rs/crossterm/blob/master/examples/event-stream-tokio.rs](https://github.com/crossterm-rs/crossterm/blob/master/examples/event-stream-tokio.rs)  
[https://ratatui.rs/tutorials/counter-async-app/async-event-stream/](https://ratatui.rs/tutorials/counter-async-app/async-event-stream/)  
[BlueskyのTUI Client Appを作り始めてしまった](https://memo.sugyan.com/entry/2024/07/17/230008)  
