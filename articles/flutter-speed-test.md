---
title: "Flutterでインターネットの通信速度を測定する"
emoji: "🏃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart]
published: true
---

![](/images/speed_test.gif)

こんにちは、こんぶです。いつも見ていただいてありがとうございます。

今回はインターネット通信速度の測定を Flutter でやってみようという記事になります。

全国の WiFi 速度を計測して共有するアプリを作りたいと思っていので、その調査のために行いました。

# パッケージの選定

[pub.dev](https://pub.dev) で `speed test` と検索してみると `speed_test_dart` というパッケージが見つかりましたのでこちらを使用します。

# インストール

flutter にパッケージを追加するときはターミナルで `flutter pub add` コマンドを実行します。

```sh:パッケージのインストール
flutter pub add speed_test_dart
```

# 使い方

## サンプルコードの紹介

最初にまるっと全体像をお見せしましょう。

```dart: 通信速度測定のサンプルコード
import 'package:flutter/material.dart';
import 'package:speed_test_dart/classes/server.dart';
import 'package:speed_test_dart/speed_test_dart.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: HomePage());
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final speedTeatDart = SpeedTestDart();
  List<Server> bestServersList = [];
  double? speedMBs;
  bool isLoading = false;

  /// 測定にベストなサーバーを設定する
  Future<void> setBestServers() async {
    setState(() {
      isLoading = true;
    });

    final settings = await speedTeatDart.getSettings();

    // Russiaのサーバーからははじかれるためリストから取り除く
    final servers = settings.servers
        .where((element) => element.country != 'Russia')
        .toList();
    bestServersList = await speedTeatDart.getBestServers(servers: servers);
    setState(() {
      isLoading = false;
    });
  }

  /// ダウンロードスピードを測定する
  Future<void> testDownloadSpeed() async {
    setState(() {
      isLoading = true;
    });
    if (bestServersList.isEmpty) {
      throw Exception('あらかじめ bestServersList を取得する必要があります');
    }

    /// MB/s で結果が得られる
    speedMBs = await speedTeatDart.testDownloadSpeed(servers: bestServersList);

    setState(() {
      isLoading = false;
    });
  }

  @override
  void initState() {
    super.initState();
    setBestServers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Builder(
          builder: (context) {
            final speedMBs = this.speedMBs;

            if (isLoading) {
              return const CircularProgressIndicator();
            }

            if (speedMBs != null) {
              /// Mbps の単位に変換する
              return Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text.rich(
                    TextSpan(
                      children: [
                        TextSpan(
                          text: '${(speedMBs * 8).floor()}',
                          style: const TextStyle(
                            fontSize: 56,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const TextSpan(
                          text: 'Mbps',
                        ),
                      ],
                    ),
                  ),
                  ElevatedButton(
                    onPressed: () {
                      testDownloadSpeed();
                    },
                    child: const Text('再測定'),
                  ),
                ],
              );
            }

            return ElevatedButton(
              onPressed: () async {
                testDownloadSpeed();
              },
              child: const Text('測定'),
            );
          },
        ),
      ),
    );
  }
}
```

## 解説

次の手順で使用できます。

1. [設定情報を取得](#設定情報を取得)
2. [測定のためにベストなサーバーを特定](#測定のためにベストなサーバーを特定)
3. [通信速度の測定](#通信速度の測定)

ではひとつずつ見ていきましょう。

### 設定情報を取得

まずは設定情報を取得してきます。

```dart:設定情報を取得する
/// インスタンス生成
final speedTeatDart = SpeedTestDart();

/// 設定情報を取得する
final settings = speedTeatDart.getSettings();
```

この `getSettings` の中身をみると `https://www.speedtest.net` からなんらかの設定データを引っ張ってくる実装になっていました。

### 測定のためにベストなサーバーを特定

設定情報を取得すると測定に使用するサーバーがいくつか得られるのですが、そこから今回測定に使用するベストなサーバーを決定します。

```dart:測定にベストなサーバーを決定する
final  bestServersList = await speedTeatDart.getBestServers(servers: settings.servers);
```

`getBestServers` が何をしているかというとレイテンシが 500ms より小さいものを選別し、短い順にソートをかけています。

### 通信速度の測定

ここでようやく通信速度を測定します。測定のために使用するサーバーを引数で渡すインターフェースになっていました。

```dart:通信速度の測定
final speedMBs = await speedTeatDart.testDownloadSpeed(servers: bestServersList);
```

`testDownloadSpeed` の中身を見に行くと与えたサーバーリストの頭から順番に実行して、失敗したら次のサーバーを実行してというふうに、成功するまで順番に実行するようです。

単位は `MB/s` として最終的な結果が得られます。
わたしたちが普段よく目にする `Mbps` とは次のような関係性になっています。

`1 Mbps = 0.125 MB/s`

つまり、得られた結果を 8 倍してあげると `Mbps` としての速度になります。

以上！

# 余談 internet_speed_test もある

ほかにも `internet_speed_test` というパッケージも人気がありましたが、こちらはサンプルコードがうまく動かなかったため、今回は採用しませんでした。

このパッケージは測定中の速度情報も見られるみたいなので、その点では`speed_test_dart`より優れています。エラーがなくなるように修正して使うのもありですね。
