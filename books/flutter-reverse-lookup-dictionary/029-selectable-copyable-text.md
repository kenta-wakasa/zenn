---
title: 文字列を選択したい
---

# A. SelectableText を使いましょう

![](/images/q29/1.gif)

Flutter の Text Widget はなぜか選択できません。アプリならあまり気にならないですが、Web だと選択できた方が嬉しいよなと思います。選択させるだけなら、SelectableText を使うことで簡単に実装できます。使い方は Text を SelectableText に置き換えるだけです。

```dart
Text('これを選択したい');

// ただ置き換えるだけです
SelectableText('これを選択したい');
```

## CopyableText を作ってみよう

これだけでは記事として寂しいので、タップしたらテキスト全体をクリップボードにコピーしてくれる CopyableText も作ってみましょう。

クリップボードへのコピーも実は簡単にできます。Clipboard.setData を呼び出すだけです。

```dart
  Clipboard.setData(ClipboardData(text: 'ここにコピーしたい文字列'));
```

これを上手く使って、CopyableText を定義していきます。Text を InkWell でラップして onLongPress されたときにクリップボードにコピーしているだけですね。

```dart
class CopyableText extends Text {
  const CopyableText(
    String data, {
    Key? key,
    TextStyle? style,
  }) : super(data, key: key, style: style);
  @override
  Widget build(BuildContext context) {
    return InkWell(
      onLongPress: () => Clipboard.setData(ClipboardData(text: data)),
      child: super.build(context),
    );
  }
}
```

## コピペで動くサンプルコード

では最後にサンプル用のコードを示しておしまいにします。

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('文字列を選択したい'),
        ),
        body: Column(
          children: [
            const Padding(
              padding: EdgeInsets.all(16),
              child: CopyableText(
                'ロングプレスでクリップボードにコピー',
                style: TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
              ),
            ),
            const Padding(
              padding: EdgeInsets.all(16),
              child: SelectableText(
                '部分選択でクリップボードにコピー',
                style: TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(16),
              child: TextFormField(
                decoration: InputDecoration(
                  hintStyle: const TextStyle(fontSize: 12, color: Colors.blue),
                  fillColor: Colors.blue[100],
                  filled: true,
                  focusedBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(16),
                    borderSide: const BorderSide(
                      color: Colors.blue,
                      width: 2.0,
                    ),
                  ),
                  enabledBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(16),
                    borderSide: BorderSide(
                      color: Colors.blue[100]!,
                      width: 1.0,
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class CopyableText extends Text {
  const CopyableText(
    String data, {
    Key? key,
    TextStyle? style,
  }) : super(data, key: key, style: style);
  @override
  Widget build(BuildContext context) {
    return InkWell(
      onLongPress: () => Clipboard.setData(ClipboardData(text: data)),
      child: super.build(context),
    );
  }
}
```
