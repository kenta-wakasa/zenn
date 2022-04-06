---
title: "ダイアログ上でページ遷移したい"
---

# A. Navigator を適切に使いましょう

たまに dialog の中でページ遷移させたいことがあると思います。

最終的な動きは次のようにしたいです。

![](/images/q6/1.gif)

素直に push すると次のような動きになると思います。

![](/images/q6/2.gif)

素直にやった場合のダイアログのコードはこのようになっています。

```dart
class CustomDialog extends StatelessWidget {
  const CustomDialog({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      contentPadding: EdgeInsets.zero,
      insetPadding: const EdgeInsets.all(32),
      content: SizedBox(
        width: 400,
        child: Scaffold(
          appBar: AppBar(
            centerTitle: true,
            title: const Text('1'),
          ),
          body: Center(
            child: ElevatedButton(
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (context) {
                      return const NextDialog();
                    },
                  ),
                );
              },
              child: const Text('2のページに進む'),
            ),
          ),
        ),
      ),
    );
  }
}
```

これだと大元の Navigator が反応し、ダイアログを無視してその上に新しい画面が表示されます。

この問題を Navigator をつかって解決します。次のように使ってあげるとうまくいきます。上の例と比べてみてください。

```dart
class CustomDialog extends StatelessWidget {
  const CustomDialog({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      contentPadding: EdgeInsets.zero,
      insetPadding: const EdgeInsets.all(32),
      content: Navigator(
        onGenerateRoute: (_) {
          return MaterialPageRoute(
            builder: ((context) => SizedBox(
                  width: 400,
                  child: Scaffold(
                    appBar: AppBar(
                      centerTitle: true,
                      title: const Text('1'),
                    ),
                    body: Center(
                      child: ElevatedButton(
                        onPressed: () {
                          Navigator.of(context).push(
                            MaterialPageRoute(
                              builder: (context) {
                                return const NextDialog();
                              },
                            ),
                          );
                        },
                        child: const Text('2のページに進む'),
                      ),
                    ),
                  ),
                )),
          );
        },
      ),
    );
  }
}
```

このように Navigator でラップしてあげることにより、どの文脈での push なのかを制御することができます。これを Navigator のネストなどと呼ばれることがありますが、非常に全体の把握が難しくバグの温床になりそうなコードです。

わかりづらいこのようなネストなどを宣言的に定義するという試みがありました。それが Navigator2.0 です。

ですが Navigator2.0 も理解が難しく、ついていけた Flutter エンジにはあまりいませんでした。そこでもっと使いやすくパッケージ化された go_router が誕生しました。

またの機会に go_router の紹介もしたいと思っています。お楽しみに！

# コピペで動くサンプルコード

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
      home: SamplePage(),
    );
  }
}

class SamplePage extends StatefulWidget {
  const SamplePage({Key? key}) : super(key: key);

  @override
  State<SamplePage> createState() => _SamplePageState();
}

class _SamplePageState extends State<SamplePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            showDialog(
              context: context,
              builder: (context) {
                return const CustomDialog();
              },
            );
          },
          child: const Text('ダイアログ'),
        ),
      ),
    );
  }
}

class CustomDialog extends StatelessWidget {
  const CustomDialog({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      contentPadding: EdgeInsets.zero,
      insetPadding: const EdgeInsets.all(32),
      content: Navigator(
        onGenerateRoute: (_) {
          return MaterialPageRoute(
            builder: ((context) => SizedBox(
                  width: 400,
                  child: Scaffold(
                    appBar: AppBar(
                      centerTitle: true,
                      title: const Text('1'),
                    ),
                    body: Center(
                      child: ElevatedButton(
                        onPressed: () {
                          Navigator.of(context).push(
                            MaterialPageRoute(
                              builder: (context) {
                                return const NextDialog();
                              },
                            ),
                          );
                        },
                        child: const Text('2のページに進む'),
                      ),
                    ),
                  ),
                )),
          );
        },
      ),
    );
  }
}

class NextDialog extends StatelessWidget {
  const NextDialog({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 400,
      child: Scaffold(
        appBar: AppBar(
          centerTitle: true,
          title: const Text('2'),
        ),
        body: Center(
          child: ElevatedButton(
            onPressed: Navigator.of(context).pop,
            child: const Text('1のページに戻る'),
          ),
        ),
      ),
    );
  }
}
```
