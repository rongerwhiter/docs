## はじめに

このガイドでは、さまざまな工具の使用方法について説明します。

### 必要なもの

- はさみ
- ドライバー
- ハンマー

## はさみの使い方

1. はさみを手に取ります。
2. 刃の部分を使いたいものに合わせて開きます。
3. 切りたいものを刃の間に入れて、ゆっくりと力を加えて切ります。

## ドライバーの使い方

1. 適切なビットを選んでドライバーに取り付けます。
2. ドライバーをねじに差し込み、適度な力を加えてねじを締めます。
3. ドライバーを反対に回して緩めることもできます。

## ハンマーの使い方

1. ハンマーを手に取ります。
2. 打ちたい場所にハンマーの頭をあて、適度な力で叩きます。
3. 注意して、素材や周囲の状況に気をつけながら作業してください。
## yasd

[yasd](https://github.com/swoole/yasd)

单步调试工具，可用于`Swoole`协程环境，支持`IDE`以及命令行的调试模式。
## tcpdump

ネットワーク通信プログラムをデバッグする際に、tcpdumpは必須ツールです。tcpdumpは非常に強力で、ネットワーク通信のすべての詳細を見ることができます。たとえば、TCPの場合、3回のハンドシェイク、PUSH/ACKデータの転送、4回のエンディングハンドシェイクなど、すべての詳細を見ることができます。それに加えて、各ネットワーク受信パケットのバイト数や時間なども見ることができます。
### 使用方法

最も簡単な使用例：

```shell
sudo tcpdump -i any tcp port 9501
```
* -i パラメータはネットワークカードを指定し、any はすべてのネットワークカードを示します
* tcp はTCPプロトコルのみを監視することを指定します
* port は監視するポートを指定します

!> tcpdumpにはroot権限が必要です；通信のデータ内容を表示するには、`-Xnlps0` パラメータを追加する必要があります。その他の詳細なパラメータについては、オンラインの記事をご覧ください
### 実行結果

```
13:29:07.788802 IP localhost.42333 > localhost.9501: Flags [S], seq 828582357, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 0,nop,wscale 7], length 0
13:29:07.788815 IP localhost.9501 > localhost.42333: Flags [S.], seq 1242884615, ack 828582358, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 2207513,nop,wscale 7], length 0
13:29:07.788830 IP localhost.42333 > localhost.9501: Flags [.], ack 1, win 342, options [nop,nop,TS val 2207513 ecr 2207513], length 0
13:29:10.298686 IP localhost.42333 > localhost.9501: Flags [P.], seq 1:5, ack 1, win 342, options [nop,nop,TS val 2208141 ecr 2207513], length 4
13:29:10.298708 IP localhost.9501 > localhost.42333: Flags [.], ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:10.298795 IP localhost.9501 > localhost.42333: Flags [P.], seq 1:13, ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 12
13:29:10.298803 IP localhost.42333 > localhost.9501: Flags [.], ack 13, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:11.563361 IP localhost.42333 > localhost.9501: Flags [F.], seq 5, ack 13, win 342, options [nop,nop,TS val 2208457 ecr 2208141], length 0
13:29:11.563450 IP localhost.9501 > localhost.42333: Flags [F.], seq 13, ack 6, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
13:29:11.563473 IP localhost.42333 > localhost.9501: Flags [.], ack 14, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
```
* `13:29:11.563473` の時間はマイクロ秒まで精度があります
*  localhost.42333 > localhost.9501 は通信の方向を示します。42333 がクライアントで、9501 がサーバーです
* [S] は SYN リクエストを表します
* [.] は ACK 確認パケットを表し、(client)SYN->(server)SYN->(client)ACK が3回のハンドシェイクプロセスです
* [P] はデータプッシュを示し、サーバーからクライアント、あるいはクライアントからサーバーにプッシュされるデータになります
* [F] は FIN パケットであり、接続を閉じる操作を示します。client/server いずれかが発信する可能性があります
* [R] は RST パケットを表し、F パケットと同じ機能ですが、RST は接続が閉じられる際にまだ処理されていないデータがあることを示します。接続を強制的に切断すると理解できます
* win 342 はウィンドウサイズを示します
* length 12 はデータパケットのサイズを示します
## strace

straceはシステムコールの実行状況をトレースすることができ、プログラムに問題が発生した場合にstraceを使用して問題を分析およびトレースすることができます。

!> FreeBSD/MacOSではtrussが使用できます。
### 使用方法

```shell
strace -o /tmp/strace.log -f -p $PID
```

* -f 表示跟踪多线程和多进程，如果不加-f参数，无法抓取到子进程和子线程的运行情况
* -o 表示将结果输出到一个文件中
* -p $PID，指定跟踪的进程ID，通过ps aux可以看到
* -tt 打印系统调用发生的时间，精确到微秒
* -s 限定字符串打印的长度，如recvfrom系统调用收到的数据，默认只打印32字节
* -c 实时统计每个系统调用的耗时
* -T 打印每个系统调用的耗时
## gdb

GDBはGNUオープンソース機構によってリリースされた強力なUNIX向けプログラムデバッグツールであり、C/C++プログラムや、PHPやSwooleのようにC言語で開発されたプログラムのデバッグに使用できます。

gdbデバッグはコマンドラインインタラクティブであり、一般的なコマンドを把握する必要があります。
### 使用方法

```shell
gdb -p <プロセスID>
gdb php
gdb php core
```

gdbには3つの使用方法があります：

- 実行中のPHPプログラムをトレースするには、`gdb -p <プロセスID>`を使用します。
- PHPプログラムを実行してデバッグするには、`gdb php`を使用して `run server.php`を実行します。
- PHPプログラムがコアダンプした後は、`gdb php core`を使用してcoreメモリイメージをロードしてデバッグを行います。

!> PATH環境変数にphpが含まれていない場合は、gdbを実行する際に絶対パスを指定する必要があります。例：`gdb /usr/local/bin/php`
### 常用コマンド

* `p`：print，C変数の値を印刷する
* `c`：continue，中断されたプログラムを継続する
* `b`：breakpoint，ブレークポイントを設定する。関数名で設定することもできます。例：`b zif_php_function`。ソースコードの行数でブレークポイントを指定することもできます。例：`b src/networker/Server.c:1000`
* `t`：thread，スレッドを切り替える。プロセスに複数のスレッドがある場合、tコマンドを使用して異なるスレッドに切り替えることができます。
* `ctrl + c`：現在実行中のプログラムを中断する。cコマンドと併用して使用します。
* `n`：next，次の行を実行し、ステップ実行する
* `info threads`：実行中のすべてのスレッドを確認する
* `l`：list，ソースコードを表示する。`l 関数名`または`l 行番号`を使用できます。
* `bt`：backtrace，実行時の関数呼び出しスタックを確認する
* `finish`：現在の関数を完了する
* `f`：frame，btと組み合わせて使用し、関数呼び出しスタックの特定のレイヤーに切り替えることができます。
* `r`：run，プログラムを実行する
### zbacktrace

zbacktraceはPHPソースパッケージに含まれるgdbのカスタムコマンドであり、btコマンドと同様の機能を持っていますが、異なる点はzbacktraceが見るスタックトレースがPHP関数呼び出しスタックであることです。

php-srcをダウンロードして展開し、ルートディレクトリから`.gdbinit`ファイルを見つけて、gdbシェルで次のように入力します。

```shell
source .gdbinit
zbacktrace
```
`.gdbinit`には他の多くのコマンドもあり、詳細な情報を知るためにソースコードを確認することができます。
#### gdb+zbacktraceを使用してデッドロックの問題をトレースする

```shell
gdb -p <プロセスID>
```

* `ps aux`コマンドを使用してデッドロックが発生しているWorkerプロセスのIDを特定する
* `gdb -p`を使用して特定のプロセスを追跡する
* `ctrl + c`、`zbacktrace`、`c`を繰り返し使用して、どのPHPコード部分でループが発生しているかを確認する
* 該当するPHPコードを特定して問題を解決する
## lsof

Linuxプラットフォームでは、`lsof`ツールを使用して特定のプロセスが開いているファイルハンドルを表示できます。これは、Swooleのワーカープロセスが開いているすべてのソケット、ファイル、リソースを追跡するために使用できます。
### 使用方法

```shell
lsof -p [プロセスID]
```
```shell
lsof -p 26821
lsof: 警告: 无法统计追踪文件系统/sys/kernel/debug/tracing
      输出信息可能不完整。
COMMAND   PID USER   FD      TYPE             DEVICE SIZE/OFF    NODE NAME
php     26821  htf  cwd       DIR                8,4     4096 5375979 /home/htf/workspace/swoole/examples
php     26821  htf  rtd       DIR                8,4     4096       2 /
php     26821  htf  txt       REG                8,4 24192400 6160666 /opt/php/php-5.6/bin/php
php     26821  htf  DEL       REG                0,5          7204965 /dev/zero
php     26821  htf  DEL       REG                0,5          7204960 /dev/zero
php     26821  htf  DEL       REG                0,5          7204958 /dev/zero
php     26821  htf  DEL       REG                0,5          7204957 /dev/zero
php     26821  htf  DEL       REG                0,5          7204945 /dev/zero
php     26821  htf  mem       REG                8,4   761912 6160770 /opt/php/php-5.6/lib/php/extensions/debug-zts-20131226/gd.so
php     26821  htf  mem       REG                8,4  2769230 2757968 /usr/local/lib/libcrypto.so.1.1
php     26821  htf  mem       REG                8,4   162632 6322346 /lib/x86_64-linux-gnu/ld-2.23.so
php     26821  htf  DEL       REG                0,5          7204959 /dev/zero
php     26821  htf    0u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    1u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    2u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    3r      CHR                1,9      0t0      11 /dev/urandom
php     26821  htf    4u     IPv4            7204948      0t0     TCP *:9501 (LISTEN)
php     26821  htf    5u     IPv4            7204949      0t0     UDP *:9502 
php     26821  htf    6u     IPv6            7204950      0t0     TCP *:9503 (LISTEN)
php     26821  htf    7u     IPv6            7204951      0t0     UDP *:9504 
php     26821  htf    8u     IPv4            7204952      0t0     TCP localhost:8000 (LISTEN)
php     26821  htf    9u     unix 0x0000000000000000      0t0 7204953 type=DGRAM
php     26821  htf   10u     unix 0x0000000000000000      0t0 7204954 type=DGRAM
php     26821  htf   11u     unix 0x0000000000000000      0t0 7204955 type=DGRAM
php     26821  htf   12u     unix 0x0000000000000000      0t0 7204956 type=DGRAM
php     26821  htf   13u  a_inode               0,11        0    9043 [eventfd]
php     26821  htf   14u     unix 0x0000000000000000      0t0 7204961 type=DGRAM
php     26821  htf   15u     unix 0x0000000000000000      0t0 7204962 type=DGRAM
php     26821  htf   16u     unix 0x0000000000000000      0t0 7204963 type=DGRAM
php     26821  htf   17u     unix 0x0000000000000000      0t0 7204964 type=DGRAM
php     26821  htf   18u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   19u  a_inode               0,11        0    9043 [signalfd]
php     26821  htf   20u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   22u     IPv4            7452776      0t0     TCP localhost:9501->localhost:59056 (ESTABLISHED)
```

* soファイルはプロセスがロードしているダイナミックリンクライブラリです
* IPv4/IPv6 TCP (LISTEN) はサーバーが監視しているポートです
* UDP はサーバーが監視しているUDPポートです
* unix type=DGRAM のときはプロセスが作成したunixSocketです
* IPv4 (ESTABLISHED) はサーバーに接続しているTCPクライアントを示し、クライアントのIPとPORT、および状態(ESTABLISHED)が含まれます
* 9u / 10u はそのファイルハンドルのfd値(ファイル記述子)を示します
* その他の詳細情報についてはlsofのマニュアルを参照してください
## perf

`perf`ツールは、Linuxカーネルが提供する非常に強力なダイナミックトレースツールであり、`perf top`コマンドは実行中のプログラムのパフォーマンス問題をリアルタイムで分析するために使用できます。`callgrind`、`xdebug`、`xhprof`などのツールとは異なり、`perf`はコードの変更なしにプロファイル結果ファイルをエクスポートする必要がありません。
### 使用方法

```shell
perf top -p [プロセスID]
```
### 出力結果

![パフォーマンス上のトップ結果](../_images/other/perf.png)

perfの結果は、現在のプロセスの実行中における各C関数の実行時間を明確に示しており、どのC関数がCPUリソースをより多く消費しているかを把握できます。

Zend VMに精通している場合、特定のZend関数の呼び出しが多いと、プログラムで特定の関数を大量に使用していることを示し、CPU使用率が高くなっている可能性があります。その場合は、対策を講じて最適化を行うと良いでしょう。
