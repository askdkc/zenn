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

<img width="707" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/82f77f67-6c8c-4084-a325-3c90f11a558c">

<br><br>

3. `.env`のDATABASE関係の情報を取得：Setting（左下の歯車） > Database の Connection info を参照

<img width="1426" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/3db3395c-c33a-4dc8-aa68-a07e98cf0c24">

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

<br>


4. `.env`のVITE_SUPABASE関係の情報を取得：Setting（左下の歯車） > API の API Settings を参照

<img width="1426" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/8bc508f4-dd78-4a12-baa4-9f2666e06703">

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
<img width="563" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/79208cd7-31e5-4b54-9388-666d83791a52">

## SupabaseのTable editorにテーブルが作成されていることを確認します

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/995266b0-8ff3-47cf-b342-c892b4a83605">

## テーブルに読み取り権限を付与します

> 手順としては[オフィシャルドキュメント](https://supabase.com/docs/guides/realtime/postgres-changes)通りです
>

SQL Editor からNew query > New blank queryをクリックします

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/a2370d9d-e22c-4f93-b777-c77f69a74f27">

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

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/1262e14f-eb2f-4266-8a5f-22b9b3b78166">


## リアルタイム機能をONにする

Database > Table を開き、usersテーブルの設定をクリックします

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/0de906d6-c737-473d-8a3b-890d3c4792b0">

Enable RealtimeをONにします

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/51b601f4-da57-4cb9-a306-ae94f1a5f7b4">

## Laravelからユーザ作成し動作確認

#### Laravelを起動させて動作確認します

```bash
php artisan key:generate

npm run build
php artisan serve
```

http://127.0.0.1:8000 にアクセス

#### Registerをクリックしてユーザを作成します

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/7ab1e534-dd3f-4feb-bdc6-14f60640e987">

ユーザ情報を適当に入力

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/076acc1f-e1fb-4fe1-af73-f14668739b6b">

#### 画面上部のステータスボタンを押すとステータスが変わります

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/dd20e251-40cb-42f4-8b30-10fbde91718f">

#### 他のユーザも作ってログインしてみましょう

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/98f5d392-74a7-4561-95d5-9bca0f58e292">

#### 全ユーザとステータスが見えます

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/c2e6a55c-6c09-4609-b522-eb76f52fdaa5">

#### adminユーザの方もリアルタイムに新規ユーザの登録やステータス変更が分かります

<img width="1428" alt="image" src="https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/49512f1e-e201-4276-8b51-d964b3f1c2cf">

### 動作GIF

![supa](https://github.com/askdkc/Realtime-SupaLaraSvelte/assets/7894265/8788724b-3fa5-4688-a773-d69ada0eb772)
