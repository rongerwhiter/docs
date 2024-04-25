```python
def greet(name):
    return f'Hello, {name}!'

print(greet('World'))
```

この関数は、引数として与えられた名前に挨拶を返すものです。
## __construct()

[非同期IO](/learn?id=同期io非同期io)のTCP Serverオブジェクトを作成します。

```php
Swoole\Server::__construct(string $host = '0.0.0.0', int $port = 0, int $mode = SWOOLE_PROCESS, int $sockType = SWOOLE_SOCK_TCP): \Swoole\Server
```

  * **パラメーター**

    * `string $host`

      * 機能：リッスンするIPアドレスを指定します。
      * デフォルト値：なし。
      * その他の値：なし。

      !> IPv4では `127.0.0.1` はローカルをリッスンし、`0.0.0.0` はすべてのアドレスをリッスンします。
      IPv6では `::1` はローカルをリッスンし、`::`（`0:0:0:0:0:0:0:0`と同等）はすべてのアドレスをリッスンします。

    * `int $port`

      * 機能：リッスンするポートを指定します。例：`9501`。
      * デフォルト値：なし。
      * その他の値：なし。

      !> もし `$sockType` が[UnixSocket Stream/Dgram](/learn?id=IPC)の場合、このパラメータは無視されます。
      `1024`より小さいポートをリッスンするには`root`権限が必要です。
      このポートが使用されていると、`server->start` すると失敗します。

    * `int $mode`

      * 機能：実行モードを指定します。
      * デフォルト値：[SWOOLE_PROCESS](/learn?id=swoole_process) マルチプロセスモード（デフォルト）。
      * その他の値：[SWOOLE_BASE](/learn?id=swoole_base) ベースモード。

      !> Swoole5以降、実行モードのデフォルト値は`SWOOLE_BASE`です。

    * `int $sockType`

      * 機能：このサーバーのタイプを指定します。
      * デフォルト値：なし。
      * その他の値：
        * `SWOOLE_TCP/SWOOLE_SOCK_TCP` tcp ipv4 socket
        * `SWOOLE_TCP6/SWOOLE_SOCK_TCP6` tcp ipv6 socket
        * `SWOOLE_UDP/SWOOLE_SOCK_UDP` udp ipv4 socket
        * `SWOOLE_UDP6/SWOOLE_SOCK_UDP6` udp ipv6 socket
        * [SWOOLE_UNIX_DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) unix socket dgram
        * [SWOOLE_UNIX_STREAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/stream_server.php) unix socket stream

      !> `$sock_type` | `SWOOLE_SSL`を使用して`SSL`トンネル暗号化を有効にできます。`SSL`を有効にした場合は必ず設定する必要があります。[ssl_key_file](/server/setting?id=ssl_cert_file) と [ssl_cert_file](/server/setting?id=ssl_cert_file)

  * **例**

```php
$server = new \Swoole\Server($host, $port = 0, $mode = SWOOLE_PROCESS, $sockType = SWOOLE_SOCK_TCP);

//UDP/TCPを混合して使用し、内部と外部のポートを同時にリッスンし、複数ポートのリッスンについてはaddlistenerセクションを参照してください。
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP); // TCPを追加
$server->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP); // Web Socketを追加
$server->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP); // UDPを追加
$server->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM); //UnixSocket Streamを追加
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP | SWOOLE_SSL); //TCP + SSL

$port = $server->addListener("0.0.0.0", 0, SWOOLE_SOCK_TCP); // システムがランダムにポートを割り当て、戻り値は割り当てられたランダムなポートです
echo $port->port;
```
## set()

ランタイムのさまざまなパラメータを設定するために使用されます。サーバーが起動した後、`Server->set`メソッドで設定されたパラメータ配列にアクセスするには`$serv->setting`を使用します。

```php
Swoole\Server->set(array $setting): void
```

!> `Server->set`は`Server->start`の前に呼び出す必要があります。各設定の意味については[このセクション](/server/setting)を参照してください。

  * **例**

```php
$server->set(array(
    'reactor_num'   => 2,     // 线程数
    'worker_num'    => 4,     // 进程数
    'backlog'       => 128,   // 设置Listen队列长度
    'max_request'   => 50,    // 每个进程最大接受请求数
    'dispatch_mode' => 1,     // 数据包分发策略
));
```
```php
Swoole\Server->on(string $event, callable $callback): bool
```

!> Calling the `on` method multiple times will override the previous setting.

