---
title: "Hy"
date: 2020-10-30
tags: ["Hy"]
---

# Midnight Hy Way

[必ず理解しておくべきこと](https://github.com/hylang/hy/issues/1324)

## バージョン

```
$ hy -v
hy 0.19.0
```

## REPL

```
$ hy --repl-output-fn=hy.contrib.hy-repr.hy-repr
```

~/.bashrc に

```
alias hyr="hy --repl-output-fn=hy.contrib.hy-repr.hy-repr
```

などとしておき

```
$ hyr
```

で起動するのがいいだろう

## 言語基礎

### 演算

```
=> (+ 7 3)
10
=> (- 7 3)
4
=> (* 7 3)
21
=> (/ 7 3)
2.3333333333333335
=> (% 7 3)
1
```

### 変数

```
=> (setv x 42)
=> x
42
```

### 入れ子操作

`リスト` ['1 '2 '3 '4 '5] と区別するため  
'(1 2 3 4 5) のことを  
こちらのドキュメントでは `入れ子` と呼ぶことにする

```
=> (first '(1 2 3 4 5))
'1
=> (second '(1 2 3 4 5))
'2
=> (rest '(1 2 3 4 5))
<itertools.islice object at 0x...
'(2 3 4 5) 相当のものが生成されているはず
=> (last '(1 2 3 4 5))
'5

=> (cut '(1 2 3 4 5) 2)
'(3 4 5)
=> (cut '(1 2 3 4 5))
'(1 2 3 4 5)
=> (cut (rest '(1 2 3 4 5)))
TypeError: 'itertools.islice' object is not subscriptable
これはなぜかできない

=> (list '(1 2 3 4 5))
['1 '2 '3 '4 '5]
=> (list (rest '(1 2 3 4 5)))
['2 '3 '4 '5]
```

### 条件分岐

```
=> (defn check [x]
...  (if (> x 0) "positive"
...      (= x 0) "zero"
...      "negative"))
=> (check 42)
"positive"
=> (check 0)
"zero"
=> (check -42)
"negative"
```

### 関数

```
=> (defn add [x y]
...  (+ x y))

=> (add 2 3)
5
```

### マクロ

```
=> (defmacro add [&rest args]
...  `(+ ~@args))
=> (add 2 3 5)
10
```

### メイン関数

```
; main.hy

(defn hello [name]
  (+ "hello, " name))

; defmain は，プログラムの最後に置くのがいいらしい
(defmain [&rest args]
  (print "hello, world")
  (print args)
  (print (hello (second args))))
```

```
$ hy main.hy alice bob carol
hello, world
('main.hy', 'alice', 'bob', 'carol')
hello, alice
```

ちなみに，hyc は，壊れているようでなにも生成してくれない  
(`__pycache__` 内が生成物?)

```
hy.lex.exceptions.PrematureEndOfInput: Premature end of input
```

カッコの数が合っていないので確認する

### グローバル変数

```
=> (setv *x* 42)
=> (defn get-*x* []
...  *x*)
=> (get-*x*)
42

=> (defn set-*y* []
...  (global *y*)
...  (setv *y* 43))
=> (set-*y*)
=> *y*
43
```

### ラムダ関数

```
=> ((fn [x y] (+ x y)) 2 3)
5
```

### キーワード引数

```
=> (defn sub [x y]
...  (- x y))
=> (sub 7 3)
4
=> (sub 3 7)
-4
=> (sub :y 7 :x 3)
-4
```

### 辞書

```
=> (setv d {1 10  2 20  3 30})
=> (. d [2])
20
=> (setv (. d [3]) 42)
=> (print d)
{1: 10, 2: 20, 3: 42}
=> (try (. d [4])
...     (except [e KeyError] (print e)))
4

(setv (. label ["text"]) "hello, world")
(assoc label "text" "hello, world")
```

## スペシャルフォーム

### -> (最初の引数に入れる)

```
=> (defn f [x y]
...  (- x y))
=> (-> 43 (f 1))
42
```

### ->> (最後の引数に入れる)

```
=> (defn f [x y]
...  (- x y))
=> (->> 1 (f 43))
42
```

### as-> (いわゆる畳み込み (?))

```
=> (defn f [x y]
...  (print (- x y))
...  (- x y))
=> (as-> 59 it (f it 1)
...            (f 100 it))
58
42
42
```

### py (Python の式)

```
=> (py "41 + 1")
42
```

### pys (Python の文)

```
=> (pys "print('hoge')")
hoge
```
