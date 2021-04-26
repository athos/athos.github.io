+++
title = "ClojureScriptでテストが失敗したときにexit(1)する"
date = 2018-08-12T11:23:53+09:00
categories = ["tech","programming"]
tags = ["TIL","ClojureScript"]
+++

CIでテストを回す場合、テストに失敗したときにCI自体も失敗してくれると嬉しい。

Clojureだと、`lein test`でテストする場合でも[`test-runner`](https://github.com/cognitect-labs/test-runner)でテストする場合でも、
テストが失敗すると`exit(1)`してくれるのでテストの失敗をCIも認識してくれる。

自分で簡単なテストランナーを書く場合でも以下のようにすれば実現できる：

<!--more-->

```clj
(require '[clojure.test :as t])

(defn -main []
  (let [{:keys [fail error]} (t/run-tests <テスト対象の名前空間> ...)]
    (System/exit (if (zero? (+ fail error)) 0 1))))
```

一方、ClojureScriptの場合を考えてみよう。

テストを実行する環境がブラウザの場合は事情が厄介なので、ここでは Node.js や Nashorn のようなスタンドアローンな処理系上でのテストを考える
(ので、ClojureScript全般のテストの話というより、それらの処理系でテストすれば十分なClojureScriptライブラリのテストの話になる)。

ClojureScriptの `clojure.test/run-tests` はClojureの場合と違ってテスト結果のサマリーを戻り値として返してくれない。
`run-tests`のドキュメント文字列にも以下のように書いてある：

>   Runs all tests in the given namespaces; prints results.
>   Defaults to current namespace if none given. Does not return a meaningful
>  value due to the possiblity of asynchronous execution.

つまり、非同期に実行されるテストもあるので、戻り値でテスト結果を返すインターフェースだとうまくないと。

じゃあどうするかというと、ドキュメント文字列の続きに解決策が書いてあって、

> To detect test completion add a :end-run-tests method case to the cljs.test/report multimethod.

`cljs.test/report`を通して `:end-run-tests` としてテストの終了が通知されてくるらしい
(ちなみに、テストの開始や個々のテストケースの開始・終了も同様にして
[フックすることができる](https://github.com/clojure/clojurescript/blob/dab61a6f2d66a6353003724745dd55b0ef93d216/src/main/cljs/cljs/test.cljs#L355-L361))。
このときに、テスト結果も引数として渡ってくるので、これを使えば求めている挙動を実現することができる：

```clj
(ns example.core-test
  (:require [cljs.test :as t]))

(def js-exit
  (cond (exists? js/exit) js/exit                  ;; Nashornで実行される場合
        (exists? js/process.exit) js/process.exit  ;; Node.jsで実行される場合
        :else identity))
  
(defmethod t/report [::t/default :end-run-tests] [{:keys [fail error]}]
  (js-exit (if (zero? (+ fail error)) 0 1)))

(defn -main []
  (t/run-tests <テスト対象の名前空間> ...))
```
