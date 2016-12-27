+++
date = "2016-12-27T10:15:59+09:00"
title = "ksnctf_onion"
tags = ["IT","CTF"]
thumbnail = "images/geek/ctf.jpg"
draft = false

+++

# 問題

>Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSV01WbDNXa1JTV0ZKdGVGWlZNakExVmpBeFYySkVU...

# 解法

英数字のみなのでbase64ぽい。

デコードすると

```
Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSWFJteFZVMjA1VjAxV2JETlhhMk0xVmpGYWMySkVU\nbGhoTVVwVVZtcEdTMlJIVmtW...
```

あれ？

でも文字数は減っている。onionという名前からして繰り返すのだろう。
繰り返すと次の文字列が手に入る。

```
b'begin 666 <data>\n51DQ!1U]&94QG4#-3:4%797I74$AU\n \nend\n'
```

uuencodeなので、ファイルに書き出したあとデコードする。

```
❯ ./onion.py > inner

~/Projects/CodeSite/CTF/ksnctf/05_onion
❯ uudecode inner

~/Projects/CodeSite/CTF/ksnctf/05_onion
❯ ls
'<data>'   __pycache__/   inner   onion.py   onion.txt

~/Projects/CodeSite/CTF/ksnctf/05_onion
❯ cat '<data>'
FLAG_FeLgP3SiAWezWPHu
```

## スクリプト

<script src="https://gist.github.com/vintersnow/a43576fbbfaf08757dd5f8c443002fed.js"></script>

# flag

```
FLAG_FeLgP3SiAWezWPHu
```

# point

**221**

# 参考

+ https://ja.wikipedia.org/wiki/Uuencode
+ http://docs.python.jp/3/library/base64.html
+ http://docs.python.jp/3.5/library/uu.html
