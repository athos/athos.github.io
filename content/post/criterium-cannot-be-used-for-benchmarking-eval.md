+++
title = "Criteriumはevalのベンチマークには使えない"
date = 2017-07-05T09:59:06+09:00
tags = ["TIL","Clojure"]
+++

ほとんどの人にとっては誰得情報かと思いますが。。。

[Criterium](https://github.com/hugoduncan/criterium)といえば、Clojure界隈ではベンチマークツールの決定版的な位置づけのライブラリ。
外れ値の検出等の統計的な処理はもちろん、ベンチマーク結果がGCの影響を受けないようにGCをオフにしたり、JITコンパイルによってコードが
十分に最適化された状態になるまでウォームアップをしてくれたりと、JVM上でベンチマークをとるうえで気をつける必要がある点について
かなりしっかりとケアしてくれる。

で、今回の問題はそんな手厚いケアがアダになったケースといえそう。

<!--more-->

今やっていたのは、「一般的には`eval`が必要になるが条件が揃うと`eval`する必要がある部分を大幅に(もしくは完全に)削減できる」
という問題に対して、その対策になるようなライブラリを作ったらナイーブに`eval`を呼ぶ場合と比較してどれくらい速くなるのか、というのを
ひとまずプロトタイプを作ってベンチマークをとってみるということ。

まず、ライブラリ側のコードをベンチマークしてみる。小さいサイズの入力について、理想的な条件が揃う場合には`eval`なしに直接実行しているのと
ほとんど遜色ないくらいの速度が出る。目論見どおり。

次に`eval`を使ったコードの場合。ライブラリと同じようベンチマークを始める。

…待てども待てどもベンチマークが全然終わらない。

Criteriumはベンチマークするコードを何回も繰り返して実行時間のサンプルをとるので結果が出るまでに多少がかかるのだけど、最初に
1回あたりの実行時間を計測してうえで何回ベンチマークコードを実行するのかを見積もるので、(ベンチマークコード1回の実行が延々かかる
ということでもなければ)ベンチマークが終わらないということはないはず。

何かがおかしい。

そこで、このベンチマークをとっている間にCriterium内部で何が起こっているのか調べるために `with-progress-reporting` をつけて
進捗を報告してもらう。

```
Warming up for JIT optimisations 5000000000 ...                                 
  classes loaded before 1 iterations                                            
  classes loaded before 434 iterations                                          
  compilation occurred before 434 iterations                                    
  classes loaded before 867 iterations                                          
  classes loaded before 1300 iterations                                         
  classes loaded before 1733 iterations                                         
  classes loaded before 2166 iterations 
  classes loaded before 2599 iterations 
  classes loaded before 3032 iterations 
  classes loaded before 3465 iterations 
  classes loaded before 3898 iterations 
  classes loaded before 4331 iterations 
  classes loaded before 4764 iterations 
  classes loaded before 5197 iterations 
  classes loaded before 5630 iterations 
  classes loaded before 6063 iterations
  ...
```

この "classes loaded before ... iterations" の行が延々と続く。なぜ？

Criteriumのコードを読むと、この行は要するにクラスロードが新たに生じるうちはJITコンパイルが安定した状態にないと判断して
ウォームアップを続けていることを表しているようだ。

Clojureのコードは通常(AOTコンパイルされていなければ)実行時にコンパイルされ、JVMバイトコードが生成される。
つまり、そのタイミングで新たにクラスが生成され、ロードされる。ただし、同一の関数は一度コンパイルされてしまえば、
その後何度呼び出してもクラスが新たに生成されることはない。Criteriumはベンチマークコードを関数にくるんで実行するので、この点について問題はない。

しかし、`eval`を使った場合は別だ。`eval`を使うと、`eval`を呼び出した時点でコンパイルが走り、バイトコードが生成され、クラスがロードされる。
`eval`を呼び出すたびにクラスがロードされるので、Criteriumのウォームアップはいつまでも終わらない。

この現象は`eval`の呼び出し以外では意図的にクラスロードするようなコードでしか発生しないので大半の人は遭遇しないだろうけど、
なかなか思い当たらない意外な落とし穴なので注意したい。
