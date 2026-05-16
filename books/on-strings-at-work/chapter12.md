---
title: 小さな関数を育てる――リファクタリングの章
---
ミナのコードは増えてきた。

最初は短い式だった。

```lisp
(string-trim " " value)
(string-downcase key)
(remove #\, amount)
(jget json "orders" 0 "amount")
```

だが、同じ処理が何度も出てきた。

ミナは言った。

「三回出たら、名前をつけます。」

## 11.1 正規化ユーティリティ

```lisp
(defun normalize-token (s)
  (string-downcase
   (string-trim " " s)))
```

## 入力例

```lisp
(normalize-token " CONFIRMED ")
```

## 出力例

```lisp
"confirmed"
```

## 失敗例

```lisp
(normalize-token nil)
;; エラー
```

## リファクタリング

```lisp
(defun normalize-token* (s)
  (if (stringp s)
      (string-downcase (string-trim " " s))
      ""))
```

昼過ぎ、高村がミナの画面を見て言った。

「その `normalize-name-space`、また出てきたな。」

ミナは名簿のバッファからCSVのバッファへ視線を移した。

「名前の汚れは、部署が変わってもだいたい同じなんです。社員名簿でも、営業CSVでも、DB投入前でも出ます。」

「だから、いちいちその場で書き直さずに、関数として残す。」

「はい。こういう小さい道具が増えると、次の地獄が少しだけ楽になります。」

## 11.2 哲学

リファクタリングは、短くするためだけではない。

```text
重複した処理に名前をつける
  ↓
意味が見える
  ↓
間違いを一箇所で直せる
  ↓
次の問題でも使える
```

ミナは高村に言った。

「こういう小さい関数は、ちゃんと名前をつけて残すと長生きします。Common LispやEmacs Lispは、流行り廃りで毎年書き換えるタイプの道具じゃないです。二十年後でも、かなりの確率でそのまま読めて動きます。」

「二十年？」

「業務コードではそれ、めちゃくちゃ価値あります。担当者が異動しても、退職しても、`normalize-name-space` とか `jget` みたいな小さい関数が残っていれば、何をしていたか追えます。人が消えても、処理の考え方が残るんです。」

---

