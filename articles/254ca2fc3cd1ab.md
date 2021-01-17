---
title: "GitHub Actions で Flutter アプリを iOS 向けにデプロイする"
emoji: "🍎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, ios, vscode, xcode, githubactions]
published: true
---

# はじめに

Flutter build iOS して Xcode で archive してそれをまた App Store Connect にアップロードして...という作業は面倒くさい。GitHub Actions を使えば push などをトリガーにこのやっかいな作業を自動化してくれるというじゃあないか。しかも、パブリックリポジトリなら無料で使わせていただけるという。これはもう、やるしかない。

すでに周辺知識がある人に向けて main.yml をはじめに載せておく。
いろいろ設定はできるが極力シンプルな形にまとめた。

```yml:main.yml
name: CI

on:
main branch
  push:
    branches: [ main ]

jobs:
  #
  # Build for iOS
  #
  build_iOS: # jobに名前をつけられる

    name: Build for iOS # GitHubに表示されるジョブの名前
    runs-on: macos-latest # ジョブを実行するマシーンの種類

    steps: #一連のタスク

      # チェックアウト
      - name: Checks-out my repository # step に名前をつけられる なくてもいい
        uses: actions/checkout@v2

      # Flutter のインストール
      - name: Install Flutter
        run: git clone https://github.com/flutter/flutter.git # ターミナルでこれが実行されるイメージ

      # PATH を通す
      - name: Add path
        run: echo "$(pwd)/flutter/bin" >> $GITHUB_PATH # GITHUB_PATH にどんどんぶちこめばいいらしい

      # パッケージのダウンロード
      - name: Download Flutter packages
        run: flutter pub get

      # 証明書の生成
      - name: Import Provisioning Profile
        run: | # 複数行の run を書きたい場合はこうする 以下、Provisioning Profilesを置くべきディレクトリにデコードしている
          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          touch ~/Library/MobileDevice/Provisioning\ Profiles/distribution.mobileprovision
          echo -n '${{ secrets.PROVISIONING_PROFILE }}' | base64 -d -o ~/Library/MobileDevice/Provisioning\ Profiles/sadistributionmple_dist.mobileprovision

      # 署名をする
      - name: Import Code-Signing Certificates
        uses: Apple-Actions/import-codesign-certs@v1 # 外部パッケージを使っている
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

ここからは、あまり詳しくない人に向けてできるだけ丁寧に説明していきたい。

# まずはプロジェクトを作る

テストでやっていくならビルド時間などを節約したい。
というわけで Flutter で新規プロジェクトを作りそれをそのまま GitHub にプッシュする。
今回作業していくのは以下のリポジトリになる。
https://github.com/kenta-wakasa/github_actions_sample

# 事前に必要なもの

必要なものがいくつかあるのでそれらを準備していこう。

1. Apple ID と App 用パスワード
2. Identifiers
3. Certificates と p12 と パスワード
4. Provisioning profile
5. 新規 App ページ

## AppleID と App 用パスワード

当然まずはディベロッパー登録をしておかなければいけない。
ここは流石に割愛する。

それをしたら次は App 用パスワードを入手する。
以下のページが参考になる。
https://support.apple.com/ja-jp/HT204397

この App 用パスワードは [App Store Connect](https://appstoreconnect.apple.com) にアップロードするときに必要になる。

## Identifiers

バンドル ID などと呼ばれているもの。
https://developer.apple.com/account/resources/identifiers/list
ここから作成する。いろいろ付け方はあるが、ユニークなものであればとりあえずは大丈夫。

## Certificates と p12 と パスワード

このページがわかりやすいと思う。
https://i-app-tec.com/ios/apply-application.html

今回は Apple Distribution を選択した。
![](https://storage.googleapis.com/zenn-user-upload/jswkrskfmsn2783i5nqb3pxao5gy)

作成が終わったらこれを .p12 ファイルとして書き出す。
アプリケーション検索で k と入力するとキーチェーンアクセスが見つかるのでこれを開く。
![](https://storage.googleapis.com/zenn-user-upload/d2i2rhfu2pzku5r64n6bznnqu3nc)
こんな感じで書き出すことができる。
パスワードを要求されるので任意のものを入力して控えておく。

書き出しについてわからない場合はこのページが参考になるかもしれない。
https://mmorley.hatenablog.com/entry/2015/09/07/131745

## Provisioning profile

ここから追加できる。
https://developer.apple.com/account/resources/profiles/add

今回は App Store を選択。
ファイル名は distribution.mobileprovision とした。
![](https://storage.googleapis.com/zenn-user-upload/2lvqfjqvpg224lepaijg0fn1bdqq)

## 新規 App ページ

ここから追加できる。
先ほど作成した Identifiers を選択しよう。
https://appstoreconnect.apple.com/apps

![](https://storage.googleapis.com/zenn-user-upload/927wqg4w3ghd70u4wqb2ythd6wky)

### 準備 OK

長かったけれどこれで下準備は完了した。
次は Xcode での設定。

# Xcode でいろいろ設定する

まずは [ プロジェクトの path ]/ios/Runner.xcworkspace を開く。

この画面で先ほど設定した Identifiers や Provisioning profile を指定する。
![](https://storage.googleapis.com/zenn-user-upload/quiwzpjc41295wesisouds8o6m3z)

最低限これだけやっておけば大丈夫だと思う。

Flutter の公式に詳しく載っている。
https://flutter.dev/docs/deployment/ios

# ExportOptions.plist をつくる

Xcode で一度アーカイブをしてそれを手元にダウンロードするとその中に ExportOptions.plist というものが入っている。 ipa ファイルを作成するときに必要になるので手に入れておく。

自分の場合はこのようになっていた。
コメントしたところを書き換えればそのまま使えるかもしれない。

```xml:ExportOptions.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>destination</key>
	<string>export</string>
	<key>method</key>
	<string>app-store</string>                // <- 出力方式
	<key>provisioningProfiles</key>
	<dict>
		<key>github.actions.sample</key>  // <- バンドルID
		<string>distribution</string>     // <- Provisioning profile
	</dict>
	<key>signingCertificate</key>
	<string>Apple Distribution</string>
	<key>signingStyle</key>
	<string>manual</string>
	<key>stripSwiftSymbols</key>
	<true/>
	<key>teamID</key>
	<string>42XYZEWKER</string>              // <- チームID
	<key>uploadBitcode</key>
	<false/>
	<key>uploadSymbols</key>
	<true/>
