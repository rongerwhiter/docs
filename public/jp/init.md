```plaintext
# サーバー側（非同期スタイル）

非同期サーバープログラムを簡単に作成できます。`TCP`、`UDP`、[unixSocket](/learn?id=什么是IPC) の 3 種類のソケットタイプ、`IPv4` および `IPv6`、`SSL/TLS` 単方向および双方向証明書のトンネル暗号をサポートしています。ユーザーは低レイヤーの実装の詳細を気にする必要がなく、ネットワーク[イベント](/server/events)のコールバック関数を設定するだけで使えます。例は[クイックスタート](/start/start_tcp_server)を参照してください。

!> `Server`側のスタイルは非同期です（つまり、すべてのイベントにコールバック関数を設定する必要があります）、しかしながら、同時にコルーチンもサポートされています。[enable_coroutine](/server/setting?id=enable_coroutine)を有効にした後はコルーチンもサポートされます（デフォルトで有効）。[コルーチン](/coroutine)では、すべてのビジネスコードは同期的に書かれます。

以下をご覧ください：
[2種類のServerの実行モードの紹介](/learn?id=server的两种运行模式介绍 ':target=_blank')  
[Process、ProcessPool、UserProcessの違いは何ですか](/learn?id=process-diff ':target=_blank')  
[Masterプロセス、Reactorスレッド、Workerプロセス、Taskプロセス、Managerプロセスの違いと関係](/learn?id=diff-process ':target=_blank')  

### 実行フローチャート <!-- {docsify-ignore} --> 

![running_process](https://wiki.swoole.com/_images/server/running_process.png ':size=800xauto')

### プロセス/スレッド構造図 <!-- {docsify-ignore} --> 

![process_structure](https://wiki.swoole.com/_images/server/process_structure.png ':size=800xauto')

![process_structure_2](https://wiki.swoole.com/_images/server/process_structure_2.png)
```
