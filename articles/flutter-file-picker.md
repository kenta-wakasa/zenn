---
title: "【Flutter】外部ファイルを取り込む"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, file, googledrive, image, music]
published: true
---

# はじめに

Flutter でデバイスに保存されたファイルを選択して取り扱う方法をまとめました。
Google Drive などのクラウドファイルもとってくることができます。

# 作ったもの

.txt ファイルを取り込んで中身を表示する。
https://twitter.com/pressedkonbu/status/1351836395099234305

:::details 環境
マシン: M1 MacBook Air
エディタ: VSCode
リポジトリ: https://github.com/kenta-wakasa/flutter_playground
:::

# 実装

さっそくコードです。

```dart:file_picker_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'file_picker_controller.dart';

class FilePickerPage extends ConsumerWidget {
  const FilePickerPage({Key key}) : super(key: key);
  static const String title = 'ファイルピッカー';

  @override
  Widget build(BuildContext context, ScopedReader watch) {
    final _provider = watch(filePickerProvider);
    const textStyle = TextStyle(
      fontSize: 16,
      fontWeight: FontWeight.bold,
    );
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
          mainAxisAlignment: MainAxisAlignment.start,
          children: [
            SizedBox(
              width: 360,
              height: 200,
              child: Center(
                  child: Text(
                _provider.fileName,
                style: textStyle,
              )),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () async {
                  if (await _provider.pickFileIsSuccess) {
                    const snackBar = SnackBar(content: Text('ピックしたよ！'));
                    ScaffoldMessenger.of(context).showSnackBar(snackBar);
                  } else {
                    const snackBar = SnackBar(content: Text('ピックしなかったよ！'));
                    ScaffoldMessenger.of(context).showSnackBar(snackBar);
                  }
                },
                child: const Text('ファイルを選択する'),
              ),
            ),
            Container(
              margin: const EdgeInsets.all(16),
              height: 2,
              color: Theme.of(context).primaryColor,
            ),
            SizedBox(
              width: 300,
              height: 200,
              //スクロールに対応
              child: SingleChildScrollView(
                child: Center(child: Text(_provider.fileContents)),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: _provider.pickSuccess
                    ? () async {
                        await _provider.readFileContents();
                      }
                    : null, // 使えない時は null をいれると色が変わってくれる
                child: const Text('選択したファイルを読み込む'),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

```dart:ile_picker_controller.dart
import 'dart:io';

import 'package:file_picker/file_picker.dart';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final filePickerProvider =
    ChangeNotifierProvider.autoDispose<FilePickerController>(
  (ref) => FilePickerController(),
);

class FilePickerController extends ChangeNotifier {
  FilePickerController() {
    /// 初期化処理をここに書く
    pickSuccess = false;
    fileName = '選択したファイル名がここに表示されます';
    fileContents = 'ファイルの中身がここに表示されます';
  }

  /// このプロバイダーが廃棄されるよきに呼ばれる
  @override
  void dispose() {
    super.dispose();
  }


  bool pickSuccess; // 読み込みが成功したら true

  File file;
  String fileName;
  String fileContents;

  Future<bool> get pickFileIsSuccess async {
    final filePickerResult = await FilePicker.platform.pickFiles(
      type: FileType.custom,
      allowedExtensions: ['txt'], // ピックする拡張子を限定できる。
    );
    if (filePickerResult != null) {
      pickSuccess = true;
      file = File(filePickerResult.files.single.path);
      fileName = filePickerResult.files.single.name;
    } else {
      pickSuccess = false;
      file = null;
      fileName = '何も選択されませんでした';
      fileContents = 'ファイルの中身がここに表示されます';
    }
    notifyListeners();
    return pickSuccess;
  }

  /// ファイルを読み込む
  Future<void> readFileContents() async {
    if (file != null) {
      fileContents = await file.readAsString();
      notifyListeners();
    }
  }
}
```

# 結論 file_picker を使えばいける

[file_picker](https://pub.dev/packages/file_picker) を使おう。パッケージがちゃんとあって助かる。ありがたい。

# iOS で使うための準備

:::message
Android では試していないけれど特殊な設定は必要ないはず。
:::
iOS では <project root>/ios/Runner/Info.plist に追記する必要がある。

```xml:info.plist

<!-- バックグラウンドでデータを落としてきたりするのに必要 -->
<key>UIBackgroundModes</key>
<array>
   <string>fetch</string>
   <string>remote-notification</string>
</array>

<!-- 音楽ファイルにアクセスするために必要 -->
<key>NSAppleMusicUsageDescription</key>
<string>ここに何に使うのかの理由を書く</string>

<!-- 写真ファイルにアクセスするために必要 -->
<key>NSPhotoLibraryUsageDescription</key>
<string>ここに何に使うのかの理由を書く</string>
```

# 使い方

## ファイル選択画面を表示したい

```dart
// これで選択画面が開く
// もちろん設定によっては複数ファイルを選択するなんてことも可能
FilePickerResult result = await FilePicker.platform.pickFiles()

// もし何も選択しなければ result に null が返ってくる
if(result != null) {
    // null じゃなければ io パッケージをつかって file を開こう
    // result の中にはファイル名やパス名などの情報が入っている

    File file = File(result.files.single.path);
} else {
   // キャンセル場合だったときの処理はここに書こう
}
```

## 複数ファイルを選択したい

```dart
FilePickerResult result = await FilePicker.platform.pickFiles(allowMultiple: true);
```

## 拡張子を限定したい

```dart
FilePickerResult result = await FilePicker.platform.pickFiles(
          type: FileType.custom,
          allowedExtensions: ['jpg', 'pdf', 'doc'], // 選びたい拡張子を並べる　ひとつでもいい
        );
```

# おしまいに

簡単で使いやすくて、ほとんど書くことがありませんでした。
活用の方法はたくさんある気がするので、覚えておいて損はないかなと思います。

ファイルを選んできてアプリのディレクトリにコピーしてくるみたいな処理をやってみたいので、できたらまたここに書きます。
また次回の記事でお会いしましょう。

# おまけ

- Button の onPressed に null を与えると色が勝手に変わってくれて便利なことに気づいた。
- 長い文字列をある範囲内でスクロールしたいときは SizedBox と SingleChildScrollView でいける。
- スナックバーも使ってみたかったので書いてみました。気になる方はそこもチェック。
