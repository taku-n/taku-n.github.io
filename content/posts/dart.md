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

## Global Variable

```
var msg = 'hello, world';

main() async {
  print(msg);  //=> hello, world
}
```

## Sleep

```
main() async {
  print('hello');
  await sleeping(3);
  print('world');
}

// Sleep for sec seconds.
Future<void> sleeping(int sec) async {
  await Future.delayed(Duration(seconds: sec));
}
```
