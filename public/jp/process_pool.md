# Swoole\Process\Pool

プロセスプールは、[Swoole\Server](/server/init)のManagerがプロセスを管理するモジュールです。複数のワーカープロセスを管理できます。このモジュールの中心機能はプロセス管理であり、`Process`よりも複数のプロセスを実現する`Process\Pool`はシンプルで、より高いカプセル化を実現していて、開発者はプロセス管理機能を容易に実装することができます。[Co\Server](/coroutine/server?id=complete-example)と組み合わせることで、複数のCPUコアを活用した純粋なコルーチンスタイルでサーバープログラムを作成できます。
## プロセス間通信

`Swoole\Process\Pool`は、合計3種類のプロセス間通信方法を提供しています。
### メッセージキュー

`Swoole\Process\Pool->__construct` の第二引数を`SWOOLE_IPC_MSGQUEUE` に設定すると、プロセス間通信にメッセージキューが使用されます。情報を投递するには、`php sysvmsg` 拡張機能を使用でき、メッセージの最大容量は`65536`までです。

* **注意**

  * `sysvmsg` 拡張機能を使用してメッセージを送信する場合、コンストラクタに `msgqueue_key` を渡す必要があります
  * `Swoole` の内部では、`sysvmsg` 拡張機能の `msg_send` の第二引数 `mtype` はサポートされていません。`0`以外の値を渡してください。
### ソケット通信
`Swoole\Process\Pool->__construct`の2番目の引数を`SWOOLE_IPC_SOCKET`に設定すると、`ソケット通信`が使用されることを意味し、クライアントとサーバーが同じマシンにない場合、この方法を使用して通信できます。

[Swoole\Process\Pool->listen()](/process/process_pool?id=listen)メソッドを使用してポートをリッスンし、[Message事件](/process/process_pool?id=on)を使用してクライアントから送られてきたデータを受け取り、[Swoole\Process\Pool->write()](/process/process_pool?id=write)メソッドを使用してクライアントに応答を返します。

`Swoole`は、クライアントがデータをこの方法で送信する場合、実際のデータの前に 4 バイトのネットワークバイト順の長さ値を追加する必要があります。

```php
$msg = 'Hello Swoole';
$packet = pack('N', strlen($msg)) . $msg;
```
### UnixSocket
2番目の引数で`SWOOLE_IPC_UNIXSOCK`を設定した場合、[UnixSocket](/learn?id=什么是IPC)を使用しています。 **プロセス間通信にはこの方法を強くお勧めします**。

この方法は非常に簡単で、[Swoole\Process\Pool->sendMessage()](/process/process_pool?id=sendMessage)メソッドと[Messageイベント](/process/process_pool?id=on)を使用してプロセス間通信を行うことができます。

また、`コルーチンモード`を有効にしている場合、[Swoole\Process\Pool->getProcess()](/process/process_pool?id=getProcess)を使用して`Swoole\Process`オブジェクトを取得し、[Swoole\Process->exportsocket()](/process/process?id=exportsocket)で`Swoole\Coroutine\Socket`オブジェクトを取得し、このオブジェクトを使用してプロセス間通信を実現することもできます。ただし、この場合は[Messageイベント](/process/process_pool?id=on)を設定できません。

!> パラメーターと環境設定については、[construct関数](/process/process_pool?id=__construct)と[設定パラメーター](/process/process_pool?id=set)を参照してください。
## 定数

定数 | 説明
---|---
SWOOLE_IPC_MSGQUEUE | システム[メッセージキュー](/learn?id=什么是IPC)通信
SWOOLE_IPC_SOCKET | ソケット通信
SWOOLE_IPC_UNIXSOCK | [Unixソケット](/learn?id=什么是IPC)通信(v4.4+)
## 协程支持

`v4.4.0` バージョンでのコルーチンのサポートが追加されました。詳細については、[Swoole\Process\Pool::__construct](/process/process_pool?id=__construct) をご覧ください。
## 使用例

