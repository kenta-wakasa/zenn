---
title: "Flutter Web を🚀爆速🚀で Firebase Hosting にデプロイする"
emoji: "🚅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, firebase, githubactions, web, vscode]
published: true
---

![](https://storage.googleapis.com/zenn-user-upload/w9ijnzv2u3fotmxjevc3l2d7ku5g)
_この記事でできあがるもの_

# はじめに

自分の作ったものが web に公開されるのはうれしいことだ。
そのうれしさまず体験することで開発モチベが高まると思う。
10 分くらいでできるのでお試しあれ。

# やっていくこと

**Flutter のサンプルアプリを Firebase Hosting を使って web 上に公開する。**

# 開発環境

    マシン: M1 MacBook Air
    エディタ: VScode
    Flutter: 1.26.0-2.0.pre.402 • channel master

# 作業の全体像

1. Flutter アプリを作成

2. GitHub と連携

3. Firebase に新規プロジェクト作成

4. Firebase CLI のインストール

5. GitHub Actions の手直し

6. PUSH & DEPLOY

# 1. Flutter アプリを作成

```shell:command
# <puipui.hugahuga> と <app-name> は好きな文字列をあたえる
flutter create --org <puipui.hugahuga>  <app-name>
```

# 2. GitHub と連携

新規リポジトリの作成を行う。
画像は全角文字が入っているけど、よい子は半角英数で作ろう。

![](https://storage.googleapis.com/zenn-user-upload/cbfx8j5hyaogbf7dtlm1zspu53l1)

あとで GitHub Actions を使うので public で作っておく。

この記事で作業していくのはこのリポジトリ。
https://github.com/kenta-wakasa/material_icons

:::details 名前について
material_icons という名前なのは flutter で使える icon が一覧できるサイトを作ろうと思っているからです。
:::

いつものお決まりを入力してローカルとリモートの連携を済ませておく。

```shell:command
git init # 初期化
git add -A # ファイル全突っ込み
git commit -am "first commit" # コミット
git branch -M main # メインブランチをmainに
git remote add origin https://github.com/<githubのユーザーネーム>/<プロジェクトネーム>.git # リモートリポジトリにあげる
git push origin HEAD # プッシュする
```

# 3. Firebase に新規プロジェクト作成

[ここ](https://firebase.google.com)から使ってみるをクリック。
![](https://storage.googleapis.com/zenn-user-upload/pl7ly4vx10bja1j4cgi8x0wb4z4y)

プロジェクトを追加。
![](https://storage.googleapis.com/zenn-user-upload/wneo4s9624gz3vom2g5uw18kmdxd)

プロジェクト名を入力して続行。
:::message
app_name と同じにしておくと管理がしやすいと思う。
:::
![](https://storage.googleapis.com/zenn-user-upload/jnv2za140ane2h64gcjgemgmky3j)

ここも続行。
![](https://storage.googleapis.com/zenn-user-upload/1vvkkktu7wrgqimnzkm0blcyuh9h)

好きなアカウントを選ぶ
![](https://storage.googleapis.com/zenn-user-upload/lhq4taojtnf0t7wptwev60pfqrya)

**これで Firebase プロジェクトの作成は完了！**

# 4. Firebase CLI のインストール

ローカルの作業ディレクトリにもどってくる。

次のコマンドの実行

```shell:command
curl -sL https://firebase.tools | bash # CLI のインストール

firebase login # ログイン!

firebase projects:list # ちゃんと動いているか確認。 動いていれば先ほどつくったプロジェクトが表示されるはず。
```

[Firebase CLI リファレンス](https://firebase.google.com/docs/cli?hl=ja#mac-linux-npm)

ここからはコマンドライン上でいろいろ聞かれるので答えていく。
自分がやったときの履歴をここに載せるので参考にしてほしい。

```md:Firebase CLI
# まずは初期化
[x] mba:~/development/material_icons% firebase init hosting

     ######## #### ########  ######## ########     ###     ######  ########
     ##        ##  ##     ## ##       ##     ##  ##   ##  ##       ##
     ######    ##  ########  ######   ########  #########  ######  ######
     ##        ##  ##    ##  ##       ##     ## ##     ##       ## ##
     ##       #### ##     ## ######## ########  ##     ##  ######  ########

You're about to initialize a Firebase project in this directory:

  /Users/kenta/development/material_icons


=== Project Setup

First, let's associate this project directory with a Firebase project.
You can create multiple project aliases by running firebase use --add,
but for now we'll just set up a default project.

# すでにプロジェクトはあるのでそれを選ぶ
? Please select an option: Use an existing project
? Select a default Firebase project for this directory: material-icons-4aa2e (material-icons)
i  Using project material-icons-4aa2e (material-icons)

=== Hosting Setup

Your public directory is the folder (relative to your project directory) that
will contain Hosting assets to be uploaded with firebase deploy. If you
have a build process for your assets, use your build's output directory.

# ディレクトリはbuild/web
? What do you want to use as your public directory? build/web

# ちゃうはず
? Configure as a single-page app (rewrite all urls to /index.html)? No

# GitHub での自動デプロイやってくれるん？？ これをやってみよう
? Set up automatic builds and deploys with GitHub? Yes
✔  Wrote build/web/404.html
✔  Wrote build/web/index.html

# ブラウザが勝手に開かれるので承認する
i  Detected a .git folder at /Users/kenta/development/material_icons
i  Authorizing with GitHub to upload your service account to a GitHub repository's secrets store.

Visit this URL on this device to log in:
https://github.com/login/oauth/authorize?client_id=89cf50f02ac6aaed3484&state=752106639&redirect_uri=http%3A%2F%2Flocalhost%3A9005&scope=read%3Auser%20repo%20public_repo

Waiting for authentication...

✔  Success! Logged into GitHub as kenta-wakasa

# 作っておいたリポジトリの名前をにゅうりょく 今回は kenta-wakasa/material_icons
? For which GitHub repository would you like to set up a GitHub workflow? (format: user/repository) kenta-wakasa/material_icons

✔  Created service account github-action-330684705 with Firebase Hosting admin permissions.
✔  Uploaded service account JSON to GitHub as secret FIREBASE_SERVICE_ACCOUNT_MATERIAL_ICONS_4AA2E.
i  You can manage your secrets at https://github.com/kenta-wakasa/material_icons/settings/secrets.

# やってくれるならやってほしいので Yes たぶん勝手ににワークフローを書いてくれる。
? Set up the workflow to run a build script before every deploy? Yes

# どうせ動かないけどいったん flutter build web と答えておく
? What script should be run before every deploy? flutter build web

✔  Created workflow file /Users/kenta/development/material_icons/.github/workflows/firebase-hosting-pull-request.yml

# PR がマージされたとき deploy するやんな
? Set up automatic deployment to your site's live channel when a PR is merged? Yes

# main です
? What is the name of the GitHub branch associated with your site's live channel? main

✔  Created workflow file /Users/kenta/development/material_icons/.github/workflows/firebase-hosting-merge.yml

i  Action required: Visit this URL to revoke authorization for the Firebase CLI GitHub OAuth App:
https://github.com/settings/connections/applications/89cf50f02ac6aaed3484
i  Action required: Push any new workflow file(s) to your repo

i  Writing configuration info to firebase.json...
i  Writing project information to .firebaserc...

✔  Firebase initialization complete!
```

**これで Firebase Hosting の導入完了！**

:::details うれしい誤算
まさか GitHub Actions の設定までやってくれるとは思っていなかった。
この設定をしておけば push したりしたタイミング自動的にデプロイまでやってくれる。
なんてありがたい世界なんだ。 Google さんありがとう。
:::

# 5. GitHub Actions の手直し

さきほど CLI の流れで

```
? What script should be run before every deploy? flutter build web
```

と入力したけれどこのままでは動かない。

workflow ファイルを少し手直しする必要がある。

次の 2 つのファイルが追加されているはずなのでそれらを修正していく。

```yml:<各々のプロジェクトネーム>/.github/workflows/firebase-hosting-merge.yml
# This file was auto-generated by the Firebase CLI
# https://github.com/firebase/firebase-tools

name: Deploy to Firebase Hosting on merge
"on":
  push:
    branches:
      - main
jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      # -- ここから追記部分！ -- #
      # Flutter のインストール clone がはやい
      - name: Install Flutter
        run: git clone https://github.com/flutter/flutter.git

      # PATH を通す
      - name: Add path
        run: echo "$(pwd)/flutter/bin" >> $GITHUB_PATH

      # パッケージのダウンロード
      - name: Download Flutter packages
        run: flutter pub get

      # -- ここまで追記部分！ -- #
      # 解説:
      #   ようは flutter build web をたたいても flutterがインストールされていないので動かないということ
      #   インストールしてPATH通してパッケージダウンロードしているだけ。

      # ビルド
      - run: flutter build web

      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: "${{ secrets.GITHUB_TOKEN }}"
          firebaseServiceAccount: "${{ secrets.FIREBASE_SERVICE_ACCOUNT_MATERIAL_ICONS_4AA2E }}"
          channelId: live
          projectId: material-icons-4aa2e
        env:
          FIREBASE_CLI_PREVIEWS: hostingchannels
```

```yml:<各々のプロジェクトネーム>/.github/workflows/firebase-hosting-pull-request.yml
# This file was auto-generated by the Firebase CLI
# https://github.com/firebase/firebase-tools

name: Deploy to Firebase Hosting on PR
"on": pull_request
jobs:
  build_and_preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      # -- ここから追記部分！ -- #
      # Flutter のインストール clone がはやい
      - name: Install Flutter
        run: git clone https://github.com/flutter/flutter.git

      # PATH を通す
      - name: Add path
        run: echo "$(pwd)/flutter/bin" >> $GITHUB_PATH

      # パッケージのダウンロード
      - name: Download Flutter packages
        run: flutter pub get

      # -- ここまで追記部分！ -- #
      # 解説:
      #   firebase-hosting-merge.yml のプルリクエストバージョンなのでやることは同じ

      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: "${{ secrets.GITHUB_TOKEN }}"
          firebaseServiceAccount: "${{ secrets.FIREBASE_SERVICE_ACCOUNT_MATERIAL_ICONS_4AA2E }}"
          projectId: material-icons-4aa2e
        env:
          FIREBASE_CLI_PREVIEWS: hostingchannels
```

GiHub Actions についてはこちらの記事も参考にしてほしい。

- [GitHub Actions で Flutter アプリを iOS 向けにデプロイする](https://zenn.dev/pressedkonbu/articles/254ca2fc3cd1ab)
- [GitHub Actions で Flutter アプリを Android 向けにデプロイする](https://zenn.dev/pressedkonbu/articles/github-actions-for-android)

**これで手直しは完了！**

# 6. PUSH & DEPLOY

ここまでくればゴールは目前。
プロジェクトの作業ディレクトリで push するだけ！

```shell:command
git add -A
git commit -am "first deploy"
git push origin HEAD
```

GiHub 上のリモートリポジトリにアクセスしていそいそとデプロイ作業をやってくれているところを眺めよう。

すると最後の方に URL が出てくる。
![](https://storage.googleapis.com/zenn-user-upload/7y51pzvox1i78a98ey5hi81m3wqk)

ここにアクセスしてみる。
https://material-icons-4aa2e.web.app/#/

![](https://storage.googleapis.com/zenn-user-upload/w9ijnzv2u3fotmxjevc3l2d7ku5g)

**うごいた！**

# おしまいに

これから Flutter 勉強してみたいというひとにこそやってみてほしいです！
また次回の記事でお会いしましょう。
