---
title: シングルトンを作りたい
---

# A. コンストラクタをプライベートにしよう

シングルトン（singleton）とはプログラム中にそのクラスのインスタンスがただひとつだけ存在させるような設計パターンです。言語によって様々な書き方がありますが dart では驚くほど簡単にシングルトンを作れるので紹介します。

## Counter クラスをシングルトンにしてみる

数字があって、カウントアップしていく Counter クラスを作ってみましょう。
シングルトンではない場合このような感じになります。

```dart
/// 内部的なvalueという状態を1ずつ増やすだけのCounterクラス
class Counter {
  var value = 0;
  int increment() {
    return value++;
  }
}
```

この状態なら複数のインスタンスを作ることができますね。
複数インスタンスを作ってみましょう。

```dart
void main() {
  final counter0 = Counter();
  final counter1 = Counter();
  final counter2 = Counter();

  print(counter0.increment());
  print(counter0.increment());
  print(counter1.increment());
  print(counter2.increment());
}
```

では次はこれをシングルトンにしてみましょう。

```dart
/// 内部的なvalueという状態を1ずつ増やすだけのCounterクラス（Singletonバージョン）
class Counter {
  /// ._() でプライベートコンストラクタが作れる！
  /// これでこのファイル以外ではCounterインスタンスは作れなくなる
  Counter._();

  /// 内部的にただひとつだけのインスタンスを static な変数として持たせる
  /// これでこのプログラムではこのクラスのインスタンスがただひとつであることが保証される
  static final instance = Counter._();

  var value = 0;

  int increment() {
    return value++;
  }
}
```

これで Counter のインスタンスはただ一つだけしか生成できなくなりました。
先ほどの例を書き換えてみて結果を見比べてみてください。

```dart
void main() {
  final counter0 = Counter.instance;
  final counter1 = Counter.instance;
  final counter2 = Counter.instance;

  print(counter0.increment());
  print(counter0.increment());
  print(counter1.increment());
  print(counter2.increment());
}
```

print の結果を見ると理解が深まるかと思います。

## グローバルな変数、関数ではダメなのか

シングルトンでやっていることはグローバルな変数や関数で同じことが実現できます。クラスではないので厳密には同じではないですが、機能として同じような振る舞いは可能です。たとえば次のようにかけます。

```dart
var value = 0;

int increment() {
  return value++;
}
```

ただこれはクラスではないので、変数や関数などのネームスペースは圧迫してしまいます。別ファイルに同じ変数名の value がいたら面倒ですよね。またクラスにすることで明確な意味づけができるので、その点もシングルトンの利点だと思います。

## コピペで動くサンプルコード

```dart
/// 内部的なvalueという状態を1ずつ増やすだけのCounterクラス（Singletonバージョン）
class Counter {
  /// ._() でプライベートコンストラクタが作れる！
  /// これでこのファイル以外ではCounterインスタンスは作れなくなる
  Counter._();

  /// 内部的にただひとつだけのインスタンスを static な変数として持たせる
  /// これでこのプログラムではこのクラスのインスタンスがただひとつであることが保証される
  static final instance = Counter._();

  var value = 0;

  int increment() {
    return value++;
  }
}

void main() {
  final counter0 = Counter.instance;
  final counter1 = Counter.instance;
  final counter2 = Counter.instance;

  print(counter0.increment());
  print(counter0.increment());
  print(counter1.increment());
  print(counter2.increment());
}
```
