---
title: "Laravel Breezeを一瞬で日本語化するパッケージ"
emoji: "📦"
type: "tech" # tech: 技術記事
topics: ['Laravel','php']
published: true
---
# Laravel Breezeを手軽に日本語化出来るパッケージを作ったよ
## これは何？
Laravelの認証機能用パッケージ：Breezeはとても便利なのですけど、そのままだと基本メニューとか表記が全て英語です

このパッケージを使うと一瞬で日本語になって便利です👍

もし使ってみて気に入ったらサポートしてね💕

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X7X8O7KCU)

 [![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/askdkc)

## 何が凄いの？
こんな感じで、ものの数秒で日本語化できます💪

![](https://storage.googleapis.com/zenn-user-upload/87b1bb61b90f-20221028.gif)

## 使い方は？
Laravel Breezeのbladeテンプレートをインストールします
```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
```

その後、以下の2コマンドを実行し、この日本語化パッケージを導入

```bash
composer require askdkc/breezejp --dev
php artisan breezejp
```

> **言語設定も自動化**
> 
> このパッケージインストールするとLaravelの設定ファイル
> `/config/app.php`のロケールを下記のように自動で日本語化します👍

```vim
---before---
'locale' => 'en',

'faker_locale' => 'en_US',
------------
↓
---after---
'locale' => 'ja',

'faker_locale' => 'ja_JP',
-----------
```

## 仕組みは？
Laravelでは`/lang`ディレクトリ配下に言語ファイルを突っ込むと、それに応じて翻訳してくれる機能が標準で動いてます

なので、認証パッケージであるLaravel Breezeがstubs内のファイルを書き出してる動きであるのと同様、このパッケージも事前にstubsに仕込んでおいた日本語用設定ファイルを`breezejp`コマンドで書き出してるだけです

ファイルを書き出してるだけなので、パッケージの更新の影響とかは特に受けません。最初にサクッと日本語化した後は、中身を煮るなり焼くなりご自由に✌️

## カスタマイズも楽々

下記ディレクトリ配下に出力されたファイルを好きに弄ってね

```bash
.
└─ lang
   ├── ja.json ← Breezeの各画面の日本語ファイル / メール通知の翻訳もこちら
   └─ ja
        ├── auth.php ← 認証画面の警告メッセージの日本語ファイル
        ├── pagination.php ← ページ送りの日本語ファイル
        ├── auth.php ← 認証画面のパスワード関係の日本語ファイル
        └── validation.php ← 各種バリデーションの日本語ファイル
```

## 実は各種バリデーションメッセージも日本語化しちゃいます
[上記のカスタマイズ](#カスタマイズも楽)の箇所に `validation.php` があるのに気づいた人は、これによってLaravelの標準のバリデーションメッセージも一緒に全部日本語化されることに気づいたと思います👀

そうなんです✨

このパッケージを最初に入れておくとLaravel標準機能のバリデーションを使うだけで簡単に日本語でユーザに警告メッセージが出せちゃうんです👍

## パッケージはGitHubで公開中

[こちらのリポジトリ](https://github.com/askdkc/breezejp)で公開しております

## おまけ
ついでにJetstreamの日本語化もできちゃいます🤫

![](https://user-images.githubusercontent.com/7894265/208773006-2feea23e-ca45-4d40-9911-49f03db9ed4d.png)

