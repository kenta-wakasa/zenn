---
title: "Q.2 popして戻ってきたときに処理を実行したい"
---

# A. push は await で待つことができます。

ユーザーの編集画面に遷移して、ユーザー情報を更新した後、ユーザーページに戻ってきたときに値が更新されない。このような問題は Flutter の初学者であれば誰もが一度は通ります。

この問題は、画面遷移時に実行する push 関数を await で待つことによって解決できます。

# コピペで試せるサンプルコード

サンプルコードを作成したので、試してみましょう。

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ユーザーデータを保存するために shared_preferences を使用します。
late final SharedPreferences prefs;

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // main関数の中でインスタンスを生成してしまいます。
  prefs = await SharedPreferences.getInstance();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: UserPage());
  }
}

/// ユーザーページ
class UserPage extends StatefulWidget {
  const UserPage({Key? key}) : super(key: key);

  @override
  State<UserPage> createState() => _UserPageState();
}

class _UserPageState extends State<UserPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ユーザー'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('ユーザー名'),
            // ユーザー名が未入力の時は 名無し と表示する。
            Text(prefs.getString('name') ?? '名無し'),
            ElevatedButton(
              onPressed: () async {
                // push は await で待てる。
                await Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (context) {
                      return const EditUserPage();
                    },
                  ),
                );
                // EditUserPage から UserPage に戻ってきたら、これ以下の処理が実行されます。
                // 今回は画面を再描画するだけです。
                // await のあるなし setState のあるなしでどのような変化があるか試してみてください。
                setState(() {});
              },
              child: const Text('編集'),
            ),
          ],
        ),
      ),
    );
  }
}

/// ユーザー編集ページ
class EditUserPage extends StatefulWidget {
  const EditUserPage({Key? key}) : super(key: key);

  @override
  State<EditUserPage> createState() => _EditUserPageState();
}

class _EditUserPageState extends State<EditUserPage> {
  final controller = TextEditingController();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('編集')),
      body: Center(
        child: TextFormField(
          controller: controller,
          // 完了を押したときに実行されます。
          // デスクトップやブラウザならEnterキーを押した時に実行されます。
          onFieldSubmitted: (value) {
            prefs.setString('name', value);
          },
        ),
      ),
    );
  }
}
```

# 型を確認する習慣をつけよう

push は await で待てる。
この事実を知っているだけ今回の問題は解決できたのですが、これがなかなか気づけません。

ですが型に注目することができれば簡単に知ることができます。

Visual Studio Code で型を確認するには、その関数の上にマウスカーソルを重なるだけです。

![](/images/q2/1.gif)

push は Future<T?>型を返すことがわかりますので、await で待てるんだなと気づくことができます。

型を確認する習慣をつけると、よりプログラミングが楽になると思います。
ぜひ実践してみてください！