```php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(5);
$pool->set(['enable_coroutine' => true]);
$pool->on('WorkerStart', function (Process\Pool $pool, $workerId) {
    /** 現在のプロセスはワーカーです */
    static $running = true;
    Process::signal(SIGTERM, function () use (&$running) {
        $running = false;
        echo "TERM\n";
    });
    echo("[Worker #{$workerId}] WorkerStart, pid: " . posix_getpid() . "\n");
    while ($running) {
        Coroutine::sleep(1);
        echo "sleep 1\n";
    }
});
$pool->on('WorkerStop', function (\Swoole\Process\Pool $pool, $workerId) {
    echo("[Worker #{$workerId}] WorkerStop\n");
});
$pool->start();
```
このセクションでは、私たちの製品を使用する方法を説明します。
### __construct()

構造関数。

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **パラメータ**

  * **`int $worker_num`**
    * **機能**：ワーカープロセスの数を指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $ipc_type`**
    * **機能**：プロセス間通信のモード【デフォルトは`SWOOLE_IPC_NONE`で、プロセス間通信機能を使用しないことを示します】
    * **デフォルト値**：`SWOOLE_IPC_NONE`
    * **その他の値**：`SWOOLE_IPC_MSGQUEUE`、`SWOOLE_IPC_SOCKET`、`SWOOLE_IPC_UNIXSOCK`

    !> -`SWOOLE_IPC_NONE`に設定した場合は`onWorkerStart`コールバックを設定する必要があり、`onWorkerStart`内でループロジックを実装する必要があります。`onWorkerStart`関数が終了するとワーカープロセスはすぐに終了し、その後に`Manager`プロセスがプロセスを再起動します。  
    -`SWOOLE_IPC_MSGQUEUE`に設定すると、システムのメッセージキュー通信が使用され、`$msgqueue_key`を指定してメッセージキューの`KEY`を設定できます。メッセージキュー`KEY`を設定しないと、プライベートキューが取得されます。  
    -`SWOOLE_IPC_SOCKET`に設定すると、通信に`Socket`を使用し、[listen](/process/process_pool?id=listen)メソッドを使用してリッスンするアドレスとポートを指定する必要があります。  
    -`SWOOLE_IPC_UNIXSOCK`に設定すると、[unixSocket](/learn?id=什么是IPC)を使用して通信し、コルーチンモードを使用します。**プロセス間通信にこの方法を強くお勧めします**。具体的な使用方法は後述をご覧ください。  
    -`SWOOLE_IPC_NONE`以外の設定を使用する場合、`onMessage`コールバックを設定する必要があり、`onWorkerStart`はオプションとなります。

  * **`int $msgqueue_key`**
    * **機能**：メッセージキューの `key`
    * **デフォルト値**：`0`
    * **その他の値**：なし

  * **`bool $enable_coroutine`**
    * **機能**：コルーチンサポートを有効にするかどうか【コルーチンを使用すると`onMessage`コールバックを設定できなくなります】
    * **デフォルト値**：`false`
    * **その他の値**：`true`

* **コルーチンモード**
    
`v4.4.0`バージョンでは、`Process\Pool`モジュールにコルーチンサポートが追加され、4番目の引数を`true`に設定することで有効にできます。コルーチンを有効にすると、内部では`onWorkerStart`時に自動的にコルーチンと[コルーチンスケジューラ](/coroutine/scheduler)が作成され、コールバック関数内で直接コルーチン関連の`API`を使用できます。例：

```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

コルーチンを有効にすると、Swooleは`onMessage`イベントコールバックの設定を禁止します。プロセス間通信が必要な場合は、2番目を`SWOOLE_IPC_UNIXSOCK`に設定して[unixSocket](/learn?id=什么是IPC)を使用し、その後、`$pool->getProcess()->exportSocket()`を使用して[Swoole\Coroutine\Socket](/coroutine_client/socket)オブジェクトをエクスポートし、`Worker`プロセス間の通信を実現します。例：

