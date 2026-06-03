---
title: "Common Lispのリスト思考が、SQLやアプリ開発に効く理由"
emoji: "🌀"
type: "tech" # tech: 技術記事
topics: ['commonlisp','postgresql','lisp','sql']
published: true
---
# # Common Lispのリスト思考が、SQLやアプリ開発に効く理由

## 前提：Common Lispで日頃からリストで考えることが、なぜアプリ開発に効くのか

Common Lispでリストを扱う訓練は、単に「Lispが書けるようになる」ためだけのものではない。

より大きな効果は、アプリ開発で扱うデータを、

```text
文字列
オブジェクト
配列
JSON
SQL
APIレスポンス
画面表示用データ
```

としてバラバラに見るのではなく、

```text
入力データを、目的の出力データへ変形する流れ
```

として見られるようになることにある。

アプリ開発では、ほとんどの作業がこの形になる。

```text
DBから取る
↓
必要な形に絞る
↓
並べ替える
↓
まとめる
↓
入れ子にする
↓
APIとして返す
↓
画面で表示する
```

これは、Common Lispで日常的にやるリスト操作そのものに近い。

```lisp
(remove-if-not #'条件 データ) ; 絞る
(mapcar #'変換 データ)        ; 形を変える
(reduce #'集約 データ)        ; 畳む
(sort データ #'比較)          ; 並べる
(walk 木構造)                 ; ネストを辿る
```

つまり、Lispのリスト思考は、アプリ開発における次の能力を鍛える。

```text
データの形を読む力
必要な出力形から逆算する力
中間データを想像する力
複雑な処理を小さな変換に分解する力
ネスト構造を怖がらない力
SQLやAPIを「文字列」ではなく「構造」として見る力
```

特にAPI開発では、DBのテーブルは平らなのに、返したいレスポンスはネストしたJSONになることが多い。

DB側：

```text
users
orders
comments
tags
```

API側：

```json
{
  "id": 1,
  "name": "田中",
  "orders": [
    {
      "id": 101,
      "total": 3200
    }
  ]
}
```

このギャップを埋めるには、

```text
平らな表をどうJOINするか
どこで親子関係を作るか
どこで複数行を配列に畳むか
どこでJSONオブジェクトにするか
```

を想像する必要がある。

Common Lispでリストを扱うと、この「形を変える想像力」が鍛えられる。

だから、Lispのリスト思考はSQLにも効くし、Laravel、Svelte、SwiftUI、GoなどでAPIや画面用データを組み立てるときにも効く。

---

## 1. SQL初心者が本当に躓く場所

SQL初心者が躓く原因は、単にSQL文法を知らないことではない。

本質はここにある。

```text
自分が欲しいデータを取り出すために、
どんな中間データを作ればいいのか分からない。
```

特にAPI用のデータを出力するときに問題が出る。

```text
欲しいJSONの形は分かる
必要そうなテーブルも分かる
でも、どこからSQLを書けばいいのか分からない
JOINすべきか
GROUP BYすべきか
json_aggすべきか
サブクエリにすべきか
CTEにすべきか
分からない
```

これはSQL文法以前に、

```text
データの形を段階的に変形する想像力
```

が足りない状態である。

Common Lispで日頃からリストで考えることは、この想像力を鍛える。

---

## 2. SQLを「文」ではなく「データ変換」として見る

SQL初心者はSQLをこう見がち。

```sql
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
ORDER BY ...
```

しかし、実際にはこう見るべき。

```text
元データ
↓
JOINして材料を集める
↓
WHEREで絞る
↓
GROUP BYで親単位にまとめる
↓
json_aggで子配列を作る
↓
SELECTでAPI出力の形にする
```

Lisp的に言えば、SQLはこれに近い。

```lisp
(-> データ
    絞る
    変換する
    グループ化する
    畳む
    出力形にする)
```

つまり、SQLは単なる文法ではなく、

```text
データの形を変える処理
```

である。

Common Lispでリストを扱うと、この見方が自然になる。

---

## 3. 欲しいAPI出力をまずリストとして見る

例えば、APIでこう返したいとする。

```json
{
  "id": 1,
  "name": "田中",
  "orders": [
    {
      "id": 101,
      "total": 3200
    },
    {
      "id": 102,
      "total": 1500
    }
  ]
}
```

Common Lisp的には、まずこれをリストとして見る。

```lisp
(:user
 :id 1
 :name "田中"
 :orders ((:id 101 :total 3200)
          (:id 102 :total 1500)))
```

さらに分解する。

```lisp
(:user
 :id user-id
 :name user-name
 :orders order-list)
```

