+++
thumbnail = "images/geek/ctf.jpg"
draft = false
date = "2016-12-29T19:42:07+09:00"
title = "ksnctf: #13 proverb"
tags = ["IT","CTF"]

+++

# 問題

>SSH: ctfq.sweetduet.info:10022
ID: q13
Pass: 8zvWx00MakSCQuGq

# 解法

sshで接続すると以下のようなファイルがあるホームディレクトリに接続する。

```
[q13@localhost ~]$ ll
total 28
-r-------- 1 q13a q13a    22 Jun  1  2012 flag.txt
---s--x--x 1 q13a q13a 14439 Jun  1  2012 proverb
-r--r--r-- 1 root root   755 Jun  1  2012 proverb.txt
-r--r--r-- 1 root root   151 Jun  1  2012 readme.txt
```

見ての通りflag.txtはq13aユーザーでしか見れない。
proverbは実行可能で実行するとことわざを出力してくれる。
でこのことわざはproverb.txtにかかれているものと同じみたい。

readme.txtは、

```
[q13@localhost ~]$ cat readme.txt
You are not allowed to connect internet and write the home directory.
If you need temporary directory, use /tmp.
Sometimes this machine will be reset.
```

なにかするならtmpでやれとのこと。

```
[q13@localhost ~]$ cd /tmp
[q13@localhost tmp]$ ls
ls: cannot open directory .: Permission denied
```

とりあえずディレクトリ作ってみる。

```
[q13@localhost tmp]$ mkdir q13
mkdir: cannot create directory `q13': File exists
[q13@localhost tmp]$ cd q13
[q13@localhost q13]$ ls
proverb  proverb.txt
```

んんん？

```
[q13@localhost q13]$ ./proverb
FLAG_XoK9PzskYedj/T&B
[q13@localhost q13]$ ll
total 0
lrwxrwxrwx 1 q13 q13 17 Dec 21 13:08 proverb -> /home/q13/proverb
lrwxrwxrwx 1 q13 q13 18 Dec 21 13:10 proverb.txt -> /home/q13/flag.txt
```

なるほど。proverbはproverb.txtから読み取っているのでflag.txtにシンボリックリンクを貼ればいいんだね。

...消していおいてよ:<

## setuid bit

```
---s--x--x 1 q13a q13a 14439 Jun  1  2012 proverb
```

proverbについている`s`はsetuid bitと言ってどのユーザーが実行してもそれをowner権限で実行するというもの。
今回これがあるからothersのq13が実行したのにflag.txtを読むことが出来た。

このsetuid bitはpasswdなどに使われている。

# point

**681**
