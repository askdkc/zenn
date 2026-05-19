---
title: JSONは一行の文字列ではなく、入れ子構造である
---
ログ調査の翌日、開発中の社内アプリで別の問題が起きた。

営業画面では顧客名が表示されるのに、注文金額だけが空欄になる。画面側のバグなのか、APIが変な値を返しているのか、DBに値がないのか分からない。

黒瀬が言った。

「APIの返事を見よう。」

高村がブラウザの開発者ツールを開き、レスポンスをコピーした。

ミナはEmacsの左バッファに貼り付けた。

「JSONですね。まず整形します。読めないJSONは、読む前に負けです。」

整形後のレスポンスはこうだった。

```json
{
  "customer": {
    "id": 120,
    "name": "Yamada",
    "tags": ["vip", "invoice"]
  },
  "orders": [
    {
      "id": "A-1029",
      "amount": 12000,
      "items": [
        {"sku": "P-01", "name": "Printer"},
        {"sku": "I-09", "name": "Ink"}
      ]
    }
  ]
}
```

「JSONは文字列として届きます。でも、読むと構造になります。特に入れ子が重要です。」

## 9.1 JSONの形を図にする

```text
root
  ├─ customer
  │    ├─ id
  │    ├─ name
  │    └─ tags
  │         ├─ 0: vip
  │         └─ 1: invoice
  │
  └─ orders
       └─ 0
          ├─ id
          ├─ amount
          └─ items
               ├─ 0
               │   ├─ sku
               │   └─ name
               └─ 1
                   ├─ sku
                   └─ name
```

高村は図を見て言った。

「でも、Lispに読ませると、この木はどうなるの？」

「JSONのobjectは、名前で値を引ける辞書みたいなものになります。Common Lispではhash-tableとして扱うことが多いです。」

```text
{"name": "Yamada"}
  ↓
hash-table
  "name" -> "Yamada"
```

「配列は？」

「配列は、番号で取り出せる箱の列です。Lisp側ではvectorになることが多いです。」

```text
["vip", "invoice"]
  ↓
vector
  0 -> "vip"
  1 -> "invoice"
```

「つまり、JSONを読むには、名前で引く場所と、番号で引く場所が混ざるんだね。」

「そうです。だからpathで辿る関数が欲しくなります。」

ミナはそこで一つ注釈を入れた。

「ここでは、JSON object のキーが文字列として入るパーサーを使っている前提です。たとえば `"customer"` というキーを文字列で取り出す想定です。」

``` text
この章では、JSON文字列はすでにCommon Lispの hash-table / vector に変換済みとします。
実際のparseには com.inuoe.jzon などのJSONライブラリを使います。

;; 例: com.inuoe.jzon を使う場合
;; (ql:quickload :com.inuoe.jzon)
;; (defparameter json (com.inuoe.jzon:parse json-string))
```

高村が聞いた。

「ライブラリによって違うの？」

「違います。JSONライブラリによっては、キーを `:customer` みたいなキーワードに変換するものもあります。その場合は、`(jget json :customer :name)` のように、pathの書き方も変わります。」

```text
この章の前提:
  JSON objectのキーは文字列
  例: "customer", "orders", "amount"

別ライブラリの可能性:
  キーがキーワードになる
  例: :customer, :orders, :amount
```

「つまり、`jget` の考え方は同じでも、キーの形は使うJSONライブラリに合わせる必要があります。」



## 9.2 `jget`で入れ子を辿る

高村はコードを見て聞いた。

「これ、Emacsで動くの？」

「Emacs Lispじゃなくて、Common Lispです。EmacsではSLIMEを使って、SBCLに式を送って動かします。」

高村が聞いた。

「また知らない名前が出てきた。SLIMEとSBCLって何？」

「SBCLはCommon Lispを実行する処理系です。つまりLispのエンジンです。SLIMEは、EmacsからそのLispエンジンへコードを送って、結果を受け取るための橋です。」

```lisp
(defun hget (key table &optional default)
  (multiple-value-bind (value foundp)
      (gethash key table)
    (if foundp value default)))

(defun jget (object &rest path)
  (reduce
   (lambda (current key)
     (cond
       ((null current) nil)
       ((hash-table-p current) (hget key current))
       ((and (vectorp current) (integerp key))
        (when (< -1 key (length current))
          (aref current key)))
       (t nil)))
   path
   :initial-value object))
```

## 記号の読み方

```text
gethash
  hash-tableから値を取り出す。

multiple-value-bind
  gethash が返す「値」と「見つかったかどうか」を別々の名前で受け取る。

&optional default
  値がなかったときの予備の値を指定できる。

&rest path
  残りの引数をまとめて path として受け取る。
  例: "orders" 0 "amount"

reduce
  path を左から順番に処理して、JSONの奥へ進む。

hash-table-p
  今見ているものが hash-table か調べる。

vectorp
  今見ているものが vector か調べる。

aref
  vector の何番目かを取り出す。
```

この関数は、JSONの中を「道順」に沿って歩くための道具である。

## 入力例

```lisp
(jget json "customer" "name")
```

## 出力例

```lisp
"Yamada"
```

## 入力例2

```lisp
(jget json "orders" 0 "items" 1 "sku")
```

## 出力例2

```lisp
"I-09"
```

## ミナのメモ

ミナは右下のメモバッファに、ここまでの変換を整理した。

```text
現実:
  APIレスポンスが大きな入れ子JSONで、人間が必要な値を見つけにくい。

抽象化:
  JSON文字列
    ↓ parse
  hash-table / vector
    ↓ jget
  必要な値

次へ:
  APIレスポンスを、人間が読める報告へ戻す。
```

---
