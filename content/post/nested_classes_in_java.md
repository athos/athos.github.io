+++
title = "Javaのネストしたクラスたち"
date = "2019-06-18T22:43:10+09:00"
categories = ["tech","programming"]
tags = ["TIL","Java"]
+++

最近Javaのネストしたクラス周辺について[言語仕様](https://docs.oracle.com/javase/specs/jls/se12/html/index.html)
を調べる機会があったのでまとめてみた。用語はなんとなく知っていたけど、オフィシャルな定義はよく知らなかったので。

<!--more-->

## ネストしたクラス (nested classes)

> A nested class is any class whose declaration occurs within the body of another class or interface ([§8](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html)).

ネストしたクラスは、別のクラスやインタフェース内で定義されるクラス。

## トップレベルクラス (top level classes)

> A top level class is a class that is not a nested class ([§8](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html)).

ネストしたクラス以外、つまり、トップレベルで定義されるクラスはトップレベルクラスという。

## メンバークラス (member classes)

> A member class is a class whose declaration is directly enclosed in the body of another class or interface declaration ([§8.5](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html#jls-8.5))

ネストしたクラスの中でも、別のクラスやインタフェースの定義本体の直下に定義されるクラスをメンバークラスという。

## staticなメンバークラス (static member classes)

*static nested class* とも呼ばれる。言語仕様の中にそのものズバリな定義はなさそうだが、
[§8.5.1](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html#jls-8.5.1) Static Member Type Declarationsが
一番詳細な記述だと思う。

> The static keyword may modify the declaration of a member type C within the body of a non-inner class or interface T.

内部クラス(後述)でない他のクラス内に定義されたメンバークラスのうち `static` と指定されたもの。

`static` 修飾子はメンバークラスにしかつけられないので、staticなメンバークラスはトップレベルクラスか別のstaticなメンバークラス内でしか定義できない。

## 内部クラス (inner classes)

> An inner class is a nested class that is not explicitly or implicitly declared static ([§8.1.3](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html#jls-8.1.3)).

内部クラスはネストしたクラスのうち `static` でないもの。

巷では「static内部クラス」「非static内部クラス」という用語を見かけることもあるが、定義に照らすと「static内部クラス」は誤りで
「非static内部クラス」は冗長な表現ということになる(内部クラスは常に非staticなので)。
一方で、「内部メンバークラス (inner member classes)」という用語は非staticなメンバークラスを表す用語として仕様の中でも使われている。

内部クラスには次の3つがある：
- 非staticメンバークラス
- ローカルクラス
- 匿名クラス

## ローカルクラス (local classes)

> A local class is a nested class that is not a member of any class and that has a name ([§14.3](https://docs.oracle.com/javase/specs/jls/se12/html/jls-14.html#jls-14.3)).

ローカルクラスはネストしたクラスのうち、別のクラスのメンバークラスではないもので、かつ名前をもつもの。

## 匿名クラス (anonymous classes)

> An anonymous class declaration is automatically derived from a class instance creation expression by the Java compiler ([§15.9.5](https://docs.oracle.com/javase/specs/jls/se12/html/jls-15.html#jls-15.9.5)).

匿名クラスは定義される場所による分類ではなく構文的な分類。class instance creation expression (つまり `new`)により暗黙的に作られるクラス。

「匿名クラスはローカルクラスの特殊な形態」という説明も見かけるけど(実現の仕方としてはそうだろうけど)、「ローカルクラスは名前を持つ」と明言されているので、
分類的には匿名クラスはローカルクラスにはならない。