!> Starting from PHP 8.2, a warning will be thrown if `$event` is not a specified event in Swoole, because PHP 8.2 does not support directly setting dynamic properties.

  * **Parameters**

    * `string $event`

      * Function: Event name for the callback.
      * Default value: None
      * Other values: None

      !> Case-insensitive. Refer to [this section](/server/events) for the list of event callbacks. Do not include `on` in the event name string.

    * `callable $callback`

      * Function: Callback function
      * Default value: None
      * Other values: None

      !> It can be a string of function name, class static method, object method array, or anonymous function. Refer to [this section](/learn?id=ways-to-set-callback-functions) for more information.
  
  * **Return Value**

    * `true` indicates success, `false` indicates failure.

  * **Example**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('connect', function ($server, $fd){
    echo "Client: Connect.\n";
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->on('close', function ($server, $fd) {
    echo "Client: Close.\n";
});
$server->start();
```
## addListener()

ポートのリスニングを追加します。ビジネスコードでは、[Swoole\Server->getClientInfo](/server/methods?id=getclientinfo)を呼び出すことで、特定の接続がどのポートから来たかを取得できます。

```php
Swoole\Server->addListener(string $host, int $port, int $sockType): bool|Swoole\Server\Port
```

!> ポート`1024`以下をリッスンするには`root`権限が必要です  
メインサーバーが`WebSocket`または`HTTP`プロトコルである場合、新しくリスンする`TCP`ポートはデフォルトでメイン`Server`のプロトコル設定を継承します。新しいプロトコルを有効にするには、新しいプロトコルを設定するために`set`メソッドを個別に呼び出す必要があります [詳細はこちら ](/server/port)。
`Swoole\Server\Port`の詳細については[こちら](/server/server_port)をクリックしてください。 

  * **パラメータ**

    * `string $host`

      * 機能：`__construct()`の`$host`と同じ
      * デフォルト値：`__construct()`の`$host`と同じ
      * その他の値：`__construct()`の`$host`と同じ

    * `int $port`

      * 機能：`__construct()`の`$port`と同じ
      * デフォルト値：`__construct()`の`$port`と同じ
      * その他の値：`__construct()`の`$port`と同じ

    * `int $sockType`

      * 機能：`__construct()`の`$sockType`と同じ
      * デフォルト値：`__construct()`の`$sockType`と同じ
      * その他の値：`__construct()`の`$sockType`と同じ
  
  * **返り値**

    * `Swoole\Server\Port`が成功したことを示し、`false`が操作に失敗したことを示します。

!> -`Unix Socket`モードでは、`$host`パラメータにはアクセス可能なファイルパスを指定する必要があり、`$port`パラメータは無視されます  
-`Unix Socket`モードでは、クライアントの`$fd`は数字ではなくファイルパスの文字列となります  
-`Linux`システムでは、`IPv6`ポートを監視した後に`IPv4`アドレスを使用しても接続できます
```php
Swoole\Server->listen(string $host, int $port, int $type): bool|Swoole\Server\Port
```
## addProcess()

ユーザー定義のプロセスを追加します。通常、この関数は、監視、レポート、または他の特殊なタスクを行うための特別なプロセスを作成するために使用されます。

```php
Swoole\Server->addProcess(Swoole\Process $process): int
```

!> `start`を実行する必要はありません。`Server`が起動するときにプロセスが自動的に作成され、指定されたサブプロセス関数が実行されます。

  * **Parameters**
  
    * [Swoole\Process](/process/process)

      * 機能：`Swoole\Process` オブジェクト
      * デフォルト値：なし
      * その他の値：なし

  * **Return Value**

    * プロセスID番号が返され、操作が成功したことを示します。そうでない場合、プログラムは致命的なエラーをスローします。

  * **注意**

    !> -作成されたサブプロセスは、`$server`オブジェクトが提供する各種メソッド（`getClientList`、`getClientInfo`、`stats`など）を呼び出すことができます。                                   
    -`Worker/Task`プロセス内では、`$process`を使用してサブプロセスと通信することができます。        
    -ユーザー定義のプロセス内では、`$server->sendMessage`および`Worker/Task`プロセスと通信することができます。      
    -ユーザープロセス内で`Server->task/taskwait`インタフェースを使用することはできません。              
    -ユーザープロセス内で`Server->send/close`などのインタフェースを使用することはできます。         
    -ユーザープロセス内では、`while(true)`（下記の例のように）または[EventLoop](/learn?id=什么是eventloop)ループ（例：タイマーの作成）を行う必要があります。そうしない場合、ユーザープロセスは続けて終了と再起動が繰り返されます。 

  * **Life Cycle**

    ?> -ユーザープロセスの寿命は`Master`および [Manager](/learn?id=manager进程) と同じであり、[reload](/server/methods?id=reload)の影響を受けません。     
    -ユーザープロセスは`reload`命令に影響を受けず、`reload`時にユーザープロセスに情報は送信されません。        
    -サーバーを`shutdown`クローズする際、`SIGTERM`シグナルがユーザープロセスに送信されてプロセスが終了します。            
    -カスタムプロセスは`Manager`プロセスによって管理され、致命的なエラーが発生した場合、`Manager`プロセスが新しいプロセスを作成します。         
    -カスタムプロセスは`onWorkerStop`などのイベントもトリガーしません。 

  * **例**

    ```php
    $server = new Swoole\Server('127.0.0.1', 9501);
    
    /**
     * ユーザープロセスにブロードキャスト機能を実装し、unixSocketからのメッセージを受信し、サーバーのすべての接続に送信します
     */
    $process = new Swoole\Process(function ($process) use ($server) {
        $socket = $process->exportSocket();
        while (true) {
            $msg = $socket->recv();
            foreach ($server->connections as $conn) {
                $server->send($conn, $msg);
            }
        }
    }, false, 2, 1);
    
    $server->addProcess($process);
    
    $server->on('receive', function ($serv, $fd, $reactor_id, $data) use ($process) {
        // 受信したメッセージを一斉送信する
        $socket = $process->exportSocket();
        $socket->send($data);
    });
    
    $server->start();
    ```

    [プロセス間通信セクション](/process/process?id=exportsockets)を参照してください。
## start()

サーバーを起動し、すべての`TCP/UDP`ポートをリッスンします。

```php
Swoole\Server->start(): bool
```

!> 注意: 以下は[SWOOLE_PROCESS](/learn?id=swoole_process) モードを例としています

  * **ヒント**

    - 起動後、`worker_num+2` 個のプロセスが作成されます。`Master`プロセス+`Manager`プロセス+`serv->worker_num`個の`Worker`プロセスです。  
    - 起動に失敗した場合はすぐに`false`が返されます。
    - 起動に成功すると、イベントループに入り、クライアントの接続要求を待機します。`start`メソッドの後にコードが実行されることはありません。  
    - サーバーが閉じられると、`start`関数は`true`を返し、次のステップに進みます。  
    - `task_worker_num` を設定すると対応する数の[Taskプロセス](/learn?id=taskworkerプロセス)が増加します。   
    - メソッドリスト内の`start`より前のメソッドは、`start`呼び出し前にのみ使用でき、`start`より後のメソッドは[onWorkerStart](/server/events?id=onworkerstart)、[onReceive](/server/events?id=onreceive)などのイベントコールバック内でのみ使用できます。

  * **拡張**
  
    * Master 主プロセス

      * 主プロセス内に複数の[Reactor](/learn?id=reactorスレッド)スレッドがあり、`epoll/kqueue/select`に基づいてネットワークイベントをポーリングします。データを受信した後、`Worker`プロセスに処理を委譲します。
    
    * Manager プロセス

      * すべての`Worker`プロセスを管理し、`Worker`プロセスの寿命が終了したりエラーが発生した場合は自動的にリサイクルし、新しい`Worker`プロセスを作成します。
    
    * Worker プロセス

      * 受信したデータを処理し、プロトコル解析とリクエストへの応答を含みます。`worker_num`を設定しない場合、下位レベルでは`CPU`の数と同じ数の`Worker`プロセスが起動します。
      * 起動に失敗した場合は、この拡張で致命的なエラーがスローされますので、関連する`php error_log`の情報を確認してください。`errno={number}` は標準の`Linux Errno` であり、関連文書を参照できます。
      * `log_file`を設定している場合、情報は指定された`Log`ファイルに出力されます。

  * **戻り値**

    * `true`を返すと操作に成功し、`false`を返すと操作に失敗します。

  * **起動に失敗する一般的なエラー**

    * `bind`に失敗しました。他のプロセスがこのポートをすでに使用しているため。
    * 必須のコールバック関数が設定されていないため、起動に失敗しました。
    * `PHP`コードに致命的なエラーがあります。PHPのエラー情報`php_errors.log`を確認してください。
    * `ulimit -c unlimited`を実行し、`core dump`を有効にして、セグメンテーション違反があるかどうかを確認してください。
    * `daemonize`を無効にし、`log`を閉じて、エラー情報を画面に出力できるようにします。
## reload()

安全地 (す_ふ語_) 重启所有Worker/Task进程。

```php
Swoole\Server->reload(bool $only_reload_taskworker = false): bool
```

!> 例: 一台繁忙なバックエンドサーバーはいつもリクエストを処理しています。管理者が`kill`プロセスの方法でサーバープログラムを終了/再起動しようとすると、ちょうどコードが途中まで実行された場合があります。  
このような状況ではデータの不整合が発生します。たとえば、取引システムでは、支払いロジックの次には出荷があります。支払いロジックの後でプロセスが終了されたと仮定すると、ユーザーが通貨を支払ったが商品が発送されていない可能性があり、重大な結果が生じます。  
`Swoole`は柔軟な終了/再起動メカニズムを提供し、管理者は特定のシグナルを`Server`に送信するだけで`Server`の`Worker`プロセスを安全に終了させることができます。参考: [サービスを正しく再起動する方法](/question/use?id=swoole如何正确的重启服务)。

  * **パラメータ**

    * `bool $only_reload_taskworker`
      
      * 機能: [Taskプロセス](/learn?id=taskworker进程)を単に再起動するかどうか
      * デフォルト値: false
      * その他の値: true

!> - `reload`には保護機構があり、一度の`reload`が進行中の場合、新しい再起動シグナルを破棄します。
- `user/group`が設定されている場合、`Worker`プロセスに`master`プロセスに情報を送信する権限がないかもしれません。この場合、`root`アカウントで`shell`で`kill`コマンドを実行する必要があります。
- `reload`命令は[userProcess](/server/methods?id=addProcess)に追加したユーザープロセスには無効です。

  * **返り値**
  
    * `true`を返すと操作が成功したことを示し、`false`を返すと操作が失敗したことを示します。

  * **拡張**

    * **シグナルの送信**
    
      * `SIGTERM`: マスタープロセス/管理プロセスにこのシグナルを送信すると、サーバーは安全に終了します。
      * PHPコードで`$serv->shutdown()`を呼び出すことで、この操作を完了できます。
      * `SIGUSR1`: マスタープロセス/管理プロセスに`SIGUSR1`シグナルを送信すると、すべての`Worker`プロセスと`TaskWorker`プロセスを平和に`restart`できます。
      * `SIGUSR2`: マスタープロセス/管理プロセスに`SIGUSR2`シグナルを送信すると、すべての`Task`プロセスを平和に再起動できます。
      * PHPコードで`$serv->reload()`を呼び出すことで、この操作を完了できます。

    ```shell
    # すべてのworkerプロセスを再起動する
    kill -USR1 マスタープロセスPID
    
    # taskプロセスのみを再起動する
    kill -USR2 マスタープロセスPID
    ```
    
      > [参考: Linuxシグナルリスト](/other/signal)

    * **Processモード**
    
        `Process`で起動されたプロセスでは、クライアントからの`TCP`接続は`Master`プロセス内で維持され、`Worker`プロセスの再起動や異常終了は接続自体に影響しません。

    * **Baseモード**
    
        `Base`モードでは、クライアント接続は直接`Worker`プロセス内で維持されるため、`reload`時にすべての接続が切断されます。

    !> `Base`モードは[Taskプロセス](/learn?id=taskworker进程)の`reload`をサポートしていません。
    
    * **Reloadの有効範囲**
      
      `Reload`操作は、`Worker`プロセスが起動した後に読み込まれたPHPファイルのみを再ロードできます。`get_included_files`関数を使用して、`WorkerStart`の前に読み込まれたPHPファイルがどれかをリストアップすることで、このリスト内のPHPファイルは再ロードされても再度読み込まれません。再起動してサーバーを再起動することで有効になります。

    ```php
    $serv->on('WorkerStart', function(Swoole\Server $server, int $workerId) {
        var_dump(get_included_files()); //この配列に含まれるファイルはプロセス開始前に読み込まれたものなので、reloadで再読み込みされません
    });
    ```

    * **APC/OPcache**
    
      `PHP`が`APC/OPcache`を有効にしている場合、`reload`での再ロードが影響を受けます。解決策は2つあります。
      
      * `APC/OPcache`の`stat`チェックを有効にし、ファイルが更新された場合、`APC/OPcache`が自動的に`OPCode`を更新します。
      * `onWorkerStart`でファイルをロード(require, includeなどの関数)する前に`apc_clear_cache`または`opcache_reset`を実行して`OPCode`キャッシュをリフレッシュします。

  * **注意**

    !> - 平滑な再起動は[onWorkerStart](/server/events?id=onworkerstart)や[onReceive](/server/events?id=onreceive)などの`Worker`プロセス内で`include/require`されたPHPファイルにのみ有効です。
    - `Server`が起動する前に`include/require`されたPHPファイルは平滑な再起動で再度読み込むことができません。
    - `Server`の設定（すなわち`$serv->set()`で渡されたパラメータ設定）は、`Server`全体を再起動しないと再読み込みできません。
    - `Server`は内部ネットワークポートをリッスンし、リモート制御コマンドを受け付け、すべての`Worker`プロセスを再起動できます。
## stop()

現在の`Worker`プロセスを停止し、すぐに`onWorkerStop`コールバック関数をトリガーします。

```php
Swoole\Server->stop(int $workerId = -1, bool $waitEvent = false): bool
```

  * **引数**

    * `int $workerId`

      * 機能：`worker id`を指定します
      * デフォルト値：-1（現在のプロセスを表す）
      * その他の値：なし

    * `bool $waitEvent`

      * 機能：終了戦略を制御します。`false`は即時終了を意味し、`true`はイベントループが空になるまで終了を待機します
      * デフォルト値：false
      * その他の値：true

  * **戻り値**

    * `true`を返すと操作成功、`false`を返すと操作失敗

  * **ヒント**

    !> -[非同期IO](/learn?id=同步io异步io)サーバーは`stop`を呼び出しても、まだ待っているイベントがあるかもしれません。例えば`Swoole\MySQL->query`を使用して`SQL`クエリを送信したが、まだ`MySQL`サーバーからの結果を待っている場合です。この状況でプロセスを強制終了させると、`SQL`の実行結果が失われます。  
    -`$waitEvent = true`に設定すると、下層は[非同期セーフ再起動](/question/use?id=swoole如何正确的重启服务)戦略を使用します。まず`Manager`プロセスに通知して、新しいリクエストを処理するための新しい`Worker`を再起動します。古い`Worker`はイベントを待ち、イベントループが空になるか`max_wait_time`を超えるまで、プロセスを終了させ、非同期イベントの安全性を最大限に保持します。
## shutdown()

サービスをシャットダウンします。

```php
Swoole\Server->shutdown(): bool
```

  * **Return Value**

    * `true`を返すと操作が成功したことを示し、`false`を返すと操作が失敗したことを示します。

  * **Tips**

    * この関数は`Worker`プロセス内で使用できます。
    * マスタープロセスに`SIGTERM`を送信することでもサービスのシャットダウンを実現できます。

```shell
kill -15 マスタープロセスのPID
```
## tick()

`tick`タイマーを追加し、カスタムコールバック関数を設定できます。この関数は [Swoole\Timer::tick](/timer?id=tick) のエイリアスです。

```php
Swoole\Server->tick(int $millisecond, callable $callback): void
```

  * **Parameters**

    * `int $millisecond`

      * 機能：間隔時間【ミリ秒】
      * デフォルト値：なし
      * その他の値：なし

    * `callable $callback`

      * 機能：コールバック関数
      * デフォルト値：なし
      * その他の値：なし

  * **注意**
  
    !> - `Worker`プロセスが終了すると、すべてのタイマーが自動的に破棄されます  
    - `tick/after`タイマーは`Server->start`の前に使用できません  
    - `Swoole5`以降、このエイリアスの使用方法は削除されましたので、直接`Swoole\Timer::tick()`を使用してください。

  * **例**

    * [onReceive](/server/events?id=onreceive) 内で使用

    ```php
    function onReceive(Swoole\Server $server, int $fd, int $reactorId, mixed $data)
    {
        $server->tick(1000, function () use ($server, $fd) {
            $server->send($fd, "hello world");
        });
    }
    ```

    * [onWorkerStart](/server/events?id=onworkerstart) 内で使用

    ```php
    function onWorkerStart(Swoole\Server $server, int $workerId)
    {
        if (!$server->taskworker) {
            $server->tick(1000, function ($id) {
              var_dump($id);
            });
        } else {
            //task
            $server->tick(1000);
        }
    }
    ```
```php
Swoole\Server->after(int $millisecond, callable $callback)
```  

  * **Parameters**

    * `int $millisecond`

      * Function: Execution time in milliseconds
      * Default: None
      * Other values: None
      * Version impact: In versions below `Swoole v4.2.10`, maximum value must not exceed `86400000`

    * `callable $callback`

      * Function: Callback function, must be callable, `callback` function does not accept any parameters
      * Default: None
      * Other values: None

  * **Note**
  
    !> -The lifecycle of timers is at the process level. When a process is restarted or killed using `reload` or `kill`, all timers will be destroyed  
    -If certain timers have critical logic and data, please implement them in the `onWorkerStop` callback function or refer to [How to restart the service correctly](/question/use?id=swoole如何正确的重启服务)  
    -After `Swoole5`, this alias usage has been removed, please use `Swoole\Timer::after()` directly
```
## defer()

[Swoole\Event::defer](/event?id=defer)のエイリアスで、関数を遅延実行します。

```php
Swoole\Server->defer(Callable $callback): void
```

  * **パラメータ**

    * `Callable $callback`

      * 機能：コールバック関数【必須】、実行可能な関数変数、文字列、配列、無名関数のいずれか
      * デフォルト値：なし
      * その他の値：なし

  * **注意**

    !> -この関数は、[EventLoop](/learn?id=什么是eventloop)ループが完了した後に実行されます。この関数は、一部のPHPコードを遅れさせて実行し、プログラムが他の`IO`イベントを優先的に処理できるようにします。たとえば、あるコールバック関数がCPU集約型計算でかつ時間が余裕がある場合、プロセスが他のイベントを処理した後にCPU集約型計算を行います  
    -`defer`された関数はすぐに実行されることを保証しません。システムの重要なロジックである場合は、速やかに実行する必要がある場合は、`after`タイマーを使用して実現してください  
    -`onWorkerStart`コールバック内で`defer`を実行する場合、イベントが発生するまでコールバックが行われません  
    -`Swoole5`以降、このエイリアスの使用法は削除されました。代わりに`Swoole\Event::defer()`を直接使用してください

  * **例**

```php
function query($server, $db) {
    $server->defer(function() use ($db) {
        $db->close();
    });
}
```
## clearTimer()

`tick/after`タイマーをクリアします。この関数は [Swoole\Timer::clear](/timer?id=clear) の別名です。

```php
Swoole\Server->clearTimer(int $timerId): bool
```

  * **Parameters**

    * `int $timerId`

      * Description：タイマーIDを指定します
      * Default value：なし
      * Other values：なし

  * **Return Value**

    * `true` を返すと操作が成功し、`false` を返すと操作が失敗します

  * **Note**

    !> - `clearTimer` は現在のプロセスのタイマーをクリアするためにのみ使用できます    
    - `Swoole5`以降、この別名の使用方法は削除されました。代わりに `Swoole\Timer::clear()` を直接使用してください

  * **Example**

```php
$timerId = $server->tick(1000, function ($timerId) use ($server) {
    $server->clearTimer($timerId);//$idはタイマーのIDです
});
```
## close()

クライアント接続を閉じます。

```php
Swoole\Server->close(int $fd, bool $reset = false): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：閉じる`fd` (ファイル記述子) を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `bool $reset`

      * 機能：`true`に設定すると、接続を強制的に閉じ、送信キュー内のデータは破棄されます
      * デフォルト値：false
      * その他の値：true

  * **戻り値**

    * `true`を返すと操作が成功、`false`を返すと操作が失敗となります

  * **注意**

  !> -`Server`が`close`をアクティブに呼び出すと、[onClose](/server/events?id=onclose) イベントがトリガーされます  
-`close`の後にクリーンアップロジックを記述しないでください。[onClose](/server/events?id=onclose) コールバックに配置するべきです  
-`HTTP\Server`の`fd`は、上位のコールバックメソッドの`response`から取得されます

  * **例**

```php
$server->on('request', function ($request, $response) use ($server) {
    $server->close($response->fd);
});
```
## send()

データをクライアントに送信します。

```php
Swoole\Server->send(int|string $fd, string $data, int $serverSocket = -1): bool
```

  * **パラメータ**

    * `int|string $fd`

      * 機能：クライアントのファイルディスクリプタまたはunixソケットのパスを指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $data`

      * 機能：送信するデータ、`TCP`プロトコルでは最大で`2M`を超えることはできません。[buffer_output_size](/server/setting?id=buffer_output_size) を変更することで送信可能な最大パケット長を変更できます
      * デフォルト値：なし
      * その他の値：なし

    * `int $serverSocket`

      * 機能：[UnixSocket DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php)の対向先にデータを送信する場合、このパラメータが必要です。TCPクライアントでは記入する必要はありません
      * デフォルト値：-1、現在のUDPポートを表します
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功していることを示し、`false`を返すと操作が失敗していることを示します

  * **ヒント**

    !> 送信プロセスは非同期です。下部では自動的に書き込み可能を監視し、データをクライアントに逐次送信します。つまり、`send`が戻った後、対向先が直ちにデータを受信しているわけではありません。

    * セキュリティ
      * `send`操作は原子性を持ち、複数のプロセスが同時に`TCP`接続に対して`send`を呼び出すと、データの混在は発生しません

    * 長さ制限
      * `2M`を超えるデータを送信する場合、データを一時ファイルに書き込み、`sendfile`インターフェースを使用して送信できます
      * [buffer_output_size](/server/setting?id=buffer_output_size) パラメータを設定することで送信長さの制限を変更できます
      * `8K`以上のデータを送信する場合、下層では`Worker`プロセスの共有メモリが有効になり、`Mutex->lock`操作が必要です

    * バッファリング
      * `Worker`プロセスの[unixSocket](/learn?id=什么是IPC) バッファがいっぱいになった場合、`8K`のデータを送信すると一時ファイルが使用されます
      * 同じクライアントに連続して多量のデータを送信すると、クライアントが受信する暇がなく`Socket`メモリバッファが一杯になる可能性があります。この場合、Swooleの下部は即座に`false`を返し、`false`が返された場合はデータをディスクに保存し、クライアントが送信済みのデータを受信した後に再び送信します

    * [コルーチンスケジュール](/coroutine?id=协程调度)
      * `send`はバッファがいっぱいになった場合、`send_yield`が有効な場合、自動的に一時停止し、対向先が一部のデータを読み取った後にコルーチンを復元し、データ送信を続行します。

    * [UnixSocket](/learn?id=什么是IPC)
      * [UnixSocket DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) ポートを監視している場合、`send`を使用して対向にデータを送信できます。

      ```php
      $server->on("packet", function (Swoole\Server $server, $data, $addr){
          $server->send($addr['address'], 'SUCCESS', $addr['server_socket']);
      });
      ```
## sendfile()

`TCP`クライアント接続にファイルを送信します。

```php
Swoole\Server->sendfile(int $fd, string $filename, int $offset = 0, int $length = 0): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：クライアントのファイルディスクリプタを指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $filename`

      * 機能：送信するファイルのパス。ファイルが存在しない場合は`false`を返します
      * デフォルト値：なし
      * その他の値：なし

    * `int $offset`

      * 機能：ファイルのオフセットを指定し、ファイルの特定の位置からデータを送信できます
      * デフォルト値：0 【デフォルトは`0`で、ファイルの先頭から送信を開始します】
      * その他の値：なし

    * `int $length`

      * 機能：送信する長さを指定します
      * デフォルト値：ファイルのサイズ
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します

  * **注意**

  !> この関数は`Server->send`と同様にデータをクライアントに送信しますが、`sendfile`は指定されたファイルからデータを取得します
## sendto()

任意のクライアント`IP:PORT`に`UDP`データパケットを送信する。

```php
Swoole\Server->sendto(string $ip, int $port, string $data, int $serverSocket = -1): bool
```

  * **パラメータ**

    * `string $ip`

      * 機能：クライアントの`IP`を指定する
      * デフォルト値：なし
      * その他の値：なし

      ?> `$ip`は`IPv4`または`IPv6`の文字列である必要があります。無効な`IP`の場合、エラーが返されます。

    * `int $port`

      * 機能：クライアントの`port`を指定する
      * デフォルト値：なし
      * その他の値：なし

      ?> `$port`は `1-65535`のネットワークポート番号である必要があります。ポートが誤っている場合、送信に失敗します。

    * `string $data`

      * 機能：送信するデータの内容。テキストまたはバイナリ内容を指定できます。
      * デフォルト値：なし
      * その他の値：なし

    * `int $serverSocket`

      * 機能：データパケットを送信する際に使用する対応するポート`server_socket`ディスクリプタを指定する【`$clientInfo`で取得できる[onPacketイベント](/server/events?id=onpacket)】
      * デフォルト値：-1、現在の監視中のUDPポートを意味する
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功したことを示し、`false`を返すと操作が失敗したことを示す

      ?> サーバーは複数の`UDP`ポートを同時に監視するかもしれません。[複数ポート監視](/server/port)を参照し、このパラメータでデータパケットを送信するポートを指定できます。

  * **注意**

  !> `IPv4`アドレスにデータを送信するには`UDP`ポートを監視する必要があります。  
  `IPv6`アドレスにデータを送信するには`UDP6`ポートを監視する必要があります。

  * **例**

```php
// IPアドレスが220.181.57.216のホストの9502ポートに"hello world"という文字列を送信する。
$server->sendto('220.181.57.216', 9502, "hello world");
// IPv6サーバーにUDPデータパケットを送信する
$server->sendto('2600:3c00::f03c:91ff:fe73:e98f', 9501, "hello world");
```
## sendwait()

データをクライアントに同期的に送信します。

```php
Swoole\Server->sendwait(int $fd, string $data): bool
```

  * **パラメーター**

    * `int $fd`

      * 機能：クライアントのファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $data`

      * 機能：送信するデータ
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作成功、`false`を返すと操作失敗

  * **ヒント**

    * 特定のシナリオでは、`Server`はクライアントに連続してデータを送信する必要がありますが、`Server->send`データ送信メソッドは純粋に非同期です。大量のデータ送信はメモリ送信キューを詰まらせる可能性があります。

    * `Server->sendwait`を使用すると、この問題を解決できます。`Server->sendwait`は接続が書き込み可能になるまで待機します。データ送信が完了すると戻ります。

  * **注意**

  !> `sendwait`は現在[SWOOLE_BASE](/learn?id=swoole_base)モードでのみ使用可能です  
  `sendwait`はローカルネットワークまたは内部ネットワーク通信にのみ使用され、外部接続では`sendwait`を使用しないでください。`enable_coroutine`=>true(デフォルトで有効)のときもこの関数を使用しないでください。他のコルーチンをブロックする可能性があります。同期ブロッキングサーバーのみが使用できます。
## sendMessage()

任意の`Worker`プロセスまたは [Taskプロセス](/learn?id=taskworkerプロセス)にメッセージを送信します。メインプロセスや管理プロセス以外で呼び出すことができます。メッセージを受け取ったプロセスは`onPipeMessage`イベントをトリガーします。

```php
Swoole\Server->sendMessage(mixed $message, int $workerId): bool
```

  * **パラメータ**

    * `mixed $message`

      * 機能：送信するメッセージのデータ内容です。長さに制限はありませんが、`8K`を超える場合は一時的なメモリファイルが作成されます。
      * デフォルト値：なし
      * その他の値：なし

    * `int $workerId`

      * 機能：ターゲットプロセスの`ID`を指定します。範囲は[$worker_id](/server/properties?id=worker_id)を参照してください。
      * デフォルト値：なし
      * その他の値：なし

  * **ヒント**

    * `Worker`プロセス内で`sendMessage`を呼び出すと、それは[非同期IO](/learn?id=同期io非同期io)となり、メッセージはまずバッファに保存され、書き込み可能な状態であれば、このメッセージが[unixSocket](/learn?id=IPC)に送信されます。
    * [Taskプロセス](/learn?id=taskworkerプロセス)内で`sendMessage`を呼び出すと、通常は[同期IO](/learn?id=同期io非同期io)ですが、一部の場合は自動的に非同期IOに変換されることがあります。[同期IOを非同期IOに変換](/learn?id=同期ioを非同期ioに変換)を参照してください。
    * [Userプロセス](/server/methods?id=addprocess)内で`sendMessage`をTaskと同様に呼び出すと、デフォルトで同期ブロッキングとなります。[同期IOを非同期IOに変換](/learn?id=同期ioを非同期ioに変換)を参照してください。

  * **注意**

  !> - `sendMessage()`が[非同期IO](/learn?id=同期ioを非同期ioに変換)である場合、相手プロセスが何らかの理由でデータを受信しない場合、`sendMessage()`を繰り返し呼び出すことは避けてください。これにより大量のメモリリソースが消費される可能性があります。応答メカニズムを追加して、相手が応答しない場合には呼び出しを一時停止してください。  
-`MacOS/FreeBSD`では、`2K`を超えると一時ファイルにデータが保存されます。  
-[sendMessage](/server/methods?id=sendMessage)を使用する場合は、必ず`onPipeMessage`イベントのコールバック関数を登録してください。  
- [task_ipc_mode](/server/setting?id=task_ipc_mode)を`3`に設定すると、特定のタスクプロセスにメッセージを送信することができなくなります。

  * **例**

```php
$server = new Swoole\Server('0.0.0.0', 9501);

$server->set(array(
    'worker_num'      => 2,
    'task_worker_num' => 2,
));
$server->on('pipeMessage', function ($server, $src_worker_id, $data) {
    echo "#{$server->worker_id} message from #$src_worker_id: $data\n";
});
$server->on('task', function ($server, $task_id, $src_worker_id, $data) {
    var_dump($task_id, $src_worker_id, $data);
});
$server->on('finish', function ($server, $task_id, $data) {

});
$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
    if (trim($data) == 'task') {
        $server->task("async task coming");
    } else {
        $worker_id = 1 - $server->worker_id;
        $server->sendMessage("hello task process", $worker_id);
    }
});

