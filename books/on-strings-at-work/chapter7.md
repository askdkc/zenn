---
title: メール本文を項目のリストへほどく
---
前章で必要な行は残った。次は、その行を項目名と値に分ける。

```text
申請番号: A-1029
氏名: 山田 太郎
部署: 営業一課
金額: 12000
希望日: 2024-04-01
```

目標はこれである。

```lisp
(:request-id "A-1029"
 :name "山田 太郎"
 :department "営業一課"
 :amount 12000
 :date "2024-04-01")
```

高村は首を傾げた。

「この `:request-id` とか `:name` って何？」

「plistです。property listの略です。名前と値を交互に並べたメモだと思ってください。」

ミナは紙に書いた。

```text
:request-id  -> "A-1029"
:name        -> "山田 太郎"
:department  -> "営業一課"
:amount      -> 12000
:date        -> "2024-04-01"
```

「メールの行は、最初はただの文字列です。でも、項目名と値に分けると、こういう名前付きのメモにできます。ここまで来ると、Excelの列にも、JSONにも、DBの行にもつなげやすくなります。」

## 行を名前と値に分ける

ミナはメール本文を見ながら、まず一行だけを指でなぞった。

```text
金額: 12000
```

「この一行は、左が項目名で、右が値です。人間は一瞬で分かる。でも、プログラムには境目を教えないと分かりません。」

境目は `:` だ。だから、まず `:` の位置を探し、左と右に切る。

```lisp
(defun split-pair (line)
  (let ((pos (position #\: line)))
    (when pos
      (list (subseq line 0 pos)
            (subseq line (1+ pos))))))
```

## 記号の読み方

```text
defun
  関数を作る。

line
  処理したい一行の文字列。

position
  指定した文字が、文字列の何番目にあるかを探す。

#\:
  コロン文字そのものを表す書き方。

subseq
  文字列の一部を切り出す。

0 pos
  先頭からコロンの直前まで。

(1+ pos)
  コロンの次の位置。
```

つまり、この関数は「コロンの左側」と「コロンの右側」に一行を分ける。

## 入力例

```lisp
(split-pair "金額: 12000")
```

## 出力例

```lisp
("金額" " 12000")
```

## 失敗例

```lisp
(split-pair "金額 12000")
;; => NIL
```

`:` がないため、名前と値に分けられない。

## リファクタリング

全角コロン `：` も扱う。

```lisp
(defun normalize-colon (line)
  (replace-char #\： #\: line))

(defun split-pair* (line)
  (split-pair (normalize-colon line)))
```

## 入力例

```lisp
(split-pair* "金額： 12000")
```

## 出力例

```lisp
("金額" " 12000")
```

## 項目名を内部キーへ変える

```lisp
(defparameter *field-map*
  '(("申請番号" . :request-id)
    ("氏名" . :name)
    ("名前" . :name)
    ("担当" . :name)
    ("部署" . :department)
    ("部門" . :department)
    ("所属" . :department)
    ("金額" . :amount)
    ("申請金額" . :amount)
    ("希望日" . :date)
    ("納期希望" . :date)))

(defun field-key (label)
  (cdr (assoc (string-trim " " label) *field-map* :test #'string=)))
```

## 記号の読み方

```text
defparameter
  後で使う変数を作る。

*field-map*
  項目名の対応表。
  Common Lispでは、慣習としてグローバル変数を *...* で囲むことが多い。

("氏名" . :name)
  "氏名" という文字を、内部では :name として扱うという対応。

assoc
  対応表の中から、左側が一致するものを探す。

cdr
  見つかった対応の右側を取り出す。

:test #'string=
  文字列を比べるときは string= を使う、という指定。
```

## 何をしているか

メール内の表記ゆれを、内部の統一キーへ変換する。

```text
氏名     -> :name
名前     -> :name
担当     -> :name
部署     -> :department
部門     -> :department
所属     -> :department
金額     -> :amount
申請金額 -> :amount
```

## 入力例

```lisp
(field-key "申請金額")
```

## 出力例

```lisp
:AMOUNT
```

## 失敗例

```lisp
(field-key "費用")
;; => NIL
```

未知のラベルは変換できない。

## ミナのメモ

ミナは右下のメモバッファに、ここまでの変換を整理した。

```text
現実:
  メール本文の中で、同じ意味の項目が別名で書かれている。

処理:
  1行を「項目名」と「値」に分ける。
  表記ゆれを内部キーへ寄せる。

抽象化:
  "金額: 12000"
    ↓
  ("金額" "12000")
    ↓
  (:amount 12000)

使った道具:
  split-pair
  field-key

次へ:
  複数行をまとめると、1件分の申請データになる。
```

高村は言った。

「ここまで分けられたら、CSV形式に近いから簡単でしょ。」

ミナは少しだけ嫌な予感がした。

---

