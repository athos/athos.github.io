---
title: "ClojureScriptコンパイラにまつわる「環境」あれこれ"
date: 2017-08-14T21:00:17+09:00
tags: ["TIL","ClojureScript"]
draft: false
backtotop: true
---

ClojureScriptのREPLやeval、マクロ等のコンパイラが関わる部分には「環境」がいろいろ出てくる。
compiler envやanalysis env、compiler stateというのもあるし、マクロの`&env`、
ClojureScriptで怪しげなことをやってるところでは `env/*compiler*` というのもよく見かける。

ClojureScriptコンパイラの実装を理解するうえで、これらのうち何が同じもので何が違うものなのか、
違うとしたらどう違うのかを一旦整理しておく必要がありそうだったので調べてまとめてみた。

<!--more-->

## compiler env

コンパイラ環境。コンパイルに必要な情報がすべて入っているマップ(を保持するアトム)。
コンパイルの各過程(analyze(AST構築)/compile(JSコード生成)/closure(最適化))で生成された
情報を書き出し、後工程で利用する。

中身としては以下のような値を持つ(他にもいろいろある)：

|キー名                          |説明 |
|-------------------------------|-----|
|`:cljs.analyzer/namespaces`    |依存する名前空間の情報や各名前空間の解析結果を保持する。          |
|`:cljs.analyzer/constant-table`|コンスタントテーブル。定数値から定数インデックスへのマップ。       |
|`:cljs.analyzer/data-readers`  |タグつきリテラルのタグからリーダ実装のVarを表すシンボルへのマップ。 |
|`:cljs.compiler/compiled-cljs` |コンパイル中間結果のキャッシュ。                              |
|`:js-dependency-index`         |シンボル間の依存関係？|

この中だと`:cljs.analyzer/namespaces`が一番使いでがあって、Clojureだと`ns-*`系の関数を介して
やるような処理はだいたいこの情報を介してできるようになっている。

### compiler state

compiler stateというのも実はcompiler envと同じものを指しているらしい。
呼び分けに何か区別があるのかはよく分かっていないけど、`cljs.analyzer.api`や`cljs.js`等の
ユーザ向けのAPIではこの名前が使われていることが多い印象。

### env/\*compiler\*

`cljs.env`で定義されている動的変数。現在のコンパイラ環境が束縛されている。
実行時には何も取得できないが、コンパイル時には有効になっているのでマクロからは参照可能。

## analysis env

コンパイラ環境がコンパイルのプロセス全般に渡って必要となる情報をまとめている一方、analysis env(解析環境)
の方はトップレベルフォーム1つの解析時に必要となる情報を持っている。
analyzerの文脈でenvというと大抵こちら。

「現在の名前空間」について、`:cljs.analyzer/namespaces`が持つデータをコピーして持っているほか、
今解析しているフォームから見えているローカルな束縛の解析結果等も持つ。

### &env

マクロの暗黙の引数`&env`には解析環境が渡ってくる。これはClojureの場合の`&env`とは
構造が違い、互換性はない(この差異を利用して、マクロがClojureの文脈で呼び出されているのか、
ClojureScriptの文脈で呼び出されているのかを判別するテクニックがある[らしい](https://github.com/cgrand/macrovich))。