$server->start();
```
## exist()

`fd`に対応する接続が存在するかどうかを検出します。

```php
Swoole\Server->exist(int $fd): bool
```

  * **Parameters**

    * `int $fd`

      * 機能: ファイル記述子
      * デフォルト値: なし
      * 他の値: なし

  * **Return Value**

    * `true`を返すと存在することを示し、`false`を返すと存在しないことを示します

  * **Note**
  
    * このインターフェースは共有メモリを使用して計算されるため、`IO`操作は行われません
## pause()

データの受信を停止します。

```php
Swoole\Server->pause(int $fd): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：指定されたファイル記述子
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します

  * **ヒント**

    * この関数を呼び出すと、接続が[EventLoop](/learn?id=什么是eventloop)から削除され、クライアントからのデータを受け取らなくなります。
    * この関数は送信キューの処理に影響を与えません
    * `SWOOLE_PROCESS`モードでのみ使用できます。`pause`を呼び出した後、一部のデータが`Worker`プロセスに到達している可能性があるため、引き続き[onReceive](/server/events?id=onreceive)イベントが発生する可能性があります
## resume()

データの受信を再開します。`pause` メソッドとペアで使用されます。

```php
Swoole\Server->resume(int $fd): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：ファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true` を返すと操作は成功、`false` を返すと操作は失敗です

  * **ヒント**

    * この関数を呼び出すと、接続が再び[EventLoop](/learn?id=什么是eventloop)に追加され、クライアントからのデータの受信を続けます
