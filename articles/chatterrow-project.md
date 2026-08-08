---
title: "チャットにプロジェクト管理を詰め込んだchatterrowを作ったよ"
emoji: "🍵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','svelte','inertia','opensource']
published: true
---

# チャットにプロジェクト管理を詰め込んだchatterrowを作ったよ

チャットをしながら、タスクやファイルもまとめて管理できるグループウェア`chatterrow`（茶多楼）を作りました🍵

「チャット、タスク、ファイルが全部別のサービスにあるのは面倒だな〜」

『それ、chatterrowなら一つにまとめられます』

という感じのアプリです👍

[GitHubのリポジトリはこちら](https://github.com/askdkc/chatterrow)

![chatterrowのサービス紹介](https://raw.githubusercontent.com/askdkc/chatterrow/main/docs/assets/chatterrow-introduction.gif)

## これは何？

Discord風のUIで、プロジェクトごとにチャンネルを作って使います。

チャンネルの中でチャットをしたり、タスクを作ったり、ファイルを共有したりできます。プロジェクト全体のガントチャートも確認できます。

つまり、チャットを入口にして、プロジェクトの情報を一か所に集めるアプリです🤯

## プロジェクトとチャンネル

まずプロジェクトを作成し、その中に用途ごとのチャンネルを作ります。

例えば、こんな感じです。

```text
プロジェクト
├── 雑談チャンネル
├── 開発チャンネル
├── タスクチャンネル
└── ファイルチャンネル
```

会話の中で決まったことをタスクにし、関連する資料を同じチャンネルに添付できます。

プロジェクト名、内容、開始日、終了日、メンバーも設定できます。

## 主な機能

### リアルタイムチャット

Laravel Reverbを使ったリアルタイムチャットです。

メッセージだけでなく、スレッドや返信数もリアルタイムに同期されます。

Markdownにも対応していて、コードブロックはShikiでいい感じにハイライトされます✨

HTMLエスケープやHTTP(S) URLの制限も行っているので、安全にMarkdownを表示できます。

ちなみに、メッセージ送信は`Cmd+Enter`または`Ctrl+Enter`です。

IMEで日本語変換を確定するためのEnterでは送信されません。これは地味に大事です👀

### タスク管理とガントチャート

タスクには下記の情報を設定できます。

- 開始日、開始時刻
- 終了日、終了時刻
- 優先度
- メモ
- 完了状態

作成したタスクは、プロジェクト単位またはチャンネル単位のガントチャートで確認できます。

期限リマインダーにも対応しています。スケジューラとキューワーカーで自動通知します。

### ファイルの添付とプレビュー

ファイルやフォルダをドラッグ＆ドロップで添付できます。

画像、PDF、Officeファイルにはサムネイルが生成され、画像とPDFはビューアで確認できます。

OfficeファイルのプレビューにはONLYOFFICE Document Serverを使っています。

読み取り専用のプレビューなので、チームで資料を共有する用途にちょうどよいです👍

### OfficeやPDFをMarkdownに変換

PDF、DOCX、XLSX、PPTXは、Microsoft MarkItDownを使ってMarkdownに変換できます。

変換処理はバックグラウンドで実行され、Redisのキューに入れて処理します。

元ファイルだけでなくテキストとしても保存できるので、資料の再利用やAIでの活用にも使いやすくなっています。

### その他

- ダークモード、ライトモード
- プロジェクトメンバーの管理
- ファイルやフォルダのドラッグ＆ドロップ
- 画像、PDF、Officeファイルのプレビュー
- 期限リマインダー

## 使っている技術

バックエンドはLaravel、フロントエンドはInertiaとSvelteです。

```text
ブラウザ
  ↓
Inertia 3 + Svelte 5 + Tailwind CSS 4
  ↓
Laravel 13 / PHP 8.5+
  ├── Laravel Reverb（リアルタイム通信）
  ├── Redis（キュー）
  ├── ONLYOFFICE（Officeプレビュー）
  └── MarkItDown（文書のMarkdown変換）
```

技術構成は下記の通りです。

- Backend: Laravel 13 / PHP 8.5+
- Frontend: Inertia 3 / Svelte 5 / Tailwind CSS 4 / Vite 8
- Database: SQLiteまたはPostgreSQL
- Realtime: Laravel Reverb
- Queue: Redis / Laravel queue worker
- Preview: Shiki / ONLYOFFICE / Poppler / ImageMagick
- Conversion: Microsoft MarkItDown 0.1.7

## Ubuntuに一瞬でセットアップ

Ubuntu Serverで動かす場合は、リポジトリに入っている`setup.sh`を実行します。

```bash
apt install -y git

git clone https://github.com/askdkc/chatterrow.git
cd chatterrow
./setup.sh
```

ドメインやデータベース、Let's Encryptのメールアドレスなどを聞かれるので、環境に合わせて入力してください。

基本的には質問に答えていくだけで、Laravel、Redis、PostgreSQL、ONLYOFFICE、Reverb、nginx、Supervisorなどをまとめてセットアップしてくれます。

HTTPSが不要なローカル検証環境なら、下記のように`--no-ssl`を指定します。

```bash
./setup.sh --domain chatterrow.test --database sqlite --no-ssl
```

`chatterrow.test`を使う場合は、事前にDNSまたは`/etc/hosts`で名前解決できるようにしておいてください。

### 本番環境で使う場合

本番で使う場合は、READMEに書かれている要件を確認してから実行してください。

- Ubuntu 24.04 LTSまたは26.04 LTS（amd64）
- 2 CPU、2 GB RAM、30 GB以上の空きディスク
- インターネットから到達できる80/443ポート
- アプリ用のドメイン

ONLYOFFICEやReverbなどの内部サービスは外部公開せず、基本的に80/443だけを公開する構成になっています。

## macOSでも試せます

Apple siliconのmacOS 26以降なら、Apple Container上でONLYOFFICEを動かせます。

まず[Apple Container](https://github.com/apple/container)をインストールし、ImageMagickも入れておきます。

```bash
brew install imagemagick
cd /path/to/chatterrow
./setup.sh
```

macOSでは`setup.sh`がApple Containerを起動し、ONLYOFFICEを`127.0.0.1:8086`で公開します。

Laravel Valet、Laravel Herd、`php artisan serve`も自動判定してくれるので、macOSで開発している人にも便利です😊

## ソースコードを触ってみる

通常のLaravelアプリと同じようにローカル開発環境を作れます。

```bash
composer install
composer markitdown:install
cp .env.example .env
php artisan key:generate
touch database/database.sqlite
php artisan migrate
npm ci
```

フロントエンドを起動します。

```bash
npm run dev
```

別のターミナルでLaravel、Reverb、キューワーカーも起動します。

```bash
php artisan serve
php artisan reverb:start --port=8081
php artisan queue:work redis
```

Redis Serverが必要なので、macOSなら下記などで起動しておきます。

```bash
brew services start redis
```

## こんな人におすすめ

- チャットだけでなくタスクやファイルも一緒に管理したい人
- プロジェクトごとに情報を整理したい人
- 自分たちのサーバーでグループウェアを運用したい人
- オンプレミスや閉域網で使えるアプリを探している人
- LaravelとSvelteを使ったアプリのサンプルを探している人

チャット、タスク、ファイルをそれぞれ別サービスで管理するのではなく、一つのプロジェクトの中でまとめて扱いたい人には、かなり相性が良いと思います👍

## ライセンス

chatterrow本体のライセンスはMITです。

ただし、ONLYOFFICE Docs Community Editionを導入する場合は、AGPLv3の条件も確認してください。

## おわりに

チャットをしながらタスクを作り、ファイルを共有し、ガントチャートで進捗を確認する。

そんな感じで、業務の情報を一か所にまとめられるようにしています。

気になった方は、ぜひGitHubのリポジトリを覗いてみてください⭐

https://github.com/askdkc/chatterrow
