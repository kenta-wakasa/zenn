---
title: "ダイアログでもホットリロードがしたい"
---

# Q.1 dialog でも hot reload がしたい

![](/images/q1/1.gif)

## A. 独自 Widget として切り出してみましょう

簡単なダイアログを出すときは、次のようにその場で Widget を作成することも多いです。

```dart
  ElevatedButton(
    onPressed: () {
      showDialog(
        context: context,
        builder: (context) {
          return const AlertDialog(
            content: Text('ここのテキストを変えてHotReload'),
          );
        },
      );
    },
    child: const Text('HotReloadできない😭'),
  ),
```

しかしこのように記述すると HotReload が効かず困った経験はありませんか？

こういうときは Extract Widget で切り出してあげるとうまくいきます。

```dart
  ElevatedButton(
    onPressed: () {
      showDialog(
        context: context,
        builder: (context) {
          return const DemoDialog();
        },
      );
    },
    child: const Text('HotReloadできる😄'),
  ),

/// Dialogを独自Widgetとして切り出す。
class DemoDialog extends StatelessWidget {
  const DemoDialog({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) { // このメソッドは rebuild時に再実行される。
    return const AlertDialog(
      content: Text('ここのテキストを変えてHotReload'),
    );
  }
}
```

## onPressed の中は再実行されない

なぜこんなことが起きるのでしょうか？
実は onPressed などに与えた関数は HotReload 時には再実行されません。

一方で、切り出された Widget の build()メソッドは再実行されます。

ちょっとした違いですが、意識すると予期せぬ不具合に対応できるかもしれませんね。

## コピペで試せるサンプルコード

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: DemoPage());
  }
}

class DemoPage extends StatefulWidget {
  const DemoPage({Key? key}) : super(key: key);

  @override
  State<DemoPage> createState() => _DemoPageState();
}

class _DemoPageState extends State<DemoPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  builder: (context) {
                    return const AlertDialog(
                      content: Text('ここのテキストを変えてHotReload'),
                    );
                  },
                );
              },
              child: const Text('HotReloadできない😭'),
            ),
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  builder: (context) {
                    return const DemoDialog();
                  },
                );
              },
              child: const Text('HotReloadできる😄'),
            ),
          ],
        ),
      ),
    );
  }
}

class DemoDialog extends StatelessWidget {
  const DemoDialog({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const AlertDialog(
      content: Text('ここのテキストを変えてHotReload'),
    );
  }
}

```

## 参考

- https://medium.com/flutter-community/flutter-hot-reload-doesnt-re-run-code-c34b7d558be0
