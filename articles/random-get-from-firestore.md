---
title: "仙人「弟子よ。Firestoreからランダムなドキュメントを取得することはできるかの？」"
emoji: "🧔🏼‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [firebase, flutter, firestore]
published: true
---

# 登場人物

仙人 🧔🏼‍♀️: Firebase に詳しい
弟子 🧑‍💻: Firebase は不得意

# ある日のこと

🧔🏼‍♀️「弟子よ。Firestore からランダムなドキュメントを取得することはできるかの？」

🧑‍💻「それくらいのメソッドは公式が用意してくれてそうですよね。ちょっと調べてみます。....あれ、ない。ないのか。」

🧔🏼‍♀️「ないんじゃ」

🧑‍💻「とりあえずドキュメントを全て取得して、その中からランダムでひとつ取り出せばできそうですよね」

🧔🏼‍♀️「確かに。じゃが Firestore はドキュメントの読み取りにコストがかかる。ドキュメントの総数が 10 万件だとするとランダムに 1 件取得するために、10 万回の読み取りコストがかかってしまうんじゃ」

🧑‍💻「10 万回の読み取りのコストは 0.038 ドルだから日本円にして 6 円くらいかかってしまうのか」

🧔🏼‍♀️「1 件のランダムなドキュメント取得をもっと少ないコストで実装してみてほしい」

🧑‍💻「んー 1 回でランダムな値を取得するなら、まずクライアント側でランダムな数字を生成して、その数字をもとに取得してくればよさそうな気がするな」

🧔🏼‍♀️「ふむふむ。ランダムな数字が取得できたとして、それでどうやってドキュメントを取得するのじゃ？」

🧑‍💻「たとえばすべてのドキュメントに 0 から N までの数字が振られていれば、0 から N までのランダムな数字を生成した後、where クエリを使ってそのランダムな数と一致するドキュメントを取得できそうです」

🧔🏼‍♀️「よいぞ。じゃがそのためにはドキュメントを追加するとき、必ず 1 ずつ増加する数字を合わせて保存する必要があるな」

🧑‍💻「現在登録されているドキュメントの数がわかればそれをシリアルナンバーとして利用できそうですね」

🧔🏼‍♀️「よし、ではコードを書いてみよう」

🧑‍💻「ぼくは Flutter が好きなので dart でやってみます」

🧔🏼‍♀️「よろしい」

# Firestore にデータを追加する

book コレクションに本データを保存していくことを想定して考えてみよう。

何冊の本がすでに保存されているかは `setting` という別コレクションの中に保存しておくとしよう。

本を追加するときにはまず book コレクションに何件のドキュメントがあるか調べて、その数をもとにシリアルナンバーを決めて保存してあげる。

その後に `setting` コレクションに保存してあるドキュメントの件数も更新してあげればよさそうだな。

```dart
  /// Firestoreに本を保存する
  Future<void> addBook(String name) async {
    final db = FirebaseFirestore.instance;
    final settingReference = db.collection('setting').doc('book');
    final newBookReference = db.collection('book').doc();
    /// 設定データから現在保存されている本の冊数を取得する
    final documentSnapshot = await settingReference.get();

    /// bookが一件も登録されていないときはnullのはずなのでその場合は0とする
    final bookCount = documentSnapshot.data()?['bookCount'] as int? ?? 0;

    /// bookを追加する
    newBookReference.set({
      'name': name,
      // シリアルナンバーは0はじまりとなるので、登録されているドキュメントの数をそのままシリアルナンバーとしてよい
      'serialNumber': bookCount,
    });

    /// bookCountを更新する
    await settingReference.set({
      'bookCount': bookCount + 1,
    });
  }
```

# Firestore からランダムなドキュメントを取得する

まずは現在登録されている本の冊数を調べて、その冊数-1 が最大値となるようなシリアルナンバーをランダムに生成する。

最後に where クエリを使って、生成されたシリアルナンバーと一致するドキュメント取得すれば完成だ。

こんな感じかな。

```dart
  /// ランダムな本をひとつ取得する
  Future<void> randomlyFetchBook() async {
    final db = FirebaseFirestore.instance;
    final settingReference = db.collection('setting').doc('book');
    final bookReference = db.collection('book');

    /// 設定データから登録されている本の冊数を取得する
    final documentSnapshot = await settingReference.get();
    final boolCount = documentSnapshot.data()?['bookCount'] as int? ?? 0;

    /// bookCountが0であれば登録されているデータはないので `Not Found` を返す
    if (boolCount == 0) {
      print('Not Found');
      return;
    }

    /// (boolCount-1)を最大値として、ランダムな数字を生成する
    final randomNumber = Random().nextInt(boolCount);

    /// whereクエリを使って生成されたserialNumberと一致するドキュメントを取得する
    final document = (await bookReference
            .where('serialNumber', isEqualTo: randomNumber)
            .get())
        .docs
        .first;

    print(document['name']);
  }
```

