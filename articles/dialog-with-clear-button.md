---
title: "【Flutter】閉じるボタンのあるダイアログを作る"
emoji: "😵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dialog]
published: true
---

# はじめに

左上などに閉じるボタンがついたダイアログを作りたいと思い、やってみました。

:::details 環境
マシン: M1 MacBook Air
エディタ: VSCode
リポジトリ: https://github.com/kenta-wakasa/flutter_playground
:::

# 作ったもの

https://twitter.com/pressedkonbu/status/1352803800155742208

notifyListeners() を使ってダイアログに状態を伝搬させるということもやっています。
普通にやるとちょっとつまづいたので。

# 実装

さっそくコードです。

**ページ**

```dart:clear_button_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'clear_button_controller.dart';

class ClearButtonPage extends ConsumerWidget {
  const ClearButtonPage({Key key}) : super(key: key);
  static const String title = '閉じるボタン';

  @override
  Widget build(BuildContext context, ScopedReader watch) {
    final _provider = watch(clearButtonProvider);
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          title,
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
      ),
      body: Center(
        child: SizedBox(
          height: 100,
          child: ElevatedButton(
            onPressed: () {
              showDialog<void>(
                context: context,
                builder: (_) {
                  return ClearButtonDialog();
                },
              );
            },
            child: Column(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                const Text(
                  'ダイアログを開く',
                  style: const TextStyle(
                    fontSize: 20,
                  ),
                ),
                Text(
                  '${_provider.count}',
                  style: const TextStyle(
                    fontSize: 20,
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

/// 閉じるボタンのあるダイアログ
/// ここが今回のキモ
class ClearButtonDialog extends ConsumerWidget {
  @override
  Widget build(BuildContext context, ScopedReader watch) {
    final _provider = watch(clearButtonProvider);
    const _buttonSize = 24.0;
    final _dialogWidth = MediaQuery.of(context).size.width * 3 / 4; // 画面サイズから相対的に大きさを決めている。
    return Dialog(
      insetPadding: const EdgeInsets.all(0),
      elevation: 0,
      backgroundColor: Colors.transparent,
      // SizedBoxでダイアログそのものの大きさをまずは決めています。
      child: SizedBox(
        width: _dialogWidth,
        height: _dialogWidth * 3 / 4,
        child: Stack(
          alignment: Alignment.center,
          children: [
            // LayoutBuilder を使って実際に描画されている大きさを取得しています
            LayoutBuilder(
              builder: (BuildContext context, BoxConstraints constraints) {
                // この Container がダイアログの本体です
                return Container(
                  // 閉じるボタンの大きさだけ小さくしています
                  width: constraints.maxWidth - _buttonSize,
                  height: constraints.maxHeight - _buttonSize,
                  decoration: BoxDecoration(
                    color: Colors.white,
                    borderRadius: BorderRadius.circular(24),
                  ),
                  child: Column(
                    // mainAxisSize: ,
                    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                    children: <Widget>[
                      const Text(
                        'カウントアップ',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 20,
                        ),
                      ),
                      const Divider(),
                      SizedBox(
                        width: 32,
                        height: 32,
                        child: Center(
                            child: Text(
                          '${_provider.count}',
                          style: const TextStyle(
                            fontWeight: FontWeight.bold,
                            fontSize: 20,
                          ),
                        )),
                      ),
                      const Divider(),
                      FloatingActionButton(
                        onPressed: () {
                          _provider.increment();
                        },
                        child: const Icon(Icons.plus_one),
                      )
                    ],
                  ),
                );
              },
            ),
            // Align ウィジェットを使えば 配置は簡単ですね。
            // ボタンサイズ分だけダイアログを小さくしたおかげでちょっとはみ出した感じにできています。
            Align(
              alignment: Alignment.topLeft,
              child: closeButton(
                context,
                _buttonSize,
                () {
                  Navigator.pop(context);
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// 閉じるボタンは使いまわせそうだったので切り出しています
Widget closeButton(
  BuildContext context,
  double buttonSize,
  Function() onPressed,
) {
  return SizedBox(
    width: buttonSize * 1.2,
    height: buttonSize * 1.2,
    child: FloatingActionButton(
      child: Icon(
        Icons.clear,
        size: buttonSize,
        color: Colors.white,
      ),
      onPressed: () {
        onPressed();
      },
    ),
  );
}

```

**コントローラー**

```dart:clear_button_controller.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final clearButtonProvider =
    ChangeNotifierProvider.autoDispose<ClearButtonController>(
  (ref) => ClearButtonController(),
);

class ClearButtonController extends ChangeNotifier {
  ClearButtonController() {
    count = 0;
  }

  int count;

  void increment() {
    count++;
    notifyListeners();
  }
}

```

# 解説

コードにコメントをたくさん書いたので詳細はそちらを読んでいただければと思います。

このような方針で達成しています。

1. **デフォルトのダイアログの背景を透明にする**
   これで描画範囲が決まってしまうので、こいつをはみ出すようなウィジェットは配置できません。
   そのため、透明にした上で描画範囲を広くとっています。
1. **Container()を使ってダイアログをつくる**
   デフォルトは透明にしてしまったので、かわりにダイアログを自分で作っています。
1. **Align を使って閉じるボタンを配置する**
   何かのウィジェットの位置を基準に配置したい時に便利です。
1. **状態を伝搬したいのでダイアログは ConsumerWidget として宣言する**
   このようにか書くと provider を watch できます。

特にに 4. についてですが、Riverpod を使うことで context などを特に渡すことなく達成できています。
グローバルに provider がいることの恩恵ですね。

# おしまいに

もっとスマートな方法がある気がしますね。
少ないコードでかける方法を知っている方がいましたら、教えてください。
