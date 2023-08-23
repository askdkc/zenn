---
title: "Laravelを3コマンドで他言語化対応🇺🇸🇯🇵"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','language','localization']
published: true
---
# Laravelを3コマンドで他言語化対応

Laravelを3コマンドだけで他言語化対応する方法を紹介します

![Laravel multi-lingual image](https://storage.googleapis.com/zenn-user-upload/2a2b10186d6e-20230709.gif)

## 事前準備：　Laravel Breezeのインストール

他言語対応したのが分かりやすいように事前にLaravel Breezeをインストールしておきます

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
php artisan migrate
```

![Breeze Default](https://storage.googleapis.com/zenn-user-upload/0b6c5e67f8a0-20230709.png)

## 日本語化対応 (2コマンド)

下記のコマンドを実行します

```bash
composer require askdkc/breezejp --dev
php artisan breezejp
```

これで日本語化対応が完了です🇯🇵

Laravel Breezeの各画面にアクセスしてみてください。日本語化されています👀

![Breeze Japanese](https://storage.googleapis.com/zenn-user-upload/6eafa5f1efb7-20230709.png)

## 多言語化対応 (1コマンド)

下記のコマンドを実行します

```bash
php artisan breezejp --langswitch
```

これで多言語化対応が完了です🤯

言語切替は

- http://localhost:8000/language/en
- http://localhost:8000/language/ja

で行えます🇺🇸🇯🇵

![Breeze English](https://storage.googleapis.com/zenn-user-upload/06d4be1311fc-20230709.png)

## おわりに

「認証機能(ログイン、ユーザ登録、パスワードリセット、メールの確認、パスワードの確認)の搭載って大変だよね？」

『それLaravelなら2コマンドですね』

「えぇ？でも英語でしょ？」

『日本語化も2コマンドですね』

「多言語化は無理でしょ？」

『1コマンドですね』

🤯

## この記事が気に入ったらサポートしてね💕

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X7X8O7KCU)

[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/askdkc)
