---
title: クレジットの入力フォームを作りたい
---

![](/images/q26/1.gif)

# A. TextFormField をシンプルに組み合わせてみましょう

決済の絡むサービスでは、クレジット情報の入力をしなければならないことがよくあります。面倒な作業なのでできるだけユーザーの負担を軽くしてあげたいものです。今回はできるだけシンプルな形で、クレジットカードの入力フォームを作っていきます。

## InputDecoration.collapsed でシンプルな入力フォームにする

シンプルな TextFormField の組み合わせで作っていきたいのですが、枠線などが入っていて扱いずらい場合があります。そんなときは、decoration に InputDecoration.collapsed()を与えることで、何の装飾もない TextFormField を作成することが可能です。

## 入力できる文字を制限する

keyboardType と inputFormatters をつかって制限をかけていきます。inputFormatter は自作することもできるので、もっと強固なバリデーションをしたい場合はこちらに挑戦してみるとよいでしょう。今回の要件は文字数と数字のみしか入力しないように制限することとしました。クレジットカード番号は大抵 16 桁ですのでその場合は次のように書きます。

```dart
TextFormField(
  // キーボードのタイプを数字にする
  keyboardType: TextInputType.number,
  inputFormatters: [
    // 文字数を制限できる
    LengthLimitingTextInputFormatter(16),
    // 数字のみしか入力させないようにする
    FilteringTextInputFormatter.digitsOnly,
  ],
)
```

## 入力が終わったら次のフォームに勝手に遷移させたい

クレジットカード番号を入力したら有効期限のフォームへ、それが終わったら、セキュリティ番号の入力へと自動で遷移させたいものです。そんなときは FocusScope.of(context).nextFocus() を使いましょう。

```dart
  onChanged: (value) {
    // 指定した文字列になったら次のフォームに移動する。
    if (value.length == 16) {
      FocusScope.of(context).nextFocus();
    }
  },
```

## コピペで動くサンプルコード

これらを組み合わせたサンプルコードを作りましたので活用していただければと思います。

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

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
  var position = const Offset(0, 0);
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ダイアログ'),
      ),
      body: Center(
          child: ElevatedButton(
        onPressed: () {
          showDialog(
            context: context,
            builder: (context) => const InputCardDialog(),
          );
        },
        child: const Text('クレジットカードの登録'),
      )),
    );
  }
}

class InputCardDialog extends StatefulWidget {
  const InputCardDialog({Key? key}) : super(key: key);

  @override
  State<InputCardDialog> createState() => _InputCardDialogState();
}

class _InputCardDialogState extends State<InputCardDialog> {
  late final creditCardNumberController = TextEditingController()
    ..addListener(() {
      setState(() {});
    });

  late final numberController = TextEditingController()
    ..addListener(() {
      setState(() {});
    });

  late final expirationMonthController = TextEditingController()
    ..addListener(() {
      setState(() {});
    });

  late final expirationYearController = TextEditingController()
    ..addListener(() {
      setState(() {});
    });
  late final cvcController = TextEditingController()
    ..addListener(() {
      setState(() {});
    });

  bool get validate {
    return numberController.text.length == 16 &&
        expirationMonthController.text.length == 2 &&
        expirationYearController.text.length == 2 &&
        cvcController.text.length == 3;
  }

  bool isProcessing = false;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      buttonPadding: const EdgeInsets.all(32),
      backgroundColor: Colors.blue[50],
      title: const Text(
        '支払いカードを登録する',
        textAlign: TextAlign.center,
        style: TextStyle(
          fontSize: 18,
          fontWeight: FontWeight.bold,
        ),
      ),
      content: Padding(
        padding: const EdgeInsets.symmetric(vertical: 32),
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          decoration: const BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.all(
              Radius.circular(8),
            ),
          ),
          child: Row(
            children: [
              SizedBox(
                width: 200,
                child: TextFormField(
                  autofocus: true,
                  controller: numberController,
                  keyboardType: TextInputType.number,
                  inputFormatters: [
                    // 文字数を制限できる
                    LengthLimitingTextInputFormatter(16),
                    FilteringTextInputFormatter.digitsOnly,
                  ],
                  decoration: const InputDecoration.collapsed(
                    hintText: 'Card number',
                  ),
                  onChanged: (value) {
                    // 指定した文字列になったら次のフォームに移動する。
                    if (value.length == 16) {
                      FocusScope.of(context).nextFocus();
                    }
                  },
                ),
              ),
              SizedBox(
                width: 32,
                child: TextFormField(
                  controller: expirationMonthController,
                  keyboardType: TextInputType.number,
                  inputFormatters: [
                    // 文字数を制限できる
                    LengthLimitingTextInputFormatter(2),
                    FilteringTextInputFormatter.digitsOnly,
                  ],
                  decoration: const InputDecoration.collapsed(hintText: 'MM'),
                  onChanged: (value) {
                    if (value.length == 2) {
                      FocusScope.of(context).nextFocus();
                    }
                  },
                ),
              ),
              const Padding(
                padding: EdgeInsets.symmetric(horizontal: 8),
                child: Text('/'),
              ),
              SizedBox(
                width: 32,
                child: TextFormField(
                  controller: expirationYearController,
                  keyboardType: TextInputType.number,
                  inputFormatters: [
                    // 文字数を制限できる
                    LengthLimitingTextInputFormatter(2),
                    FilteringTextInputFormatter.digitsOnly,
                  ],
                  decoration: const InputDecoration.collapsed(hintText: 'YY'),
                  onChanged: (value) {
                    if (value.length == 2) {
                      FocusScope.of(context).nextFocus();
                    }
                  },
                ),
              ),
              SizedBox(
                width: 40,
                child: TextFormField(
                  controller: cvcController,
                  keyboardType: TextInputType.number,
                  inputFormatters: [
                    // 文字数を制限できる
                    LengthLimitingTextInputFormatter(3),
                    FilteringTextInputFormatter.digitsOnly,
                  ],
                  decoration: const InputDecoration.collapsed(hintText: 'CVC'),
                ),
              ),
            ],
          ),
        ),
      ),
      actions: [
        ElevatedButton(
          onPressed: validate ? () {} : null,
          child: const Text('登録'),
        )
      ],
    );
  }
}
```
