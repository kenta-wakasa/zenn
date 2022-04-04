---
title: "Stringをenumに変換したい"
---

# A. byName を使いましょう

ある Web API からの次のような JSON レスポンスが返ってきました。

```json
{
  "name": "こんぶ",
  "job": "engineer"
}
```

バックエンドのエンジニアから、
「job は engineer, teacher, fireman, dancer, policeman, actor のどれかしか返ってこないよ」
と言われました。

こういうときは enum を定義したくなりますね。
String 型だとどんな文字列でも入る余地がありますが、enum にすることでより範囲を狭めることができ、さまざまな場面で便利になります（Switch での分岐など）。コードから仕様を察することができますし、予期せぬエラーに気付きやすいです。

dart で enum を定義するときは次のように書きます。

```dart
enum Job {
  engineer,
  teacher,
  fireman,
  dancer,
  policeman,
  actor,
}
```

では JSON で受け取った文字列を enum に変換してみましょう。

あなたならどう書きますか？

switch 文が使えそうだと思われるかもしれません。
その場合、およそこのようになるでしょう。

```dart
Job string2Job(String value) {
  switch (value) {
    case 'engineer':
      return Job.engineer;
    case 'teacher':
      return Job.teacher;
    case 'fireman':
      return Job.fireman;
    case 'dancer':
      return Job.dancer;
    case 'policeman':
      return Job.policeman;
    case 'actor':
      return Job.actor;
    default:
      throw Exception('No enum value');
  }
}
```

しかしもっと簡単に書く方法があります。

```dart
Job.values.byName(value);
```

です。

この機能は dart 2.15 から使用可能になりました。

ではこれを使って JSON データから User インスタンスを作成する fromJson メソッドをつくってみましょう。

```dart
factory User.fromJson(String json) {
  final map = jsonDecode(json) as Map<String, dynamic>;
  return User(
    name: map['name'],
    job: Job.values.byName(map['job']), // byNameを使えば簡単にenumに変換できる。
  );
}
```

わざわざ自前で string2Job のような関数を定義しなくてもよいので便利です。

このように enum は少しずつ使いやすくなっています。
ですがまだまだ機能が豊富とは言えません。

またどこかの機会で extension をつかった拡張方法を紹介したいと思いますのでお楽しみに！

# コピペで動くサンプルコード

```dart
import 'dart:convert';
import 'dart:developer';

import 'package:flutter/material.dart';

void main() {
  final user = User.fromJson(source);
  log(user.toString());
}

enum Job {
  engineer,
  teacher,
  fireman,
  dancer,
  policeman,
  actor,
}

const source = '''
{
  "name": "こんぶ",
  "job": "engineer"
}
''';

class User {
  User({
    required this.name,
    required this.job,
  });

  factory User.fromJson(String json) {
    final map = jsonDecode(json) as Map<String, dynamic>;
    return User(
      name: map['name'],
      job: Job.values.byName(map['job']),
    );
  }
  @override
  String toString() {
    return 'User : name=$name, job=${job.name}'; // .nameで元の文字列を取り出せる。
  }

  final String name;
  final Job job;
}
```