# 仙人によるレビュー

🧑‍💻「コード書けました！ レビューお願いします」

🧔🏼‍♀️「ふむ、よく書けておるの。ただ今のコードだと正しくシリアルナンバーがインクメントされないかもしれないのお」

🧑‍💻「どうしてでしょう？」

🧔🏼‍♀️「たとえば、こんなふうに実行するとどうなるかの？」

```dart
Future.wait([
  addBook('Book A'),
  addBook('Book B'),
]);
```

🧑‍💻「Book A と Book B のシリアルナンバーが同じ数字になってしまいました。同時に実行されると同じシリアルナンバーが複数存在してしまうんですね」

🧔🏼‍♀️「そうなのじゃ。データを追加するのが 1 人であればいいのじゃが、サービスの規模が大きくなるほど同時に書き込み処理が行われる確率もあがってしまう」

🧑‍💻「いったいどうすれば...」

🧔🏼‍♀️「こういうときはトランザクションを使うとよい」

🧑‍💻「トランザクション？」

🧔🏼‍♀️「トランザクションは同時にデータの編集が起きてしまった場合にもう一度実行し直してくれるものじゃ」

# トランザクションを使った改善版

```dart
  /// Firestoreに本を保存する
  Future<void> addBook(String name) async {
    final db = FirebaseFirestore.instance;
    final settingReference = db.collection('setting').doc('book');
    final newBookReference = db.collection('book').doc();

    db.runTransaction((transaction) async {
      /// 設定データから現在保存されている本の冊数を取得する
      final documentSnapshot = await transaction.get(settingReference);

      /// bookが一件も登録されていないときはnullのはずなのでその場合は0とする
      final bookCount = documentSnapshot.data()?['bookCount'] as int? ?? 0;

      /// bookを追加する
      transaction.set(newBookReference, {
        'name': name,
        // シリアルナンバーは0はじまりとなるので、登録されているドキュメントの数をそのままシリアルナンバーとしてよい
        'serialNumber': bookCount,
      });

      /// bookCountを更新する
      transaction.set(settingReference, {
        'bookCount': bookCount + 1,
      });
    });
  }
```

🧑‍💻「おーすごい。今度は同時に実行しても正しくシリアルナンバーが振られています」

🧔🏼‍♀️「もうひとつ、この手法で気を付けておかなければいけないことがある」

🧑‍💻「なんでしょう？」

🧔🏼‍♀️「ドキュメントが削除された場合どうなる？」

🧑‍💻「それはまずいですね....。bookCount は単純にマイナス 1 してあげればいいですが、シリアルナンバーに欠番が出てしまいます」

🧔🏼‍♀️「そうなのじゃ。だからこの手法では、削除された場合にシリアルナンバーをもう一度降り直す必要がある」

🧑‍💻「めんどくさそう。もっと他にいい方法はないんでしょうか...」

🧔🏼‍♀️「一番簡単な方法は、欠番を欠番のまま置いておくことじゃな」

🧑‍💻「どういうことでしょう？」

🧔🏼‍♀️「もし欠番のデータに運悪くあたってしまったら、もう一度引き直せばいいのじゃよ。そうすればいつかは当たるじゃろ」

🧑‍💻「なるほど、取得時の関数も修正してみます！」

# 欠番が出てしまった場合の対処

```dart
  /// ランダムな本をひとつ取得する
  Future<void> randomlyFetchBook() async {
    final db = FirebaseFirestore.instance;
    final settingReference = db.collection('setting').doc('book');
    final bookReference = db.collection('book');

    /// 設定データから登録されている本の冊数を取得する
    final documentSnapshot = await settingReference.get();
    final boolCount = documentSnapshot.data()?['bookCount'] as int? ?? 0;

    /// bookCountが0であれば登録されているデータはないので `Not Found` を返す
    if (boolCount == 0) {
      print('Not Found');
      return;
    }

    /// (boolCount-1)を最大値として、ランダムな数字を生成する
    final randomNumber = Random().nextInt(boolCount);

    /// 値が取得できるまで繰り返す
    while (true) {
      /// whereクエリを使って生成されたserialNumberと一致するドキュメントを取得する
      final docs = (await bookReference
              .where('serialNumber', isEqualTo: randomNumber)
              .get())
          .docs;

      /// 値が取得できたら return してループから抜ける
      if (docs.isNotEmpty) {
        print(docs.first['name']);
        return;
      }
    }
  }
```

🧑‍💻「おー欠番があっても値を取得できてる！」

🧔🏼‍♀️「よくやったの。さて次の問題じゃが...」

**To Be Continued...**
