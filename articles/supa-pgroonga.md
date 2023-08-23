---
title: "SupabaseでPGroongaを使う"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['supabase','pgroonga','svelte','postgresql']
published: false
---

# SupabaseでPGroongaを使ってみよう

Supabaseで日本語のいい感じな全文検索拡張機能である`PGroonga`が使えるので、その使い方の例を紹介します

## まずはサインアップ

まずは[supabase](https://supabase.com)にサインアップしてアカウントを作成し、適当に組織を作り、下記のようなダッシュボードにアクセスしDBを作ります

<img width="1418" alt="create-new-project" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/e3ff10ad-047c-4774-9033-1ce03c9edee1">


## ExtensionをONにする

Supabaseは日本語検索に強いPGroongaを使えるので、こいつをONにします

左側のDatabaseのExtensionを開いて検索フィールドにPGroongaと入れると出てきます（左側のPGroongaを選択してね）

![pgronnnga-extension](https://github.com/askdkc/askdkc.github.io/assets/7894265/9beb3c6a-bb05-44e1-9883-1aaf9208fb73)


## サンプルデータの作成

PGroongaのオフィシャルドキュメントのハウツー内に[「PostgRESTでPGroongaを使う方法」](https://pgroonga.github.io/ja/how-to/postgrest.html)という記事がありますので、これを参考にサンプルデータを作っていきます

### サンプルデータ

<img width="1418" alt="table-creation" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/ae9ab209-0eee-4817-805d-942c88b5df24">

```sql
CREATE TABLE memos (
  id integer,
  title text,
  content text
);

INSERT INTO memos VALUES (1, 'PostgreSQLはリレーショナル・データベース管理システムです。','すごいでしょう');
INSERT INTO memos VALUES (2, 'Groongaは日本語対応の高速な全文検索エンジンです。','スゴイデショウ');
INSERT INTO memos VALUES (3, 'PGroongaはインデックスとしてGroongaを使うためのPostgreSQLの拡張機能です。','ハバナイスデー');
INSERT INTO memos VALUES (4, 'groongaコマンドがあります。','今日はコンバンワこんにちわ');

```

### サンプルデータへの検索用`PGroonga`インデックス作成

<img width="1418" alt="create-indexes" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/44377810-dc0a-4407-b659-5357f8e61510">

```sql
CREATE INDEX pgroonga_title_search_index ON memos USING pgroonga (title) 
  WITH (
   normalizers = 'NormalizerNFKC150(
    "unify_to_romaji", true,
    "unify_hyphen_and_prolonged_sound_mark", true
   )',
   tokenizer='TokenNgram(
    "unify_alphabet", false,
    "unify_symbol", false,
    "unify_digit", false,
    "report_source_location", true
  )',
);
  
CREATE INDEX pgroonga_content_search_index ON memos USING pgroonga (content) 
  WITH (
   normalizers = 'NormalizerNFKC150(
    "unify_to_romaji", true,
    "unify_hyphen_and_prolonged_sound_mark", true
   )',
   tokenizer='TokenNgram(
    "unify_alphabet", false,
    "unify_symbol", false,
    "unify_digit", false,
    "report_source_location", true
   )',
);

```

## `PGroonga`検索用ストアドファンクション作成

<img width="1418" alt="create-function" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/db6bfeca-b581-45ee-aefc-930b29d5bec5">

```sql
-- Title検索用
CREATE FUNCTION find_title(keywords text) RETURNS SETOF memos AS $$
BEGIN
  RETURN QUERY SELECT * FROM memos WHERE title &@~ keywords;
END;
$$ LANGUAGE plpgsql;

-- Content検索用
CREATE FUNCTION find_content(keywords text) RETURNS SETOF memos AS $$
BEGIN
  RETURN QUERY SELECT * FROM memos WHERE content &@~ keywords;
END;
$$ LANGUAGE plpgsql;

```

## アクセス権限の付与

作成した`memos`テーブルに読み取り専用のアクセス権限を設定します

<img width="1418" alt="create-policy" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/89403b68-a349-47a6-b86f-a24b75f2e706">

```sql
-- 1. Enable RLS
alter table memos
  enable row level security;

-- 2. Create Policy
create policy "Public memos are viewable by everyone."
  on memos for select using (
    true
);

```

## フロントエンドの準備

Supabaseのオフィシャルドキュメントを参考にSvelteでフロントエンドを作成します：

https://supabase.com/docs/guides/getting-started/tutorials/with-svelte

### Svelteの準備

```bash
npm create vite@latest supabase-svelte -- --template svelte-ts
cd supabase-svelte
npm install
```
### `supabase-js`のインストール

```bash
npm install @supabase/supabase-js
```

### `.env`ファイルの作成

```bash
touch .env
vi .env
```

Supabaseの `Project Settings > API` から`SUPABASE_URL`と`SUPABASE_ANON_KEY`をゲットします

<img width="1418" alt="env-keys" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/9381aee1-ec04-4115-906d-18cf73d173d9">

#### `.env`の中身に記載

```vim
VITE_SUPABASE_URL=YOUR_SUPABASE_URL
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

### Supabaseへの接続用クライアントファイル準備

`src/supabaseClient.ts`ファイルを作成し、次の中身を記載：

```ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseAnonKey)

```

### 検索用フロントエンド作成

`src/App.svelte`を次のように変更：

```html
<script lang="ts">
  import { supabase } from './supabaseClient'

  let keyword = ''
  let results = []

  const handleSearch = async () => {
    if (keyword.length === 0) {
      results = []
      return
    }

    try {
      const { data, error } = await supabase.rpc('find_title', { keywords: keyword })
      results = data
      if (error) throw error
    } catch (error) {
      if (error instanceof Error) {
        alert(error.message)
      }
    }
  }
</script>

<main>
  <h1>Supabase + Svelte</h1>

  <form on:submit|preventDefault={handleSearch}>
    <input type="text" bind:value={keyword} placeholder="Search for a title">
    <button type="submit">
      Search
    </button>
  </form>
  {#if keyword.length > 0}
    {#if results.length > 0}
      <ul>
        {#each results as result (result.id)}
          <li>
            Title: {result.title}<br>
            Content: {result.content}
          </li>
        {/each}
      </ul>
    {/if}
  {/if}
</main>

<style>
  input {
  font-size: 1em;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: none;
  transition: border-color 0.3s;
}
</style>
```

##  動作確認

Svelteをコンパイルしてアクセス

```bash
npm run dev
```

http://localhost:5173 へブラウザでアクセス

<img width="1418" alt="image" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/e137aef2-9335-40b2-85e3-1e816842fe95">

検索すると検索結果が下に表示されます(ローマ字で入力しても検索可能！)

<img width="1418" alt="image" src="https://github.com/askdkc/askdkc.github.io/assets/7894265/7a8bccb9-57bb-4bc0-8246-f62f834a7053">

