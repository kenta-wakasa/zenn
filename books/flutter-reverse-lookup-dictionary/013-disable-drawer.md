---
title: "スワイプでドロワーが出ないようにしたい"
---

![](/images/q13/1.gif)

# A. スワイプで Drawer を出さないのはどうでしょう？

Android や iOS では画面左端から右へスワイプすると前のページに戻るスワイプバック機能が備わっています。それと同じように flutter の Drawer Widget も画面左端からスワイプすることで表示させることができますが、スワイプバックと干渉します。

1. Drawer が開く
1. スワイプバックが実行されて Drawer が閉じる

これが繰り返されてなかなか Drawer が開かず、最終的にはスワイプバックだけが反応し、アプリが終了してしまうこともしばしば。ユーザーに高負荷のストレスを与えてしまう原因となっています。

そこで今回はそもそも Drawer をスワイプで開かないようにする方法を紹介します。
根本的な解決にはなっていないのですが、Drawer は左上のハンバーガーメニューをタップしても開けますので、干渉するよりもマシという判断です。

## drawerEdgeDragWidth を 0 にする

方法はとても簡単で、タイトルの通り Scaffold の drawerEdgeDragWidth プロパティに 0 を指定するだけです。これで Drawer がスワイプでは出なくなります。

```dart
Scaffold(
  drawer: const Drawer(
    backgroundColor: Colors.blue,
  ),
  drawerEdgeDragWidth: 0, // これを0にする。
)
```

もっといい解決策を思いつきましたら、また共有いたします！

## コピペで動くサンプルコード

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
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      drawer: const Drawer(
        backgroundColor: Colors.blue,
      ),
      drawerEdgeDragWidth: 0, // これを0にする。
      appBar: AppBar(
        title: const Text(
          'スワイプバックが干渉する',
          style: TextStyle(
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }
}
```
