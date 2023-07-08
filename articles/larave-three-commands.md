---
title: "Laravelを3コマンドで他言語化対応"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','language','localization']
published: false
---
# Laravelを3コマンドで他言語化対応

Laravelを3コマンドだけで他言語化対応する方法を紹介します。

## 事前準備：　Laravel Breezeのインストール

他言語対応したのが分かりやすいように事前にLaravel Breezeをインストールしておきます。

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
php artisan migrate
```

## 日本語化対応 (2コマンド)

下記のコマンドを実行します。

```bash
composer require askdkc/breezejp --dev
php artisan breezejp
```

これで日本語化対応が完了です。

Breezeの各画面にアクセスしてみてください。日本語化されています。

## 多言語化対応 (1コマンド)

下記のコマンドを実行します。

```bash
php artisan breezejp --langswitch
```

これで多言語化対応が完了です。

言語切替は

- http://localhost:8000/language/en
- http://localhost:8000/language/ja

で行えます。

## おわりに

「認証機能(ログイン、ユーザ登録、パスワードリセット、メールの確認、パスワードの確認)の搭載って大変だよね？」

『それLaravelなら2コマンドですね』

「えぇ？でも英語でしょ？」

『日本語化も2コマンドですね』

「多言語化は無理でしょ？」

『1コマンドですね』

🤯