## getCallback()

Serverの指定された名前のコールバック関数を取得します。

```php
Swoole\Server->getCallback(string $event_name): \Closure|string|null|array
```

  * **パラメータ**

    * `string $event_name`

      * 機能：イベントの名前。`on`を追加する必要はありません。大文字と小文字は区別されません。
      * デフォルト値：なし
      * 他の値：[イベント](/server/events)を参照してください。

  * **戻り値**

    * 対応するコールバック関数が存在する場合は、異なる[コールバック関数の設定方法](/learn?id=四种设置回调函数的方式)に基づいて `Closure` / `string` / `array` を返します。
    * 対応するコールバック関数が存在しない場合は、`null` を返します。
## getClientInfo()

接続情報を取得します。別名は`Swoole\Server->connection_info()`です

```php
Swoole\Server->getClientInfo(int $fd, int $reactorId = -1, bool $ignoreError = false): false|array
```

  * **パラメータ**

    * `int $fd`

      * 機能：ファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `int $reactorId`

      * 機能：接続が存在する[Reactor](/learn?id=reactor线程)スレッドの`ID`です。現在は何の影響もありません。API互換性を維持するためだけのものです
      * デフォルト値：-1
      * その他の値：なし

    * `bool $ignoreError`

      * 機能：エラーを無視するかどうか。`true`に設定すると、接続が閉じていても接続情報を返します。`false`は接続が閉じている場合は`false`を返します
      * デフォルト値：false
      * その他の値：なし

  * **ヒント**

    * クライアント証明書

      * [onConnect](/server/events?id=onconnect)でトリガーされたプロセス内でのみ証明書を取得できます
      * 形式は`x509`形式で、`openssl_x509_parse`関数を使用して証明書情報を取得できます

    * [dispatch_mode](/server/setting?id=dispatch_mode) = 1/3 を使用する場合、このようなデータパケット配信戦略はステートレスサービスに使用されるため、接続が切断された後、関連情報は直接メモリから削除されます。そのため、`Server->getClientInfo`では関連する接続情報を取得できません。

  * **戻り値**

    * 失敗した場合は`false`を返します
    * 成功した場合は、クライアント情報を含む`array`を返します