```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 具体的な使い方については[Swoole\Coroutine\Socket](/coroutine_client/socket)および[Swoole\Process](/process/process?id=exportsocket)の関連セクションを参照してください。

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```
### set()

設定パラメータ。

```php
Swoole\Process\Pool->set(array $settings): void
```

オプション|タイプ|目的|デフォルト値
---|---|----|----
enable_coroutine|bool|コルーチンを有効にするかを制御します|false
enable_message_bus|bool|メッセージバスを有効にします。この値が`true`の場合、大きなデータを送信するとき、データは小さなチャンクに分割され、一つずつ対向端に送信されます|false
max_package_size|int|プロセスが受信できる最大データ量を制限します|2 * 1024 * 1024

* **注意**

  * `enable_message_bus`が`true`の場合、`max_package_size`は効果がありません。なぜなら、データは小さなチャンクに分割されて送信され、受信側も同じです。
  * `SWOOLE_IPC_MSGQUEUE`モードでは、`max_package_size`も効果がありません。一度に受信できるデータ量は最大で`65536`です。
  * `SWOOLE_IPC_SOCKET`モードでは、`enable_message_bus`が`false`の場合、取得するデータ量が`max_package_size`を超えると、接続が直ちに切断されます。
  * `SWOOLE_IPC_UNIXSOCK`モードでは、`enable_message_bus`が`false`の場合、データが`max_package_size`を超えると、`max_package_size`を超えるデータは切り捨てられます。
  * コルーチンモードを有効にした場合、`enable_message_bus`が`true`の場合、`max_package_size`も効果がありません。データは適切に分割（送信）および結合（受信）されます。それ以外の場合は、`max_package_size`により受信データ量が制限されます。

!> Swooleのバージョン >= v4.4.4 で使用可能
### on()

プロセスプールのコールバック関数を設定します。

```php
Swoole\Process\Pool->on(string $event, callable $function): bool;
```