そして `order-list` はこういう複数要素のリストである。

```lisp
((:id order-id :total total)
 (:id order-id :total total)
 ...)
```

ここまで分解すると、必要な処理が見える。

```text
1. users から user を取る
2. orders から user_id が一致する注文を取る
3. orders をリストとして集める
4. user の中に orders として入れる
```

SQLではこうなる。

```sql
SELECT
  u.id,
  u.name,
  json_agg(
    json_build_object(
      'id', o.id,
      'total', o.total
    )
    ORDER BY o.id
  ) AS orders
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.id = 1
GROUP BY u.id, u.name;
```

重要なのは、`json_agg` という関数を知っていることだけではない。

その前に、

```text
orders は複数行なので、1つのリストに畳む必要がある
```

と想像できること。

この想像力が、Lispのリスト思考で育つ。

---

## 4. JOINは「平らにする」、GROUP BYは「畳む」

SQL初心者はJOINを魔法のように見る。

```sql
SELECT *
FROM users
JOIN orders ON orders.user_id = users.id;
```

しかしJOINの結果はこうなる。

```text
user_id | name | order_id | total
--------+------+----------+------
1       | 田中 | 101      | 3200
1       | 田中 | 102      | 1500
```

ユーザー情報が注文数だけ重複する。

これは壊れているのではない。

```text
親と子をJOINした結果、親が子の数だけ展開された
```

だけである。

Lisp的に見ると、JOIN後のデータはこういう平らなリストである。

```lisp
'((:user-id 1 :name "田中" :order-id 101 :total 3200)
  (:user-id 1 :name "田中" :order-id 102 :total 1500))
```

APIで欲しいのはこれではない。

欲しい形はこう。

```lisp
'((:id 1
   :name "田中"
   :orders ((:id 101 :total 3200)
            (:id 102 :total 1500))))
```

つまり、必要な処理はこれ。

```text
JOINで一度平らにする
↓
user_idごとにGROUP BYする
↓
ordersをjson_aggで配列に畳む
```

SQL：

```sql
SELECT
  u.id,
  u.name,
  json_agg(
    json_build_object(
      'id', o.id,
      'total', o.total
    )
    ORDER BY o.id
  ) AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

この見方ができると、SQLのJOINで行数が増えても混乱しにくい。

---

## 5. 欲しいAPI出力から「木」を描く

例えば、欲しいAPI出力がこれだとする。

```json
{
  "id": 1,
  "title": "Lisp入門",
  "author": {
    "id": 5,
    "name": "佐藤"
  },
  "tags": ["lisp", "sql", "postgresql"],
  "comments": [
    {
      "id": 10,
      "body": "分かりやすい",
      "user": {
        "id": 8,
        "name": "鈴木"
      }
    }
  ]
}
```

Lispのリストで見るとこうなる。

```lisp
(:id 1
 :title "Lisp入門"
 :author (:id 5 :name "佐藤")
 :tags ("lisp" "sql" "postgresql")
 :comments ((:id 10
             :body "分かりやすい"
             :user (:id 8 :name "鈴木"))))
```

この時点で、必要な関係が見える。

```text
posts
  ├─ author: users
  ├─ tags: post_tags -> tags
  └─ comments
       └─ comment user: users
```

つまり、出力構造はこういう木である。

```text
post
├── author
├── tags[]
└── comments[]
    └── user
```

SQLを書く前に、この木を描けるかが重要。

---

## 6. 「1個なのか、複数なのか」を分ける

API出力で一番大事なのはこれ。

```text
その項目は1個か？
それとも複数個か？
```

例えば：

```json
{
  "post": {
    "id": 1,
    "title": "記事A",
    "author": {
      "id": 10,
      "name": "田中"
    },
    "tags": ["lisp", "sql"],
    "comments": [
      {"id": 100, "body": "good"},
      {"id": 101, "body": "nice"}
    ]
  }
}
```

ここでは：

```text
post      = 1個
author    = 1個
tags      = 複数
comments  = 複数
```

この区別がSQL設計に直結する。

| API上の形 | SQL上の処理 |
|---|---|
| 1個のオブジェクト | JOINして列を足す |
| 複数の子要素 | `json_agg` / `array_agg` |
| 数値の合計 | `SUM` |
| 件数 | `COUNT` |
| 最大・最新 | `MAX` / `ORDER BY ... LIMIT 1` / `DISTINCT ON` |
| 存在確認 | `EXISTS` |

初心者はこの単数・複数を混ぜてしまう。

例えばタグが複数なのに、単にJOINする。

```sql
SELECT p.id, p.title, t.name
FROM posts p
JOIN post_tags pt ON pt.post_id = p.id
JOIN tags t ON t.id = pt.tag_id;
```

結果：

```text
id | title | name
---+-------+-------
1  | 記事A | lisp
1  | 記事A | sql
```

これは「記事が重複した」のではなく、複数のタグを行として展開しただけである。

APIで配列にしたいなら畳む。

```sql
SELECT
  p.id,
  p.title,
  json_agg(t.name ORDER BY t.name) AS tags
