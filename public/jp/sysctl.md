# カーネルパラメータの調整

## ulimitの設定

`ulimit -n`を100000またはそれ以上に設定する必要があります。以下のコマンドを実行して変更できます： `ulimit -n 100000`。変更できない場合は、`/etc/security/limits.conf`ファイルに設定する必要があります。

```
* soft nofile 262140
* hard nofile 262140
root soft nofile 262140
root hard nofile 262140
* soft core unlimited
* hard core unlimited
root soft core unlimited
root hard core unlimited
```

注意：`limits.conf`ファイルを変更した後は、システムを再起動して有効にする必要があります。

## カーネル設定

`Linux`オペレーティングシステムでカーネルパラメータを変更するには、3つの方法があります：

- `/etc/sysctl.conf`ファイルを変更し、設定オプションを追加します。フォーマットは`key = value`です。変更を保存した後は、`sysctl -p`を呼び出して新しい設定をロードします
- `sysctl`コマンドを使用して一時的に変更します。例：`sysctl -w net.ipv4.tcp_mem="379008 505344 758016"`
- `/proc/sys/`ディレクトリ中のファイルを直接変更します。例：`echo "379008 505344 758016" > /proc/sys/net/ipv4/tcp_mem`

> 最初の方法は、オペレーティングシステムを再起動した後に自動的に有効になります。2番目と3番目の方法は再起動後に失効します。

### net.unix.max_dgram_qlen = 100

swooleは、プロセス間通信にUnixソケットのdgramを使用しています。リクエストが多い場合は、このパラメータを調整する必要があります。システムはデフォルトで10に設定されていますが、100またはそれ以上に設定することができます。または、workerプロセスの数を増やすか、個々のworkerプロセスが割り当てるリクエスト量を減らすこともできます。

### net.core.wmem_max

このパラメータを変更して、ソケットのバッファサイズを増やします

```
net.ipv4.tcp_mem  =   379008       505344  758016
net.ipv4.tcp_wmem = 4096        16384   4194304
net.ipv4.tcp_rmem = 4096          87380   4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

### net.ipv4.tcp_tw_reuse

ソケットの再利用を設定します。この機能の目的は、サーバーが再起動したときにリッスンポートを迅速に再利用できるようにすることです。このパラメータを設定しないと、サーバーが再起動する際にポートが適切に解放されず、起動に失敗する可能性があります。

### net.ipv4.tcp_tw_recycle

クイック回収を使用すると、ショートコネクションサーバーでこのパラメータを有効にする必要があります。このパラメータは、Linuxシステムではデフォルトで0に設定されており、TCP接続のTIME-WAITソケットのクイック回収を開始します。このパラメータをオープンにすると、NATユーザー接続が不安定になる可能性があるため、慎重にテストしてから有効にしてください。

## メッセージキューの設定

プロセス間通信方法としてメッセージキューを使用する場合、このカーネルパラメータを調整する必要があります

- kernel.msgmnb = 4203520、メッセージキューの最大バイト数
- kernel.msgmni = 64、作成できるメッセージキューの最大数
- kernel.msgmax = 8192、メッセージキューの単一のデータの最大長

## FreeBSD/MacOS

- sysctl -w net.local.dgram.maxdgram=8192
- sysctl -w net.local.dgram.recvspace=200000
  Unixソケットのバッファ領域を変更します

## CoreDumpの有効化

カーネルパラメータを設定します

```
kernel.core_pattern = /data/core_files/core-%e-%p-%t
```

現在のcoredumpファイルの制限を確認するには、 `ulimit -c`コマンドを使用します。

```shell
ulimit -c
```

これが0の場合は、/etc/security/limits.confを変更して制限を設定する必要があります。

> Core-dumpを有効にすると、プログラムに異常が発生した場合にプロセスがファイルにエクスポートされます。プログラムの問題を調査するのに非常に役立ちます

## その他の重要な設定

- net.ipv4.tcp_syncookies=1
- net.ipv4.tcp_max_syn_backlog=81920
- net.ipv4.tcp_synack_retries=3
- net.ipv4.tcp_syn_retries=3
- net.ipv4.tcp_fin_timeout = 30
- net.ipv4.tcp_keepalive_time = 300
- net.ipv4.tcp_tw_reuse = 1
- net.ipv4.tcp_tw_recycle = 1
- net.ipv4.ip_local_port_range = 20000 65000
- net.ipv4.tcp_max_tw_buckets = 200000
- net.ipv4.route.max_size = 5242880

## 設定が有効になっているか確認する

例： `net.unix.max_dgram_qlen = 100` を変更した後、以下を使用して確認できます：

```shell
cat /proc/sys/net/unix/max_dgram_qlen
```

変更が成功した場合、ここに新しい設定値が表示されます。