</dict>
</plist>
```

このファイルをプロジェクトのルートに配置した。

# VScode に拡張機能をいれる

GitHub Actions を触りやすくなる拡張機能があるのでインストールする。
https://marketplace.visualstudio.com/items?itemName=cschleiden.vscode-github-actions
![](https://storage.googleapis.com/zenn-user-upload/40j9mwxx2dh05dwrllihy8ylm6v4)

# Secrets を入力してく

GitHub Actions では Secrets に非公開情報を保存しておくことができる。
今回は次のものを設定する。

1. APPLE_APP_PASS (App 用 パスワード)
2. APPLE_ID (Apple ID)
3. CERTIFICATES_P12 (p12 ファイル)
4. CERTIFICATE_PASSWORD (p12 ファイルのパスワード)
5. PROVISIONING_PROFILE (Provisioning profile)

さきほどインストールしておいた拡張機能を使うことで VScode からでも簡単に設定できる。
![](https://storage.googleapis.com/zenn-user-upload/wh3j336qh90ym1j2um9t72byxo73)

Repository Secrets の横の + アイコンをクリックすればどんどん追加できる。
けれどここでひとつ問題がある。
文字列はそのまま入力できるが CERTIFICATES_P12 と PROVISIONING_PROFILE はどうしようか。

これは Base64 に変換することで解決できる。

ターミナルをひらいて

```shell:command
% base64 [変換したいファイル名] | tr -d  "\n" | pbcopy
```

と入力すればクリップボードに保存される。
あとはそれを cmd + v でペーストしてあげればよい。

ちなみに tr -d "\n" は改行コードを削除している。

# ようやく Actions のワークフローを書いていく

編集していくファイルは [root]/.github/workflows/main.yml で、ここに命令を書き込んでいけばリモートにある mac 環境でそれを実行してくれる。
このファイルそのものは GitHub の Actions タブなどからも作成できる。
![](https://storage.googleapis.com/zenn-user-upload/225vin5m0g9ieg13zole8rirn69x)

ここまでくればおそらくコピペでも動くはず。

1. distribution.mobileprovision
2. ./build/ios/ipa/github_actios_sample.ipa

この二つのファイル名はそれぞれ自分が名付けたものに変更してほしい。

これらもハードコードする必要はなく、すでにあるリポジトリの情報から引っ張ってくることも可能なので、次の学習のステップとして試してみるといいかもしれない。

できるだけシンプルに書いたのであまり解説するところもない。
わからないコマンドはそのコマンド名で調べると出てくると思う。

そんなわけで、ふたつに絞って補足の説明をする。

```
flutter build ipa
```

これは mono さんの記事を参考にさせていただたいた。
https://medium.com/flutter-jp/ipa-e176de0276c6
ipa ファイルを生成するのは面倒だったが、このコマンドをたたくだけで実現できるようにった。

```
xcrun altool --upload-app
```

アップロードはなんとこれだけでできてしまう。
この記事を参考にさせていただいた。
https://qiita.com/messhi/items/cb8c6f2b4b6540995189

# おまけ

アップロードに成功するとテストフライトなどでそのままダウンロードしてこれる。
しかし、輸出コンプライアンスがありません などと言われて App に暗号化が使用されていないことを回答しなければ TestFlight などに落としてくることができない。

info.plist に

```xml:info.plist
  <key>ITSAppUsesNonExemptEncryption</key>
  <false/>
```

を追加すれば OK らしい。
これで、何の設定もすることなくアプリを落としてくることができる。

この記事を参考にさせていただいた。
https://tommy10344.hatenablog.com/entry/2020/04/29/025809

# おしまいに

わからないこと、間違っていることなどあれば気軽にコメントいただけるとうれしいです。
次は Android 向けにも書いていこうとお思います。

# 参考記事

https://www.mum-meblog.com/entry/research-detail/flutter-ios-ci-cd