```php
$fd_info = $server->getClientInfo($fd);
var_dump($fd_info);

array(15) {
  ["server_port"]=>
  int(9501)
  ["server_fd"]=>
  int(4)
  ["socket_fd"]=>
  int(25)
  ["socket_type"]=>
  int(1)
  ["remote_port"]=>
  int(39136)
  ["remote_ip"]=>
  string(9) "127.0.0.1"
  ["reactor_id"]=>
  int(1)
  ["connect_time"]=>
  int(1677322106)
  ["last_time"]=>
  int(1677322106)
  ["last_recv_time"]=>
  float(1677322106.901918)
  ["last_send_time"]=>
  float(0)
  ["last_dispatch_time"]=>
  float(0)
  ["close_errno"]=>
  int(0)
  ["recv_queued_bytes"]=>
  int(78)
  ["send_queued_bytes"]=>
  int(0)
}
```

パラメータ | 役割
---|---
server_port | サーバーのリッスンポート
server_fd | サーバーのfd
socket_fd | クライアントのfd
socket_type | ソケットタイプ
remote_port | クライアントのポート
remote_ip | クライアントのIP
reactor_id | どのリアクタースレッドから来たか
connect_time | クライアントがサーバーに接続した時間（秒単位、マスタープロセスによって設定されます）
last_time | 最後にデータを受信した時間（秒単位、マスタープロセスによって設定）
last_recv_time | 最後にデータを受信した時間（秒単位、マスタープロセスによって設定）
last_send_time | 最後にデータを送信した時間（秒単位、マスタープロセスによって設定）
last_dispatch_time | ワーカープロセスがデータを受信した時間
close_errno | 接続が閉じられた際のエラーコード、接続が異常に閉じられた場合、close_errnoの値はゼロ以外となります。Linuxエラー情報リストを参照できます
recv_queued_bytes | 処理待ちのデータ量
send_queued_bytes | 送信待ちのデータ量
websocket_status | [Optional] WebSocket接続状態、サーバーがSwoole\WebSocket\Serverの場合にこの情報が追加されます
uid | [Optional] ユーザーIDをバインドした場合にこの情報が追加されます
ssl_client_cert | [Optional] SSLトンネル暗号を使用し、かつクライアントが証明書を設定した場合にこの情報が追加されます
## getClientList()

現在の`Server`のすべてのクライアント接続を反復処理する`Server::getClientList`メソッドは、共有メモリに基づいており、`IOWait`は存在せず、反復処理のスピードは非常に速いです。また`getClientList`は、現在の`Worker`プロセスの`TCP`接続だけでなく、すべての`TCP`接続を返します。別名は`Swoole\Server->connection_list()`

```php
Swoole\Server->getClientList(int $start_fd = 0, int $pageSize = 10): false|array
```

  * **Parameters**

    * `int $start_fd`

      * 機能：開始`fd`を指定する
      * デフォルト値：0
      * 他の値：なし

    * `int $pageSize`

      * 機能：1ページあたりの取得数を指定する。最大100を超えてはならない
      * デフォルト値：10
      * 他の値：なし

  * **Return Value**

    * 呼び出し成功時、取得した`$fd`の数値インデックス配列が返されます。配列は小さい順にソートされます。最後の`$fd`は新しい`start_fd`として再取得します
    * 呼び出し失敗時は`false`が返されます

  * **Tips**

    * `Server::$connections`イテレーターを使用して接続を反復処理することをお勧めします
    * `getClientList`は`TCP`クライアントにのみ使用でき、`UDP`サーバーではクライアント情報を自分で保存する必要があります
    * [SWOOLE_BASE](/learn?id=swoole_base)モードでは現在のプロセスの接続のみを取得できます

  * **Example**
  
```php
$start_fd = 0;
while (true) {
  $conn_list = $server->getClientList($start_fd, 10);
  if ($conn_list === false || count($conn_list) === 0) {
      echo "finish\n";
      break;
  }
  $start_fd = end($conn_list);
  var_dump($conn_list);
  foreach ($conn_list as $fd) {
      $server->send($fd, "broadcast");
  }
}
```
## bind()

`bind`メソッドはユーザーが定義した`UID`に接続をバインドし、[dispatch_mode](/server/setting?id=dispatch_mode) = 5と設定することでこの値を使用して`hash`ベースの固定分配を行うことができます。これにより、特定の`UID`の接続がすべて同じ`Worker`プロセスに割り当てられることが保証されます。

```php
Swoole\Server->bind(int $fd, int $uid): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：接続の`fd`を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `int $uid`

      * 機能：バインドする`UID`、非`0`の数字である必要があります
      * デフォルト値：なし
      * その他の値：`UID`は最大で`4294967295`を超えず、最小で`-2147483648`より小さくない必要があります

  * **戻り値**

    * `true`を返すと操作が成功したことを示し、`false`を返すと操作が失敗したことを示します

  * **ヒント**

    * `$serv->getClientInfo($fd)`を使用して、接続にバインドされた`UID`の値を確認できます
    * デフォルトの[dispatch_mode](/server/setting?id=dispatch_mode)=2では、`Server`は`socket fd`ごとに異なる`Worker`プロセスに接続データを割り当てます。`fd`は不安定であり、クライアントが切断して再接続すると、`fd`が変わります。そのため、このクライアントのデータは別の`Worker`に割り当てられます。`bind`を使用すると、ユーザーが定義した`UID`に従って割り当てることができます。クライアントが切断して再接続しても、同じ`UID`の`TCP`接続データは同じ`Worker`プロセスに割り当てられます。

    * タイミングの問題

      * クライアントがサーバーに接続し、複数のパケットを連続して送信すると、タイミングの問題が発生する可能性があります。`bind`操作時には、後続のパケットは既に`dispatch`されており、これらのデータパケットは引き続き`fd`に基づいて現在のプロセスに割り当てられます。新たに受信したデータパケットは`UID`に基づいて割り当てられます。
      * そのため、`bind`メカニズムを使用する場合は、ネットワーク通信プロトコルにハンドシェイク手順を設計する必要があります。クライアントが接続した後、まずハンドシェイク要求を送信し、その後クライアントは任意のパケットを送信しないでください。サーバーが`bind`を完了し、応答した後に、クライアントが新しいリクエストを送信します。 

    * 再バインド

      * 一部の場合、ビジネスロジックがユーザーの接続を新しい`UID`に再バインドする必要がある場合があります。その場合は、接続を切断し、新しい`TCP`接続を確立してハンドシェイクを行い、新しい`UID`にバインドします。

    * 負の`UID`のバインド

      * バインドされた`UID`が負の場合、これは内部的に`32ビット符号なし整数`に変換され、PHPレベルで`32ビット符号付き整数`に変換する必要があります。以下のように使用できます：

  ```php
  $uid = -10;
  $server->bind($fd, $uid);
  $bindUid = $server->connection_info($fd)['uid'];
  $bindUid = $bindUid >> 31 ? (~($bindUid - 1) & 0xFFFFFFFF) * -1 : $bindUid;
  var_dump($bindUid === $uid);
  ```

  * **注意**

!> - `dispatch_mode=5`に設定されている場合にのみ有効です  
- `UID`がバインドされていない場合、デフォルトでは`fd`に基づいて割り当てが行われます  
- 同じ接続は一度だけ`bind`できます。すでに`UID`がバインドされている場合は、再度`bind`を呼び出しても`false`が返されます

  * **例**

```php
$serv = new Swoole\Server('0.0.0.0', 9501);

