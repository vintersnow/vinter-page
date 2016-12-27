+++
date = "2016-12-27T01:01:39+09:00"
title = "ksnctf: #3 crawling_chaos"
draft = false

thumbnail = "images/geek/ctf.jpg"

tags = ['IT', 'CTF', 'ksnctf']
+++

# 問題

> http://ksnctf.sweetduet.info/q/3/unya.html

# 解法

開くとformが一つだけ、最初はsqlインジェクションかと思ったけど、htmlを見るとjsスクリプトだった。

```js
(ᒧᆞωᆞ)=(/ᆞωᆞ/),(ᒧᆞωᆞ).ᒧうー=-!!(/ᆞωᆞ/).にゃー,(〳ᆞωᆞ)=(ᒧᆞωᆞ),(〳ᆞωᆞ).〳にゃー=- -!(ᒧᆞωᆞ).ᒧうー,(ᒧᆞωᆞ).ᒧうーｰ=(〳ᆞωᆞ).〳にゃー- -!(ᒧᆞωᆞ).ᒧうー,(〳ᆞωᆞ).〳にゃーｰ=(ᒧᆞωᆞ).ᒧうーｰ- -(〳ᆞωᆞ).〳にゃー,(ᒧᆞωᆞ) ...
```

[某某時的な存在](https://www.google.co.jp/search?q=%E3%83%8B%E3%83%A3%E3%83%AB%E5%AD%90%E3%81%95%E3%82%93&rlz=1C5CHFA_enJP689JP690&source=lnms&tbm=isch&sa=X&ved=0ahUKEwjsysrpnpLRAhXFWrwKHUB3AAkQ_AUICCgB&biw=1680&bih=926)が這い寄ってくるのを感じる…

htmlでエラーが出ないことから、なぜかjavascriptとして動くらしい。

```nohighlight
❯ node chaos.js
undefined:2
$(function(){$("form").submit(function(){var t=$('input[type="text"]').val();var p=Array(70,152,195,284,475,612,791,896,810,850,737,1332,1469,1120,1470,832,1785,2196,1520,1480,1449);var f=false;if(p.length==t.length){f=true;for(var i=0;i<p.length;i++)if(t.charCodeAt(i)*(i+1)!=p[i])f=false;if(f)alert("(」・ω・)」うー!(/・ω・)/にゃー!");}if(!f)alert("No");return false;});});
^

ReferenceError: $ is not defined
    at eval (eval at <anonymous> (/Users/vinter/Projects/CodeSite/CTF/ksnctf/03_crawling_chaos/chaos.js:1:17299), <anonymous>:2:1)
    at Object.<anonymous> (/Users/vinter/Projects/CodeSite/CTF/ksnctf/03_crawling_chaos/chaos.js:1:17333)
    at Module._compile (module.js:570:32)
    at Object.Module._extensions..js (module.js:579:10)
    at Module.load (module.js:487:32)
    at tryModuleLoad (module.js:446:12)
    at Function.Module._load (module.js:438:3)
    at Module.runMain (module.js:604:10)
    at run (bootstrap_node.js:394:7)
    at startup (bootstrap_node.js:149:9)
```


これがflagぽい。

```js
var p=Array(70,152,195,284,475,612,791,896,810,850,737,1332,1469,1120,1470,832,1785,2196,1520,1480,1449);
```

## decode

<script src="https://gist.github.com/vintersnow/8742430bab271a052b6c19325e14777b.js"></script>

# flag

```
FLAG_fqpZUCoqPb4izPJE
```

# point

**151**
