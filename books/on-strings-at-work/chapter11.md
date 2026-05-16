---
title: APIレスポンスを人間向けレポートへ戻す
---
APIレスポンスを解析できても、人間が読めなければ仕事にはならない。

ミナは、JSONから注文レポートを作った。

障害対応では、JSONをそのまま貼っても営業部には伝わらない。必要なのは「誰の注文で、いくらで、何が欠けているのか」が一目で分かる報告だ。

「APIの返事を、人間が判断できる形に戻します。」

```lisp
(defun customer-name (json)
  (jget json "customer" "name"))

(defun first-order-amount (json)
  (jget json "orders" 0 "amount"))

(defun print-order-report (json)
  (format t "顧客: ~A~%" (customer-name json))
  (format t "初回注文金額: ~A~%" (first-order-amount json))
  (format t "タグ: ~{~A~^, ~}~%"
          (ensure-list (jget json "customer" "tags"))))
```

## 記号の読み方

```text
format
  人間向けの文字列を作って表示する。

t
  出力先。ここでは画面に出すという意味。

~A
  値をそこに埋め込む。

~%
  改行する。

~{~A~^, ~}
  リストの中身を順番に表示し、間にカンマを入れる。
  今は「タグをカンマ区切りで表示する」くらいに読めばよい。
```

## 出力例

```text
顧客: Yamada
初回注文金額: 12000
タグ: vip, invoice
```

## 失敗例

```json
{
  "customer": {"name": "Yamada"},
  "orders": []
}
```

この場合、初回注文金額が `nil` になる。

## リファクタリング

```lisp
(defun show-value-or-missing (label value)
  (format t "~A: ~A~%" label (or value "[missing]")))

(defun print-order-report* (json)
  (show-value-or-missing "顧客" (customer-name json))
  (show-value-or-missing "初回注文金額" (first-order-amount json))
  (show-value-or-missing "タグ" (ensure-list (jget json "customer" "tags"))))
```

## 出力例

```text
顧客: Yamada
初回注文金額: [missing]
タグ: NIL
```

この報告によって、画面側の表示バグではなく、APIレスポンス側で注文金額が欠けていることが分かった。

## ミナのメモ

ミナは右下のメモバッファに、ここまでの変換を整理した。

```text
APIレスポンス
  ↓ jget
必要な値
  ↓ format
人間が読めるレポート
```

---

