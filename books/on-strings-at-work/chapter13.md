---
title: PostgreSQL投入――staging tableで汚いデータを検品する
---
この章に入って、ミナの右下バッファは初めてSQLの実行結果置き場になった。

それまでは抽出結果や作業メモを置いていた場所である。だが、ここからはPostgreSQLへ流し込む前後の確認が必要になる。だから、同じ場所を `psql` の結果、検査SQL、エラー行の確認に使う。

ミナが文字列、CSV、ログ、JSONを扱えるようになった頃、黒瀬が青い顔でノートPCを持ってきた。

「全支店の顧客データを新システムへ移す。だが、形式が支店ごとに違う。」

```text
tokyo_customers.csv
osaka_customer_list.csv
nagoya_2024_old.csv
fukuoka_export.txt
sapporo_api_response.json
```

ミナは言った。

「これは、いきなり本番テーブルへ入れてはいけません。」

## staging tableを作る

```sql
CREATE TABLE import_customers_staging (
    source_file text,
    source_branch text,
    raw_customer_id text,
    raw_name text,
    raw_email text,
    raw_amount text,
    raw_registered_at text,

    normalized_customer_id text,
    normalized_name text,
    normalized_email text,
    amount integer,
    registered_at date,

    import_error text,
    imported_at timestamptz DEFAULT now()
);
```

## 何をしているか

本番投入前の検品用テーブルを作る。

```text
raw_*
  元データをそのまま残す。

normalized_*
  正規化後の値を入れる。

amount / registered_at
  PostgreSQLで扱いやすい型にした値。

import_error
  変換や検査で問題があった理由。
```

## 正規化してstaging rowを作る

ミナは名前欄を見て、指が自然に止まった。

```text
" 高橋　美咲 "
```

前に社員名簿で見た汚れと同じだった。全角スペース、前後空白、見た目は同じなのに機械には別物として扱われる問題。

ミナは新しく考え直さなかった。すでに作った `normalize-name-space` を使うことにした。こういう時のために、小さな関数には名前をつけて残してある。

高村は、ミナが何気なく関数名を打つのを見て言った。

「その関数、前にも見たな。」

「名前の汚れは、部署が変わってもだいたい同じなんです。」

「だから小さい道具にして残しておくのか。」

ミナは頷いた。

```lisp
(defun make-staging-row
    (&key source-file source-branch
          customer-id name email amount registered-at)
  (let* ((clean-name (normalize-name-space name))
         (clean-email (string-trim " " email))
         (amount-result (parse-amount-field amount))
         (amount-ok-p (getf amount-result :ok))
         (clean-amount (when amount-ok-p
                         (getf amount-result :value)))
         (errors '()))
    (unless amount-ok-p
      (push (getf amount-result :error) errors))
    (list
     :source-file source-file
     :source-branch source-branch
     :raw-customer-id customer-id
     :raw-name name
     :raw-email email
     :raw-amount amount
     :raw-registered-at registered-at
     :normalized-customer-id customer-id
     :normalized-name clean-name
     :normalized-email clean-email
     :amount clean-amount
     :registered-at registered-at
     :import-error (when errors
                     (format nil "~{~A~^; ~}" (nreverse errors)))))))
```

## 何を直したか

以前のように、`maybe-parse-integer` が値そのものか `nil` だけを返す設計だと、`0` のような正しい値を誤ってエラー扱いする危険がある。
ここでは `parse-amount-field` が `:ok` と `:value` を分けて返すため、`0` も正しい値として扱える。

```lisp
(parse-amount-field "0")
;; => (:OK T :VALUE 0 :RAW "0" :CLEANED "0")
```

## 出力例

```lisp
(:SOURCE-FILE "tokyo_customers.csv"
 :SOURCE-BRANCH "tokyo"
 :RAW-CUSTOMER-ID "002"
 :RAW-NAME "佐藤 花子"
 :RAW-EMAIL " sato@example.com "
 :RAW-AMOUNT "12,000円"
 :NORMALIZED-NAME "佐藤 花子"
 :NORMALIZED-EMAIL "sato@example.com"
 :AMOUNT NIL
 :IMPORT-ERROR "not an integer")
```

## SQLで検査してから本テーブルへ

```sql
SELECT *
FROM import_customers_staging
WHERE import_error IS NOT NULL;
```

## 記号の読み方

```text
SELECT *
  すべての列を見る。

FROM import_customers_staging
  import_customers_staging テーブルから見る。

WHERE import_error IS NOT NULL
  import_error が空ではない行だけを見る。
```

これは、問題があった行だけを見るSQLである。

重複顧客IDを見る。

```sql
SELECT normalized_customer_id, count(*)
FROM import_customers_staging
GROUP BY normalized_customer_id
HAVING count(*) > 1;
```

## 記号の読み方

```text
SELECT normalized_customer_id, count(*)
  顧客IDと、その件数を見る。

GROUP BY normalized_customer_id
  同じ顧客IDごとにまとめる。

HAVING count(*) > 1
  まとめた後、件数が2件以上のものだけ残す。
```

これは、同じ顧客IDが複数回出ているものを探すSQLである。

移行前日の夜、大阪支店のファイルに同じ顧客IDが二件あることが分かった。

```csv
A-005, 高橋　美咲 ,takahashi@example.com,"18,000",2024/4/5
A-005, 高橋 美咲,takahashi-new@example.com,19000,2024/04/07
```

高村は焦った。

「全部止めるしかないか？」

ミナは首を横に振った。

「壊れてる行だけ止めればいいです。全部止める必要ないっす。」

その場が静かになった。

ミナは、文字列を整えただけではない。
どこまで機械に任せ、どこから人間に返すべきかを分けた。

## ミナのメモ

ミナは右下のメモバッファに、ここまでの変換を整理した。

```text
汚いCSV/JSON
  ↓ 正規化
staging row
  ↓ COPY
staging table
  ↓ 検査SQL
本テーブル

判断:
  正常行は流す。
  危険行は止める。
  人間判断が必要なものは返す。
```

---
