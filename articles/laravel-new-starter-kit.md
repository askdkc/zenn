---
title: "Laravel 12の新しいスターターキットを一瞬で日本語化"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','livewire','japanese']
published: false
---
# Laravel 12のスターターキットを一瞬で日本語化する方法の紹介

## まずはLaravelの新規プロジェクト作成

``` bash
laravel new newproj
```

- starter kitを聞かれたらLivewireを選択します

- Voltの利用はどちらでもOK

- 作ったプロジェクトにcdします

``` bashbash
cd newproj
```

## 日本語化
次のコマンドを実行します

``` bash
composer require askdkc/breezejp

php artisan breezejp
```

## 完了
終わりです
