---
title: "はじまった"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions ]
published: false
---
# はじめに
[この記事](https://zenn.dev/pressedkonbu/articles/254ca2fc3cd1ab)の Android 版を書いていきたいと思う。 iOS より手順は少ない気がするけれど、Gradle と仲良くないのでそのあたりはコピペでなんとか動いたという感じになっている。このあたりの知見をお持ちの方がいたら補足してほしい。

さっそく main.yml を載せておく。[前回](https://zenn.dev/pressedkonbu/articles/254ca2fc3cd1ab)と同様[このリポジトリ](https://github.com/kenta-wakasa/github_actions_sample/actions)で作業をした。

```yml:main.yml
name: CI

on:
  # main branch に push があったらこの workflow がはしる
  push:
    branches: [ main ]

jobs:
  # ----------------------------------------------------------------- #
  # Build for Android
  # ----------------------------------------------------------------- #
  build_Android: # job の名前  [job = step のまとまり]

    runs-on: ubuntu-latest # ubuntu 環境でやっていく

    steps:
      # チェックアウト
      - name: Checks-out my repository 
        uses: actions/checkout@v2

      # Flutter のインストール clone がはやい
      - name: Install Flutter
        run: git clone https://github.com/flutter/flutter.git

      # PATH を通す
      - name: Add path
        run: echo "$(pwd)/flutter/bin" >> $GITHUB_PATH

      # パッケージのダウンロード
      - name: Download Flutter packages
        run: flutter pub get

      # Secrets からrelease.jks を生成
      - name: Create release.jks
        run: echo -n ${{ secrets.ANDROID_KEY_JKS }} | base64 -d > android/release.jks # -n で改行を消している

      # Secrets から service-account-ke.json を生成
      - name: Create release.jks
        run: echo -n ${{ secrets.SERVICE_ACCOUNT_KEY_JSON }} | base64 -d > android/service-account-ke.json

      # Secrets から key.properties を生成
      # key.properties には各種シークレットな文字列なり file name なりを渡している
      - name: Create key.properties
        run: |
          echo 'storeFile=release.jks' > android/key.properties
          echo 'serviceAccountFile=service-account-ke.json' >> android/key.properties
          echo 'storePassword=${{ secrets.ANDROID_STORE_PASSWORD }}' >> android/key.properties
          echo 'keyPassword=${{ secrets.ANDROID_KEY_PASSWORD }}' >> android/key.properties
          echo 'keyAlias=${{ secrets.ANDROID_KEY_ALIAS }}' >> android/key.properties

      # App Bundle を生成
      - name: Building Android AppBundle
        run: flutter build appbundle --build-number ${GITHUB_RUN_NUMBER} # build-number には run_number を渡している
        # ToDo: version-number や build-number は外部ファイルを参照するようにしたい

      # gradle-play-publisher で アップロード
      # https://github.com/Triple-T/gradle-play-publisher この外部パッケージを活用している
      - name: Upload to GooglePlayStore
        run: ./gradlew publishReleaseBundle
        working-directory: ./android
```