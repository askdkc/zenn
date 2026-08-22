---
title: "PGroonga用MCPを作ったよ"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [pgroonga, postgresql, mcp, opencode, sql]
published: true
---

# PGroonga用MCPを作ったよ

最近のAIモデルはSQLを書くのはかなり得意になり、やりたいことは大体頼めばやってくれます。インデックス設定による高速化もお手のものです。しかし、日本語検索になると日本語という厄介な壁の解決がやたら面倒だな？と思うことがあります。

日本語の全文検索に便利なPGroongaというPostgreSQL用の拡張機能があります。検索で使い勝手の良い中間一致にインデックスを利かすことができ、日本語検索も非常に高速です。ただ、インデックス作成やノーマライザー、トークナイザーの使い方に癖があり、条件次第では思ったように動作しないことも度々あリマした。AIもこの辺は苦手だったため、PGroonga用のMCPサーバーを作りました。

- [GitHub: askdkc/pgroonga-mcp](https://github.com/askdkc/pgroonga-mcp)
- [npm: @askdkc/pgroonga-mcp](https://www.npmjs.com/package/@askdkc/pgroonga-mcp)

## 何をしてくれる？

`pgroonga-mcp`はPostgreSQLとPGroongaの検索・診断を提供するMCPサーバーです。

どのご家庭にもあるAIツールからこのMCPサーバーを呼び出すと、次のような流れでPGroongaを使えます。

```text
Claude Code / Codex / OpenCode
          ↓ MCP
      pgroonga-mcp
          ↓
  PostgreSQL + PGroonga
```

主なツールは次のとおりです。

- `pgroonga_server_info`: PostgreSQL、PGroonga、Groongaの情報を確認
- `pgroonga_list_indexes`: 利用可能なPGroongaインデックスを確認
- `pgroonga_search`: PGroongaインデックスを使って検索
- `pgroonga_explain_search`: 検索を実行せずに`EXPLAIN`を確認
- `pgroonga_health`: PGroongaの状態を確認
- `pgroonga_normalize_text`: インデックスの正規化設定でテキストを正規化
- `pgroonga_lookup_variants`: NormalizerTableの変換候補を検索

## 前提条件

- Node.js 22以上
- Claude Code、Codex CLI、OpenCodeなど、MCPサーバーを利用できるAIツール

## MCPサーバーを追加する

MCPサーバーを使いたいプロジェクトのルートディレクトリで、npmパッケージをインストールします。

```zsh
npm i @askdkc/pgroonga-mcp
```

インストールすると、`pgroonga-mcp`コマンドがプロジェクトから利用できるようになります。

## MCPクライアントをセットアップする

パッケージをインストールしたら、セットアップコマンドを実行します。

```zsh
npx pgroonga-mcp setup
```

対話式の選択画面が表示されるので、使用するAIツールを選択します。Codex、Claude Code、OpenCode、DSH(Deepseek Harness)に対応しています。

対話式の入力を使わず、クライアントを指定してセットアップすることもできます。

```zsh
npx pgroonga-mcp setup --clients codex,claude,opencode
```

対応しているすべてのクライアントを設定する場合は、`--all`を指定します。

```zsh
npx pgroonga-mcp setup --all
```

セットアップで作成または更新されるプロジェクト設定ファイルは次のとおりです。

| AIツール | 設定ファイル |
| --- | --- |
| Codex | `.codex/config.toml` |
| Claude Code | `.mcp.json` |
| OpenCode | `opencode.json`、または既存の`opencode.jsonc` |
| DSH | `cordis.yml` |

プロジェクトの設定だけが変更され、ユーザー全体の設定は変更されません。

既存ファイルを変更せずに、変更内容だけ確認する場合は`--dry-run`を指定します。

```zsh
npx pgroonga-mcp setup --all --dry-run
```

既存の`pgroonga`設定がある場合は、内容を確認してから`--force`を指定します。

```zsh
npx pgroonga-mcp setup --clients claude --force
```

セットアップによって登録されるMCPサーバーの起動コマンドは、次の形式です。

```text
npx --no-install pgroonga-mcp
```

MCPを起動するたびにパッケージをネットワークからダウンロードするのではなく、プロジェクトにインストール済みのパッケージを使います。

## 手動で追加する場合

セットアップコマンドを使わずに追加する場合は、stdio形式のMCPサーバーとして次のコマンドを登録します。

```json
{
  "command": "npx",
  "args": ["--no-install", "pgroonga-mcp"]
}
```

AIツールによって設定ファイルの形式が異なるので、この`command`と`args`を各ツールのMCPサーバー設定に合わせて記載します。通常は`setup`コマンドを使う方が簡単です。

## PostgreSQLへの接続を設定する

プロジェクトのルートに`.env`を作成し、接続先を設定します。

```dotenv
PGROONGA_DATABASE_URL=postgresql://postgres:password@127.0.0.1:5432/app
PGROONGA_ALLOWED_SCHEMAS=public
PGROONGA_ALLOWED_TABLES=public.name_variants
```

`PGROONGA_ALLOWED_TABLES`には、AIから検索するテーブルを指定します。テーブル名は実際の環境に合わせて変更してください。

`.env`以外のファイルを使う場合は、`PGROONGA_ENV_FILE`で指定できます。

```zsh
export PGROONGA_ENV_FILE=.env.local
```

データベースURLが設定されていなくてもMCPサーバー自体は起動します。その状態では、データベースを使うツールが`database_unavailable`を返します。

## 実際の動作サンプル

MCPサーバーの有無を確認したサンプルを[pgroonga-mcp-sample](https://github.com/askdkc/pgroonga-mcp-sample)に作成してあります。

下記のような表記揺れや異体字を含むサンプルCSVデータ(`name_variants.csv`)を登録する指示をAIに出して、MCP無し/有りで出力されたSQLの結果を記載しています。

```text
PostgreSQLにテーブルを作成するSQLを書いて。
@name_variants.csv を読み込めるようにして。
「スーパーマン」「スーパーガール」「斎藤マン」で検索した時に表記揺れや異体字をヒット出来るSQLにして。
```

以下に内容を抜粋します。

### MCP無し版

生成されたSQLです。

```sql
-- psql -d <database> -f name_variants.sql で実行する。
-- name_variants.csv は psql を起動したカレントディレクトリに置く。

\set ON_ERROR_STOP on

BEGIN;

-- CSVに含まれる表記揺れを、CSVの各グループの先頭表記に寄せる。
CREATE OR REPLACE FUNCTION normalize_name_variant(value text)
RETURNS text
LANGUAGE sql
IMMUTABLE
PARALLEL SAFE
AS $function$
  SELECT translate(
    value,
    'ｰ-－‐‑–—―−─━斉齋齊辺邊邉',
    repeat('ー', 11) || '斎斎斎部部部'
  );
$function$;

CREATE TABLE IF NOT EXISTS name_variants (
  num integer PRIMARY KEY,
  name text NOT NULL,
  normalized_name text GENERATED ALWAYS AS (
    normalize_name_variant(name)
  ) STORED
);

CREATE INDEX IF NOT EXISTS name_variants_normalized_name_idx
  ON name_variants (normalized_name);

-- \\copy はCSVを実行元（psqlクライアント）から読み込む。
-- SQLクライアントから実行する場合は、下記をサーバー上の絶対パスを使う
-- COPY文に置き換える。
CREATE TEMP TABLE name_variants_import (
  num integer,
  name text
) ON COMMIT DROP;

\copy name_variants_import (num, name) FROM 'name_variants.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8')

INSERT INTO name_variants (num, name)
SELECT num, name
FROM name_variants_import
ON CONFLICT (num) DO UPDATE
SET name = EXCLUDED.name;

-- 正規化済みの完全一致検索。アプリケーションからは $1 を渡す。
CREATE OR REPLACE FUNCTION find_name_variants(search_text text)
RETURNS TABLE (num integer, name text)
LANGUAGE sql
STABLE
PARALLEL SAFE
AS $function$
  SELECT nv.num, nv.name
  FROM name_variants AS nv
  WHERE nv.normalized_name = normalize_name_variant($1)
  ORDER BY nv.num;
$function$;

COMMIT;

-- 検索例:
-- SELECT * FROM find_name_variants('スーパーマン');
-- SELECT * FROM find_name_variants('スーパーガール');
-- SELECT * FROM find_name_variants('斎藤マン');
```

MCP無し版ではPostgreSQLの機能だけで頑張ろうとしています。

### MCPあり版

生成されたSQLは下記です。

```sql
-- Run from the repository root with:
--   psql --set ON_ERROR_STOP=1 -f schema.sql

\set ON_ERROR_STOP on

BEGIN;

CREATE EXTENSION IF NOT EXISTS pgroonga;

-- Domain-specific canonicalization for kanji variants. NFKC does not
-- canonicalize these characters by itself.
CREATE TABLE public.name_variant_normalizations (
  target text PRIMARY KEY,
  normalized text NOT NULL
);

INSERT INTO public.name_variant_normalizations (target, normalized) VALUES
  ('斉', '斎'),
  ('齋', '斎'),
  ('齊', '斎'),
  ('邊', '辺'),
  ('邉', '辺');

CREATE INDEX name_variant_normalizations_index
  ON public.name_variant_normalizations
  USING pgroonga (
    target pgroonga_text_term_search_ops_v2,
    normalized
  );

CREATE TABLE public.name_variants (
  num integer PRIMARY KEY,
  name text NOT NULL
);

-- \copy reads the CSV from the psql client's current directory. Run this
-- script from the directory containing name_variants.csv.
\copy public.name_variants (num, name) FROM 'name_variants.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8')

CREATE INDEX name_variants_name_index
  ON public.name_variants
  USING pgroonga (name pgroonga_text_term_search_ops_v2)
  WITH (
    tokenizer = '',
    normalizers =
      'NormalizerNFKC100("unify_kana", true, "unify_hyphen_and_prolonged_sound_mark", true),
       NormalizerTable(
         "normalized",
         "${table:public.name_variant_normalizations_index}.normalized",
         "target",
         "target"
       )'
  );

COMMIT;

-- The same normalized index is used for both sides of each search.
SELECT num, name
FROM public.name_variants
WHERE name &= 'スーパーマン'
ORDER BY num;

SELECT num, name
FROM public.name_variants
WHERE name &= 'スーパーガール'
ORDER BY num;

SELECT num, name
FROM public.name_variants
WHERE name &= '斎藤マン'
ORDER BY num;
```

怪しいところとして

```sql
('邊', '辺'),
('邉', '辺');
```

がありますが、異体字対応やノーマライザーによる表記揺れにしっかりとPGroongaで対応出来ています。

## まとめ

日本語の全文検索を導入したい際に、PGroongaのインデックスや正規化設定をAIツールでいい感じにSQLを作りたい時は使ってみてください。

