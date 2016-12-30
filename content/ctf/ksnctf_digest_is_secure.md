+++
date = "2016-12-30T23:35:12+09:00"
title = "ksnctf: #9 digest is secure!"
thumbnail = "images/geek/ctf.jpg"
draft = false
tags = ["IT","CTF"]

+++

# 問題

>http://ksnctf.sweetduet.info/q/9/q9.pcap

# 解法

問題文から分かるようにdigest認証の問題。
basic認証がガバガバだったのと違い、digest認証はもう少し安全。

wikipedia曰く以下のように認証する。

>A1 = ユーザ名 ":" realm ":" パスワード  
>A2 = HTTPのメソッド ":" コンテンツのURI  
>response = MD5( MD5(A1) ":" nonce ":" nc ":" cnonce ":" qop ":" MD5(A2) )

ユーザーはサーバーからnonce、nc、conce、qopが与えられてresponseを作る。
サーバーはA1を持っているので、responseを作って照合する。

なるほど確かに逆ハッシュが出来ないと厳しそう。

hydraというツールがあるくらいなので出来ないことは無いのだろうが…  
と思ってpcapファイルを眺めていると、なんとhtdigestを覗いているではないか！

```
GET /~q9/htdigest HTTP/1.1
Host: ctfq.sweetduet.info:10080
Connection: keep-alive
Authorization: Digest username="q9", realm="secret", nonce="bbKtsfbABAA=5dad3cce7a7dd2c3335c9b400a19d6ad02df299b", uri="/~q9/htdigest", algorithm=MD5, response="d9f18946e5587401c303b34e00a059eb", qop=auth, nc=00000002, cnonce="6945eb2a7ba8cf7f"
User-Agent: Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.162 Safari/535.19
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate,sdch
Accept-Language: ja,en-US;q=0.8,en;q=0.6
Accept-Charset: Shift_JIS,utf-8;q=0.7,*;q=0.3

HTTP/1.1 200 OK
Date: Sat, 26 May 2012 20:54:53 GMT
Server: Apache/2.2.15 (CentOS)
Authentication-Info: rspauth="022023eac9b9e023d50cca5eef69c287", cnonce="6945eb2a7ba8cf7f", nc=00000002, qop=auth
Last-Modified: Sat, 26 May 2012 12:30:54 GMT
ETag: "422e4-2b-4c0efa7f441cf"
Accept-Ranges: bytes
Content-Length: 43
Connection: close
Content-Type: text/plain; charset=UTF-8

q9:secret:c627e19450db746b739f41b64097d449
```

つまり`A1`がc627e19450db746b739f41b64097d449と分かった。

ここでdigest認証のレスポンスの作り方をもう一度見ると、

>response = MD5( MD5(A1) ":" nonce ":" nc ":" cnonce ":" qop ":" MD5(A2) )

+ A1 ← 分かった
+ A2 ← 分かる
+ nonce,nc,cnonce,qop ← 相手から与えられる

というわけでレスポンスが作れる！
（http://ksnctf.sweetduet.info:10080/~q9 がアクセス可能だと気づくのにしばらく時間がかかったが...）

responseの書き換えにはburpSuiteを使用する。これはproxyを立てることでhttp通信を覗き見、書き換えすることが出来るツールだ。
セットアップの仕方は[ここらへん](http://tech.pjin.jp/blog/2016/07/15/burp-suite-1-7%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9%E3%80%80%E3%81%9D%E3%81%AE%EF%BC%92/)参考に。

{{% img src="static/images/geek/test.jpg" %}}

nonceなどをコピーしてレスポンスを生成して書き換えるとflagが手に入った。

<script src="https://gist.github.com/vintersnow/23f1c45a295eb990bccc6ec21a19d1c8.js"></script>

## おまけ htdigest

やったことがなかったので、htdigestを試してみる。

```
❯ htdigest -c htdigest_test secret vinter
Adding password for vinter in realm secret.
New password: # vinterと入力
Re-type new password:

~/Projects/CodeSite/CTF/ksnctf/09_digest_is_secure
❯ cat htdigest_test
vinter:secret:98ef267645a2168773d7b944345ecf47

~/Projects/CodeSite/CTF/ksnctf/09_digest_is_secure
❯ md5 -s vinter:secret:vinter
MD5 ("vinter:secret:vinter") = 98ef267645a2168773d7b944345ecf47
```

確かにhashが一致している。

# point

**1031**

# 参考

+ https://ja.wikipedia.org/wiki/Digest%E8%AA%8D%E8%A8%BC
+ http://tech.pjin.jp/blog/2016/07/15/burp-suite-1-7%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9%E3%80%80%E3%81%9D%E3%81%AE%EF%BC%92/
