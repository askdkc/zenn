---
title: "Laravel開発ならValet一択"
emoji: "😏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','valet']
published: true
---
# Laravel開発ならValet一択

「いや〜、Laravelでアプリを開発する方法は色々ありますが、やっぱりMac使ってて一番興奮する開発方法はValetですね！」

『間違いないね！』

## Macじゃない人

ご安心ください☺️
[こちらで解決可能](https://apple.com/jp)です☺️

## セットアップ

### brew入れます

[brewのオフィシャルページ](https://brew.sh/)に書かれている方法で初期セットアップしましょう

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 必要なパッケージ入れて行きます

```bash
brew update
brew install php
brew install composer
```

#### 環境変数整えます

```bash
vi ~/.zshrc
```

末尾に下記を入れておきます

```vim
export PATH=$PATH:$HOME/.composer/vendor/bin
```

### Laravel Installerのインストール

```bash
composer global require laravel/installer
```

### Laravel Valetのインストール

```bash
composer global require laravel/valet
```


### Laravel Valetの初期設定

Macは自分のホームディレクトリに`Sites`というディレクトリを作ると便利です（昔のMacはWEBサーバが入ってて、ここに公開用HTMLを入れておりました）

```bash
mkdir -p ~/Sites
cd ~/Sites
valet install
valet park
```

### Laravelの動作確認

```bash
mkdir ~/Sites
laravel new mynewlaravel
```

ブラウザを開いて

http://mynewlaravel.test

にアクセスすると、、、<br>
はい！もうLaravel動いてます！

### 面倒なステップを1ステップに

最新のLaraconで発表されたLaravel Herdを使いましょう

https://herd.laravel.com/
