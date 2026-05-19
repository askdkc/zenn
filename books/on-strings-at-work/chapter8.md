---
title: CSVは表ではなく、区切られた文字列である
---
Power Automateで集まったExcelをCSVとして出力すると、次のようなファイルになった。

```csv
id,name,amount,note
001,山田 太郎,12000,急ぎ
002,佐藤 花子,12,000円,確認
003,鈴木 一郎,"18,000",カンマあり
```

「CSV形式に近いから簡単でしょ。」

ミナはその言葉を思い出しながら、二行目と三行目を指した。

「見た目は表っぽいです。でもCSVの実体は、区切り文字を持つ文字列です。しかも、引用符やカンマを含む値があります。そこを雑に扱うと壊れます。」

「二行目は？」

「`12,000円` が引用符で囲まれていないので、CSVとしては列が増えます。本来、カンマを含む値は `"12,000円"` のように引用符で囲む必要があります。」

## 7.1 単純なsplitの危険

高村は、CSVを見ながら言った。

「カンマで区切ればいいんじゃない？」

ミナは、にこっと笑ってから、わざと一行を指した。

```csv
003,鈴木 一郎,"18,000",カンマあり
```

「じゃあ、カンマで単純に分割しますよね？ するとこうなります。」

```lisp
("003" "鈴木 一郎" "\"18" "000\"" "カンマあり")
```

高村は固まった。

「金額が割れた。」

「そうです。`18,000` のカンマまで区切り扱いになりました。これ、CSVとしてデータに使えない形ですよね。ダルくないですかぁ？」

「ダルい。」

「なので、CSVは見た目より面倒です。」

```lisp
;; これはCSV用の正しい分割関数ではありません。
;; 単純splitが壊れることを見るための学習用関数です。
(defun split-on-char (char string)
  (let ((result '())
        (start 0))
    (loop for pos = (position char string :start start)
          do (push (subseq string start pos) result)
          while pos
          do (setf start (1+ pos)))
    (nreverse result)))

(defun naive-split-comma (line)
  (split-on-char #\, line))
```

## 何をしているか

カンマで単純に分割する。
学習用の悪い例である。

## 入力例

```lisp
(naive-split-comma "001,山田 太郎,12000,急ぎ")
```

## 出力例

```lisp
("001" "山田 太郎" "12000" "急ぎ")
```

## 失敗例

```lisp
(naive-split-comma "003,鈴木 一郎,\"18,000\",カンマあり")
```

本当はこう欲しい。

```lisp
("003" "鈴木 一郎" "18,000" "カンマあり")
```

だが、単純splitでは金額の中のカンマまで割ってしまう。

## リファクタリング

実務ではCSVパーサーを使う。
自作splitで頑張るのは、形式が本当に単純だと確認できる場合だけである。

（ここではCSVパーサーの実装には踏み込みません。
実務では cl-csv などのライブラリを使います。
この章の `naive-split-comma` は、単純splitが壊れることを見るための悪い例です。）

## 7.2 数値変換と失敗

次に、金額欄を整数として扱いたい。

ただし、ここでミナは一つ注意を入れた。

「`nil` だけを返す関数にすると、『値がない』のか『変換に失敗した』のか分かりにくくなります。あと、金額が `0` の時に、うっかりエラー扱いする事故も起こりがちです。」

高村が頷いた。

「0円はあり得るけど、偽っぽく見えるから危ないわけか。」

「そうです。だから、成功したかどうかと、値そのものを分けて返します。」

```lisp
(defun clean-integer-string (s)
  (remove #\, (string-trim " " s)))

(defun parse-integer-result (s)
  (let ((cleaned (clean-integer-string s)))
    (handler-case
        (list :ok t
              :value (parse-integer cleaned)
              :raw s
              :cleaned cleaned)
      (error ()
        (list :ok nil
              :error "not an integer"
              :raw s
              :cleaned cleaned)))))

(defun parse-amount-field (s)
  (parse-integer-result s))
```

## 記号の読み方

```text
clean-integer-string
  前後の空白を削り、カンマを取り除く。
  例: "18,000" -> "18000"

parse-integer
  文字列を整数に変える。
  例: "18000" -> 18000

handler-case
  エラーが起きたときに、処理全体を止めずに受け止める。

:ok
  変換に成功したかどうか。

:value
  成功した時の整数値。

:error
  失敗した時の理由。

:raw
  元の文字列。

:cleaned
  変換前に掃除した文字列。
```

## 入力例

```lisp
(parse-amount-field "18,000")
```

## 出力例

```lisp
(:OK T :VALUE 18000 :RAW "18,000" :CLEANED "18000")
```

## 0円の例

```lisp
(parse-amount-field "0")
```

## 出力例

```lisp
(:OK T :VALUE 0 :RAW "0" :CLEANED "0")
```

`0` は正しい値であり、エラーではない。

## 失敗例

```lisp
(parse-amount-field "12,000円")
```

## 出力例

```lisp
(:OK NIL
 :ERROR "not an integer"
 :RAW "12,000円"
 :CLEANED "12000円")
```

`円` が残っているため、整数として読めない。

## リファクタリング

`nil` だけで成功・失敗を表すのではなく、`:ok`、`:value`、`:error` を持つ結果にする。  
これで、「値がない」と「変換に失敗した」と「0という正しい値」を区別できる。

## ミナのメモ

ミナは右下のメモバッファに、ここまでの変換を整理した。

```text
現実:
  CSVが表に見えるが、実際には区切り文字を持つ文字列である。

問題:
  カンマを含む値
  引用符
  円付き金額
  数値変換失敗

処理:
  まず壊れる例を見る。
  その後、専用パーサーや検査へ進む。

次へ:
  壊れた値は捨てず、エラー情報として残す。
```

---