$serv->fdlist = [];

$serv->set([
    'worker_num' => 4,
    'dispatch_mode' => 5,   //uid dispatch
]);

$serv->on('connect', function ($serv, $fd, $reactor_id) {
    echo "{$fd} connect, worker:" . $serv->worker_id . PHP_EOL;
});

$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {
    $conn = $serv->connection_info($fd);
    print_r($conn);
    echo "worker_id: " . $serv->worker_id . PHP_EOL;
    if (empty($conn['uid'])) {
        $uid = $fd + 1;
        if ($serv->bind($fd, $uid)) {
            $serv->send($fd, "bind {$uid} success");
        }
    } else {
        if (!isset($serv->fdlist[$fd])) {
            $serv->fdlist[$fd] = $conn['uid'];
        }
        print_r($serv->fdlist);
        foreach ($serv->fdlist as $_fd => $uid) {
            $serv->send($_fd, "{$fd} say:" . $data);
        }
    }
});

$serv->on('close', function ($serv, $fd, $reactor_id) {
    echo "{$fd} Close". PHP_EOL;
    unset($serv->fdlist[$fd]);
});

$serv->start();
```
`stats()`関数は、現在の`Server`のアクティブな`TCP`接続数、起動時間などの情報、`accept/close`（接続の確立/切断）の総数などを取得します。

```php
Swoole\Server->stats(): array
```

* **例**

```php
array(25) {
  ["start_time"]=>
  int(1677310656)
  ["connection_num"]=>
  int(1)
  ["abort_count"]=>
  int(0)
  ["accept_count"]=>
  int(1)
  ["close_count"]=>
  int(0)
  ["worker_num"]=>
  int(2)
  ["task_worker_num"]=>
  int(4)
  ["user_worker_num"]=>
  int(0)
  ["idle_worker_num"]=>
  int(1)
  ["dispatch_count"]=>
  int(1)
  ["request_count"]=>
  int(0)
  ["response_count"]=>
  int(1)
  ["total_recv_bytes"]=>
  int(78)
  ["total_send_bytes"]=>
  int(165)
  ["pipe_packet_msg_id"]=>
  int(3)
  ["session_round"]=>
  int(1)
  ["min_fd"]=>
  int(4)
  ["max_fd"]=>
  int(25)
  ["worker_request_count"]=>
  int(0)
  ["worker_response_count"]=>
  int(1)
  ["worker_dispatch_count"]=>
  int(1)
  ["task_idle_worker_num"]=>
  int(4)
  ["tasking_num"]=>
  int(0)
  ["coroutine_num"]=>
  int(1)
  ["coroutine_peek_num"]=>
  int(1)
  ["task_queue_num"]=>
  int(1)
  ["task_queue_bytes"]=>
  int(1)
}
```

パラメータ | 説明
---|---
start_time | サーバーの起動時間
connection_num | 現在の接続数
abort_count | 拒否された接続数
accept_count | 受け入れられた接続数
close_count | 閉じられた接続数
worker_num  | ワーカーのプロセス数
task_worker_num  | タスクワーカーのプロセス数【`v4.5.7`以降利用可能】
user_worker_num  | ユーザー作業者のプロセス数
idle_worker_num | アイドル状態のワーカープロセス数
dispatch_count | サーバーがワーカーに送信したパケットの数【`v4.5.7`以降、[SWOOLE_PROCESS](/learn?id=swoole_process)モードのみ有効】
request_count | サーバーが受信したリクエストの回数【onReceive、onMessage、onRequset、onPacketのみリクエストが計算される】
response_count | サーバーが返したレスポンスの回数
total_recv_bytes| データ受信の合計バイト数
total_send_bytes | データ送信の合計バイト数
pipe_packet_msg_id | プロセス間通信ID
session_round | 開始セッションID
min_fd | 最小の接続FD
max_fd | 最大の接続FD
worker_request_count | 現在のワーカープロセスが受け取ったリクエストの回数【worker_request_countがmax_requestを超えるとワーカープロセスが終了します】
worker_response_count | 現在のワーカープロセスのレスポンス回数
worker_dispatch_count | マスタープロセスが現在のワーカープロセスにタスクをディスパッチした回数、[マスタープロセス](/learn?id=reactor糸)がディスパッチを行うとカウントが増加します
task_idle_worker_num | アイドル状態のタスクプロセス数
tasking_num | 実行中のタスクプロセス数
coroutine_num | 現在のコルーチン数【コルチン用】、詳細な情報を取得するには[このセクション](/coroutine/gdb)を参照してください
coroutine_peek_num | すべてのコルーチン数
task_queue_num | タスクのメッセージキュー内のタスク数【タスク用】
task_queue_bytes | タスクキューのメモリ使用量（バイト）【タスク用】
## taskwait()

`taskwait`関数は、`task`関数と同等の機能を持っており、非同期のタスクを [タスクワーカープロセス](/learn?id=taskworkerプロセス)プールに投稿します。`task`関数と異なる点は、`taskwait`は同期的に待機し、タスクが完了するかタイムアウトするまで返ります。`$result`はタスクの実行結果であり、`$server->finish`関数から送出されます。タスクがタイムアウトした場合、ここでは`false`が返されます。

```php
Swoole\Server->taskwait(mixed $data, float $timeout = 0.5, int $dstWorkerId = -1): mixed
```

  * **引数**

    * `mixed $data`

      * 機能: 投稿するタスクデータ、任意の型であり、文字列でない場合は自動的にシリアライズされます
      * デフォルト値: なし
      * その他の値: なし

    * `float $timeout`

      * 機能: タイムアウト時間、浮動小数点数、秒単位、最小`1ms`の粒度がサポートされており、指定された時間内に [タスクワーカープロセス](/learn?id=taskworkerプロセス)がデータを返さない場合、`taskwait`は`false`を返し、以降のタスクの結果データを処理しません
      * デフォルト値: 0.5
      * その他の値: なし

    * `int $dstWorkerId`

      * 機能: どの [タスクワーカープロセス](/learn?id=taskworkerプロセス)に投稿するかを指定します。Taskプロセスの`ID`を渡します。範囲は`[0, $server->setting['task_worker_num']-1]`
      * デフォルト値: -1【デフォルトは`-1`で、ランダムに投稿され、空いている [タスクワーカープロセス](/learn?id=taskworkerプロセス)が自動的に選択されます】
      * その他の値: `[0, $server->setting['task_worker_num']-1]`

  *  **戻り値**

      * `false`は投稿に失敗したことを示します
      * `onTask`イベントで`finish`メソッドまたは`return`を実行した場合、`taskwait`は`onTask`で投稿された結果を返します。

  * **ヒント**

    * **コルーチンモード**

      * `4.0.4`バージョンから`taskwait`メソッドは [コルーチンスケジューリング](/coroutine?id=協力的マルチタスク)をサポートし、コルーチン内で`Server->taskwait()`を呼び出すと、自動的に[コルーチンスケジューリング](/coroutine?id=協力的マルチタスク)が行われ、ブロックされることはありません。
      * [コルーチンスケジューラ](/coroutine?id=協力的マルチタスク)を利用して、`taskwait`は並行呼び出しを実現できます。
      * `onTask`イベントには、`finish`メソッドまたは`return`は1つだけ存在する必要があります。それ以外の余分な`return`または`Server->finish`を実行すると、「task[1] has expired」の警告が表示されます。

    * **同期モード**

      * 同期ブロッキングモードでは、`taskwait`は[Unixソケット](/learn?id=IPC)通信と共有メモリを使用して、データを`Worker`プロセスに返します。このプロセスは同期ブロッキングされます。

    * **特殊な場合**

      * [onTask](/server/events?id=ontask)に同期IO操作がない場合、プロセス切り替えが2回だけ行われ、IO待ちが発生しません。そのため、このケースでは `taskwait` は非ブロッキングと見なすことができます。実際のテストでは、[onTask](/server/events?id=ontask)で`PHP`配列の読み書きのみを行い、`10`万回の`taskwait`操作を行った場合、合計時間はわずか`1`秒であり、1回あたりの平均消費時間は`10`マイクロ秒です。

  * **注意**

  !> -`Swoole\Server::finish`を使用せずに`taskwait`  
-`taskwait`メソッドは [タスクプロセス](/learn?id=taskworkerプロセス)内で呼び出すことはできません
## taskWaitMulti()

複数の`task`非同期タスクを並行して実行します。 このメソッドは[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)はサポートしておらず、他のコルーチンの開始を引き起こす可能性があるため、コルーチン環境では後述の`taskCo`を使用する必要があります。

```php
Swoole\Server->taskWaitMulti(array $tasks, float $timeout = 0.5): false|array
```

  * **パラメータ**

    * `array $tasks`

      * 機能：連番の数字のインデックス配列である必要があり、関連付けられたインデックス配列はサポートされていません。 レイヤーは`$tasks`を反復処理し、各タスクを個別に [タスクプロセス](/learn?id=タスクワーカープロセス) にディスパッチします。
      * デフォルト値：なし
      * その他の値：なし

    * `float $timeout`

      * 機能：浮動小数点数で、単位は秒です。
      * デフォルト値：0.5秒
      * その他の値：なし

  * **戻り値**

    * タスクが完了するかタイムアウトした場合、結果の配列を返します。 結果配列内の各タスクの結果の順序は`$tasks`と対応しており、たとえば：`$tasks[2]`に対応する結果は`$result[2]`です。
    * あるタスクが実行タイムアウトした場合、他のタスクに影響を与えず、返される結果データにはタイムアウトしたタスクは含まれません。

  * **注意**

  !> - 同時最大タスク数は`1024`を超えてはいけません。

  * **例**

```php
$tasks[] = mt_rand(1000, 9999); // タスク1
$tasks[] = mt_rand(1000, 9999); // タスク2
$tasks[] = mt_rand(1000, 9999); // タスク3
var_dump($tasks);

