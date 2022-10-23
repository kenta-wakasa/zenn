---
title: "Marvel Exampleを読んでRemiさんの気持ちに近づきたい"
emoji: "📝"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [riverpod, flutter, marvel]
published: false
---

Riverpod の公式サンプルのひとつ、Marvel をご存知でしょうか。公式ドキュメントでも紹介されているこのサンプルを読むことで、作者である Remi さん好みの使い方を習得できるのではないかと思いました。

そこで今回は公式サンプルを fork してそこに日本語コメントをメモがわりに残しつつ、なるへそと感じた部分をピックアップしてこの記事にまとめてみたいと思います。

こちらが fork したリポジトリのリンクになりますので適宜ご覧ください。

https://github.com/kenta-wakasa/riverpod/tree/master/examples/marvel

# サンプルプロジェクトをビルドして体感する

まずはコードを動かしましょう。コードを読む前にそのコードを動かすことで、全体像が掴みやすくなります。必要な手順は次の通りです。

- https://developer.marvel.com で APIKey を Get
- assets/configurations.json に取得した key を記述
- `flutter create .` で Android 向けのビルド環境を構築
- `flutter pub get` でパッケージを取得
- `flutter pub run build_runner build` で freezed 関連ファイルを生成
- `flutter run` でビルド

README に書かれている通りですね。特に問題なくビルドできました。

![](/images/marvel/build.gif)

# 使われている provider を調べる

Riverpod のサンプルコードなので

- selectedCharacterId
- character
- repository
- characterPages
- charactersCount
- characterAtIndex
- \_characterIndex
