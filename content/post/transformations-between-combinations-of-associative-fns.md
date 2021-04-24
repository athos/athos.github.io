+++
title = "Associativeな値の操作関数の等価な書き換え"
date = 2016-06-21T17:24:46+09:00
tags = ["TIL","Clojure"]
+++

`update`や`update-in`が便利すぎてついつい使いすぎてしまう。
気づくと「もっとストレートな書き方があるのに…」というパターン(`update`+`assoc`とか`update-in`+`update`とか)を書いていたりするのでちょっと整理してみた。

<!--more-->

## 前提
### get/assoc/updateの性質
- `(get (assoc m k v) k)` = `v`
- `(assoc m k (get m k))` = `m`
- `(update m k f x ...)` = `(assoc m k (f (get m k) x ...))`

### get-in/assoc-in/update-inの形式的定義

#### get-in
- `(get-in m [k])` = `(get m k)`
- `(get-in m [k ks ...])` = `(get (get-in m [ks ...]) k)`

#### assoc-in
- `(assoc-in m [k] v)` = `(assoc m k v)`
- `(assoc-in m [k ks ...] v)` = `(assoc m k (assoc-in (get m k) ks v))`

#### update-in
- `(update-in m [k] f x ...)` = `(update m k f x ...)`
- `(update-in m [k ks ...] f x ...)` = `(update m k update-in [ks ...] f x ...)`

## get-in/assoc-in/update-inの書き換え
以上の`get`/`assoc`/`update`の性質と`get-in`/`assoc-in`/`update-in`の定義から、次が導出できる(はず)。

### 単純な書き換え
- `(get-in (get m k) [ks ...])` = `(get-in m [k ks ...])`
- `(get-in (get-in m [ks1 ...]) [ks2 ...])` = `(get-in m [ks1 ... ks2 ...])`
- `(assoc-in m [ks ...] (assoc (get-in m [ks ...]) k v))` = `(assoc-in m [ks ... k] v)`
    - 特に、`(empty? (get-in m [ks ...]))`の場合、`(assoc-in m [ks ...] (assoc {} k v))` = `(assoc-in m [ks ... k] v)`
- `(assoc-in m [ks1 ...] (assoc-in (get-in m [ks1 ...]) [ks2 ...] v))` = `(assoc-in m [ks1 ... ks2 ...] v)`
    - 特に、`(empty? (get-in m [ks1 ...]))`の場合、`(assoc-in m [ks1 ...] (assoc-in {} [ks2 ...] v))` = `(assoc-in m [ks1 ... ks2 ...] v)`
- `(update-in m [ks ...] update k f x ...)` = `(update-in m [ks ... k] f x ...)`
- `(update-in m [ks1 ...] update-in [ks2 ...] f x ...)` = `(update-in [ks1 ... ks2 ...] f x ...)`

### より複雑な書き換え
- `(assoc m k1 (update (get m k1) k2 f x ...))` = `(update-in m [k1 k2] f ...)`
    - 特に、`(empty? (get m k1))`の場合、`(assoc m k1 (update {} k2 f x ...))` = `(update-in m [k1 k2] f ...)`
- `(assoc-in m [ks1 ...] (update-in (get-in m [ks1 ...]) [ks2 ...] f x ...))` = `(update-in m [ks1 ... ks2 ...] f x ...)`
    - 特に、`(empty? (get-in m [ks1 ...]))`の場合、`(assoc-in m [ks1 ...] (update-in {} [ks2 ...] f x ...))` = `(update-in m [ks1 ... ks2 ...] f x ...)`
- `(update m k1 assoc k2 v)` = `(assoc-in m [k1 k2] v)`
- `(update-in m [ks1 ...] assoc-in [ks2 ...] v)` = `(assoc-in m [ks1 ... ks2 ...] v)`

### 証明
読者への課題とする。□
