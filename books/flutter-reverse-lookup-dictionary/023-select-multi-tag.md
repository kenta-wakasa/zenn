---
title: 複数選択できるタグをつくりたい
---

![](/images/q23/1.gif)

# A. List を使いこなしましょう

検索システムなどをつくっていて、条件を複数選択するという場面はよくあります。そのような UI の作りたかをご紹介します。

## map をつかって UI を生成する

お酒を検索するシステムを作るとしましょう。検索対象となるお酒の種類を次のように決めました。String の List としてもたせます。

```dart
  /// 検索したいお酒の種類を選ぶことを想定
  final tags = [
    'ビール',
    'ワイン',
    '日本酒',
    '焼酎',
    'ウィスキー',
    'ジン',
    'ウォッカ',
    '紹興酒',
    'マッコリ',
    'カクテル',
    '果実酒',
  ];
```

これを使って、選択できるボタンを作ってきます。横幅に応じて自動で改行されてほしいので Wrap を使います。まずは Text だけを表示してみましょう。List の map メソッドを使えば、String 型の List からその数分だけの Widget に変換することができます。

```dart
  Wrap(
    runSpacing: 16,
    spacing: 16,
    // .map を活用しよう
    children: tags.map((tag) {
      return Text(tag); // <- もともと String だった tag を Text に変換しているイメージ
    }).toList(),
  )
```

Text をカスタマイズして、ボタンにしていきます。見た目は Container でつくって、tap できるようにするために InkWell で囲みます。さらに自分が選択されているかを判断するために isSelected 変数を用意しておきます。あとから中身は実装しますので、いったん常に false になるようにしておきます。この isSelected を使い、選択されていれば色が変わるように制御します。

```dart
    Wrap(
      runSpacing: 16,
      spacing: 16,
      children: tags.map((tag) {
        // selectedTags の中に自分がいるかを確かめる
        final isSelected = false // 後で実装する。
        return InkWell(
          borderRadius: const BorderRadius.all(Radius.circular(32)),
          onTap: () {
            // ここに tap されたときの処理
          },
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 200),
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            decoration: BoxDecoration(
              borderRadius: const BorderRadius.all(Radius.circular(32)),
              border: Border.all(
                width: 2,
                color: Colors.pink,
              ),
              color: isSelected ? Colors.pink : null,
            ),
            child: Text(
              tag,
              style: TextStyle(
                color: isSelected ? Colors.white : Colors.pink,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        );
      }).toList(),
    ),
```

次に自分が選択されているかどうかの判定と、タップされたときに実行される関数を定義していきましょう。まずは状態として、選択された tag を記録するための変数を用意します。

```dart
  /// 選択されたタグ
  var selectedTags = <String>[];
```

tag が選択されているかどうかは List の contains メソッドをつかえば調べられます。contains メソッドは引数として受け取ったものを持っていれば true,持っていなければ false を返すので...

```dart
  // selectedTags の中に自分がいるかを確かめる
  final isSelected = selectedTags.contains(tag);
```

選択されているかどうかはこのように書けますね。

では、タップ時の関数ですが、選択されていれば選択を解除し、選択されていなければ選択状態になるようにしましょう。

これも List の add と remove 関数を使えばできます。add は引数に与えられた要素を List に追加。remove は引数に与えられた要素を List から取り除きます。

```dart
  onTap: () {
    if (isSelected) {
      // すでに選択されていれば取り除く
      selectedTags.remove(tag);
    } else {
      // 選択されていなければ追加する
      selectedTags.add(tag);
    }
    setState(() {});
  },
```

これですべての準備が整いました。最終的に出来上がったコードを載せますので参考にしてください。

## コピペで動くサンプルコード

![](/images/q23/1.gif)

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: DemoPage(),
    );
  }
}

class DemoPage extends StatefulWidget {
  const DemoPage({Key? key}) : super(key: key);

  @override
  State<DemoPage> createState() => _DemoPageState();
}

class _DemoPageState extends State<DemoPage> {
  /// 検索したいお酒の種類を選ぶことを想定
  final tags = [
    'ビール',
    'ワイン',
    '日本酒',
    '焼酎',
    'ウィスキー',
    'ジン',
    'ウォッカ',
    '紹興酒',
    'マッコリ',
    'カクテル',
    '果実酒',
  ];

  /// 選択されたタグ
  var selectedTags = <String>[];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('複数選択できるタグ')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Wrap(
              runSpacing: 16,
              spacing: 16,
              children: tags.map((tag) {
                // selectedTags の中に自分がいるかを確かめる
                final isSelected = selectedTags.contains(tag);
                return InkWell(
                  borderRadius: const BorderRadius.all(Radius.circular(32)),
                  onTap: () {
                    if (isSelected) {
                      // すでに選択されていれば取り除く
                      selectedTags.remove(tag);
                    } else {
                      // 選択されていなければ追加する
                      selectedTags.add(tag);
                    }
                    setState(() {});
                  },
                  child: AnimatedContainer(
                    duration: const Duration(milliseconds: 200),
                    padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                    decoration: BoxDecoration(
                      borderRadius: const BorderRadius.all(Radius.circular(32)),
                      border: Border.all(
                        width: 2,
                        color: Colors.pink,
                      ),
                      color: isSelected ? Colors.pink : null,
                    ),
                    child: Text(
                      tag,
                      style: TextStyle(
                        color: isSelected ? Colors.white : Colors.pink,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                );
              }).toList(),
            ),
            Expanded(
              child: Center(
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    ElevatedButton(
                      onPressed: () {
                        selectedTags.clear();
                        setState(() {});
                      },
                      child: const Text('クリア'),
                    ),
                    const SizedBox(width: 16),
                    ElevatedButton(
                      onPressed: () {
                        // deep copy する方法
                        // selectedTags = tags だと参照を渡したことにしかならない
                        selectedTags = List.of(tags);
                        setState(() {});
                      },
                      child: const Text('ぜんぶ'),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