* **パラメーター**

  * **`string $event`**
    * **機能**：イベントを指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $function`**
    * **機能**：コールバック関数
    * **デフォルト値**：なし
    * **その他の値**：なし

* **イベント**

  * **onWorkerStart** ワーカープロセス開始

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId 現在の作業プロセス番号。プロセスには番号が付けられます
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStart', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} is started\n";
  });
  ```

  * **onWorkerStop** ワーカープロセス終了

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId 現在の作業プロセス番号。プロセスには番号が付けられます
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStop', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} stop\n";
  });
  ```

  * **onMessage** メッセージ受信

  !> 外部からのメッセージを受け取ります。接続ごとにメッセージを送信でき、`PHP-FPM`の短い接続メカニズムと同様です。

  ```php
  /**
    * @param \Swoole\Process\Pool $pool Poolオブジェクト
    * @param string $data メッセージのデータ内容
   */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('Message', function(Swoole\Process\Pool $pool, string $data){
    var_dump($data);
  });
  ```

  !> イベント名は大文字と小文字を区別しません。`WorkerStart`、`workerStart`、または`workerstart`はすべて同じです。
### listen()

`SOCKET`をリッスンし、`$ipc_mode = SWOOLE_IPC_SOCKET`時にのみ使用できます。

```php
Swoole\Process\Pool->listen(string $host, int $port = 0, int $backlog = 2048): bool
```

* **パラメータ**

  * **`string $host`**
    * **機能**：リッスンするアドレス【`TCP`および[unixSocket](/learn?id=什么是IPC)の2種類のタイプをサポートします。`127.0.0.1`は`TCP`アドレスをリッスンし、`$port`を指定する必要があります。`unix:/tmp/php.sock`は[unixSocket](/learn?id=什么是IPC)アドレスをリッスンします。】
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`int $port`**
    * **機能**：リッスンするポート番号【`TCP`モードでは必ず指定する必要があります。】
    * **デフォルト値**：`0`
    * **他の値**：なし

  * **`int $backlog`**
    * **機能**：リッスンキューの長さ
    * **デフォルト値**：`2048`
    * **他の値**：なし

* **戻り値**

  * 成功した場合は`true`を返します
  * 失敗した場合は`false`を返し、エラーコードを取得するには`swoole_errno`を呼び出すことができます。リッスンに失敗した場合、`start`を呼び出すとすぐに`false`が返されます。

* **通信プロトコル**

    リッスンポートにデータを送信するとき、クライアントはリクエストの前に4バイトのネットワークバイト順の長さ値を追加する必要があります。プロトコル形式は次のとおりです：

```php
// $msg 送信するデータ
$packet = pack('N', strlen($msg)) . $msg;
```

* **使用例**

```php
$pool->listen('127.0.0.1', 8089);
$pool->listen('unix:/tmp/php.sock');
```
### write()

データを対向プロセスに書き込むために使用され、`$ipc_mode`が`SWOOLE_IPC_SOCKET`の場合にのみ使用できます。

```php
Swoole\Process\Pool->write(string $data): bool
```

!> このメソッドはメモリ操作であり、`IO`消費はありません。データ送信操作は同期ブロッキング`IO`です。

* **パラメータ** 

  * **`string $data`**
    * **機能**：書き込まれるデータの内容【`write`を複数回呼び出すことができ、`onMessage`関数が終了した後にデータがすべて`socket`に書き込まれ、接続が`close`されます】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **使用例**

  * **サーバー側**

    ```php
    $pool = new Swoole\Process\Pool(2, SWOOLE_IPC_SOCKET);
    
    $pool->on("Message", function ($pool, $message) {
        echo "Message: {$message}\n";
        $pool->write("hello ");
        $pool->write("world ");
        $pool->write("\n");
    });
    
    $pool->listen('127.0.0.1', 8089);
    $pool->start();
    ```

  * **クライアント側**

    ```php
    $fp = stream_socket_client("tcp://127.0.0.1:8089", $errno, $errstr) or die("error: $errstr\n");
    $msg = json_encode(['data' => 'hello', 'uid' => 1991]);
    fwrite($fp, pack('N', strlen($msg)) . $msg);
    sleep(1);
    //表示されるのは hello world\n です
    $data = fread($fp, 8192);
    var_dump(substr($data, 4, unpack('N', substr($data, 0, 4))[1]));
    fclose($fp);
    ```
### sendMessage()

データを対象プロセスに送信するには、`$ipc_mode` が `SWOOLE_IPC_UNIXSOCK` のときにのみ使用できます。

```php
Swoole\Process\Pool->sendMessage(string $data, int $dst_worker_id): bool
```

* **パラメータ**

  * **`string $data`**
    * **機能**：送信するデータ
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $dst_worker_id`**
    * **機能**：対象のワーカーID
    * **デフォルト値**：`0`
    * **その他の値**：なし

* **戻り値**

  * 送信に成功した場合は `true`
  * 送信に失敗した場合は `false`

* **注意**

  * データが `max_package_size` を超え、かつ `enable_message_bus` が `false` の場合、対象プロセスはデータを受信する際にデータを切り捨てます

```php
<?php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => false, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(2048)


$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => true, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(6000)
```
### start()

ワーカープロセスを起動します。

```php
Swoole\Process\Pool->start(): bool
```

!> 起動に成功すると、現在のプロセスは`wait`状態になり、ワーカープロセスを管理します。  
起動に失敗すると、`false`が返され、`swoole_errno`を使用してエラーコードを取得できます。

* **使用例**

