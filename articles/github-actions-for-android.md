---
title: "GitHub Actions で Flutter アプリを Android 向けにデプロイする"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, githubactions, android, vscode, deploy]
published: true
---

# はじめに

[この記事](https://zenn.dev/pressedkonbu/articles/254ca2fc3cd1ab)の Android 版を書いていきたいと思う。 iOS より手順は少ない気がするけれど、Gradle と仲良くないので手こずった。なのでそのあたりはコピペでなんとか動いたという感じになっている。Gradle に関する知見をお持ちの方がいたら補足してほしい。

## この記事でやっていること

1. Flutter アプリを [Android App Bundle](https://developer.android.com/guide/app-bundle?hl=ja) で書き出す
2. gradle-play-publisher を使って Google Play Console にアップロード

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
      # > android/key.properties で 上書き
      # >> android/key.properties で　追加
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

ここからはできるだけ丁寧に解説していく。

# 作業の全体像

1. アプリの署名鍵を生成する
2. Gradle に情報を追加する
3. Google Play の API キーを手に入れる
4. Secrets の設定をする
5. ワークフローを書く
6. push して確認する

# 1. アプリの署名鍵を生成する

アプリを公開するためには署名付きで書き出す必要があるため、まずは鍵を用意する。

鍵はターミナルで次のように作成する。

```shell:command
% keytool -genkey -v -keystore キーストア名 -alias 鍵アルゴリズム名 -keyalg 署名アルゴリズム名 -validity 妥当性日数
```

例えばこんな感じで入力する。

```shell:command
% keytool -genkey -v -keystore release.jks -alias hoge -keyalg RSA -validity 10000
```

続けて次のように尋ねられるので、どんどん入力していく。

```shell:command
キーストアのパスワードを入力してください: パスワード ↵
新規パスワードを再入力してください: パスワード ↵
姓名を入力してください
    [Unknown]: hoge sama ↵
組織単位名を入力してください。
    [Unknown]: personal ↵
組織名を入力してください。
    [Unknown]: personal ↵
都市名または地域名を入力してください。
    [Unknown]: shibuya ↵
都道府県名を入力してください。
    [Unknown]: tokyo ↵
この単位に該当する 2 文字の国コードを入力してください。
    [Unknown]: JP ↵
CN=hoge sama, OU=hoge inc., O=hoge inc., L=shibuya, ST=tokyo, C=JP でよろしいですか。
    [いいえ]: はい ↵
```

すると release.jks というファイルが生成されるので、これを android フォルダの直下に配置する。

:::message
このファイルは公開したくない。
.gitignore に ./android/release.jks を追加しておこう。
:::

# 2. Gradle に情報を追加する

用意した鍵を埋め込んだり、リリース用の設定をするために Gradle ファイルを修正していく。
ついでに gradle-play-publisher を使うための設定も行う。

./android/app/build.gradle を開こう。

:::message
build.gradle は ./andorid/build.gradle もあるので注意。
:::

追記箇所を書いていく。

```groovy:./android/app/build.gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: "com.github.triplet.play" // <- 追記：自動アップロードのためのパッケージ
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

// -- 追記: ここから -------------------------------
// key.propertiesファイルの読み込み このファイルはあとで作る。
def keystorePropertiesFile = rootProject.file("key.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
// -- ここまで ------------------------------------
```

```groovy:./android/app/build.gradle
android {
    // -- 追記: ここから -------------------------------
    signingConfigs {
      // 先ほど仕込んだ keystoreProperties にいろいろ情報がはいっている
      // それを取り出して代入する
      release {
          keyAlias keystoreProperties['keyAlias']
          keyPassword keystoreProperties['keyPassword']
          storeFile rootProject.file(keystoreProperties['storeFile'])
          storePassword keystoreProperties['storePassword']
      }
    }

    buildTypes {
        release {
            //  signingConfig signingConfigs.debug から変更
            signingConfig signingConfigs.release

        }
    }

    // Google Play Console API を使ったアップロード
    play {
       serviceAccountCredentials = rootProject.file (keystoreProperties['serviceAccountFile']) // jsonのパスを設定
       // ここにはいろいろオプションを追加できる
    }
    // -- ここまで ------------------------------------
}
```

次は./android/build.gradle を開いていく。

:::message
今度は app じゃない方。
:::

```groovy:./android/build.gradle
// buildscript に変更を加える
buildscript {

    ext.kotlin_version = '1.3.50'

    repositories {
        google()
        jcenter()
        maven { url "https://plugins.gradle.org/m2/" }  // <- ここを追加するといける。
    }

    dependencies {
        // classpath 'com.android.tools.build:gradle:4.1.0' // <- このバージョンだとうまくいかない
        classpath 'com.android.tools.build:gradle:3.5.4' // <- このバージョンに帰る
        classpath "com.github.triplet.gradle:play-publisher:2.6.0" // <- 自動アップロードのためのパッケージ
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }

}
```

Gradle の変更はこれで完了！

# 3. Google Play の API キーを手に入れる

API キーを入手するにはとりあえず一度アプリを提出しなければならない。
これは面倒だがダミー情報でもいいのでひたすら入力してアプリを提出する。

提出はここから。
https://play.google.com/console/u/0/developers

特にアイコン画像などを用意するのが面倒だと思うので[ダミーファイル](https://drive.google.com/drive/folders/1ucdI1k9Cjb5otmePMin19U6Fw-9yexxg?usp=sharing)を用意した。自由に使ってほしい。

適当に apk ファイルなどをあげるとアプリが登録できる。
まずはそこまでやる。

そこまでやったら、設定 > デベロッパー アカウント > API アクセスの新しいサービスアカウントの作成をクリック。
![](https://storage.googleapis.com/zenn-user-upload/smvuyoh5mysbsnyq6wj47w3sut8z)

すると次のようなポップアップが表示されるので言われた通りに進めていく。
![](https://storage.googleapis.com/zenn-user-upload/4y5k0l2c5fnd78p5ezgw0lso6pyj)

サービスアカウントの作成をクリック。
![](https://storage.googleapis.com/zenn-user-upload/qej0a9z4lz334h5rn3bl13uul179)

必要事項を入力したあと、作成ボタンをクリック。
![](https://storage.googleapis.com/zenn-user-upload/5889b6i7lo9mqcfk8pd66i1r1iu3)

ロールはサービスアカウント ユーザーを選択。
![](https://storage.googleapis.com/zenn-user-upload/h5daxr2t5x3yp9td7ij7r6s5zlxy)

アカウントを作ったら、鍵を作成する。
![](https://storage.googleapis.com/zenn-user-upload/tz665piw0vha7ig8iy19mzvt9ol5)

JSON を選択して作成する。
![](https://storage.googleapis.com/zenn-user-upload/aeh2zba1kthoqqzts8gcajuik2f4)

このときダウンロードされたファイルをわかりやすい名前に変更する。
今回は service-account-ke.json としている。

このファイルも ./android フォルダにいれる。

:::message
service-account-ke.json も release.jks と同様に公開したくない。
.gitignore に ./android/service-account-ke.json を追加しておこう。
:::

ここまできたら Google Play Console に戻ってくる。
言われている通り完了を押すとアカウントが追加されているはず。

追加されたアカウントの閲覧宣言をクリック。

![](https://storage.googleapis.com/zenn-user-upload/1xbhzpjt639tw35aokue32f3zy9p)

アプリの権限のタブで先ほど追加したアプリを選択する。
![](https://storage.googleapis.com/zenn-user-upload/s8ddd6zv0ebn4epzxjwif00z0el2)

これで準備完了！

# 4. Secrets の設定をする

Secrets の設定は前回の [iOS 版の記事](https://zenn.dev/pressedkonbu/articles/254ca2fc3cd1ab) を参考にしてほしい。

今回追加したのは次の値。

- **ANDROID_KEY_ALIAS**
  // 署名のときに設定した -alias hoge の hoge の部分
- **ANDROID_KEY_PASSWORD**
  // 署名のときに設定したパスワード
- **ANDROID_STORE_PASSWORD**
  // 署名のときに設定したパスワード
  // ANDROID_KEY_PASSWORD と同じはず
- **ANDROID_KEY_JKS**
  // release.jks を base64 に変換したもの
- **SERVICE_ACCOUNT_KEY_JSON**
  // service-account-ke.json を base64 に変換したもの

base64 への変換はターミナルで次のコマンドを打つ。

```shell:command
% base64 <変換したいファイル名> | tr -d "\n" | pbcopy
```

たとえば...

```shell:command
% base64 service-account-ke.json | tr -d "\n" | pbcopy
```

といった感じに入力するとクリップボードに追加されるので cmd + v でペーストすればよい。

# 5. ワークフローを書く

ようやくここでワークフローを書いていく。
内容は最初に示した main.yml の通り。
コメントに補足も書いているので適宜参照してほしい。

# 6. push して確認する

実際に push してしっかり動くか確認する。
![](https://storage.googleapis.com/zenn-user-upload/1r9o6hpbh364xkcvz0wz3csjw7oq)

成功時にはこのようになるはずだ。

# おしまいに

GitHub Actions を使った Android 向けの自動デプロイの方法をまとめた。
キーの管理などが面倒だけれど一度やってしまえば楽になるため早めに導入しておきたい。

さらに Flutter なら iOS も Android も同時にリリースできる！　ステキ！

両方やるとこんな感じになる。

```yml:main.yml
name: CI

on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
    paths-ignore:
    # - 'FILE_NAME' # ここに列挙されたファイルがpushされても無視する
    - 'README.md'
    - 'analysis_options.yaml'
    - '.gitignore'
  # pull_request:
  #   branches: [ main ]
  workflow_dispatch:

jobs:
  # ----------------------------------------------------------------- #
  # Build for Android
  # ----------------------------------------------------------------- #
  build_Android:

    runs-on: ubuntu-latest
    timeout-minutes: 20

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
        run: echo -n ${{ secrets.ANDROID_KEY_JKS }} | base64 -d > android/release.jks

      # Secrets から service-account-ke.json を生成
      - name: Create release.jks
        run: echo -n ${{ secrets.SERVICE_ACCOUNT_KEY_JSON }} | base64 -d > android/service-account-ke.json

      # Secrets から key.properties を生成
      - name: Create key.properties
        run: |
          echo 'storeFile=release.jks' > android/key.properties
          echo 'serviceAccountFile=service-account-ke.json' >> android/key.properties
          echo 'storePassword=${{ secrets.ANDROID_STORE_PASSWORD }}' >> android/key.properties
          echo 'keyPassword=${{ secrets.ANDROID_KEY_PASSWORD }}' >> android/key.properties
          echo 'keyAlias=${{ secrets.ANDROID_KEY_ALIAS }}' >> android/key.properties
          cat android/key.properties
      # App Bundle を生成
      - name: Building Android AppBundle
        run: flutter build appbundle --build-number ${GITHUB_RUN_NUMBER}

      # gradle-play-publisher で アップロード 権限を変更して再挑戦 もっかい
      - name: Upload to GooglePlayStore
        run: ./gradlew publishReleaseBundle
        working-directory: ./android

      # # いったんダウンロードしたい場合
      # - name: Download AppBundle
      #   uses: actions/upload-artifact@v2
      #   with:
      #     name: app-release.aab
      #     path: ./build/app/outputs/bundle/release/app-release.aab


  # ----------------------------------------------------------------- #
  # Build for iOS
  # ----------------------------------------------------------------- #
  build_iOS: # jobに名前をつけられる

    name: Build for iOS # GitHubに表示されるジョブの名前
    runs-on: macos-latest # ジョブを実行するマシーンの種類

    steps: #一連のタスク

      # チェックアウト
      - name: Checks-out my repository
        uses: actions/checkout@v2

      # Flutter のインストール clone がはやい
      - name: Install Flutter
        run: git clone https://github.com/flutter/flutter.git

      # PATH を通す
      - name: Add path
        run: echo "$(pwd)/flutter/bin" >> $GITHUB_PATH

      # # マーケットプレイスの actions を使う場合
      # - uses: subosito/flutter-action@v1
      #   with:
      #     channel: 'beta'

      # パッケージのダウンロード
      - name: Download Flutter packages
        run: flutter pub get

      # 証明書の生成
      - name: Import Provisioning Profile
        run: |
          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          touch ~/Library/MobileDevice/Provisioning\ Profiles/distribution.mobileprovision
          echo -n '${{ secrets.PROVISIONING_PROFILE }}' | base64 -d -o ~/Library/MobileDevice/Provisioning\ Profiles/sadistributionmple_dist.mobileprovision

      # 署名をする
      - name: Import Code-Signing Certificates
        uses: Apple-Actions/import-codesign-certs@v1
        with:
          p12-file-base64: ${{ secrets.CERTIFICATES_P12 }}
          p12-password: ${{ secrets.CERTIFICATE_PASSWORD }}

      # ipa ファイルの出力
      - name: Create ipa file
        # GITHUB_RUN_NUMBER をビルドナンバーに指定することで被りがないようにしている。
        run: flutter build ipa --export-options-plist=ExportOptions.plist --build-number ${GITHUB_RUN_NUMBER}

      # AppStoreConnect にアップロードする
      - name: Upload to AppStoreConnect
        run: xcrun altool --upload-app -f "./build/ios/ipa/github_actios_sample.ipa" -u "${{ secrets.APPLE_ID }}" -p "${{ secrets.APPLE_APP_PASS }}"

```

参考になりましたらフォロー&サポートなどしていただけると励みになります！
質問や訂正などもありました気軽によろしくお願いします。

# 参考にさせていただいた記事

https://www.mum-meblog.com/entry/research-detail/flutter-android-ci-cd
https://note.com/shcahill/n/n1c7d72df3c4d
