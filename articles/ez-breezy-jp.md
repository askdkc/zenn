---
title: "Laravel Breezeを一瞬で日本語化するパッケージ"
emoji: "📦"
type: "tech" # tech: 技術記事
topics: ['Laravel','php']
published: true
---
# Laravel Breezeを手軽に日本語化出来るパッケージを作ったよ
## これは何？
Laravelの認証機能用パッケージ：Breezeは基本メニューとか表記が全て英語なのですが、このパッケージを使うことでそれらが一瞬で日本語になります。

## 何が凄いの？
ものの数秒で日本語化できます💪

![](https://storage.googleapis.com/zenn-user-upload/87b1bb61b90f-20221028.gif)

## 使い方は？
Laravel Breezeをインストール済みの環境で以下の2コマンドを実行

```bash
composer require askdkc/breezejp --dev
php artisan breezejp
```

> **注意：言語設定しておいてね**
> 
> パッケージインストール前でも後でも良いのですが
> Laravelの設定ファイル`config/app.php`で日本語を選択するのをお忘れなく

```vim
---before---
'locale' => 'en',
------------
↓
---after---
'locale' => 'ja',
-----------
```

## 仕組みは？
Laravelでは`/lang`ディレクトリ配下に言語ファイルを突っ込むと、それに応じて翻訳してくれる機能が標準で動いてます

なので、Laravel Breezeがstubs内のファイルを書き出してくれるのと同様、このパッケージも事前にstabsに仕込んでおいた日本語用設定ファイルを`breezejp`コマンドで書き出してるだけです

ファイルを書き出してるだけなので、パッケージの更新の影響とかは特に受けません。最初にサクッと日本語化した後は、煮るなり焼くなりご自由に✌️

## カスタマイズも楽々

下記ディレクトリ配下に出力されたファイルを好きに弄ってね

```bash
/lang/ja.json ← Breezeの各画面の日本語ファイル / メール通知の翻訳もこちら
/lang/ja/auth.php ← 認証画面の警告メッセージの日本語ファイル
/lang/ja/pagination.php ← ページ送りの日本語ファイル
/lang/ja/auth.php ← 認証画面のパスワード関係の日本語ファイル
/lang/ja/validation.php ← 各種バリデーションの日本語ファイル
```
## GitHubに公開中

[こちらのリポジトリ](https://github.com/askdkc/breezejp)で公開しております
