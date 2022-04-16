---
title: ダイアログでもテキスト入力がしたい
---

![](/images/q16/1.gif)

# A. StatefulWidget もダイアログに出せます

簡単な項目を編集したいとき、わざわざ編集ページに遷移させるよりも、ダイアログを出して、そこで入力さる方がスムーズなユーザー体験を提供できます。

ダイアログの中で TextFormField を使うことなんてできるのかと思われるかもしれませんが、ダイアログというのは思った以上になんでも出せます。状態を持たせることも可能です。

## タイトルのみを出す場合

簡単な文字を出すだけのダイアログは次のように書きます。

```dart
  /// タイトルのみを表示するシンプルなダイアログを表示する
  static Future<void> showOnlyTitleDialog(
    BuildContext context,
    String title,
  ) async {
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text(title),
        );
      },
    );
  }
```

## テキスト編集をする場合

この AlertDialog を　 StatefulWidget でラップしてあげれば状態を持たせることができます。
文字列を編集するための EditingDialog Widget を作成しました。

```dart
/// 状態を持ったダイアログ
class TextEditingDialog extends StatefulWidget {
  const TextEditingDialog({Key? key, this.text}) : super(key: key);
  final String? text;

  @override
  State<TextEditingDialog> createState() => _TextEditingDialogState();
}

class _TextEditingDialogState extends State<TextEditingDialog> {
  final controller = TextEditingController();
  final focusNode = FocusNode();
  @override
  void dispose() {
    controller.dispose();
    focusNode.dispose();
    super.dispose();
  }

  @override
  void initState() {
    super.initState();
    // TextFormFieldに初期値を代入する
    controller.text = widget.text ?? '';
    focusNode.addListener(
      () {
        // フォーカスが当たったときに文字列が選択された状態にする
        if (focusNode.hasFocus) {
          controller.selection = TextSelection(baseOffset: 0, extentOffset: controller.text.length);
        }
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      content: TextFormField(
        autofocus: true, // ダイアログが開いたときに自動でフォーカスを当てる
        focusNode: focusNode,
        controller: controller,
        onFieldSubmitted: (_) {
          // エンターを押したときに実行される
          Navigator.of(context).pop(controller.text);
        },
      ),
      actions: [
        TextButton(
          onPressed: () {
            Navigator.of(context).pop(controller.text);
          },
          child: const Text('完了'),
        )
      ],
    );
  }
}
```

後はこれを簡単に呼び出せるように showEditingDialog 関数として定義します。
完了ボタンが押されれば入力していた文字列が返ってきます。
ダイアログが閉じられれば null が返ります。

```dart
  /// 入力した文字列を返すダイアログを表示する
  Future<String?> showEditingDialog(
    BuildContext context,
    String text,
  ) async {
    return showDialog<String>(
      context: context,
      builder: (context) {
        return TextEditingDialog(text: text);
      },
    );
  }
```

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
  var name = 'こんぶ';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          '状態をもったダイアログ',
          style: TextStyle(
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () async {
                DialogUtils.showOnlyTitleDialog(context, 'ダイアログやで');
              },
              child: const Text('タイトルのみ'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () async {
                final result = await DialogUtils.showEditingDialog(context, name);
                setState(() {
                  name = result ?? name;
                });
              },
              child: Text('$name 👈これを編集する'),
            ),
          ],
        ),
      ),
    );
  }
}

class DialogUtils {
  DialogUtils._();

  /// タイトルのみを表示するシンプルなダイアログを表示する
  static Future<void> showOnlyTitleDialog(
    BuildContext context,
    String title,
  ) async {
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text(title),
        );
      },
    );
  }

  /// 入力した文字列を返すダイアログを表示する
  static Future<String?> showEditingDialog(
    BuildContext context,
    String text,
  ) async {
    return showDialog<String>(
      context: context,
      builder: (context) {
        return TextEditingDialog(text: text);
      },
    );
  }
}

/// 状態を持ったダイアログ
class TextEditingDialog extends StatefulWidget {
  const TextEditingDialog({Key? key, this.text}) : super(key: key);
  final String? text;

  @override
  State<TextEditingDialog> createState() => _TextEditingDialogState();
}

class _TextEditingDialogState extends State<TextEditingDialog> {
  final controller = TextEditingController();
  final focusNode = FocusNode();
  @override
  void dispose() {
    controller.dispose();
    focusNode.dispose();
    super.dispose();
  }

  @override
  void initState() {
    super.initState();
    // TextFormFieldに初期値を代入する
    controller.text = widget.text ?? '';
    focusNode.addListener(
      () {
        // フォーカスが当たったときに文字列が選択された状態にする
        if (focusNode.hasFocus) {
          controller.selection = TextSelection(baseOffset: 0, extentOffset: controller.text.length);
        }
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      content: TextFormField(
        autofocus: true, // ダイアログが開いたときに自動でフォーカスを当てる
        focusNode: focusNode,
        controller: controller,
        onFieldSubmitted: (_) {
          // エンターを押したときに実行される
          Navigator.of(context).pop(controller.text);
        },
      ),
      actions: [
        TextButton(
          onPressed: () {
            Navigator.of(context).pop(controller.text);
          },
          child: const Text('完了'),
        )
      ],
    );
  }
}
```
