---
title: "完成をイメージし課題を分割する"
---

# 完成イメージ

ゴールが定まらないと安心して走ることはできません。まずは完成イメージを明確にしましょう。

わたしがつくったサンプルを紹介します。

@[codepen](https://codepen.io/kenta-wakasa/pen/YzNWMvV)

Zenn では [CodePen](https://codepen.io) を使うことで、このように Flutter のプログラムを埋め込むことができます。実際に遊ぶことが可能です。

このプログラムのコードは 500 行くらいです。HOLD 機能や NEXT 機能などが未実装なため完全な状態ではないですが、これくらいのコード量で TETRIS を動かすことができます。意外とコンパクトですよね。

# 課題を分割する

これで、完成系のイメージを共有することができました。あとはこれを作るだけです。
といっても、何から手をつければいいか、困ってしまうと思います。

ここで重要となるのが、課題を分割する力です。TETRIS はとても複雑な仕組みに思えますが、簡単な仕組みをうまく組み合わせてつくられています。まずはこの認識を持つことが大切です。

一見、つくることが困難に思えるものを、適切に分割し、解決できそうなレベルの課題に落とし込む。この力がプログラミングをしていく上でとても重要になります。

そこでさっそくその練習をしてみましょう。TETRIS で行われている処理をできるだけ分割して列挙してみてください。どの粒度で分割するかなどは特に決めません。思いつくままに書き出してみましょう。

:::message
**WORK : TETRIS をできるだけ細かく分割してみよう**
たとえば... ブロックを画面に表示する。ボタン押すと動き始める。etc...
:::

# 分割した課題を一つひとつクリアする

さて書き出せましたでしょうか。手を動かすことが大切ですので、ぜひやってみてください。

わたしは次のように分割しました。

:::details 分割した機能一覧

- テトリスを画面に表示する

- テトリスを生成する

- テトリスを落下させる

- テトリスを動かす

- テトリスを回転させる

- テトリスの衝突を判定する

- テトリスを固める

- テトリスを消す

- ゲームスタート

- ゲームオーバー

- 消した列をカウントする

:::

ここで分割した項目はさらに細かく分割していくことができます。

次の章からは実際にこの課題を一つひとつクリアしていきますので、さらなる分割はそのときにおこなっていきます。

前置きは以上です。それではさっそくコードを書いていきましょう！