```php
$workerNum = 10;
$pool = new Swoole\Process\Pool($workerNum);

$pool->on("WorkerStart", function ($pool, $workerId) {
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while (true) {
         $msg = $redis->brpop($key, 2);
         if ( $msg == null) continue;
         var_dump($msg);
     }
});

$pool->on("WorkerStop", function ($pool, $workerId) {
    echo "Worker#{$workerId} is stopped\n";
});

$pool->start();
```

* **プロセス管理**

  * 特定のワーカープロセスが致命的なエラーや自己終了に遭遇した場合、マネージャはリサイクル処理を行い、ゾンビプロセスの発生を防ぎます
  * ワーカープロセスが終了した後、管理者は自動的に新しいワーカープロセスを起動・作成します
  * メインプロセスが`SIGTERM`シグナルを受信した場合、新規プロセスの`fork`を停止し、実行中のすべてのワーカープロセスを`kill`します
  * メインプロセスが`SIGUSR1`シグナルを受信した場合、実行中のすべてのワーカープロセスを個別に`kill`し、新しいワーカープロセスを再起動します

* **シグナル処理**

  下層では、メインプロセス（マネージャプロセス）のみがシグナル処理が設定されており、`Worker`ワーカープロセスにはシグナルが設定されていません。開発者自身がシグナルのリスナーを実装する必要があります。

  - ワーカープロセスが非同期モードの場合は、[Swoole\Process::signal](/process/process?id=signal) を使用してシグナルを監視します
  - ワーカープロセスが同期モードの場合は、`pcntl_signal`および`pcntl_signal_dispatch`を使用してシグナルを監視します

  ワーカープロセスでは`SIGTERM`シグナルを監視するべきであり、メインプロセスがそのワーカープロセスを終了させる必要がある場合、このプロセスに`SIGTERM`シグナルが送信されます。ワーカープロセスが`SIGTERM`シグナルをリッスンしていない場合、下層は現在のプロセスを強制的に終了させ、一部のロジックが失われる可能性があります。

```php
$pool->on("WorkerStart", function ($pool, $workerId) {
    $running = true;
    pcntl_signal(SIGTERM, function () use (&$running) {
        $running = false;
    });
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while ($running) {
         $msg = $redis->brpop($key);
         pcntl_signal_dispatch();
         if ( $msg == null) continue;
         var_dump($msg);
     }
});
```
### stop()

When a coroutine is started, this function removes the current process socket from the event loop.

```php
Swoole\Process\Pool->stop(): bool
```  
### shutdown()

ワーカープロセスを終了します。

```php
Swoole\Process\Pool->shutdown(): bool
```
### getProcess()

現在の作業プロセスオブジェクトを取得します。[Swoole\Process](/process/process)オブジェクトを返します。

!> Swoole バージョン >= `v4.2.0` で利用可能

```php
Swoole\Process\Pool->getProcess(int $worker_id): Swoole\Process
```

* **パラメータ** 

  * **`int $worker_id`**
    * **機能**：取得する`worker`を指定する【オプション, デフォルトは現在の`worker`】
    * **デフォルト値**：なし
    * **他の値**：なし

!> `start`の後、作業プロセスの`onWorkerStart`や他のコールバック関数内で呼び出す必要があります；  
返される`Process`オブジェクトはシングルトンパターンであり、作業プロセス内で`getProcess()`を繰り返し呼び出すと同じオブジェクトが返されます。

* **使用例**

```php
$pool = new Swoole\Process\Pool(3);

$pool->on('WorkerStart', function ($pool, $workerId) {
    $process = $pool->getProcess();
    $process->exec('/usr/local/bin/php', ['-r', 'var_dump(swoole_version());']);
});

$pool->start();
```
### detach()

現在の Worker プロセスをプロセスプールの管理から切り離し、すぐに新しいプロセスが作成されます。古いプロセスはもはやデータを処理せず、アプリケーションレイヤーのコードでライフサイクルを管理する必要があります。

!> Swoole バージョン >= `v4.7.0` で使用可能

```php
Swoole\Process\Pool->detach(): bool
```