FROM posts p
JOIN post_tags pt ON pt.post_id = p.id
JOIN tags t ON t.id = pt.tag_id
GROUP BY p.id, p.title;
```

---

## 7. 「まず中間リストを作る」と考える

SQL初心者は最初から完成SQLを書こうとする。

これは難しい。

Common Lisp的には、いきなり完成形を書かない。

まず中間データを見る。

例：

```text
ユーザーごとの購入合計金額ランキングをAPIで返したい
```

欲しい出力：

```json
[
  {
    "user_id": 1,
    "name": "田中",
    "total_amount": 12000
  },
  {
    "user_id": 2,
    "name": "佐藤",
    "total_amount": 8500
  }
]
```

まず注文のリストがある。

```lisp
'((:user-id 1 :amount 3000)
  (:user-id 1 :amount 9000)
  (:user-id 2 :amount 8500))
```

欲しいのはこれ。

```lisp
'((:user-id 1 :total-amount 12000)
  (:user-id 2 :total-amount 8500))
```

つまり必要な処理はこれ。

```text
user_idごとにgroup
amountをsum
usersと結合してnameを付ける
total_amount順に並べる
```

SQL：

```sql
SELECT
  u.id AS user_id,
  u.name,
  SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name
ORDER BY total_amount DESC;
```

---

## 8. GROUP BYは「キーごとにreduceする」

`GROUP BY` が苦手な人は多い。

理由は、SQL文法として覚えようとするから。

Lisp的には単純。

```text
同じキーのものを集めて、reduceする
```

注文リスト：

```lisp
'((:user-id 1 :amount 3000)
  (:user-id 1 :amount 9000)
  (:user-id 2 :amount 8500))
```

これを `user-id` でまとめる。

```text
1 -> (3000 9000)
2 -> (8500)
```

それぞれ `+` で畳む。

```text
1 -> 12000
2 -> 8500
```

SQL：

```sql
SELECT
  user_id,
  SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

`COUNT` も同じ。

```sql
SELECT
  user_id,
  COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```

まとめると：

```text
GROUP BY = キーごとに袋を作る
SUM/COUNT/MAX = 袋の中身を畳む
```

---

## 9. 「最新1件を取る」もリスト操作として考える

APIでよくある要求。

```text
各ユーザーの最新注文だけ欲しい
```

Lisp的に見る。

```text
user-idごとに分ける
created-atが一番新しいものを選ぶ
```

PostgreSQLなら `DISTINCT ON` が使いやすい。

```sql
SELECT DISTINCT ON (user_id)
  user_id,
  id AS order_id,
  created_at,
  total
FROM orders
ORDER BY user_id, created_at DESC;
```

または window function を使う。

```sql
SELECT *
FROM (
  SELECT
    o.*,
    row_number() OVER (
      PARTITION BY user_id
      ORDER BY created_at DESC
    ) AS rn
  FROM orders o
) x
WHERE rn = 1;
```

Lisp的には：

```text
user_idごとに並べて、各グループの1番目を取る
```

ここでも重要なのはSQL文法ではない。

```text
「グループごとに最新を1つ選ぶ」という操作を想像できること
```

である。

---

## 10. ネストしたAPIレスポンスは「木を組み立てる」問題

例えばこういうAPI。

```json
{
  "category": {
    "id": 1,
    "name": "Programming",
    "posts": [
      {
        "id": 10,
        "title": "Lisp入門",
        "comments_count": 5
      },
      {
        "id": 11,
        "title": "SQL入門",
        "comments_count": 3
      }
    ]
  }
}
```

Lispなら：

```lisp
(:category
 :id 1
 :name "Programming"
 :posts ((:id 10 :title "Lisp入門" :comments-count 5)
         (:id 11 :title "SQL入門" :comments-count 3)))
```

ここで見える処理：

```text
category は1個
posts は複数
各postには comments_count が必要
```

まず、投稿ごとのコメント数を作る。

```sql
SELECT
  p.id,
  p.category_id,
  p.title,
  COUNT(c.id) AS comments_count
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
GROUP BY p.id, p.category_id, p.title;
```

