---
title: "GoogleDriveに保存した動画を再生したい"
---

# A. video_player パッケージを使いましょう（URL に注意）

アプリ内で動画を再生したいとき、どのストレージを使うか悩ましいですよね。
FirebaseStorage でもいいし、DropBox でもいいし、Vimeo にアップロードしてもいい。
いろいろな選択肢がある中 GoogleDrive は 15GB まで無料で使えて便利かもしれません。

そこで今回は GoogleDrive に置いた mp4 形式の動画を再生する方法を解説します。

## 再生までの流れ

1. mp4 ファイルのアップロード
1. 共有リンクの生成
1. 共有リンクを再生可能な URL に変換
1. video_player パッケージにで再生

まず 1, 2 については動画を撮りましたのでそちらをご覧ください。

![](/images/q7/1.gif)

共有権限を付与することを忘れないようにしてください。

得られたリンクがこちらです。
`https://drive.google.com/file/d/1k76gmWXK7YPKKjdrtZpf5HMkJSm9QxT6/view?usp=sharing`

この共有リンクはそのままだと再生できないため、ダイレクトダウンロードリンクに変換する必要があります。

まずは id を抜き出します。
id は 次の部分に入っています。
`https://drive.google.com/file/d/[ここにidがはいっている]/view?usp=sharing`
今回の id は　`1k76gmWXK7YPKKjdrtZpf5HMkJSm9QxT6` です。

ダイレクトダウンロードリンクは次のようになっています。
`https://drive.google.com/uc?export=download&id=xxxxxxxx`
この `xxxxxxxx` に `1k76gmWXK7YPKKjdrtZpf5HMkJSm9QxT6` をあてはめれば OK です。

id をもとにダイレクトダウンロードリンクを生成する関数を作りましょう。

```dart
/// idからGoogleDriveのDirectDownloadリンクを生成する
Uri generateDirectDownloadUrl(String id) {
  final baseUrl = Uri.parse('https://drive.google.com/uc');
  final resultUrl = baseUrl.replace(
    queryParameters: {
      'export': 'download',
      'id': id,
    },
  );
  return resultUrl;
}
```

ここまできたらあとは簡単です。
video_player というパッケージがありますので、それを使って再生します。

初期化はこのようにおこないます。

```dart
Future<void> initialize() async {
  _controller = VideoPlayerController.network(
    generateDirectDownloadUrl(id).toString(),
  );
  await _controller.initialize();
  if (mounted) {
    setState(() {});
  }
}
```

使い方は簡単ですので、サンプルコードをご覧いただければと思います！
それではまた明日。

## コピペで動くサンプルコード

```dart
import 'package:flutter/material.dart';
import 'package:video_player/video_player.dart';

/// idからGoogleDriveのDirectDownloadリンクを生成する
Uri generateDirectDownloadUrl(String id) {
  final baseUrl = Uri.parse('https://drive.google.com/uc');
  final resultUrl = baseUrl.replace(
    queryParameters: {
      'export': 'download',
      'id': id,
    },
  );
  return resultUrl;
}

void main() => runApp(const VideoApp());

class VideoApp extends StatefulWidget {
  const VideoApp({Key? key}) : super(key: key);
  @override
  _VideoAppState createState() => _VideoAppState();
}

class _VideoAppState extends State<VideoApp> {
  late VideoPlayerController _controller;

  /// コピーした元のリンク
  /// https://drive.google.com/file/d/1k76gmWXK7YPKKjdrtZpf5HMkJSm9QxT6/view?usp=sharing
  static const id = '1k76gmWXK7YPKKjdrtZpf5HMkJSm9QxT6';

  Future<void> initialize() async {
    _controller = VideoPlayerController.network(
      generateDirectDownloadUrl(id).toString(),
    );
    await _controller.initialize();
    if (mounted) {
      setState(() {});
    }
  }

  @override
  void initState() {
    super.initState();
    initialize();
  }

  @override
  void dispose() {
    super.dispose();
    _controller.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Video Demo',
      home: Scaffold(
        body: Center(
          child: _controller.value.isInitialized
              ? AspectRatio(
                  aspectRatio: _controller.value.aspectRatio,
                  child: VideoPlayer(_controller),
                )
              : const SizedBox.shrink(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () async {
            if (_controller.value.isPlaying) {
              await _controller.pause();
            } else {
              await _controller.play();
            }
            setState(() {});
          },
          child: Icon(
            _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
          ),
        ),
      ),
    );
  }
}
```

## 参考

https://stackoverflow.com/questions/61691514/how-to-play-videos-from-a-google-drive-link

https://pub.dev/packages/video_player
