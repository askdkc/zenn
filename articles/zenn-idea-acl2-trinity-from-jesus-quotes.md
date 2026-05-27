---
title: "ACL2で「矛盾しそうな主張」が本当に矛盾するか確かめる"
emoji: "🧪"
type: "idea"
topics: ["acl2", "lisp", "commonlisp"]
published: true
---

## ACL2とは何か

ACL2 は、Lisp に似た形式で定義を書き、その定義から導かれる性質を機械的に証明するためのシステムである。

名前は `A Computational Logic for Applicative Common Lisp` の略であり、ACL2 は単なるプログラミング言語ではない。関数やデータを記述するための言語であると同時に、それらについて推論するための論理体系と、自動的に証明を試みる定理証明器を備えている。

通常のプログラムでは、いくつかの入力例を与えて期待どおりに動くことをテストする。しかし、テストで確認できるのは、実際に試した例に限られる。

これに対して ACL2 では、「この定義のもとでは、この条件が常に成立する」「この複数の条件は同時に成立可能である」「この前提を置くと矛盾が生じる」といった命題を、論理式として記述し、証明の対象にできる。

もちろん、ACL2 が証明できるのは、入力された定義と前提から論理的に導けることだけである。聖書の解釈が正しいか、ある教理が信仰上真実であるかを、ACL2 自体が決定するわけではない。

それでも、ある主張を明示的な条件に分解し、それらが通常の論理のもとで両立するかを検証する道具として、ACL2 は有効である。

## これは何か

この記事は、ACL2 の面白さを伝えるために、やや扱いにくい題材をあえて形式化してみる試みである。

例として使うのは、福音書に記録されたイエス自身の発言から抽出した制約である。父・イエス・聖霊についての発言群が、通常の同一性のもとで同時に成立できるかを ACL2 で確認する。

実際のソースコードと証明は、次の GitHub リポジトリで公開している。

https://github.com/askdkc/acl2-trinity-consistency-proof

この題材を選んだ理由は、自然言語のままだと「矛盾している」「いや矛盾していない」という議論になりやすいからである。ACL2 を使うと、どの主張を前提にしたのか、どこで `equal` を使ったのか、どの関係を別の述語として置いたのかが明示される。

ここでやっていないことは明確である。後世の教理語としての「三位一体」を公理にしない。イエスが「唯一の、まことの神」であることも、聖霊が「唯一の、まことの神」であることも、直接の前提としては置かない。

代わりに、福音書でイエスが父・子・聖霊について語った内容を、関係制約として形式化する。

## なぜこの題材を使うのか

ACL2 の面白さは、抽象的なロジックや小さなプログラムだけでなく、自然言語で語られる複雑な主張も、ある程度まで形式的な制約として分解できる点にある。

三位一体をめぐる議論は、その例としてちょうどよい。キリスト教において重要でありながら、誤解されやすく、しかも「同一性」「区別」「一つであること」といった論理的に扱いやすい論点を含んでいる。

争点は単純に「神は一人か、三人か」という話ではない。福音書には、イエスが父を「唯一の、まことの神」と呼ぶ発言がある一方で、イエス自身が父と一つであると語り、父と同様に敬われるべき存在として語られ、命を与える権威を持つ者として描かれる箇所がある。また、聖霊についても、父およびイエスと区別されながら、弟子たちに働き、導き、語る存在として語られている。

そのため、問題は次のようになる。

- 神が唯一であるという発言を維持できるか。
- 父とイエスを同一対象にせず、区別したまま扱えるか。
- それでも、イエスについて語られる特別な権威や父との一致を同時に維持できるか。
- 聖霊についての発言も、父やイエスと単純に同一視せずに組み込めるか。

後世の教会は、これらの問題を整理するために「三位一体」「本質」「位格」「同質」といった教理語を用いてきた。しかし、その語彙を最初から前提にすると、検証結果は「三位一体を公理として置けば三位一体が成立する」という循環的なものになりかねない。

そこでこの記事では、後世の教理定式を直接の前提には置かず、ACL2 で扱える制約の形に落とす。

## この例では何を検証するのか

本検証では、福音書に記録されたイエス自身の発言だけを対象とし、父・イエス・聖霊について語られた内容を ACL2 上の制約として形式化する。

ここで直接の前提として置かないものは、次のとおりである。

- 「三位一体」という後世の教理語そのもの。

