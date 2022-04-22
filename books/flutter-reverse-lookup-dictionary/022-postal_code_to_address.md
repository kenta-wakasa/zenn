---
title: 郵便番号から住所を補完したい
---

# A. 公開されている Web API を活用しよう

![](/images/q22/1.gif)

新規登録画面ではユーザーの手間をいかに減らせるかが重要となります。郵便番号を入力したら自動で住所が補完されたりするとストレスがなく嬉しいですよね。今回はそんなときに役立つ記事です。

郵便番号から住所を割り出す API があるはずだと思い、[郵便番号 住所 変換 api]と検索すると次のページを発見しました。

http://zipcloud.ibsnet.co.jp/doc/api

利用登録などもなく気軽に使えそうです。
これを使っていくこととしましょう。

## API の使い方

とても簡単です。

```
https://zipcloud.ibsnet.co.jp/api/search?zipcode=[ここを郵便番号にする]
```

これを実行すると、このような結果が返ってきます。

```json
{
  "message": null,
  "results": [
    {
      "address1": "北海道",
      "address2": "美唄市",
      "address3": "上美唄町協和",
      "kana1": "ﾎｯｶｲﾄﾞｳ",
      "kana2": "ﾋﾞﾊﾞｲｼ",
      "kana3": "ｶﾐﾋﾞﾊﾞｲﾁｮｳｷｮｳﾜ",
      "prefcode": "1",
      "zipcode": "0790177"
    },
    {
      "address1": "北海道",
      "address2": "美唄市",
      "address3": "上美唄町南",
      "kana1": "ﾎｯｶｲﾄﾞｳ",
      "kana2": "ﾋﾞﾊﾞｲｼ",
      "kana3": "ｶﾐﾋﾞﾊﾞｲﾁｮｳﾐﾅﾐ",
      "prefcode": "1",
      "zipcode": "0790177"
    }
  ],
  "status": 200
}
```

Web ブラウザでも試せますので、ご自身の住所でやってみてください。

## zipCodeToAddress を定義

使い方がわかったところで郵便番号を住所に変換するトップレベルの関数を作っていきます。トップレベルなのでどこへでも持ち出して実行可能です。
http の通信には [http パッケージ](https://pub.dev/packages/http)を使います。

```dart
Future<String?> zipCodeToAddress(String zipCode) async {
  if (zipCode.length != 7) {
    return null;
  }
  final response = await get(
    Uri.parse(
      'https://zipcloud.ibsnet.co.jp/api/search?zipcode=$zipCode',
    ),
  );
  // 正常なステータスコードが返ってきているか
  if (response.statusCode != 200) {
    return null;
  }
  // ヒットした住所はあるか
  final result = jsonDecode(response.body);
  if (result['results'] == null) {
    return null;
  }
  final addressMap = (result['results'] as List).first; // 結果が2つ以上のこともあるが、簡易的に最初のひとつを採用することとする。
  final address = '${addressMap['address1']} ${addressMap['address2']} ${addressMap['address3']}'; // 住所を連結する。
  return address;
}
```

これで郵便番号から住所が取得できるようになりました。
それではこれを使って住所を補完してみましょう。
大まかな流れはこのようになります。

1. 郵便番号と住所を入力するための 2 つの TextEditingController を用意
1. 郵便番号が 7 桁の数字になったときに API を通して住所を取得
1. 正常に住所が返って来れば住所のテキスト欄を上書きする。

## コピペで動くサンプルコード

この手順で住所を取得したサンプルコードを作りましたので、参考にしてください。それではまた。

````dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:http/http.dart';

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
  final zipCodeController = TextEditingController();
  final addressController = TextEditingController();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('郵便番号から住所を補完したい'),
      ),
      body: Center(
        child: ConstrainedBox(
          constraints: const BoxConstraints(
            maxWidth: 400,
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              TextFormField(
                decoration: const InputDecoration(
                  hintText: '郵便番号',
                ),
                maxLength: 7,
                onChanged: (value) async {
                  // 入力された文字数が7以外なら終了
                  if (value.length != 7) {
                    return;
                  }
                  final address = await zipCodeToAddress(value);
                  // 返ってきた値がnullなら終了
                  if (address == null) {
                    return;
                  }
                  // 住所が帰ってきたら、addressControllerの中身を上書きする。
                  addressController.text = address;
                },
                controller: zipCodeController,
                keyboardType: TextInputType.number,
                inputFormatters: [FilteringTextInputFormatter.digitsOnly],
              ),
              TextFormField(
                decoration: const InputDecoration(
                  hintText: '住所',
                ),
                controller: addressController,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 郵便番号から住所を取得する
///
/// API: http://zipcloud.ibsnet.co.jp/doc/api
///
/// 利用規約: http://zipcloud.ibsnet.co.jp/rule/api
///
/// レスポンスサンプル
/// ```json
/// {
/// 	"message": null,
/// 	"results": [
/// 		{
/// 			"address1": "北海道",
/// 			"address2": "美唄市",
/// 			"address3": "上美唄町協和",
/// 			"kana1": "ﾎｯｶｲﾄﾞｳ",
/// 			"kana2": "ﾋﾞﾊﾞｲｼ",
/// 			"kana3": "ｶﾐﾋﾞﾊﾞｲﾁｮｳｷｮｳﾜ",
/// 			"prefcode": "1",
/// 			"zipcode": "0790177"
/// 		},
/// 		{
/// 			"address1": "北海道",
/// 			"address2": "美唄市",
/// 			"address3": "上美唄町南",
/// 			"kana1": "ﾎｯｶｲﾄﾞｳ",
/// 			"kana2": "ﾋﾞﾊﾞｲｼ",
/// 			"kana3": "ｶﾐﾋﾞﾊﾞｲﾁｮｳﾐﾅﾐ",
/// 			"prefcode": "1",
/// 			"zipcode": "0790177"
/// 		}
/// 	],
/// 	"status": 200
/// }
/// ```
Future<String?> zipCodeToAddress(String zipCode) async {
  if (zipCode.length != 7) {
    return null;
  }
  final response = await get(
    Uri.parse(
      'https://zipcloud.ibsnet.co.jp/api/search?zipcode=$zipCode',
    ),
  );
  // 正常なステータスコードが返ってきているか
  if (response.statusCode != 200) {
    return null;
  }
  // ヒットした住所はあるか
  final result = jsonDecode(response.body);
  if (result['results'] == null) {
    return null;
  }
  final addressMap = (result['results'] as List).first; // 結果が2つ以上のこともあるが、簡易的に最初のひとつを採用することとする。
  final address = '${addressMap['address1']} ${addressMap['address2']} ${addressMap['address3']}'; // 住所を連結する。
  return address;
}
````
