---
title: "GWにGoをはじめてみた"
date: 2021-05-16T16:35:00+09:00
categories: ["tech","programming"]
tags: ["Go"]
---

最近気づくと身の回りのOSSで、Goで書かれているものの割合が結構増えていた。
DockerやKubernetes、Terraform等の有名所はいうに及ばず、このブログを書くのに使ってるHugoもそうだし、アナリティクスサービスで使っている[GoatCounter](https://www.goatcounter.com/)もGo製らしい。まったく触れないままでいるのもいろいろと不便だし、GWに時間もあったのでせっかくなのでGoに入門してみた。

<!--more-->

やったことは[プログラミング言語Go](https://www.amazon.co.jp/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E8%A8%80%E8%AA%9EGo-ADDISON-WESLEY-PROFESSIONAL-COMPUTING-Donovan/dp/4621300253)を読んだのと、いくつか簡単なプログラムを書いたぐらい(テストコード含めて2000行ちょっと)。VS Codeでの開発環境も整えてデバッガも使えるようになった🙌

- [athos/go-playground - GitHub](https://github.com/athos/go-playground)

今のところそんなに悪い印象はない。
言語仕様をシンプルに保っているという触れ込みどおり、コアはかなり小さな言語という印象。
同じくシンプルを指向するClojureとも通じるところが多くあるように思う。プログラミング言語Goの文中にあった「他の言語から来たプログラマはGoの◯◯機能のミニマリズムに驚くかもしれない」みたいな記述も、Clojurianとしては親近感というか既視感を覚えるレベル。言語自体はかなりopinionatedだと思うけど(個人的にはopinionatedな言語は好きだけど)、これほどまでに世間でウケてるのはわりと不思議。背景にGoogleがいるからなのか、世の需要にうまくマッチしているからなのか？

言語がシンプルな分、周辺のツールチェーンをリッチに作り込むバランス感覚は今どきの言語という感じ。
そういったツールを作りやすくするために言語のスキャナやパーサ、ASTなんかが標準ライブラリに含まれているのもいいなぁと思う。

よく言われるジェネリクスがないというのは今のところあまり不便に感じてない。
といっても、そういうのはたとえば何かしらのデータ構造をライブラリとして提供したいとか、もうちょっとジェネラルに使えるものを構築するときに活きてくる機能だと思うので、小さなアプリケーションを作ってるだけの今の時点では必要性を感じづらいところかもしれない。個人的にはそれよりも、代数的データ型というか、そこまで行かなくても共用体みたいなのがもっとお手軽に作れると嬉しいかなと思った。

エラーハンドリングについては、前評判どおりたしかに煩雑になりがちだなと思う。
エディタに`if err != nil { ... }`を入力するためのスニペットは用意したけど、Rustの`Result` / `Option`に対する`?`みたいな、言語側のサポートがもうちょっとあると嬉しいかなぁとは思ったりする。

一方で、`error`をむやみに返さずに適切に`panic`したり、ときには無視したりする判断も必要だなと思う。たとえば、`bufio`の[`UnreadByte()`](https://golang.org/pkg/bufio/#Reader.UnreadByte) / [`UnreadRune()`](https://golang.org/pkg/bufio/#Reader.UnreadRune)はメソッドを呼び出す直前の操作に条件がついていて、それが満たされない場合にはエラーを返す。こういう事前条件違反は正しくコーディングできていれば起きないし、`panic`を使った方が適当な気がするけど、エラーを返すようになっているらしい。
こういうものまで含めてやみくもに`if err != nil { return err }`していると、無用なエラーが呼び出し側まで伝播して本当にそこらじゅうエラーハンドリングのコードばかりになってしまう。
ここで言ってるのは、ようするに「エラーは適切なタイミングで適切に処理する」っていうGoに限らないごく当たり前の話なんだけど、 Goには`Error`と`Exception`の違いもないし、どんなエラーが返ってくるかも戻り値型を見ただけでは分からないので、どんなエラーが返ってくる可能性があるかはドキュメントを読むなり実装コードに当たるなりしてより意識的に確認していく必要がありそう。

Go言語については今後も継続的に触っていきたい所存💪