- 「イエスは唯一の、まことの神である」という命題。

- 「聖霊は唯一の、まことの神である」という命題。

- 父・イエス・聖霊を単純に同一の存在として扱う定義。

代わりに、イエスの発言から読み取れる関係だけを制約として与える。

たとえば、父が唯一のまことの神として語られること、父とイエスが区別されること、イエスが父と一つであると語ること、イエスが父と同様に敬われること、聖霊が父およびイエスに関係しながら弟子たちに働くこと、などである。

検証したい命題は、次の一点に集約される。

> 福音書に記録された父・イエス・聖霊についてのイエスの発言群は、三者を相互に `equal` とせず、通常の同一性を維持したまま、同時に成立可能であるか。

この例が示そうとするのは、三位一体の教理全体を ACL2 によって証明することではない。

示そうとするのは、より限定された、しかし重要な点である。すなわち、イエス自身の発言から抽出した条件群は、「父・イエス・聖霊をすべて同一対象としなければ矛盾する」というものではなく、三者の区別を保ったまま同時に成立しうるか、という論理的一貫性の確認である。

三位一体を巡る議論は、しばしば後世の神学用語や教派的立場の対立から始まる。ここでは、いったんそこから離れ、福音書に記録されたイエス自身の発言を出発点として、そこに含まれる制約の組合せを機械的に検証する。

## モデルの範囲

ACL2 ファイルでは、扱う対象を名前付きの記号として置いている。

```lisp
(defun gospel-agent-p (x)
  (if (member-equal x '(father jesus holy-spirit))
      t
      nil))

(defun disciple-p (x)
  (if (member-equal x '(disciple-a disciple-b))
      t
      nil))
```

父・イエス・聖霊は、最初から相互に区別される。

```lisp
(defthm named-agents-are-distinct
    (and
     (not (equal 'father 'jesus))
     (not (equal 'father 'holy-spirit))
     (not (equal 'jesus 'holy-spirit)))
  :rule-classes nil)
```

この時点で、モデルは「父 = イエス = 聖霊」という形の同一化を採用していない。

## イエスの発言から抽出した制約

詳細な対応表では、それぞれの聖句と ACL2 上の制約を対応させている。この記事では要点だけを挙げる。

マルコ 12:29 では、イエスは神が一つであることを語る。

```lisp
(confesses-one-god 'jesus)
```

ヨハネ 17:3 では、イエスは父を「唯一の、まことの神」と呼び、自分を父から遣わされた者として語る。

```lisp
(only-true-god 'father)
(sends 'father 'jesus)
```

ヨハネ 10:30 では、イエスは父と自分が「一つ」であると語る。

```lisp
(one-with 'jesus 'father)
```

ただし、ヨハネ 17:21-22 では、互いに異なる弟子たちについても「一つ」となることが語られる。そのため、このモデルでは `one-with` を ACL2 の `equal` とは別の関係として扱う。

```lisp
(one-with 'disciple-a 'disciple-b)
(not (equal 'disciple-a 'disciple-b))
```

ヨハネ 5:21-23 からは、父とイエスが命を与えること、またイエスが父と同様に敬われることを制約にする。

```lisp
(gives-life 'father)
(gives-life 'jesus)
(honor-as 'jesus 'father)
```

ヨハネ 17:5 からは、イエスが世界以前に父のそばで栄光を持っていた、という関係を置く。

```lisp
(has-precreation-glory-with 'jesus 'father)
```

聖霊については、ヨハネ 14-16 章の発言を関係として入れる。父はイエスの名によって聖霊を遣わし、イエスも父のみもとから聖霊を遣わす。聖霊は弟子たちのうちにおり、教え、導き、イエスについて証言し、イエスに栄光を得させる。

```lisp
(sends-in-name-of 'father 'holy-spirit 'jesus)
(sends-from 'jesus 'holy-spirit 'father)

(indwells 'holy-spirit 'disciple-a)
(indwells 'holy-spirit 'disciple-b)

(teaches 'holy-spirit 'disciple-a)
(teaches 'holy-spirit 'disciple-b)

(guides-into-truth 'holy-spirit 'disciple-a)
(guides-into-truth 'holy-spirit 'disciple-b)

(testifies-about 'holy-spirit 'jesus)
(glorifies 'holy-spirit 'jesus)
```

