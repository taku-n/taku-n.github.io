---
title: "Clang"
date: 2019-12-27T01:42:16+09:00
draft: false
---

https://apt.llvm.org/

```
$ sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
$ clang-9 -v

# これ以降は，いらなかったみたい

$ curl -OL https://apt.llvm.org/llvm-snapshot.gpg.key
$ sudo apt-key add llvm-snapshot.gpg.key
$ sudo apt update
$ sudo apt install clang-9 lldb-9 lld-9
```
