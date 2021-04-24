+++
title = "「2つのキーのどちらか一方のみを含むマップ」を表現するスペック"
date = 2017-08-03T14:30:57+09:00
tags = ["TIL","Clojure"]
+++

たとえば、「整数をとる`:x`というキー」と「文字列をとる`:y`というキー」をもつマップのスペックは以下のように表現できる：

<!--more-->

```clj
(s/def ::x integer?)
(s/def ::y string?)

(s/keys :req-un [::x ::y])

;; (s/valid? (s/keys :req-un [::x ::y]) {:x 1})
;; => false
;; 両方のキーがないとダメ
```

もし、「`:x`と`:y`の少なくともどちらか一方をもつ」という条件ならこう書ける：

```clj
(s/keys :req-un [(::x ::y)]

;; (s/valid? (s/keys :req-un [(or ::x ::y)]) {:x 1})
;; => true
;; 片方のキーだけでもOK
```

`:req`や`:req-un`のベクタ内では、`or`だけでなく`and`も使えるようになっていて、キーの複雑な存在条件を記述することだできるようになっている。
この機能はあまり知られていないが`s/keys`のdocstringにもちゃんと書いてある：

    The :req key vector supports 'and' and 'or' for key groups:

    (s/keys :req [::x ::y (or ::secret (and ::user ::pwd))] :opt [::z])
    
ところで、上の例では`or`を使っているので、当然`:x`も`:y`も両方とも持つ場合もいいことになる：

```clj
(s/valid? (s/keys :req-un [(or ::x ::y)]) {:x 1 :y "foo"})
;; => true
```

たまに聞く話として「どちらか一方のキー**のみ**をもつ場合しか受けつけたくないときはどうすればいいんだ？」というのがあるようだ
(個人的には、今のところそういうスペックがほしいと思うケースに遭遇したことはない)。

そのような場合には、以下のようなスペックを書くのが常套的かと思う：

```clj
(s/and (s/keys :req-un [(or ::x ::y)])
       (fn [m] (or (and (contains? m :x) (not (contains? m :y)))
                   (and (not (contains? m :y)) (contains? m :y)))))
```

つまり、`s/keys`だけでは所望の条件を表現できないので、`s/and`ないし`s/merge`で追加の条件をつけてやるという話。

そういうものかと納得してしまえばそれまでの話なんだけど、たまたま`s/keys`の実装を読んでいたら、
`s/keys`内で使える`and`や`or`の位置には、実は**そのスコープで見えてる任意の関数・マクロが使える**ということのようだ。

これは単に今の時点で`s/keys`の構文チェックを厳密にしていないための不具合というか未定義動作なので、この挙動を利用して
何をかしようとするのはまったくもってお薦めしないのだけど、この挙動を使えばさっきの例は`s/keys`の中でしれっと`not`を使ってやることで
以下のようにすっきりと書くことができる：

```clj
(s/keys :req-un [(or (and ::x (not ::y)) (and (not ::x) ::y))])

;; 両方のキーがあるとダメ
;; (s/valid? (s/keys :req-un [(or (and ::x (not ::y)) (and (not ::x) ::y))]) {:x 1 :y "foo"})
;; => false
;; 片方のキーのみならOK
;; => false
;; (s/valid? (s/keys :req-un [(or (and ::x (not ::y)) (and (not ::x) ::y))]) {:x 1})
;; => true
;; (s/valid? (s/keys :req-un [(or (and ::x (not ::y)) (and (not ::x) ::y))]) {:y "foo"})
;; => true
```
