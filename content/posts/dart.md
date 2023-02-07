---
title: "Dart"
date: 2023-02-07T11:01:42+09:00
draft: false
---

# Dart 3

[DartPad](https://dartpad.dev/)

## hello, world

```
main() async {
  print('hello, world');  //=> hello, world
}
```

## Asynchronous Function

```
main() async {
  var msg = await getMsg();
  print(msg);  //=> hello, world
}

/* Asynchronous Function */
Future<dynamic> getMsg() async {
  return 'hello, world';
}
```
