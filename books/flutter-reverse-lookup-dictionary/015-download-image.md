---
title: "ネット画像をバイトデータに変換したい"
---

![](/images/q15/1.gif)

# A. http パッケージで get するだけでいい

上の gif がチカチカするので少しスペースを開けます。

|
|
|
|
|
|
|
|

インターネット上のイメージデータをバイトデータに変換したいことがあったので、今回はその方法を紹介します。といいましても、とても簡単で http パッケージで get した後、その response の bodyBytes に格納されているようです。関数として表すと次のようになります。

```dart
  /// ネットワーク上の画像をBytesデータに変換
  Future<void> networkImageToBytes(Uri uri) async {
    // http で取得すると普通に bytes として取り出せる。
    final response = await get(uri);
    bytes = response.bodyBytes;
    setState(() {});
  }
```

当たり前の話なのかもしれないですが、こういうのをみるとインターネットってすごいよなあと感心してしまいます。

この中身をログ表示して、jpg などの生データを読み解いていくのも勉強になりそうです。

今回のサンプルコードでは ネットワークイメージを Image.network で表示せず、一度 Uint8List 型のバイトデータに変換してから Image.memory で表示させるということをやっていますので何かの参考になれば幸いです。

## コピペで動くサンプルコード

```dart
import 'dart:typed_data';

import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'package:http/http.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        textTheme: GoogleFonts.sawarabiGothicTextTheme(),
      ),
      home: const DemoPage(),
    );
  }
}

class DemoPage extends StatefulWidget {
  const DemoPage({Key? key}) : super(key: key);

  @override
  State<DemoPage> createState() => _DemoPageState();
}

class _DemoPageState extends State<DemoPage> {
  Uint8List? bytes;

  /// ネットワーク上の画像をBytesデータに変換
  Future<void> networkImageToBytes() async {
    // http で取得すると普通に bytes として取り出せる。
    final response = await get(Uri.parse('https://i.gzn.jp/img/2018/01/15/google-gorilla-ban/00.jpg'));
    bytes = response.bodyBytes;
    setState(() {});
  }

  @override
  void initState() {
    super.initState();
    networkImageToBytes();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          'ネット画像をバイトデータに変換',
          style: TextStyle(
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
      body: Center(
        child: bytes == null
            ? const Text(
                '変換中',
                style: TextStyle(
                  fontSize: 40,
                ),
              )
            : Image.memory(bytes!),
      ),
    );
  }
}
```
