+++
draft = false
date = "2016-12-27T21:55:08+09:00"
title = "ksnctf: #8 basic is secure?"
tags = ["IT","CTF"]
thumbnail = "images/geek/ctf.jpg"

+++

# 問題

>http://ksnctf.sweetduet.info/q/8/q8.pcap

# 解法

basicと言っているのでベーシック認証なのだろう。

pcapファイルをwiresharkで開く。
２回分の通信しかない短い記録なので上から見ていく。
二つ目のTCPストリームを見るとbasic 認証を通っているのが分かる。

```
GET /~q8/ HTTP/1.1
Host: ctfq.sweetduet.info:10080
Connection: keep-alive
Authorization: Basic cTg6RkxBR181dXg3eksyTktTSDhmU0dB
User-Agent: Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.162 Safari/535.19
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate,sdch
Accept-Language: ja,en-US;q=0.8,en;q=0.6
Accept-Charset: Shift_JIS,utf-8;q=0.7,*;q=0.3

HTTP/1.1 200 OK
Date: Sat, 26 May 2012 20:54:05 GMT
Server: Apache/2.2.15 (CentOS)
Last-Modified: Sat, 26 May 2012 12:24:46 GMT
ETag: "422da-b8-4c0ef920b3f8e"
Accept-Ranges: bytes
Content-Length: 184
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Q8</title>
  </head>
  <body>
    <p>Congratulations!</p>
    <p>The flag is q8's password.</p>
  </body>
</html>
```

basic認証はbase64でエンコードされているのでデコードする。

```
❯ echo 'cTg6RkxBR181dXg3eksyTktTSDhmU0dB' | base64 -D
q8:FLAG_5ux7zK2NKSH8fSGA
```

# flag

```
FLAG_5ux7zK2NKSH8fSGA
```

# point

**521**
