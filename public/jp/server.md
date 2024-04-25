# TCPサーバー

?> `Swoole\Coroutine\Server` は、完全な[コルーチン](/coroutine)クラスです。これを使用すると、コルーチン`TCP`サーバーを作成し、TCPおよび[unixSocket](/learn?id=什么是IPC)タイプをサポートします。

[Server](/server/tcp_init)モジュールとの違い：

* 動的な作成と破棄が可能で、実行時にポートを動的にリッスンし、サーバーを動的に閉じることもできる
* 接続処理は完全に同期的であり、プログラムは`Connect`、`Receive`、`Close`のイベントを順番に処理できる

!> 4.4以上のバージョンで利用可能
## Short Alias

`Co\Server` が使用可能な短い名前です。
```python
def add(a, b):
    return a + b
```

このメソッドは、2つの数値を加算します。
### __construct()

?> **Constructor method.**

```php
Swoole\Coroutine\Server::__construct(string $host, int $port = 0, bool $ssl = false, bool $reuse_port = false);
```

  * **Parameters**

    * **`string $host`**
      * **Function**: Address to listen on
      * **Default**: None
      * **Other values**: None

    * **`int $port`**
      * **Function**: Port to listen on [if 0, a random port will be assigned by the operating system]
      * **Default**: None
      * **Other values**: None

    * **`bool $ssl`**
      * **Function**: Enable SSL encryption or not
      * **Default**: `false`
      * **Other values**: `true`

    * **`bool $reuse_port`**
      * **Function**: Enable port reuse, same effect as the configuration in [this section](/server/setting?id=enable_reuse_port)
      * **Default**: `false`
      * **Other values**: `true`
      * **Version impact**: Swoole version >= v4.4.4

  * **Tips**

    * **The `$host` parameter supports 3 formats**

      * `0.0.0.0/127.0.0.1`: IPv4 address
      * `::/::1`: IPv6 address
      * `unix:/tmp/test.sock`: [UnixSocket](/learn?id=什么是IPC) address

    * **Exceptions**

      * `Swoole\Exception` exception will be thrown in case of parameter error, failed to bind address and port, or failure in `listen`. 

Translated by: Shun.
### set()

?> **設定プロトコル処理パラメータ。**

```php
Swoole\Coroutine\Server->set(array $options);
```

  * **設定パラメータ**

    * パラメータ`$options`は、[setprotocol](/coroutine_client/socket?id=setprotocol) メソッドで受け入れられる設定項目と完全に一致する必要があります。

    !> パラメータを設定する前に [start()](/coroutine/server?id=start) メソッドが呼び出される必要があります

    * **長さプロトコル**

    ```php
    $server = new Swoole\Coroutine\Server('127.0.0.1', $port, $ssl);
    $server->set([
      'open_length_check' => true,
      'package_max_length' => 1024 * 1024,
      'package_length_type' => 'N',
      'package_length_offset' => 0,
      'package_body_offset' => 4,
    ]);
    ```

    * **SSL証明書設定**

    ```php
    $server->set([
      'ssl_cert_file' => dirname(__DIR__) . '/ssl/server.crt',
      'ssl_key_file' => dirname(__DIR__) . '/ssl/server.key',
    ]);
    ```
### handle()

?> **Set the connection handling function.**

!> The handling function must be set before calling [start()](/coroutine/server?id=start).

```php
Swoole\Coroutine\Server->handle(callable $fn);
```

  * **Parameters** 

    * **`callable $fn`**
      * **Description**: Set the connection handling function.
      * **Default**: None
      * **Other values**: None
      
  * **Example** 

    ```php
    $server->handle(function (Swoole\Coroutine\Server\Connection $conn) {
        while (true) {
            $data = $conn->recv();
        }
    });
    ```

    !> - After a successful `Accept` (connection establishment), the server will automatically create a [coroutine](/coroutine?id=coroutine-scheduling) and execute `$fn`.  
    - `$fn` is executed in a new sub-coroutine space, so there is no need to create coroutines again within the function.  
    - `$fn` takes one argument, an object of type [Swoole\Coroutine\Server\Connection](/coroutine/server?id=coroutineserverconnection).  
    - You can use [exportSocket()](/coroutine/server?id=exportsocket) to get the Socket object of the current connection.
### shutdown()

?> **サーバーをシャットダウンします。** 

?> ベースとなるサーバーは `start` および `shutdown` を複数回呼び出すことをサポートしています。

```php
Swoole\Coroutine\Server->shutdown(): bool
```
### start()

?> **サーバーを起動します。**

```php
Swoole\Coroutine\Server->start(): bool
```

  * **Return Value**

    * 起動に失敗すると`false`を返し、`errCode`プロパティが設定されます
    * 起動に成功すると、ループに入り、接続を`Accept`します
    * `Accept`(接続の確立)後、新しいコルーチンが作成され、そのコルーチン内で指定された関数の`handle`メソッドが呼び出されます

  * **Error Handling**

    * `Accept`(接続の確立)時に`Too many open file`エラーが発生した場合、またはサブコルーチンを作成できない場合、`1`秒一時停止してから`Accept`を再開します
    * エラーが発生した場合、`start()`メソッドは戻り、エラー情報は`Warning`形式で報告されます。
## オブジェクト
### Coroutine\Server\Connection

`Swoole\Coroutine\Server\Connection`オブジェクトには、4つのメソッドがあります：
#### recv()

データを受信し、プロトコル処理が設定されている場合、毎回完全なパケットが返されます。

```php
function recv(float $timeout = 0)
```
#### send()

データを送信する

```php
function send(string $data)
```  
#### close()

接続を閉じる

```php
function close(): bool
```
#### exportSocket()

現在の接続されているSocketオブジェクトを取得します。より低レベルのメソッドを呼び出すことができます。詳細は [Swoole\Coroutine\Socket](/coroutine_client/socket) を参照してください。

```php
function exportSocket(): Swoole\Coroutine\Socket
```
```php
use Swoole\Process;
use Swoole\Coroutine;
use Swoole\Coroutine\Server\Connection;

//多进程管理モジュール
$pool = new Process\Pool(2);
//すべてのOnWorkerStartコールバックでコルーチンを自動作成する
$pool->set(['enable_coroutine' => true]);
$pool->on('workerStart', function ($pool, $id) {
    //それぞれのプロセスが9501ポートを監視する
    $server = new Swoole\Coroutine\Server('127.0.0.1', 9501, false, true);

    //15シグナルを受信してサービスをシャットダウンする
    Process::signal(SIGTERM, function () use ($server) {
        $server->shutdown();
    });

    //新しい接続要求を受信して自動的にコルーチンを作成する
    $server->handle(function (Connection $conn) {
        while (true) {
            //データを受信
            $data = $conn->recv(1);

            if ($data === '' || $data === false) {
                $errCode = swoole_last_error();
                $errMsg = socket_strerror($errCode);
                echo "errCode: {$errCode}, errMsg: {$errMsg}\n";
                $conn->close();
                break;
            }

            //データを送信
            $conn->send('hello');

            Coroutine::sleep(1);
        }
    });

    //ポートを監視開始
    $server->start();
});
$pool->start();
```

!> Cygwin環境で実行する場合は、単一プロセスに変更してください。`$pool = new Swoole\Process\Pool(1);`
