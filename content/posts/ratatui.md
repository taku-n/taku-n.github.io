---
title: "Ratatui"
date: 2025-02-07T16:37:47+09:00
draft: true
---

# Ratatui

## Installation

```
sudo aptitude update
sudo aptitude full-upgrade
sudo aptitude install pkg-config
sudo aptitude install libssl-dev

rustup self update
rustup update
cargo install cargo-generate
```

## Create a Project

```
cargo generate ratatui/templates simple
cargo add tokio@1 --features=full
cargo add anyhow@1
cargo add async-recursion@1
cargo add tui-textarea@0.7
```

main.rs  

```
use anyhow::*;
use async_recursion::*;

pub use app::App                .direction(Direction::Vertical)
                .constraints(vec![Constraint::Percentage(50), Constraint::Percentage(50)])
                .split(f.area());

        let lines = strings_to_lines(self.text_area.lines());
        let b = Paragraph::new(lines).block(Block::new().title("You typed:"));

        f.render_widget(&self.text_area, rects[0]);
        f.render_widget(b, rects[1]);

pub mod app;

#[tokio::main]
async fn main() -> color_eyre::Result<()> {
    color_eyre::install()?;
    let terminal = ratatui::init();

    let handle = tokio::spawn(async move {
        App::new().run(terminal).await
    });

    let result = handle.await?;
    ratatui::restore();
    result
}
```

app.rs  

```
use ratatui::prelude::*;
use ratatui::widgets::*;

#[derive(Debug, Default)]
pub struct App<'a> {
    /// Is the application running?
    running: bool,

    text_area: tui_textarea::TextArea<'a>,

    lines: Vec<Line<'a>>,
}

impl<'a> App<'a> {
    ...

    pub async fn run(mut self, mut terminal: DefaultTerminal) -> Result<()> {
        self.text_area = tui_textarea::TextArea::default();
        self.running = true;
        while self.running {
            terminal.draw(|frame| self.draw(frame))?;
            self.handle_crossterm_events()?;
            tokio::task::yield_now().await;
        }
        Ok(())
    }

    ...

    fn on_key_event(&mut self, key: KeyEvent) {
        match (key.modifiers, key.code) {
            (_, KeyCode::Esc)
            | (KeyModifiers::CONTROL, KeyCode::Char('c') | KeyCode::Char('C')) => self.quit(),
            // Add other key handlers here.
            _ => {}
        }
        self.text_area.input(key);
    }

    ...

    // Get the last max_lines lines from App::lines
    fn get_last_lines(&self, max_lines: usize) -> Vec<Line> {
        let mut v = vec![Line::raw(""); max_lines];
        let starting_line: usize;
        if max_lines < self.lines.len() {
            starting_line = self.lines.len() - max_lines;
            v.clone_from_slice(&self.lines[starting_line..]);
        } else {
            return self.lines.clone();
        }
        v
    }

    fn add_line(&mut self) {
        let string = self.text_area.clone().into_lines().remove(0);
        self.lines.push(string.into());
    }

    fn delete_text_area_line(&mut self) {
        self.text_area.move_cursor(tui_textarea::CursorMove::Head);
        self.text_area.delete_line_by_end();
    }
}

fn strings_to_lines(strings: &[String]) -> Vec<Line> {
    let mut v: Vec<Line> = Vec::new();
    for x in strings {
        v.push(Line::raw(x));
    }
    v
}
```

## hello, block

Draw an empty block  

app.rs  

```
fn draw(&mut self, f: &mut Frame) {
    f.render_widget(Block::new(), f.area());
}
```

```

```

You can quit by Esc or Ctrl-C  

Set the title of this block  

```
fn draw(&mut self, f: &mut Frame) {
    let b = Block::new().title("hello, block");
    f.render_widget(b, f.area());
}
```

```
hello, block
```

Add a paragraph to the block  

```
fn draw(&mut self, f: &mut Frame) {
    let b = Paragraph::new("block").block(Block::new().title("hello"));
    f.render_widget(b, f.area());
}
```

```
hello
block
```

2 rects vertically, each of them having a block  

```
fn draw(&mut self, f: &mut Frame) {
    let rects = Layout::default()
            .direction(Direction::Vertical)
            .constraints(vec![Constraint::Percentage(50), Constraint::Percentage(50)])
            .split(f.area());
    let b_top = Paragraph::new("top").block(Block::new().title("hello"));
    let b_bottom = Paragraph::new("bottom").block(Block::new().title("hello"));
    f.render_widget(b_top, rects[0]);
    f.render_widget(b_bottom, rects[1]);
}
```

```
hello
top

hello
bottom

```

Split the bottom rect horizontally  

```
fn draw(&mut self, f: &mut Frame) {
    let rects_v = Layout::default()
            .direction(Direction::Vertical)
            .constraints(vec![Constraint::Percentage(50), Constraint::Percentage(50)])
            .split(f.area());
    let rects_h = Layout::default()
            .direction(Direction::Horizontal)
            .constraints(vec![Constraint::Percentage(50), Constraint::Percentage(50)])
            .split(rects_v[1]);

    let b_top = Paragraph::new("top").block(Block::new().title("hello"));
    let b_bottom_left = Paragraph::new("bottom left").block(Block::new().title("hello"));
    let b_bottom_right = Paragraph::new("bottom right").block(Block::new().title("hello"));

    f.render_widget(b_top, rects_v[0]);
    f.render_widget(b_bottom_left, rects_h[0]);
    f.render_widget(b_bottom_right, rects_h[1]);
}
```

```
hello
top

hello       hello
bottom left bottom right
```

A block having padding  

```
fn draw(&mut self, f: &mut Frame) {
    let b = Paragraph::new("block").block(Block::new()
            .title("hello")
            .padding(Padding::new(1, 1, 1, 1)));  // (left, right, top, bottom)
    f.render_widget(b, f.area());
}
```

```
hello

 block

```

Text area  

```
fn draw(&mut self, f: &mut Frame) {
    f.render_widget(&self.text_area, f.area());
}
```

Print what you typed  

```
fn draw(&mut self, f: &mut Frame) {
    let rects = Layout::default()
            .direction(Direction::Vertical)
            .constraints(vec![Constraint::Percentage(50), Constraint::Percentage(50)])
            .split(f.area());

    let lines = strings_to_lines(self.text_area.lines());
    let b = Paragraph::new(lines).block(Block::new().title("You typed:"));

    f.render_widget(&self.text_area, rects[0]);
    f.render_widget(b, rects[1]);
}
```

```
hello
world

You typed:
hello
world
```

Add a line you typed when you press Enter  

```
fn draw(&mut self, f: &mut Frame) {
    let rects = Layout::default()
            .direction(Direction::Vertical)
            .constraints(vec![Constraint::Percentage(99), Constraint::Length(1)])
            .split(f.area());

    let max_lines = rects[0].height;
    let b = Paragraph::new(self.get_last_lines(max_lines.into())).block(Block::new());

    f.render_widget(b, rects[0]);
    f.render_widget(&self.text_area, rects[1]);
}

// Add other key handlers here.
(_, KeyCode::Enter) | (KeyModifiers::CONTROL, KeyCode::Char('m')) => {
    self.add_line();
    self.delete_text_area_line();
    return;
}
```
