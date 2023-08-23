---
title: "SupabaseのRealtime機能をLaravelから利用する"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Supabase','Laravel','Svelte','PostgreSQL']
published: true
---
# Supabaseのリアルタイム機能をLaravelから利用しよう

ここではSupabaseで使えるリアルタイムにDBへの変更点を通知する機能を、LaravelとSvelteで使っていく方法を案内します👍

![](https://storage.googleapis.com/zenn-user-upload/34960a3242a4-20230809.gif)

### 必要なもの
- Supabaseのアカウント（無料！）
- Laravelが動く環境（無料！）
- npm（無料！）

## サンプルコードのセットアップ

下記の手順でセットアップ出来ます

```bash
git clone git@github.com:askdkc/Realtime-SupaLaraSvelte.git
cd Realtime-SupaLaraSvelte
cp .env.example .env
composer install
npm i
```

### `.env`用設定取得

Supabaseに無料枠を作ってある前提で話を進めます

1. ログインしてDashboardにアクセス


2. New Projectで新規プロジェクトを作成(NameとPasswordは適当に)

![](https://storage.googleapis.com/zenn-user-upload/5c17b39d6653-20230809.png)

3. `.env`のDATABASE関係の情報を取得：Setting（左下の歯車） > Database の Connection info を参照

![](https://storage.googleapis.com/zenn-user-upload/5aef65f659a9-20230809.png)

上記を参照に`.env`を下記のようにします

- Before
```env
DB_CONNECTION=pgsql
DB_HOST="ここにProject Settings > Database SettingのHostのURLを入れる"
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres
DB_PASSWORD="自分でSupabaseに設定したパスワード"
```

- After (パスワードをmy-supa-secret-passwordにした場合)
```env
DB_CONNECTION=pgsql
DB_HOST=db.nymypvoodrylamygipip.supabase.co
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres
DB_PASSWORD=my-supa-secret-password
```

4. `.env`のVITE_SUPABASE関係の情報を取得：Setting（左下の歯車） > API の API Settings を参照

![](https://storage.googleapis.com/zenn-user-upload/46d360d58b94-20230809.png)

`.env`を下記のようにします

- Before
```env
VITE_SUPABASE_URL="ここにProject Settings>API>Project URLのhttps://xxx.supabase.coを入れる"
VITE_SUPABASE_KEY="ここにProject Settings>API>Project API keysののanon publicのkeyを入れる"
```

- After
```env
VITE_SUPABASE_URL=https://nymypvoodrylamygipip.supabase.co
VITE_SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3Mi(省略)
```

## Laravelでmigrationを実行します

```bash
php artisan migrate
```

![](https://storage.googleapis.com/zenn-user-upload/a143562bdbf7-20230809.png)

## SupabaseのTable editorにテーブルが作成されていることを確認します

![](https://storage.googleapis.com/zenn-user-upload/f6436aa48073-20230809.png)

## テーブルに読み取り権限を付与します

> 手順としては[オフィシャルドキュメント](https://supabase.com/docs/guides/realtime/postgres-changes)通りです
>

SQL Editor からNew query > New blank queryをクリックします

![](https://storage.googleapis.com/zenn-user-upload/9c384aa59743-20230809.png)

次のSQLを実行します
```sql
alter table "users"
enable row level security;

create policy "Allow anonymous access"
  on users
  for select
  to anon
  using (true);
```

![](https://storage.googleapis.com/zenn-user-upload/53655b57aa18-20230809.png)


## リアルタイム機能をONにする

Database > Table を開き、usersテーブルの設定をクリックします

![](https://storage.googleapis.com/zenn-user-upload/f25b2f128f22-20230809.png)

Enable RealtimeをONにします

![](https://storage.googleapis.com/zenn-user-upload/26f2d74ca591-20230809.png)

## Laravelからユーザ作成し動作確認

#### Laravelを起動させて動作確認します

```bash
php artisan key:generate

npm run build
php artisan serve
```

http://127.0.0.1:8000 にアクセス

#### Registerをクリックしてユーザを作成します

![](https://storage.googleapis.com/zenn-user-upload/1a52388491c6-20230809.png)

ユーザ情報を適当に入力

![](https://storage.googleapis.com/zenn-user-upload/57a090fff9d4-20230809.png)

#### 画面上部のステータスボタンを押すとステータスが変わります

![](https://storage.googleapis.com/zenn-user-upload/a065d5f4eba7-20230809.png)

#### 他のユーザも作ってログインしてみましょう

![](https://storage.googleapis.com/zenn-user-upload/92618df14b11-20230809.png)

#### 全ユーザとステータスが見えます

![](https://storage.googleapis.com/zenn-user-upload/1e2900935e07-20230809.png)

#### adminユーザの方もリアルタイムに新規ユーザの登録やステータス変更が分かります

![](https://storage.googleapis.com/zenn-user-upload/8af3d3a54b00-20230809.png)

### 動作GIF

![](https://storage.googleapis.com/zenn-user-upload/34960a3242a4-20230809.gif)

## この記事が気に入ったらサポートしてね💕

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X7X8O7KCU)

[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/askdkc)
