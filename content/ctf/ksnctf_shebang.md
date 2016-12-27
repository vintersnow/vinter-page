+++
date = "2016-12-27T21:45:46+09:00"
title = "ksnctf #10 #!"
tags = ["IT","CTF"]
thumbnail = "images/geek/ctf.jpg"
draft = false

+++

# 問題

>What's this?
```
↓
#!/usr/bin/python
print "Hello world"
```
>The flag is FLAG_S?????? (in capital letters).

# 解法

shellscriptの先頭にあるおまじないの名称。
ググり力が試される。

「shellscript 先頭」で調べたら出て来た。

+ https://moneyforward.com/engineers_blog/2015/05/21/bash-script-tips/

# flag

```
FLAG_SHEBANG
```

# point

**471**
