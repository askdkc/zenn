---
title: "Emacs初心者が利用1ヶ月でパッケージにコントリビュートした話"
emoji: "🦄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [emacs,lisp,elisp]
published: false
---
# Emacs利用1ヶ月でパッケージにコントリビュートした話
宗教として知られるVim、Emacsの宗派において、Emacs初心者が何故お手軽にパッケージの一つにコントリビュート出来たか？のお話

## きっかけ：パッケージがmacOSで動かん🤦‍♂️
Emacsには様々なメモやToDOを作るのに便利なOrgmodeなるモードがある🦄

このOrgmode用パッケージに`org-web-tools`なるものがあり、これを使うと`org-web-tools-insert-link-for-url`コマンドでURLのリンクをタイトル付きで手軽にメモれる

## 問題点：macosのEmacsだと動かん
Linux上では無問題に動作するこのパッケージだが、何故かmacOSだと`curl`がDNSエラーを吐いて死ぬ💀

## 解決に至るまで

### ソースを見る
問題があるならソースを見てみようということで、GitHubにソースを見に行く（むしろこれはダウンロードされてるパッケージのディレクトリの中身でも良い）🔍

はい、それがこちら：
```elisp
(defun org-web-tools-insert-link-for-url (url)
  "Insert Org link to URL using title of HTML page at URL.
If URL is not given, look for first URL in `kill-ring'."
  (interactive (list (org-web-tools--get-first-url)))
  (insert (org-web-tools--org-link-for-url url)))
```

`curl`は`org-web-tools--get-first-url`から呼ばれてるっぽいので、そっちを見に行く🏃‍♀️💨

はい、それがこちら：
```elisp
(defun org-web-tools--get-first-url ()
  "Return URL in clipboard, or first URL in the `kill-ring', or nil if none."
  (cl-loop for item in (append (list (gui-get-selection 'CLIPBOARD))
                               kill-ring)
           when (and item (string-match (rx bol "http" (optional "s") "://") item))
           return item))
```

### 動かしてみる
Emacsはテキストエディタ兼Lisp実行環境でもあるので、Lisp(Elisp)をそのまま実行できる👍

なので、スクラッチバッファを開き、上記のコードを貼り、下記のようにして読み込ませる：

```elisp
(defun org-web-tools-insert-link-for-url (url)
  "Insert Org link to URL using title of HTML page at URL.
If URL is not given, look for first URL in `kill-ring'."
  (interactive (list (org-web-tools--get-first-url)))
  (insert (org-web-tools--org-link-for-url url))) ← ここにカーソルを置いて C-x C-e

(defun org-web-tools--get-first-url ()
  "Return URL in clipboard, or first URL in the `kill-ring', or nil if none."
  (cl-loop for item in (append (list (gui-get-selection 'CLIPBOARD))
                               kill-ring)
           when (and item (string-match (rx bol "http" (optional "s") "://") item))
           return item)) ← ここにカーソルを置いて C-x C-e
```

そして下記のようにして実行
```elisp
(org-web-tools--get-first-url "https://zenn.dev/zenn/articles/zenn-cli-guide") ← ここにカーソルを置いて C-x C-e
```

結果
```
[[https://zenn.dev/zenn/articles/zenn-cli-guide][Zenn CLIで記事・本を管理する方法]]
```

動きます👍

### 問題点を探す
`org-web-tools-insert-link-for-url`は動きそうなので、こやつにURLを渡す`org-web-tools--get-first-url`を見てみます

最初にクリップボードのデータを取得してる箇所を試します

スクラッチバッファに下記を書きます：

```elisp
(gui-get-selection 'CLIPBOARD)
```

macOSでブラウザから適当なURLをコピーし、Emacsで記載した処理を実行します

```elisp
(gui-get-selection 'CLIPBOARD) ←ここでC-x C-e
```

結果：`nil`

**いや、nilじゃねーよ** 👀⁉️

## 犯人は`(gui-get-selection 'CLIPBOARD)`

なんということでしょう、Emacsの`(gui-get-selection 'CLIPBOARD)`はmacOS側のクリップボードデータを取ることが出来ないのでした😨

## 修正とPR

はい修正push、あとよろ〜

https://github.com/alphapapa/org-web-tools/pull/66

## まとめ

こうしてEmacs初心者の自分は、Emacsの便利機能に助けられ、Emacs初心者でもElispのパッケージにコントリビュート出来たのでした

Happy Hacking!!

