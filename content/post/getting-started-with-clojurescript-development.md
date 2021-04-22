---
title: "ClojureScript自身の開発のはじめかた"
date: 2017-06-24T17:32:29+09:00
tags: ["TIL","ClojureScript"]
draft: false
backtotop: true
---

ClojureScriptの[不具合らしき挙動](https://dev.clojure.org/jira/browse/CLJS-2119)に遭遇したのでパッチを作ろうとするも、
普段cljsbuildやらFigwheelなんかのツールを介してしかClojureScriptに触れていないので、素のClojureScriptをどう動かせばいいのか
すら分からない状態。

とりあえず必要最低限、素のClojureScriptでの動作確認とテストをするところまではやったので、備忘録として残しておく。

<!--more-->

## ブートストラップ

以下のスクリプトを実行するだけ：

```
$ ./script/bootstrap
```

実際には、依存ライブラリなんかをダウンロードしてきて`lib/`以下に置いている。依存ライブラリ、特にClojureのバージョンが違ったりすると
REPLの起動でコケたりするので忘れずにやる。

`lib/`以下も含め、一旦綺麗にしたい場合は、あらかじめ以下も実行しておく：

```
$ ./script/clean
```

## REPL起動

一旦ブートストラップできたら以下でREPLが起動できる：

```
$ ./script/repljs
```

このスクリプトはJSランタイムとしてRhinoを使ってREPLを起動する。

Node.jsやNashornを使ってREPLを起動したい場合には、それぞれ `script/noderepljs`や`script/nashornrepljs`を使う。

`script/repl`というスクリプトもあるんだけど、こっちは実際にはClojureのREPLが立ち上がってそこからClojureScriptの
REPL環境やらを自前でロードして使う感じなので、サクッとREPLを起動したいだけの用途には向かない。

## テスト実行

ClojureScriptコンパイラ等、Clojureで書かれてたコードをテストする場合には以下を実行する：

```
$ lein test
```

ClojureScriptで書かれたコード(コアライブラリやセルフホスティングコンパイラ)等に関するテストは上のコマンドには含まれない。

コアライブラリのテストをしたい場合には以下を実行する：

```
$ ./script/test
```

ただし、このスクリプトはテストで使うJSランタイムが見つからないと**テストコードのビルドだけをして終了してしまう**ので注意が必要。
JSランタイムは環境変数等で指定してやる必要がある。
詳しくは[こちら](https://github.com/clojure/clojurescript/wiki/Running-the-tests#testing-javascript-engines)を参照。
