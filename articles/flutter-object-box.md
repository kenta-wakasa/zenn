---
title: "Flutter で ObjectBox つかってみた"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, objectbox]
published: true
---

# ObjectBox とは

処理速度が速いローカルデータベースです。

![](/images/benchmarks.png)
_https://pub.dev/packages/objectbox より引用_

Hive や sqflite と比較した図を見るとその性能のよさがわかります。膨大なデータを扱わなければ実感できないレベルだとは思いますが、速いにこしたことはありません。公式ドキュメントを見ながら、その使用感を試します。

https://docs.objectbox.io

# パッケージの導入

pubspec.yaml を編集して必要なパッケージを入れます。

```yaml
name: demoobjectbox
description: A new Flutter project.

publish_to: "none"
version: 1.0.0+1

environment:
  sdk: ">=2.16.2 <3.0.0"

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2

  objectbox: ^1.4.0 # ここ追加
  objectbox_flutter_libs: any # ここ追加

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^1.0.0

  build_runner: ^2.0.0 # ここ追加
  objectbox_generator: any # ここ追加

flutter:
  uses-material-design: true
```

自動生成をおこなうので build_runner と objectbox_generator も忘れずに追加しましょう。

# Entity クラスの作成

扱いたいオブジェクトを定義します。とてもシンプルな User クラスを定義してみましょう。

```dart
import 'package:objectbox/objectbox.dart';

@Entity() // アノテーションが必要
class User {
  User({this.name});
  int id = 0; // id をこのように定義すると自動でインクリメントしてくれる
  String? name;
}
```

id と name しか持たないシンプルな User クラスを定義しました。
型と変数名を決めるだけなのでとても簡単ですね。

これをもとにコードを自動生成します。ターミナルを開き次のコマンドを実行しましょう。

```sh
flutter pub run build_runner build
```

成功すると、objectbox-model.json, objectbox.g.dart が生成されます。

# store を open する

データを追加したり取得するためには store を open する必要があります。

```dart
final store = await openStore();
```

この store はグローバルに持っていても問題ありません。
main 関数の中で呼ぶことでその後は自由に使用できます。

```dart
import 'package:flutter/material.dart';

import 'objectbox.g.dart';
import 'user.dart';

late Store store;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  store = await openStore();

  runApp(const MyApp());
}
```

# Box を取得

User クラスが格納される Box をまずは取得します。

```dart
final userBox = store.box<User>();
```

# User を追加

userBox の put メソッド簡単に追加できます。

```dart
final user = User(name: '適切なユーザー名');
userBox.put(user);
```

put は同期関数のため、await は必要ありません。
hive では非同期関数でしたね。

# User 一覧を取得

getAll メソッドを呼ぶだけです。

```dart
userBox.getAll() // 型は List<User> なので便利
```

# Query をつかって名前検索

query が使えるのも魅力です。
name が 'こんぶ' と一致するユーザーを取得するには次のようにします。

```dart
  final query = userBox.query(User_.name.equals('こんぶ')).build();
  final results = query.find();
  query.close();
```

query の詳細は次のドキュメントを参照してください。

https://docs.objectbox.io/queries

# Hive との違い

使用感が似ている Hive との違いについて感想です。

## Entity に id が含まれている

Hive は Entity に id を含めなくても、key を自由に指定して put したり、add すれば自動でインクリメントしてくれたりします。ObjectBox は id が必須となっているので、好みが分かれそうです。

id が入ってしまうことの不便な点は、immutable なクラスにしたい場合に自動インクリメントが使えないところですかね。というのもインスタンス生成時に必ず id を決めなくてはいけないので、uuid などを使って新規 id を生成する必要があります。

freezed と一緒に使うこともできるそうなので、freezed を普段使っている方との相性はいいですね。

https://pub.dev/documentation/objectbox/latest/objectbox/Entity/realClass.html

## box を open ではなく store を open するだけでいい

hive の場合は box に名前をつけて open しなければいけませんが、ObjectBox は store を open すればすべての box が使用可能です。box は Entity の型と関連づいているので、なんでも追加できる hive よりも安全です。

細かな違いはありますが、大差はないと感じましたので、Hive を使っている方は ObjectBox を採用するのもありかと思います。

# コピペで動くサンプルコード

名前を入力してユーザーを追加するだけの簡単なサンプルを作りましたのでご紹介しておしまいにします。
また次の記事でお会いしましょう。

![](/images/objectbox.gif)

```dart:user.dart
import 'package:objectbox/objectbox.dart';

/// freezedと合わせてimmutableにする
/// https://pub.dev/documentation/objectbox/latest/objectbox/Entity/realClass.html

@Entity()
class User {
  User({this.name});
  // Annotate with @Id() if name isn't "id" (case insensitive).
  int id = 0;
  final String? name;
}
```

```dart:main.dart
import 'dart:async';

import 'package:flutter/material.dart';

import 'objectbox.g.dart';
import 'user.dart';

late Store store;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  store = await openStore();

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Builder(builder: (context) {
        return const DemoPage();
      }),
    );
  }
}

class DemoPage extends StatefulWidget {
  const DemoPage({Key? key}) : super(key: key);

  @override
  State<DemoPage> createState() => _DemoPageState();
}

class _DemoPageState extends State<DemoPage> {
  final userBox = store.box<User>();

  final controller = TextEditingController();

  List<User> query() {
    // nameが'こんぶ'と一致したUserを返す。
    final query = userBox.query(User_.name.equals('こんぶ')).build();
    final results = query.find();
    query.close();
    return results;
  }

  StreamSubscription? _sub;

  @override
  void initState() {
    // watchで変更を監視できる
    _sub = store.watch<User>().listen((event) {
      setState(() {});
    });

    super.initState();
  }

  @override
  void dispose() {
    controller.dispose();
    _sub?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ObjectBox'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(32),
            child: TextFormField(
              controller: controller,
              decoration: const InputDecoration(hintText: 'ユーザー名'),
            ),
          ),
          ElevatedButton(
            onPressed: () {
              final user = User(name: controller.text);
              userBox.put(user);
            },
            child: const Text('ユーザーを追加する'),
          ),
          const SizedBox(height: 16),
          const Text('データ全体'),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: userBox
                .getAll()
                .map(
                  (e) => Text('ID: ${e.id}, name: ${e.name ?? '名無し'}'),
                )
                .toList(),
          ),
          const SizedBox(height: 16),
          const Text('検索'),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: query()
                .map(
                  (e) => Text('ID: ${e.id}, name: ${e.name ?? '名無し'}'),
                )
                .toList(),
          ),
        ],
      ),
    );
  }
}
```
