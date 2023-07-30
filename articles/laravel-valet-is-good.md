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

#### 何が便利って

「ちょっと開発者さん？たこ焼き屋やってるんだけど、冷凍たこ焼きをネットで売りたいんだ」

『それならLaravelでStripe使ってちゃっちゃと作りますか！』

```bash
cd ~/Sites
laravel new takoyakikun
```

ブラウザを開いて

http://takoyakikun.test

にアクセスすると、、、<br>
はい！もうLaravel動いてます！

そのまま

http://mynewlaravel.test

にもアクセス可能です

複数のプロジェクトをなんのコマンドも叩かずに切り替え可能🤯

楽〜😍

### 面倒なステップを1ステップに

最新のLaraconUSで発表されたLaravel Herdを使いましょう

https://herd.laravel.com/

## 冷凍たこ焼きで何かと楽したい人へ

こちらにユーザ認証機能や多言語対応をコマンド数回叩くだけでOKという[記事](https://zenn.dev/circleback/articles/larave-three-commands)があります


## 何故かPostgreSQLでエラーになるのだが？と言う人

最近のhomebrewのアップデートでPostgreSQL使用時のvaletでエラーが発生するようになってる感じです

直し方を見つけたのでLaravel Valetの該当Issueに[解決方法を報告](https://github.com/laravel/valet/issues/1433#issuecomment-1653419658)してます
