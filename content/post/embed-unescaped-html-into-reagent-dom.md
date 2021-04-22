---
title: "ReagentでエスケープされていないHTMLを埋め込む"
date: 2016-05-18T16:24:59+09:00
tags: ["TIL","ClojureScript"]
draft: false
backtotop: true
---

ClojureScriptのOmラッパーのひとつであるReagentでは、DOMをClojureScriptのデータ(ベクタやマップ)として書く。

このDOMを表すデータに含まれる文字列はデフォルトでHTMLエスケープされる。これを回避するにはReactの`dangerouslySetInnerHTML`の機能を使う。

<!--more-->

```clj
[:div {:dangerouslySetInnerHTML {:__html "<b>This is an unescaped HTML!!</b>"}}]
```

## 参考
- https://github.com/reagent-project/reagent/issues/14
- https://facebook.github.io/react/tips/dangerously-set-inner-html.html
