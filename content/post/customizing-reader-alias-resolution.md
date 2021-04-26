+++
title = "解決できないエイリアスをリードしたときにエラーにならないようにする"
date = 2018-08-24T15:15:04+09:00
categories = ["tech","programming"]
tags = ["TIL","Clojure"]
+++

Clojureの[MLからのネタ](https://groups.google.com/forum/#!topic/clojure/XrbBLynjpN8)。

Clojureでは名前空間にエイリアスがつけられて、シンボルやキーワードでそのエイリアスを使った場合に自動的に解決してくれる機能がある：

<!--more-->

```clj
(require '[clojure.string :as str])

`str/sym ;=> clojure.string/sym
::str/kw ;=> clojure.string/kw
```

ただし、キーワードのエイリアス解決については、`require` や `alias` であらかじめ定義されたエイリアスでない場合にはエラーになる。
このエラーは、そういった解決できないエイリアスのついたキーワードが **`comment` マクロやS式コメント(`#_`) でコメントアウトされていても回避できない**ことに注意：

```clj
=> ::unknown/kw
RuntimeException Invalid token: ::unknown/kw  clojure.lang.Util.runtimeException (Util.java:221)
=> (comment ::unknown/kw)
RuntimeException Invalid token: ::foo/kw  clojure.lang.Util.runtimeException (Util.java:221)
RuntimeException Unmatched delimiter: )  clojure.lang.Util.runtimeException (Util.java:221)
=> #_ ::unknown/kw
RuntimeException Invalid token: ::unknown/kw  clojure.lang.Util.runtimeException (Util.java:221)
=> 
```

それで、MLではこのエラーに対して、1.9で導入されたリーダリゾルバのプラグインの仕組みを使った回避策が述べられている。
MLの回答そのままだとエイリアスの解決をまったくしないのでそのまま実用はできないけど、エイリアス解決を組み込んで使えるように書き換えるとだいたいこんな感じになる：

```clj
(def resolver
  (reify clojure.lang.LispReader$Resolver
    (currentNS [_] (.-name *ns*))
    (resolveClass [_ sym] sym)
    (resolveAlias [_ sym]
      (or (some-> (get (ns-aliases *ns*) sym) ns-name)
          (binding [*out* *err*]
            (println (str "WARNING: unknown alias \"" sym
                          "\" found in " (ns-name *ns*) " ns"))
            sym)))
    (resolveVar [_ sym] sym)))

(alter-var-root #'*reader-resolver* (constantly resolver))
```

リーダのエイリアス解決時にリゾルバの `resolveAlias` が呼ばれるので、とりあえず普通にエイリアスの解決を試みてみて、
ダメならそのエイリアスを(警告を表示したうえで)そのまま返すようにしてやる。

これを `user.clj` などどこか適当なところに書いておいてやると、以下のように見知らぬエイリアスに遭遇してもエラーが出なくなる：

```clj
=> ::unknown/kw
WARNING: unknown alias "unknown" found in user ns
:unknown/kw
=> (comment ::unknown/kw)
WARNING: unknown alias "unknown" found in user ns
nil
=> #_ ::unknown/kw

=>
```

今回はエラーの解決にのみ注目した使い方をしたけど、たとえば解決できないエイリアスはプロジェクト名をプレフィックスとしてつける
といったカスタマイズをすればSpecを使うときに長々と名前空間名をつけたキーワードを書かなくてよくなる、みたいな応用もできるように思う。
ただし、リーダリゾルバはグローバルに影響を与えるので、ライブラリでは使うべきではないし、アプリケーションでも慎重に(そして自己責任で)使うべきでしょう。