// すべてのTaskの結果が返されるのを待つ、タイムアウトは10秒
$results = $server->taskWaitMulti($tasks, 10.0);

if (!isset($results[0])) {
    echo "タスク1はタイムアウトしました\n";
}
if (isset($results[1])) {
    echo "タスク2の実行結果は{$results[1]}\n";
}
if (isset($results[2])) {
    echo "タスク3の実行結果は{$results[2]}\n";
}
```
## taskCo()

`Task` is executed concurrently and performs coroutine scheduling to support the `taskWaitMulti` functionality in a coroutine environment.

```php
Swoole\Server->taskCo(array $tasks, float $timeout = 0.5): false|array
```
  
* `$tasks` Task list, must be an array. The underlying loop through the array will deliver each element as a `task` to the `Task` process pool
* `$timeout` Timeout, default is `0.5` seconds. If not all tasks are completed within the specified time, it will immediately stop and return the results
* When tasks are completed or timeout, the function returns an array of results. The order of each task result in the result array corresponds to `$tasks`, for example: the result corresponding to `$tasks[2]` is `$result[2]`
* If a task fails or times out, the corresponding result in the result array will be `false`, for example: if `$tasks[2]` fails, then the value of `$result[2]` is `false`

!> The maximum number of concurrent tasks should not exceed `1024`  

  * **Scheduling Process**

    * Each task in the `$tasks` list is randomly dispatched to a `Task` worker process. After dispatching, `yield` yields the current coroutine and sets a timer for `$timeout` seconds
    * Results corresponding to each task are collected in `onFinish` and saved to the result array. It checks if all tasks have returned results. If not, continue waiting. If yes, `resume` resumes the corresponding coroutine and clears the timeout timer
    * If not all tasks are completed within the specified time, the timer triggers first, and the underlying layer clears the waiting state. The unfinished task results are marked as `false`, and the corresponding coroutine is immediately `resumed`

  * **Example**

```php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 2,
]);

$server->on('Task', function (Swoole\Server $serv, $task_id, $worker_id, $data) {
    echo "#{$serv->worker_id}\tonTask: worker_id={$worker_id}, task_id=$task_id\n";
    if ($serv->worker_id == 1) {
        sleep(1);
    }
    return $data;
});

$server->on('Request', function ($request, $response) use ($server) {
    $tasks[0] = "hello world";
    $tasks[1] = ['data' => 1234, 'code' => 200];
    $result   = $server->taskCo($tasks, 0.5);
    $response->end('Test End, Result: ' . var_export($result, true));
});

$server->start();
```
## finish()

[TASKワーカープロセス](/learn?id=taskworkerプロセス)内で、投稿されたタスクが完了したことを`Worker`プロセスに通知するために使用されます。この関数は、`Worker`プロセスに結果データを渡すことができます。

```php
Swoole\Server->finish(mixed $data): bool
```

  * **Parameters**

    * `mixed $data`

      * Function: 処理結果の内容
      * Default: なし
      * 他の値: なし

  * **Returns**

    * `true`を返すと操作成功、`false`を返すと操作失敗

  * **Tips**
    * `finish`メソッドは複数回呼び出すことができ、`Worker`プロセスは複数回[onFinish](/server/events?id=onfinish)イベントをトリガします
    * [onTask](/server/events?id=ontask)コールバック関数内で`finish`メソッドを呼び出した後でも、`return`データは[onFinish](/server/events?id=onfinish)イベントをトリガします
    * `Server->finish`はオプションです。`Worker`プロセスがタスクの実行結果に関心がない場合、この関数を呼び出す必要はありません
    * [onTask](/server/events?id=ontask)コールバック関数内で`return`する文字列は、`finish`を呼び出すことと同等です

  * **注意**

  !> `Server->finish`関数を使用するには、`Server`に[onFinish](/server/events?id=onfinish)コールバック関数を設定する必要があります。この関数は、[TASKワーカープロセス](/learn?id=taskworkerプロセス)の[onTask](/server/events?id=ontask)コールバックでのみ使用できます
## heartbeat()

[heartbeat_check_interval](/server/setting?id=heartbeat_check_interval)とは異なり、このメソッドはサーバーのすべての接続を自動的にチェックし、約束された時間を超過している接続を特定します。`if_close_connection`を指定すると、タイムアウトした接続が自動的に閉じられます。指定しない場合は接続の`fd`配列が返されます。

```php
Swoole\Server->heartbeat(bool $ifCloseConnection = true): bool|array
```

  * **Parameters**

    * `bool $ifCloseConnection`

      * Function: タイムアウトした接続を閉じるかどうか
      * デフォルト値: true
      * その他の値: false

  * **Return Value**

    * 成功すると、閉じられた`$fd`を含む連続する配列が返されます
    * 失敗すると`false`が返されます

  * **Examples**

```php
$closeFdArrary = $server->heartbeat();
```
## getLastError()

最後の操作で発生したエラーコードを取得します。ビジネスコードでは、エラーコードの種類に応じて異なるロジックを実行することができます。

```php
Swoole\Server->getLastError(): int
```

  * **Return values**

Error Code | Explanation
---|---
1001 | 接続は既に`Server`側で閉じられており、このエラーが発生すると、通常はコードで`$server->close()`を実行して特定の接続を閉じた後に、まだその接続に`$server->send()`でデータを送信しようとしました。
1002 | 接続は`Client`側で閉じられており、ソケットが閉じられているため、データを送信できません。
1003 | `close`が実行中で、[onClose](/server/events?id=onclose)コールバック関数内で`$server->send()`を使用してはいけません。
1004 | 接続が既に閉じられています。
1005 | 接続が存在しない可能性があり、渡された`$fd`が間違っているかもしれません。
1007 | タイムアウトしたデータが受信され、`TCP`接続が閉じられた後、一部のデータが[unixSocket](/learn?id=什么是IPC)のキャッシュ領域に残っている場合があります。これらのデータは破棄されます。
1008 | 送信バッファがいっぱいで`send`操作を実行できません。このエラーが発生すると、接続の対向がデータを適時受信できず、送信バッファが一杯になっていることを示します。
1202 | 送信されたデータが[server->buffer_output_size](/server/setting?id=buffer_output_size)の設定を超えています。
9007 | [dispatch_mode](/server/setting?id=dispatch_mode)=3を使用している場合にのみ発生し、現在利用可能なプロセスがないことを示します。`worker_num`プロセス数を増やすことができます。
## getSocket()

このメソッドを呼び出すと、`socket`ハンドルの基礎を取得できます。返されるオブジェクトは`sockets`リソースハンドルです。

```php
Swoole\Server->getSocket(): false|\Socket
```

!> このメソッドを使用するには、PHPの`sockets`拡張機能が必要であり、`Swoole`をコンパイルする際には`--enable-sockets`オプションを有効にする必要があります

  * **ポートを聴く**

    * `listen`メソッドで追加されたポートは、`Swoole\Server\Port`オブジェクトが提供する`getSocket`メソッドを使用できます。

    ```php
    $port = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $socket = $port->getSocket();
    ```

    * `socket_set_option`関数を使用して、より低レベルの`socket`パラメータを設定できます。

    ```php
    $socket = $server->getSocket();
    if (!socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1)) {
        echo 'Unable to set option on socket: '. socket_strerror(socket_last_error()) . PHP_EOL;
    }
    ```

  * **マルチキャストをサポート**

    * `socket_set_option`を使用して`MCAST_JOIN_GROUP`パラメータを設定すると、`Socket`をマルチキャストに参加させてネットワークグループデータパケットをリッスンできます。

```php
$server = new Swoole\Server('0.0.0.0', 9905, SWOOLE_BASE, SWOOLE_SOCK_UDP);
$server->set(['worker_num' => 1]);
$socket = $server->getSocket();

