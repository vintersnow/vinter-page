+++
date = "2016-12-27T12:27:36+09:00"
title = "ksnctf: #7 programing"
thumbnail = "images/geek/ctf.jpg"
draft = false
tags = ["IT","CTF"]
+++

# 問題

>http://ksnctf.sweetduet.info/q/7/program.cpp

# 解法
c++コードが渡される。
インデントが崩れていて読めないけど、コンパイルは通る。
実行すると`FROG_This_is_wrong_:(`となった。

で、実はこれC++ではなくて[whitespace](https://ja.wikipedia.org/wiki/Whitespace)という言語。
覚えていたからいいけど、初見だと分からないと思う。スペースとタブが混じっているところから気づくのかな？

まあ分かってしまえばインタプリタを探して実行するだけなんだけど、このインタプリタがなかなか見つからない！
いやいっぱいあるんだけどまともに動くのがない。
とりあえずいかのものがまともに動いた。

+ https://github.com/hostilefork/whitespacers のC言語版
+ http://ws2js.luilak.net/interpreter.html
+ https://whitespace.kauaveel.ee/

実行するとPINコードを求められる。

```
PIN: 
```

一番最後のインタプリタはアセンブリぽいコードを表示してくれてさらにステップ実行出来るのでこれで見ると、PINが33355524だと分かる。
入力するとflagが出て来た。

# flag

```
FLAG_EmTx6FTbGLieiMcA
```

# point

**451**