ここで重要なのは、複数なのは弟子の側であり、聖霊そのものを複数にしていない点である。`holy-spirit` という同じ対象が、異なる弟子に対して働く。

さらに、マタイ 28:19 の洗礼命令では、父・子・聖霊が一つの「名」の下に置かれる。

```lisp
(baptize-in-one-name 'father 'jesus 'holy-spirit)
```

## witness model としての encapsulate

ACL2 ファイルでは、これらの関係を `encapsulate` の中で抽象関数として宣言し、ローカルな witness 実装を与えている。

```lisp
(encapsulate
 (((confesses-one-god *) => *)
  ((only-true-god *) => *)
  ((sends * *) => *)
  ((one-with * *) => *)
  ((gives-life *) => *)
  ...)

 ;; local witness implementation
 ...)
```

ローカル定義は、たとえば次のような形である。

```lisp
(local
 (defun one-with (x y)
   (or
    (and (equal x 'jesus)
         (equal y 'father))
    (and (equal x 'father)
         (equal y 'jesus))
    (and (equal x 'disciple-a)
         (equal y 'disciple-b))
    (and (equal x 'disciple-b)
         (equal y 'disciple-a)))))
```

これは「一つ」という言葉を数的同一性としてではなく、独立した関係として解釈した witness である。イエスと父が `one-with` でありながら `equal` ではない。弟子 A と弟子 B も `one-with` でありながら `equal` ではない。

この witness が存在するなら、少なくとも抽出された制約群は一階述語論理的に即矛盾ではない。

## ACL2 が確認する主な定理

最終的な定理の一つは、父・イエス・聖霊が相互に区別されたまま、発言から抽出した制約が共存することを述べる。

```lisp
(defthm sayings-coexist-with-distinct-agents
    (and
     (not (equal 'father 'jesus))
     (not (equal 'father 'holy-spirit))
     (not (equal 'jesus 'holy-spirit))

     (confesses-one-god 'jesus)
     (only-true-god 'father)
     (sends 'father 'jesus)

     (one-with 'jesus 'father)

     (gives-life 'father)
     (gives-life 'jesus)
     (honor-as 'jesus 'father)

     (has-precreation-glory-with 'jesus 'father)

     (asks-father-for-helper 'jesus 'father 'holy-spirit)
     (another-helper 'holy-spirit)
     (sends-in-name-of 'father 'holy-spirit 'jesus)
     (sends-from 'jesus 'holy-spirit 'father)

     (testifies-about 'holy-spirit 'jesus)
     (glorifies 'holy-spirit 'jesus)
     (has-all-of 'jesus 'father)

     (baptize-in-one-name 'father 'jesus 'holy-spirit))
  :hints
  (("Goal"
    :use
    (named-agents-are-distinct
     recorded-sayings-constraints)))
  :rule-classes nil)
```

また、「一つ」が `equal` を強制しないことも明示的に確認する。

```lisp
(defthm one-with-does-not-force-equal
    (and
     (one-with 'jesus 'father)
     (not (equal 'jesus 'father))

     (one-with 'disciple-a 'disciple-b)
     (not (equal 'disciple-a 'disciple-b)))
  :hints
  (("Goal"
    :use
    (named-agents-are-distinct
     named-disciples-are-distinct
     recorded-sayings-constraints)))
  :rule-classes nil)
```

同一の聖霊が、複数の異なる弟子のうちにいることも確認する。

```lisp
(defthm one-holy-spirit-indwells-multiple-disciples
    (and
     (indwells 'holy-spirit 'disciple-a)
     (indwells 'holy-spirit 'disciple-b)
     (not (equal 'disciple-a 'disciple-b)))
  :hints
  (("Goal"
    :use
    (named-disciples-are-distinct
     recorded-sayings-constraints)))
  :rule-classes nil)
```

最後に、父を「唯一の、まことの神」と呼ぶ言語が、イエスや聖霊に関する他の発言と共存できることを示す。

