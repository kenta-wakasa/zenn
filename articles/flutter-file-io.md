---
title: "【Flutter】外部ファイルを読み書きする"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, io, ios, xcode, dart]
published: true
---

# はじめに

Flutter でファイルを読み書きする方法をまとめました。

# 作ったもの

![](https://storage.googleapis.com/zenn-user-upload/12sq9ombnxe5i9eyd8cbh3p0bauj =400x)
_現在時刻をテキストデータとして書き込むアプリ_

:::details 環境
マシン: M1 MacBook Air
エディタ: VSCode
リポジトリ: https://github.com/kenta-wakasa/flutter_playground
:::

# 実装

早速ですがサンプルのコードです。

**ページ**

```dart:io_page.dart
import 'package:flutter/material.dart';
// riverpod を使う場合は忘れずに pubspec.yaml を編集して pub get
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'io_controller.dart';

// ページは描画しているだけなので詳しい内容はコントローラーへ GO
class IoPage extends ConsumerWidget {
  const IoPage({Key key}) : super(key: key);
  static const String title = '外部データの入出力';
  @override
  Widget build(BuildContext context, ScopedReader watch) {
    final _ioProvider = watch(ioProvider);
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          title,
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('アプリのローカルパス: \n${_ioProvider?.appPath}'),
            Padding(
              padding: const EdgeInsets.all(16),
              child: SizedBox(
                  height: 25,
                  child: FittedBox(
                    fit: BoxFit.fitHeight,
                    child: Text(
                      _ioProvider.content,
                    ),
                  )),
            ),
            Center(
              child: ElevatedButton(
                onPressed: () async {
                  await _ioProvider.write();
                  await _ioProvider.read();
                },
                child: const Text('いまの時間を書き込む'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**コントローラー**

```dart:io_controller.dart
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:path_provider/path_provider.dart';

final ioProvider = ChangeNotifierProvider.autoDispose<IoController>(
  (ref) => IoController(),
);

class IoController extends ChangeNotifier {
  IoController() {
    /// 初期化処理をここに書く
    /// コンストラクタで非同期処理をやりたい場合どうすのがいいんでしょう？
    () async {
      content = 'ファイルに書き込まれた時間を表示します';
      final _localPath = await localPath;
      appPath = '$_localPath/playground/';
      appDirectory = Directory(appPath);

      /// 新しくディレクトリをつくる
      await appDirectory.create(recursive: true);
    }();
  }

  @override
  void dispose() {
    super.dispose();
  }

  String appPath;
  String content;
  Directory appDirectory;

  /// ローカルパスの取得
  Future<String> get localPath async {
    final directory = await getApplicationDocumentsDirectory();
    notifyListeners();
    return directory.path;
  }

  /// 現在時刻の書き込み
  Future<void> write() async {
    final file = File('$appPath/test.txt');
    print('write: ${file.path}');
    await file.writeAsString(DateTime.now().toString());
  }

  /// 現在時刻の読み込み
  Future<void> read() async {
    final file = File('$appPath/test.txt');
    content = await file.readAsString();
    notifyListeners();
  }
}
```

# 解説

1. まずは path を取得する
1. 読み書きの方法を知る

## 1. まずは path を取得する

アプリに与えられた固有の path 情報取得しなければいけません。
path の取得には [path_provider](https://pub.dev/packages/path_provider) というパッケージを使っています。

```dart
final directory = await getApplicationDocumentsDirectory();
final path = directory.path;
```

:::message
今回は iOS ですが Android でも同じように動くそうです。
:::

## 2. 読み書きの方法を知る

ファイルの読み書きには [io](https://api.dart.dev/stable/2.10.4/dart-io/dart-io-library.html) というライブラリを使います。

### ディレクトリの作成

```dart
final directory = Directory('作りたいディレクトリの path');
await directory.create(recursive: true);
```

### ファイルの読み込み

```dart
final file = File('読み込みたいファイルの path');
final String content = await file.readAsString(); // テキストデータならこう書く
```

### ファイルの書き込み

```dart
final file = File('書き込みたいファイルの path');
await file.writeAsString(DateTime.now().toString()); // テキストデータならこう書く
```

以上！

# iOS の Files に表示させるには？

Files アプリの中には「この iPhone 内」というな名前のディレクトリがあるけど、ここに書き込んだデータを表示させたいときはどうすればいいのだろう？

## info.plist に追記する

```xml:info.plist
<!-- on my iPhone (このiPhone内) に表示させるために必要  -->
    <key>LSSupportsOpeningDocumentsInPlace</key>
    <true/>
    <key>UIFileSharingEnabled</key>
    <true/>
<!-- //////////////////////////////////////////// -->
```

この 2 つを追記すればよい！

![](https://storage.googleapis.com/zenn-user-upload/y6anjv0ezs423vxaas6a2hbya14d =240x)

# おしまいに

