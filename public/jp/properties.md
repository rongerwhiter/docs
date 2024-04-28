```html
<p>属性はオブジェクトに関する情報を提供します。</p>
```

# 属性

```html
<p>属性はオブジェクトに関する情報を提供します。</p>
```
### $ setting

[Server->set()](/server/methods?id=set)関数で設定されたパラメーターは、`Server->$ setting`プロパティに保存されます。コールバック関数内で実行パラメーターの値にアクセスできます。このプロパティは`array`タイプの配列です。

```php
Swoole\Server->setting
```

  * **例**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(array('worker_num' => 4));

echo $server->setting['worker_num'];
```
### $connections

`TCP` connection iterator that can be used to traverse all connections on the server at the moment using `foreach`. The functionality of this property is consistent with [Server->getClientList](/server/methods?id=getclientlist), but more user-friendly.

The elements traversed are individual connection `fd`.

```php
Swoole\Server->connections
```

!> The `$connections` property is an iterator object, not a PHP array. Therefore, it cannot be accessed using `var_dump` or array indices. It can only be iterated through using `foreach`.

  * **Base Mode**

    * In [SWOOLE_BASE](/learn?id=swoole_base) mode, cross-process operations on `TCP` connections are not supported, so in `BASE` mode, the `$connections` iterator can only be used within the current process.

  * **Example**

```php
foreach ($server->connections as $fd) {
  var_dump($fd);
}
echo "There are " . count($server->connections) . " connections in the current server\n";
```
### $host

`host`プロパティは、現在のサーバーがリッスンしているホストアドレスを返します。このプロパティは、`string`型の文字列です。

```php
Swoole\Server->host
```
### $port

`port`プロパティは、現在のサーバーがリッスンしているポートを示します。このプロパティは整数型の`int`です。

```php
Swoole\Server->port
```
### $type

`$type` プロパティは、現在のServerのタイプを返します。このプロパティは、`int` 型の整数です。

```php
Swoole\Server->type
```
!> このプロパティは、以下の値のいずれかを返します
- `SWOOLE_SOCK_TCP` tcp ipv4 ソケット
- `SWOOLE_SOCK_TCP6` tcp ipv6 ソケット
- `SWOOLE_SOCK_UDP` udp ipv4 ソケット
- `SWOOLE_SOCK_UDP6` udp ipv6 ソケット
- `SWOOLE_SOCK_UNIX_DGRAM` unix ソケット dgram
- `SWOOLE_SOCK_UNIX_STREAM` unix ソケット stream
### $ssl

現在のサーバーが`ssl`を有効にしているかどうかを返します。このプロパティは`bool`型です。

```php
Swoole\Server->ssl
```
### $mode

`mode`プロパティは、現在のサーバーのプロセスモードを返します。このプロパティは整数型の`int`です。

```php
Swoole\Server->mode
```

!> このプロパティは次のいずれかの値を返します
- `SWOOLE_BASE` シングルプロセスモード
- `SWOOLE_PROCESS` マルチプロセスモード
### $ports

`Server::$ports`はポートをリッスンするための配列で、サーバーが複数のポートをリッスンしている場合は、すべての`Swoole\Server\Port`オブジェクトを取得できます。

`swoole_server::$ports[0]`は構築時に指定されたメインサーバーポートです。

  * **例**

```php
$ports = $server->ports;
$ports[0]->set($settings);
$ports[1]->on('Receive', function () {
    //callback
});
```
### $master_pid

現在のサーバーのメインプロセスの`PID`を返します。

```php
Swoole\Server->master_pid
```

!> `onStart/onWorkerStart`の後でのみ取得できます

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->master_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $manager_pid

現在のサーバー管理プロセスの`PID`を返します。このプロパティは整数型の`int`です。

```php
Swoole\Server->manager_pid
```

!> `onStart/onWorkerStart` の後でのみ取得できます

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->manager_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $worker_id

現在の`Worker`プロセスの番号を取得します。これには [Task プロセス](/learn?id=taskworkerプロセス) も含まれます。このプロパティは、`int` 型の整数です。

```php
Swoole\Server->worker_id
```

  * **例**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num' => 8,
    'task_worker_num' => 4,
]);
$server->on('WorkerStart', function ($server, int $workerId) {
    if ($server->taskworker) {
        echo "task workerId：{$workerId}\n";
        echo "task worker_id：{$server->worker_id}\n";
    } else {
        echo "workerId：{$workerId}\n";
        echo "worker_id：{$server->worker_id}\n";
    }
});
$server->on('Receive', function ($server, $fd, $reactor_id, $data) {
});
$server->on('Task', function ($serv, $task_id, $reactor_id, $data) {
});
$server->start();
```

  * **ヒント**

    * このプロパティは[onWorkerStart](/server/events?id=onworkerstart)時の`$workerId`と同じです。
    * `Worker` プロセスの番号の範囲は `[0, $server->setting['worker_num'] - 1]` です。
    * [Task プロセス](/learn?id=taskworkerプロセス)の番号の範囲は `$server->setting['worker_num'], $server->setting['worker_num'] + $server->setting['task_worker_num'] - 1]` です。

!> ワーカープロセスが再起動されても `worker_id` の値は変わりません。
### $taskworker

現在のプロセスが `Task` プロセスであるかどうかを示すプロパティで、`bool` 型です。

```php
Swoole\Server->taskworker
```

  * **Return Value**

    * `true` このプロセスは `Task` ワーカープロセスであることを示します。
    * `false` このプロセスは `Worker` プロセスであることを示します。
### $worker_pid

`Worker`プロセスの現在の操作システムプロセス`ID`を取得します。`posix_getpid()`と同じ値を返します。このプロパティは、`int`型の整数です。

```php
Swoole\Server->worker_pid
```  