次に、カテゴリごとにpostsを畳む。

```sql
WITH post_comment_counts AS (
  SELECT
    p.id,
    p.category_id,
    p.title,
    COUNT(cm.id) AS comments_count
  FROM posts p
  LEFT JOIN comments cm ON cm.post_id = p.id
  GROUP BY p.id, p.category_id, p.title
)
SELECT
  c.id,
  c.name,
  json_agg(
    json_build_object(
      'id', pcc.id,
      'title', pcc.title,
      'comments_count', pcc.comments_count
    )
    ORDER BY pcc.id
  ) AS posts
FROM categories c
JOIN post_comment_counts pcc ON pcc.category_id = c.id
WHERE c.id = 1
GROUP BY c.id, c.name;
```

Lispの `let` 感覚があると、CTEが自然に見える。

---

## 11. 複雑なSQLは小さな変換に分ける

初心者は複雑なSQLを見るとこうなる。

```text
SELECTが長い
JOINが多い
GROUP BYがある
json_aggがある
もう分からない
```

Common Lispでリストを扱う人は、こう見る。

```text
このSQLは何段階の変換か？
中間データは何か？
どの段階で配列に畳んでいるか？
どの段階で親に戻しているか？
```

複雑なSQLは、だいたい次の組み合わせである。

```text
1. 平らにする
2. 絞る
3. 並べる
4. グループ化する
5. 集約する
6. 子リストを作る
7. 親に埋め込む
8. JSONにする
```

だからSQLでも、

```text
まずJOIN結果を見よう
次にGROUP BYしよう
次にjson_aggしよう
最後にAPI形状にしよう
```

と段階的に考えられる。

---

## 12. SQLを組み立てる手順

SQL初心者には、毎回この手順で考えさせるとよい。

### 手順1: 欲しい出力をリストで書く

```lisp
(:id user-id
 :name user-name
 :orders ((:id order-id :total total)
          ...))
```

### 手順2: 単数と複数を分ける

```text
user = 1個
orders = 複数
```

### 手順3: 複数の部分は集約が必要と判断する

```text
orders は json_agg が必要
```

### 手順4: 材料テーブルを列挙する

```text
users
orders
```

### 手順5: 親子関係を確認する

```text
orders.user_id = users.id
```

### 手順6: まず平らなSQLを書く

```sql
SELECT
  u.id,
  u.name,
  o.id,
  o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

### 手順7: 親単位に畳む

```sql
SELECT
  u.id,
  u.name,
  json_agg(
    json_build_object(
      'id', o.id,
      'total', o.total
    )
  ) AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

---

## 13. Lispの操作とSQLの対応

| Lispの感覚 | SQLの感覚 |
|---|---|
| `mapcar` | `SELECT` |
| `remove-if-not` | `WHERE` |
| `reduce` | `SUM` / `COUNT` / `MAX` |
| `group-by` | `GROUP BY` |
| `sort` | `ORDER BY` |
| `walk` | ネストJSON/API構造の探索 |
| `let` | CTE / 中間結果の命名 |

---

## 14. まとめ

Common Lispのリスト思考がSQLに効く理由は、SQL文法を覚えやすくなるからではない。

より本質的には、

```text
欲しい出力をリストや木として想像できるようになる
```

からである。

API用SQLで詰まる初心者に足りないのは、多くの場合これである。

```text
どのテーブルを使うか
どこでJOINするか
どこで行が増えるか
どこでGROUP BYするか
どこでjson_aggするか
どこで親子構造に戻すか
```

Common Lispでリストを扱っていると、これらを次のように考えられる。

```text
リストを見る
木を見る
入れ子を見る
親子関係を見る
単数と複数を分ける
平らにする
畳む
必要な形に作り直す
```

そしてAPI用SQLは、ほぼこの作業である。

```text
RDBの平らな表
→ JOINで材料を集める
→ GROUP BYで親単位に戻す
→ json_aggで子配列を作る
→ json_build_objectでAPIの形にする
```

つまり、Common Lispで日頃からリストで考えることは、SQL初心者が一番躓きやすい、

```text
欲しい出力から逆算して、複雑なSQLをどこから組み立てればいいのか分からない問題
```

に直接効く。

Lisp脳はSQLを上手く書くための文法知識そのものではない。

しかし、

```text
データの形を頭の中で変形する力
```

を鍛える。

この力があると、SQL、API設計、Query Builder、JSON整形、画面表示用データ生成まで、アプリ開発全体の見通しが良くなる。
