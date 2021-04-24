+++
title = "ByteBufferの使用がバイナリ互換性を破壊するケース"
date = 2018-12-12T20:05:43+09:00
tags = ["TIL","Clojure"]
+++

ClojureコードをコンパイルするとJVMバイトコードが生成される。一般的には、使用するClojureのパージョンさえ固定すれば、
どのような環境でコンパイルしてもコンパイル結果として得られるバイトコードは基本的に同じはずだ。

しかし、「どのバージョンのJava上でコンパイルしたバイトコードか」が問題になる特殊なケースがあるらしい(実際に遭遇した)ので、記録のためにメモしておく。

<!--more-->

たとえば、次のようなコードがあったとする：

```clj
(import 'java.nio.ByteBuffer)

(defn set-pos! [^ByteBuffer b ^long pos]
  (.position b pos))
```

これをJava 8上でコンパイルすると、コンパイル結果のバイトコードは以下のようになる：

```
 0: aload_0
 1: checkcast     #15                 // class java/nio/Buffer
 4: lload_1
 5: invokestatic  #21                 // Method clojure/lang/RT.intCast:(J)I
 8: invokevirtual #25                 // Method java/nio/Buffer.position:(I)Ljava/nio/Buffer;
11: areturn
```

一方で、同じコードをJava 9以上でコンパイルすると次のようなバイトコードが出力される：

```
 0: aload_0
 1: checkcast     #15                 // class java/nio/ByteBuffer
 4: lload_1
 5: invokestatic  #21                 // Method clojure/lang/RT.intCast:(J)I
 8: invokevirtual #25                 // Method java/nio/ByteBuffer.position:(I)Ljava/nio/ByteBuffer;
11: areturn                                        
```

バイトコードのインストラクションだけ見ていると気づきにくいが、下から2行目 `8: invokevirtual #25` から始まる行の呼び出している
メソッドの型が違っているのが分かるだろうか。

Java 8以前では `ByteBuffer` に対して `position(int)` メソッドを呼び出すと、`Buffer#position(int)` が呼ばれ、戻り値の型は
`Buffer`になる。Java 9以降では、 `Buffer#positon(int)` は残されつつも `ByteBuffer` に独自のオーバーライド
`ByteBuffer#position(int)` が追加され、その戻り値の型は `ByteBuffer` だ。 `ByteBuffer` 自体は `Buffer` クラスを
継承しているので型の面で特に何かが問題になるということはない。

問題なのは、バイトコード上に解決されるメソッドだ。 `ByteBuffer#position(int)` はJava 9で追加されたメソッドなので、
Java 8以前の環境に持っていくと当然そんなメソッドはないとエラーで怒られる：

```clj
;; Clojure 1.9.0
;; Java HotSpot(TM) 64-Bit Server VM 1.8.0_181-b13
user=> (set-pos! (ByteBuffer/allocate 32) 0)

NoSuchMethodError java.nio.ByteBuffer.position(I)Ljava/nio/ByteBuffer;  bytebuffer-repro.core/set-pos! (core.clj:5)
user=>
```

つまり、 `ByteBuffer#position` を使っているコードをJava 9以降の環境でコンパイルすると、生成されたバイトコードはJava 8以前の環境では動かなくなる。
同じ問題は `ByteBuffer#position` の他に `ByteBuffer#flip` や `ByteBuffer#limit`、 `ByteBuffer#clear` にもあるようだ。

調べてみると、AkkaやMongoDBのJavaドライバーなどのOSSもこの問題に遭遇している。

- https://github.com/akka/akka/issues/23755
- https://jira.mongodb.org/browse/JAVA-2559

この問題の解決自体は難しくない。Clojureでは、型ヒントをつけて `position` メソッドのレシーバが `Buffer` だと明示してやることで、
メソッドを `Buffer#position` に解決させることができる：

```clj
(import '[java.nio Buffer ByteBuffer])

(defn set-pos! [^ByteBuffer b ^long pos]
  (.position ^Buffer b pos))
```

この例だと、元の `b` の型ヒントを `Buffer` に変更してやるのでも問題ない。

もしくは、レシーバの型ヒントを完全に落としてしまって、実行時にリフレクションで解決してもらう手もあるが、わざわざ型ヒントをつけていたのを
落とすのが現実的な解決策になるケースはあまり多くないと思う：

```clj
(defn set-pos! [b ^long pos]
  (.position b pos))
```

なお、この話はあくまでAOTコンパイルをする場合の問題ということに注意。AOTコンパイルしないのであれば、コンパイル環境と実行環境が
食い違うということもないのでそもそも上のような問題は発生しない。AOTコンパイルはしないで済むならしない方が吉。