$ret = socket_set_option(
    $socket,
    IPPROTO_IP,
    MCAST_JOIN_GROUP,
    array(
        'group' => '224.10.20.30', // グループアドレス
        'interface' => 'eth0' // ネットワークインターフェースの名前、数字または文字列であり、eth0、wlan0などが使用できます
    )
);

if ($ret === false) {
    throw new RuntimeException('Unable to join multicast group');
}

$server->on('Packet', function (Swoole\Server $server, $data, $addr) {
    $server->sendto($addr['address'], $addr['port'], "Swoole: $data");
    var_dump($addr, strlen($data));
});

$server->start();
```
```php
Swoole\Server->protect(int $fd, bool $is_protected = true): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：クライアント接続`fd`を指定します
      * デフォルト値：なし
      * 他の値：なし

    * `bool $is_protected`

      * 機能：設定する状態
      * デフォルト値：true 【保護状態を示します】
      * 他の値：false 【保護しないことを示します】

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します
```
## confirm()

[enable_delay_receive](/server/setting?id=enable_delay_receive)と一緒に使用して、接続を確認します。クライアントが接続を確立すると、読み取り可能なイベントをリッスンせず、単に[onConnect](/server/events?id=onconnect)イベントコールバックがトリガーされ、[onConnect](/server/events?id=onconnect)コールバックで`confirm`接続が実行されると、サーバーがクライアント接続からのデータを受信します。

!> Swooleバージョン >= `v4.5.0` で利用可能

```php
Swoole\Server->confirm(int $fd): bool
```

  * **Parameters**

    * `int $fd`

      * 機能：接続のユニークな識別子
      * デフォルト値：なし
      * その他の値：なし

  * **Return Value**
  
    * 確認が成功した場合は`true`を返します
    * `$fd`に対応する接続が存在しない、閉じられている、または既にリッスン状態である場合は`false`を返し、確認に失敗します

  * **Purpose**
  
    このメソッドは一般的にサーバーを保護するために使用され、トラフィック過多攻撃を受けるのを防ぎます。クライアント接続を受け取るときに[onConnect](/server/events?id=onconnect)関数がトリガーされ、送信元`IP`を判断し、サーバーにデータを送信することを許可するかどうかを判断できます。

  * **Example**
    
```php
//Serverオブジェクトを作成し、127.0.0.1:9501ポートをリッスンする
$serv = new Swoole\Server("127.0.0.1", 9501); 
$serv->set([
    'enable_delay_receive' => true,
]);

//接続イベントをリッスン
$serv->on('Connect', function ($serv, $fd) {  
    //ここで$fdを確認し、問題がなければconfirmを実行
    $serv->confirm($fd);
});

//データ受信イベントをリッスン
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, "Server: ".$data);
});

//接続終了イベントをリッスン
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//サーバーを起動
$serv->start(); 
```
## getWorkerId()

```php
Swoole\Server->getWorkerId(): int|false
```

!> Swooleのバージョン >= `v4.5.0RC1` で利用可能

現在の`Worker`プロセスの`id`（プロセスの`PID`ではない）を取得します。[onWorkerStart](/server/events?id=onworkerstart)の`$workerId`と一致します。
## getWorkerPid()

指定の`Worker`プロセスの`PID`を取得します。

```php
Swoole\Server->getWorkerPid(int $worker_id = -1): int|false
```

  * **パラメーター**

    * `int $worker_id`

      * 機能：指定されたプロセスのPIDを取得します
      * デフォルト値：-1、【-1 は現在のプロセスを表す】
      * その他の値：なし

!> Swooleバージョン >= `v4.5.0RC1` で使用可能
## getWorkerStatus()

`Worker`プロセスの状態を取得します

```php
Swoole\Server->getWorkerStatus(int $worker_id = -1): int|false
```

!> Swooleバージョン >= `v4.5.0RC1` で利用可能

  * **引数**

    * `int $worker_id`

      * 機能：プロセスの状態を取得します
      * デフォルト値：-1、【-1は現在のプロセスを示します】
      * その他の値：なし

  * **戻り値**
  
    * `Worker`プロセスの状態を返します。プロセスの状態値を参照してください。
    * `Worker`プロセスでないか、プロセスが存在しない場合は`false`を返します

  * **プロセスの状態値**

    定数 | 値 | 説明 | バージョン依存
    ---|---|---|---
    SWOOLE_WORKER_BUSY | 1 | ビジー | v4.5.0RC1
    SWOOLE_WORKER_IDLE | 2 | アイドル | v4.5.0RC1
    SWOOLE_WORKER_EXIT | 3 | [reload_async](/server/setting?id=reload_async)が有効な場合、同じworker_idに2つのプロセスがある可能性があり、新しいプロセスと古いプロセスがあります。古いプロセスが読み取るステータスコードはEXITです。 | v4.5.5
## getManagerPid()

```php
Swoole\Server->getManagerPid(): int
```

!> Swooleバージョン `v4.5.0RC1` 以上で利用可能
## getMasterPid()

```php
Swoole\Server->getMasterPid(): int
```

Swooleのバージョン >= `v4.5.0RC1` で利用可能
## addCommand()

```php
Swoole\Server->addCommand(string $name, int $accepted_process_types, Callable $callback): bool
```

!> -Swoole version >= `v4.8.0` available         
  -This function can only be called before the server is started. If there is a command with the same name, it will return `false` directly.

* **Parameters**

    * `string $name`

        * Description: Name of the `command`
        * Default: None
        * Other values: None

    * `int $accepted_process_types`

      * Description: Process types that will accept the request. If you want to support multiple process types, you can connect them with `|`, for example `SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_MANAGER`
      * Default: None
      * Other values:
        * `SWOOLE_SERVER_COMMAND_MASTER`: master process
        * `SWOOLE_SERVER_COMMAND_MANAGER`: manager process
        * `SWOOLE_SERVER_COMMAND_EVENT_WORKER`: worker process
        * `SWOOLE_SERVER_COMMAND_TASK_WORKER`: task process

    * `callable $callback`

        * Description: Callback function with two parameters, one is the `Swoole\Server` class, and the other is a user-defined variable. This variable is passed as the 4th parameter of `Swoole\Server::command()`.
        * Default: None
        * Other values: None

* **Return Value**

    * `true` if the custom command is added successfully, `false` if failed.
## command()

定義されたカスタムコマンド`command`を呼び出します。

```php
Swoole\Server->command(string $name, int $process_id, int $process_type, mixed $data, bool $json_decode = true): false|string|array
```

!>Swooleバージョン >= `v4.8.0` で利用可能です。`SWOOLE_PROCESS`および`SWOOLE_BASE`モードでは、この関数は`master`プロセスでのみ使用できます。

* **パラメーター**

    * `string $name`

        * 機能：`command` の名前
        * デフォルト値：なし
        * その他の値：なし

    * `int $process_id`

        * 機能：プロセスID
        * デフォルト値：なし
        * その他の値：なし

    * `int $process_type`

        * 機能：プロセスのリクエストタイプ、以下のいずれかの値を選択します。
        * デフォルト値：なし
        * その他の値：
          * `SWOOLE_SERVER_COMMAND_MASTER` masterプロセス
          * `SWOOLE_SERVER_COMMAND_MANAGER` managerプロセス
          * `SWOOLE_SERVER_COMMAND_EVENT_WORKER` workerプロセス
          * `SWOOLE_SERVER_COMMAND_TASK_WORKER` taskプロセス

    * `mixed $data`

        * 機能：リクエストされたデータ、このデータはシリアライズ可能である必要があります。
        * デフォルト値：なし
        * その他の値：なし

    * `bool $json_decode`

        * 機能：`json_decode` を使用して解析するかどうか
        * デフォルト値：true
        * その他の値：false

* **使用例**

    ```php
    <?php
    use Swoole\Http\Server;
    use Swoole\Http\Request;
    use Swoole\Http\Response;

    $server = new Server('127.0.0.1', 9501, SWOOLE_BASE);
    $server->addCommand('test_getpid', SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_EVENT_WORKER,
        function ($server, $data) {
          var_dump($data);
          return json_encode(['pid' => posix_getpid()]);
        });
    $server->set([
        'log_file' => '/dev/null',
        'worker_num' => 2,
    ]);

    $server->on('start', function (Server $serv) {
        $result = $serv->command('test_getpid', 0, SWOOLE_SERVER_COMMAND_MASTER, ['type' => 'master']);
        Assert::eq($result['pid'], $serv->getMasterPid());
        $result = $serv->command('test_getpid', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::eq($result['pid'], $serv->getWorkerPid(1));
        $result = $serv->command('test_not_found', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::false($result);

        $serv->shutdown();
    });

    $server->on('request', function (Request $request, Response $response) {
    });
    $server->start();
    ```
