---
title: "ClojureScriptマクロからコンパイラオプションを参照する"
date: 2017-09-08T13:44:21+09:00
tags: ["TIL","ClojureScript"]
draft: false
backtotop: true
---

小ネタ。

ClojureScriptのコンパイラオプションというのは`cljsbuild`で`project.clj`に書いたり、
自前でビルドスクリプトを作る場合にはbuild APIに渡すアレ：

<!--more-->

```clj
(require 'cljs.build.api)

(cljs.build.api/build "src"
  ;; ↓コレがコンパイラオプション
  {:main 'hello-world.core
   :output-to "main.js"
   :target :nodejs})
```

ビルドの仕方によって値が変わってほしい(マクロなら展開結果が変わってほしい)というケースはたまにあって、
解決策はいろいろ考えられるがコンパイラオプションの値によって挙動が変えられれば小回りが利いて便利な場合もある。

ClojureScriptマクロから参照できるコンパイラ環境 [env/\*compiler\*](https://github.com/athos/TIL/blob/master/clojure/various-enviroments-in-clojurescript-compiler.md#envcompiler)
はコンパイラオプションも `:options` キーに保持しているので、そこからコンパイラオプションにアクセスできる。

たとえば、REPLを下のように構成して、

```clj
(require '[cljs.repl :as repl]
         '[cljs.repl.node :as node])

(repl/repl* (node/repl-env)
  {:output-dir "target/compiled/js/out"
   :optimizations :none
   :cache-analysis true
   :app/dev? true ;; ←この値をマクロから参照したい
   :source-map true})
```

以下のようなマクロを定義すると、

```clj
(ns example.macros
  (:require [cljs.env :as env]))
  
(defmacro dev?
  `'~(get-in @env/*compiler* [:options :app/dev?]))
```

REPL中から以下のようにコンパイラオプションの値を取得することができる：

```clj
=> (require-macros '[example.macros :refer [dev?]])
nil
=> (dev?)
true
=>
```

ちなみに、コンパイラオプションのキー名は標準的なオプションと近すぎるとREPLがtypoの可能性を警告してくる
(し、そうでなくても実際に標準オプションとコンフリクトする可能性がある)ので上の例のように適当なネームスペースをつけておいた方がよさそう。
