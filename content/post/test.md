+++
title = "test"
draft = true
date = "2016-12-05T13:15:39+09:00"

tags = [ "hello", "world" ]
+++


jshintで嘘のエラーが出た時の対処法

#概要
jshint便利ですよね。かなり自由に書けるjavascriptに秩序を与えてくれます。
自分はsublimeを通して使っています。
 普通のjavascriptを書く分にはいいのですが、node.jsなど外部ファイルを必要とするものを書く時は「定義されていませんよ〜」的なエラーたくさん出てきます。その対処法を調べたのでまとめておきます。
{{% img src="img/test.jpg" %}}

>require' is not defined. 'require' is not defined.と嘘のエラーが出ています。

#対処法その1
fileの先頭の以上の用に追加する。

```
/*global require */
```
これでこのファイルではrequireが定義されている事になります。
自分で作った外部ファイルの関数などはこうするしかないのかも。

#対処法その2
jshintが無視するように指定してしまいます。未定義以外のエラーにはいいかもれませんね。

```
root: __dirname + 'views',// jshint ignore:line
```

#対処法その3
実はnode.jsなどについては楽に出来る方法がアリます！

```
"node" : true
```
を.jshintrcに付け加えるだけ。以上です。

また

```
/*jshint node: true */
```
を最初に付け加えるという方法もあります。
![スクリーンショット_2015_01_22_23_27.png](https://qiita-image-store.s3.amazonaws.com/0/34727/821dd610-aaf7-f634-d5a1-d044e01c9508.png "スクリーンショット_2015_01_22_23_27.png")



jshintrcのオプションは[JSHintで気軽なコーディングを](http://blog.craftgear.net/50832ff38cdc8fb415000001/title/JSHint%E3%81%A7%E6%B0%97%E8%BB%BD%E3%81%AA%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92)が参考になると思います。
