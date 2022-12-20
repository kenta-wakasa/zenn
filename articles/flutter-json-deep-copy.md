---
title: "dartでネストされたMapをディープコピーする"
emoji: "🌀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, json, dart]
published: true
---

こんにちは、こんぶです。
いつも見ていただいてありがとうございます。

JSON 形式のデータを扱っていると Map をコピーして何かしらに変換するという操作を行うことがたまにあります。

このとき、いろいろな罠があり、参照渡しではなくディープコピーをしようと思うと大変だったので、記事でまとめてみます。

# サンプルデータ

Map がネストしているかつ、中に List が入っているデータを想定します。

```dart:テストデータ
  final originData = {
    'name': 'こんぶ',
    'company': {
      'name': 'KBOY',
      'employees': [
        'kboy',
        'こんぶ',
      ]
    }
  };
```

# 浅いコピー

「じゃあこいつをコピーしよっと」と思い、次のように書いたとします。

```dart
  test('Mapのディープコピー', () {
    final originData = {
      'name': 'こんぶ',
      'company': {
        'name': 'KBOY',
        'employees': [
          'kboy',
          'こんぶ',
        ]
      }
    };
    final copiedData = originData; // <- これでコピーやで！
    copiedData['name'] = 'ダイゴ';
    log(const JsonEncoder.withIndent('  ').convert(originData));
    expect(originData != copiedData, true);
  });
```

コピーした `copiedData` name を `ダイゴ` で上書きしました。
`expect(originData != copiedData, true);` このテストは通るでしょうか。

残念ながら通りません。

```dart
final copiedData = originData;
```

このように単純に代入しただけではそのデータの参照を渡していることにしかなっていません。

# インスタンスの再生成

だったらインスタンスを再生成すれば深いコピーになるのでは？

```dart
final copiedData = Map.of(originData);
```

これでどうでしょうか。

```diff dart
  test('Mapのディープコピー', () {
    final originData = {
      'name': 'こんぶ',
      'company': {
        'name': 'KBOY',
        'employees': [
          'kboy',
          'こんぶ',
        ]
      }
    };
-   final copiedData = originData;
+   final copiedData = Map.of(originData); // <- これなら大丈夫なはず！
    copiedData['name'] = 'ダイゴ';
    log(const JsonEncoder.withIndent('  ').convert(originData));
    expect(originData != copiedData, true);
  });
```

このテストは無事通りました。

ではこのまま `company` の 名前を `株式会社KBOY` に変えてみましょう。

そして `company` を比較してみましょう。

先ほどのテストコードの続きに、以下のコードを足して実行してみます。

```dart:companyの比較
/// companyの比較
///
/// コピーしたデータを変更してみる
(copiedData['company'] as Map)['name'] = '株式会社KBOY';

log(const JsonEncoder.withIndent('  ').convert(originData));

/// コピー元の company と比較したら違うオブジェクトとして認識されるのか？
expect(originData['company'] != copiedData['company'], true);
```

`テスト実行中...`

結果、通りません。実はこれもエラーになってしまいます。

copiedData の company のデータだけを変えたつもりが、originData のデータも書き変わってしまうのです。

実はネストされた Map は `Map.of` などを使ったとしても、参照が渡されるだけになってしまうんですね。

List だとどうでしょうか？

`copiedData` の中にある `employees` だけに `ダイゴ` を追加して比較してみましょう。

```dart
/// employees の比較
///
/// コピーしたデータのemployeesにダイゴを追加してみよう
((copiedData['company'] as Map)['employees'] as List).add('ダイゴ');

log((originData['company'] as Map)['employees'].toString());
expect(
    (originData['company'] as Map)['employees'] !=
        (copiedData['company'] as Map)['employees'],
    true);
```

実行してログを見てもらうとわかるように `originData` の `employees` にもダイゴが追加されてしまうのです。

んー困った困った、どうすれば...。

解決策はあります。

# ネストされた Map をディープコピーするための再帰関数を定義する

```dart: ディープコピーを実現する関数
  dynamic deepCopy(dynamic origin) {
    /// Map のとき
    if (origin is Map) {
      final copiedMap = {};
      for (final key in origin.keys) {
        final value = origin[key];
        copiedMap[key] = deepCopy(value);
      }
      return copiedMap;
    }

    /// List のとき
    if (origin is List) {
      final copiedList = [];
      for (final element in origin) {
        copiedList.add(deepCopy(element));
      }
      return copiedList;
    }

    /// それ以外のとき
    return origin;
  }
```

この関数を使って、今までのテストコードを実行してみましょう。
するとうまくテストが通ることが確認できます。

もっと複雑なもの、たとえば、List の中に Map が存在している場合などもみてみましょう。

検証のためのテストデータを作ります。

```dart:リストの中にMapがあるパターン
    final originMapInListData = {
      'companies': [
        {
          'name': 'KBOY',
          'employees': [
            'kboy',
            'こんぶ',
          ],
        },
      ]
    };

    final copiedMapInListData = deepCopy(originMapInListData);

    /// List の中の Map を更新してみる
    (copiedMapInListData['companies'] as List).first['name'] = '株式会社KBOY';
    expect(
        (originMapInListData['companies'] as List).first !=
            (copiedMapInListData['companies'] as List).first,
        true);
```

このようなパターンであってもうまく変換されることがわかりました。

# もうひとつの解決策

これは JSON データに限った話になりますが、deepCopy の関数をどうしても使いたくない場合はこのようにもかけます。

```dart
final copiedData = jsonDecode(jsonEncode(originData));
```

エンコードした後に、デコードして戻すという手法です。

こうすることでどんなにネストしていても強制的にディープコピーにすることができます。

見た目では気持ち悪いですが、サクッとやってしまいたいときには便利です。

今回はここまで！
