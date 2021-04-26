+++
title = "これからのClojureScriptの最もお手軽な起動方法？"
date = 2017-08-19T00:22:01+09:00
categories = ["tech","programming"]
tags = ["TIL","ClojureScript"]
+++

先日の[EuroClojure](https://www.youtube.com/playlist?list=PLZdCLR02grLpzt6WENiHe16-vx74VbCw_)
でのAlex Miller氏の基調講演で、[tools.deps.alpha](https://github.com/clojure/tools.deps.alpha)やそれを使った
インストーラ等の構想が発表された。

<!--more-->

これは、1.9でClojure自身が `spec.alpha` や `core.specs.alpha` などのライブラリに依存するようになり、
これまでのようにJARをひとつダウンロードしてこればClojureを使い始められるという前提が崩れることへの対応策という
面もあるとのこと。

`tools.deps.alpha`の副産物というか主産物というか、`clj`スクリプトが新たに提供されるように
[なるとのこと](https://github.com/clojure/tools.deps.alpha#clj-script)で、
この`clj`スクリプトが本格的に使えるようになると、`clj`コマンドを叩くだけでClojureを起動できるようになる。
スクリプティングやちょっとClojureを始めてみるような用途にはむしろこれまでのClojureの環境以上に便利に使えそうな雰囲気がある。

そんなわけで、Clojureが1.9リリースに向けて着々と準備が進んでいる一方で、`clj`スクリプトでは残念ながら
ClojureScriptのサポートはしないことが明言されている。

しかし、Mike Fikes氏(ClojureScript主要開発メンバー、Planck等のself-hosted CLJS処理系の開発者でもある)が今日、
`clj`コマンドを介したClojureScriptのREPLを簡単に起動する方法を紹介してくれていた：

> FWIW, @deg, if you create a `deps.edn` with
>
> `{:deps {org.clojure/clojurescript {:type :mvn, :version "1.9.908"}}}`
>
> Then `clj -m cljs.repl.node` starts a ClojureScript REPL.

from [Clojurian Slack #clojure channel](https://clojurians-log.clojureverse.org/clojure/2017-08-18.html#inst-2017-08-18T02:25:06.000166Z)

つまり、ClojureScriptはもともとClojureの一ライブラリであることと、ClojureScript自身はNode上で動くREPLの
実装を含んでいることから、`clj`コマンドでClojureScriptを依存ライブラリに指定してワンライナーでREPLを起動することができると。

ClojureScriptは`cljsbuild`にしろ`figwheel`にしろビルドの設定が煩雑で、使い始めるまでのハードルがだいぶ
高いので、これぐらいのお手軽さでREPLを起動できるようになると入門者にとっても教える側にとっても嬉しいことが
多いなぁと思った次第でした。