```lisp
(defthm father-only-true-god-language-coexists-with-other-sayings
    (and
     (only-true-god 'father)

     (implies
      (and (only-true-god x)
           (only-true-god y))
      (equal x y))

     (not (equal 'father 'jesus))
     (not (equal 'father 'holy-spirit))
     (not (equal 'jesus 'holy-spirit))

     (sends 'father 'jesus)
     (one-with 'jesus 'father)
     (gives-life 'jesus)
     (honor-as 'jesus 'father)
     (has-precreation-glory-with 'jesus 'father)

     (another-helper 'holy-spirit)
     (sends-in-name-of 'father 'holy-spirit 'jesus)
     (sends-from 'jesus 'holy-spirit 'father)
     (indwells 'holy-spirit 'disciple-a)
     (indwells 'holy-spirit 'disciple-b)
     (glorifies 'holy-spirit 'jesus))
  :hints
  (("Goal"
    :use
    (only-true-god-referent-is-unique
     named-agents-are-distinct
     recorded-sayings-constraints)))
  :rule-classes nil)
```

この定理は、`(only-true-god 'jesus)` や `(only-true-god 'holy-spirit)` を主張していない。そこは意図的にモデルの外に置いている。

## 結論

この例で見せたかった ACL2 の面白さは、自然言語では曖昧になりやすい論点を、明示的な制約と定理に分解できることである。

今回の ACL2 モデルが示しているのは、次の一点である。

> 福音書に記録されたイエス自身の発言から抽出した関係制約には、父・イエス・聖霊を相互に同一化せず、同一の聖霊が複数の弟子に働くことも含めて、それらを同時に満たす整合的な witness model が存在する。

したがって、次のような主張は、このモデルの範囲では成り立たない。

> 父・イエス・聖霊について語る以上、一階述語論理の通常の同一性の下では直ちに矛盾する。

矛盾が出るとすれば、それは本文に現れる「一つ」という関係を、追加の根拠なく ACL2 の `equal` へ変換した場合である。

## この証明が証明していないこと

このモデルは、以下を証明するものではない。

- イエスは「唯一の、まことの神」である
- 聖霊は「唯一の、まことの神」である
- 後世の教理語そのものが正しい
- 福音書の記録が歴史的事実として真である
- 各ギリシャ語表現について唯一可能な解釈がこれである

証明しているのは、あくまで形式的整合性である。後世に三位一体と整理される論点を生じさせたイエスの発言群について、通常の同一性を維持したまま同時成立可能な関係モデルが存在する、ということに限定される。

この限定こそが ACL2 らしいところでもある。何を前提にしたのか、何を証明したのか、何を証明していないのかを、コードと定理の形で切り分けられる。

## 実行方法

ACL2 をビルドして、証明ファイルを読み込ませる。

証明ファイルは、公開済みリポジトリに含まれている。

https://github.com/askdkc/acl2-trinity-consistency-proof

```zsh
wget https://github.com/acl2/acl2/archive/refs/tags/8.7.tar.gz
tar zxvf 8.7.tar.gz
cd acl2-8.7
make LISP=sbcl
```

生成された `saved_acl2` で次を実行する。

```zsh
${PATH}/saved_acl2 << 'EOL'
  (ld "god-son-holyspirits-from-jesus-quotes.lisp")
  :q
EOL
```

定義が受理され、列挙された定理が `Q.E.D.` になれば成功である。

## 参照した聖句

- [マルコ 12:29-30 口語訳](https://www.bible.com/ja/bible/81/MRK.12.29-30.JA1955)
- [ヨハネ 17:3 口語訳](https://www.bible.com/ja/bible/81/JHN.17.3.JA1955)
- [ヨハネ 10:30 口語訳](https://www.bible.com/ja/bible/81/JHN.10.30.JA1955)
- [ヨハネ 17:21-22 口語訳](https://www.bible.com/ja/bible/81/JHN.17.21-22.JA1955)
- [ヨハネ 5:21-23 口語訳](https://www.bible.com/ja/bible/81/JHN.5.21-23.JA1955)
- [ヨハネ 17:5 口語訳](https://www.bible.com/ja/bible/81/JHN.17.5.JA1955)
- [ヨハネ 14:16-17 口語訳](https://www.bible.com/ja/bible/81/JHN.14.16-17.JA1955)
- [ヨハネ 14:26 口語訳](https://www.bible.com/ja/bible/81/JHN.14.26.JA1955)
- [ヨハネ 15:26 口語訳](https://www.bible.com/ja/bible/81/JHN.15.26.JA1955)
- [ヨハネ 16:13-15 口語訳](https://www.bible.com/ja/bible/81/JHN.16.13-15.JA1955)
- [マタイ 28:19 口語訳](https://www.bible.com/ja/bible/81/MAT.28.19.JA1955)
