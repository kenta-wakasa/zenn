---
title: "音声波形をみたい"
---

# A. マイクから取ってきた波形を CustomPaint で表示しよう

![](/images/q10/1.gif)

音声波形を使えば、音の高さを特定したり、文字列に変換したりと様々なことができます。ただ眺めているだけでも楽しいです。そこで本日は音声波形を画面に表示するまでをやってみたいと思います。

## audio_streamer をつかう

音声を取得できるパッケージはたくさんあるかと思いますが、今回は[audio_streamer](https://pub.dev/packages/audio_streamer)を使用します。とてもシンプルに音声を取得できます。

音声収録のスタートとストップだけなら次のようなコードで実装可能です。
onAudio 関数で実際のデータを受け取っているので、何か処理をしたい場合は buffer に対して行うことになります。

```dart
  final AudioStreamer audioStreamer = AudioStreamer();

  // buffer に次々と音声データが入ってくる
  void onAudio(List<double> buffer) {
    setState(() {
      this.buffer = buffer;
    });
  }

  void handleError(PlatformException error) {
    setState(() {
      _isRecording = false;
    });
    log(error.message.toString());
    log(error.details);
  }

  void start() async {
    audioStreamer.start(onAudio, handleError);
    setState(() {
      _isRecording = true;
    });
  }

  void stop() async {
    bool stopped = await audioStreamer.stop();
    setState(() {
      _isRecording = stopped;
    });
  }
```

ここで得られた buffer は List<double>の形なので、それを折れ線グラフの形にマッピングしていきます。マッピングのために CustomPaint を使用します。グラフ作成のためのパッケージを使用しても良いですが、CustomPaint は描画処理が軽いのでリアルタイムの音声を扱うのに適しています。

List<double> のデータを受け取って波形グラフを描く WavePainter です。
コードの解説はコメントをご覧ください。

```dart
class WavePainter extends CustomPainter {
  WavePainter({
    required this.samples,
    required this.color,
    required this.constraints,
  });

  BoxConstraints constraints;
  List<double> samples;
  Color color;

  final _absMax = 1;
  static const _hightOffset = 0.5;

  @override
  void paint(Canvas canvas, Size size) {
    // 色、太さ、塗り潰しの有無などを指定
    final paint = Paint()
      ..color = color
      ..strokeWidth = 1.0
      ..style = PaintingStyle.stroke;

    // 得られたデータをオフセットのリストに変換する
    // やっていることは決められた範囲で等間隔に点を並べているだけ
    final points = toPoints(samples);

    // addPolygon で path をつくり drawPath でグラフを表現する
    final path = Path()..addPolygon(points, false);
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(oldPainting) => true;

  // 得られたデータを等間隔に並べていく
  List<Offset> toPoints(List<double> samples) {
    final points = <Offset>[];
    for (var i = 0; i < (samples.length / 2); i++) {
      points.add(
        Offset(
          i / (samples.length / 2) * constraints.maxWidth,
          project(samples[i], _absMax, constraints.maxHeight),
        ),
      );
    }
    return points;
  }

  double project(double value, int max, double height) {
    final waveHeight = (value / max) * height;
    return waveHeight + _hightOffset * height;
  }
}
```

あとはこれに取得した音声データを与えるだけです。
最後にコピペで動く全体のコードを示して終わりにします。

## コピペで動くサンプルコード

```dart
import 'dart:developer';

import 'package:audio_streamer/audio_streamer.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final AudioStreamer audioStreamer = AudioStreamer();
  bool _isRecording = false;
  List<double> buffer = [];

  @override
  void initState() {
    super.initState();
  }

  void onAudio(List<double> buffer) {
    setState(() {
      this.buffer = buffer;
    });
  }

  void handleError(PlatformException error) {
    setState(() {
      _isRecording = false;
    });
    log(error.message.toString());
    log(error.details);
  }

  void start() async {
    audioStreamer.start(onAudio, handleError);
    setState(() {
      _isRecording = true;
    });
  }

  void stop() async {
    bool stopped = await audioStreamer.stop();
    setState(() {
      _isRecording = stopped;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: LayoutBuilder(
          builder: (BuildContext context, BoxConstraints constraints) {
            return CustomPaint(
              painter: WavePainter(
                samples: buffer,
                color: Colors.blue,
                constraints: constraints,
              ),
            );
          },
        ),
        floatingActionButton: FloatingActionButton(
          backgroundColor: _isRecording ? Colors.red : Colors.green,
          onPressed: _isRecording ? stop : start,
          child: _isRecording ? const Icon(Icons.stop) : const Icon(Icons.mic),
        ),
      ),
    );
  }
}

class WavePainter extends CustomPainter {
  WavePainter({
    required this.samples,
    required this.color,
    required this.constraints,
  });

  BoxConstraints constraints;
  List<double> samples;
  Color color;

  final _absMax = 1;
  static const _hightOffset = 0.5;

  @override
  void paint(Canvas canvas, Size size) {
    // 色、太さ、塗り潰しの有無などを指定
    final paint = Paint()
      ..color = color
      ..strokeWidth = 1.0
      ..style = PaintingStyle.stroke;

    // 得られたデータをオフセットのリストに変換する
    // やっていることは決められた範囲で等間隔に点を並べているだけ
    final points = toPoints(samples);

    // addPolygon で path をつくり drawPath でグラフを表現する
    final path = Path()..addPolygon(points, false);
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(oldPainting) => true;

  // 得られたデータを等間隔に並べていく
  List<Offset> toPoints(List<double> samples) {
    final points = <Offset>[];
    for (var i = 0; i < (samples.length / 2); i++) {
      points.add(
        Offset(
          i / (samples.length / 2) * constraints.maxWidth,
          project(samples[i], _absMax, constraints.maxHeight),
        ),
      );
    }
    return points;
  }

  double project(double value, int max, double height) {
    final waveHeight = (value / max) * height;
    return waveHeight + _hightOffset * height;
  }
}
```
