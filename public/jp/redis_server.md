# Redis\Server

`Redis`サーバープロトコルに準拠した`Server`クラスで、このクラスをベースに`Redis`プロトコルのサーバープログラムを実装できます。

?> `Swoole\Redis\Server`は[Server](/server/tcp_init)を継承しているため、`Server`が提供するすべての`API`や設定項目を使用することができます。プロセスモデルも同様です。[Server](/server/init)セクションを参照してください。

* **利用可能なクライアント**

  * 任意のプログラミング言語の`redis`クライアント、PHPの`redis`拡張および`phpredis`ライブラリを含む
  * [Swoole\Coroutine\Redis](/coroutine_client/redis) コルーチンクライアント
  * `Redis`が提供するコマンドラインツール、`redis-cli`、`redis-benchmark`も含まれます。
## 方法

`Swoole\Redis\Server`は`Swoole\Server`を継承しており、親クラスが提供するすべてのメソッドを使用できます。
### setHandler

?> **Set the handler for `Redis` commands.**

!> `Redis\Server` does not require setting the [onReceive](/server/events?id=onreceive) callback. Simply use the `setHandler` method to set the handling function for the corresponding command. If an unsupported command is received, an `ERROR` response will be automatically sent to the client with the message `ERR unknown command '$command'`.

```php
Swoole\Redis\Server->setHandler(string $command, callable $callback);
```

* **Parameters**

  * **`string $command`**
    * **Description**: Name of the command
    * **Default**: None
    * **Other values**: None

  * **`callable $callback`**
    * **Description**: Handling function for the command [When the callback function returns a string type, it will be automatically sent to the client]
    * **Default**: None
    * **Other values**: None

    !> The returned data must be in `Redis` format. You can use the `format` static method for packing.
### format

?> **応答データをフォーマットします。**

```php
Swoole\Redis\Server::format(int $type, mixed $value = null);
```

* **パラメータ**

  * **`int $type`**
    * **機能**：データの種類、定数は以下の[format constants](/redis_server?id=フォーマット定数)を参照してください。
    * **デフォルト値**：なし
    * **その他の値**：なし
    
    !> `$type`が`NIL`の場合は、`$value`を渡す必要はありません。`ERROR`および`STATUS`タイプの場合、`$value`はオプションです。`INT`、`STRING`、`SET`、`MAP`は必須です。

  * **`mixed $value`**
    * **機能**：値
    * **デフォルト値**：なし
    * **その他の値**：なし
### 送信

?> **`send()`メソッドを使用して、データをクライアントに送信します。**

```php
Swoole\Server->send(int $fd, string $data): bool
```
```python
PI = 3.14159
SPEED_OF_LIGHT = 299792458
```
### 定数のフォーマットパラメータ

`Redis`のレスポンスデータをフォーマットするために使用される主な定数です。

定数 | 説明
---|---
Server::NIL | nilデータを返します
Server::ERROR | エラーコードを返します
Server::STATUS | ステータスを返します
Server::INT | 整数を返します。フォーマットには整数型の値を渡す必要があります
Server::STRING | 文字列を返します。フォーマットには文字列型の値を渡す必要があります
Server::SET | リストを返します。フォーマットには配列の値を渡す必要があります
Server::MAP | マップを返します。フォーマットには連想配列の値を渡す必要があります
```python
print("Hello, World!")
```

次のコードは、画面に "Hello, World!" と表示します。
### サーバーサイド

```php
use Swoole\Redis\Server;

define('DB_FILE', __DIR__ . '/db');

$server = new Server("127.0.0.1", 9501, SWOOLE_BASE);

if (is_file(DB_FILE)) {
    $server->data = unserialize(file_get_contents(DB_FILE));
} else {
    $server->data = array();
}

$server->setHandler('GET', function ($fd, $data) use ($server) {
    if (count($data) == 0) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'GET' command"));
    }

    $key = $data[0];
    if (empty($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    } else {
        return $server->send($fd, Server::format(Server::STRING, $server->data[$key]));
    }
});

$server->setHandler('SET', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'SET' command"));
    }

    $key = $data[0];
    $server->data[$key] = $data[1];
    return $server->send($fd, Server::format(Server::STATUS, "OK"));
});

$server->setHandler('sAdd', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sAdd' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }

    $count = 0;
    for ($i = 1; $i < count($data); $i++) {
        $value = $data[$i];
        if (!isset($server->data[$key][$value])) {
            $server->data[$key][$value] = 1;
            $count++;
        }
    }

    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('sMembers', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sMembers' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::SET, array_keys($server->data[$key])));
});

$server->setHandler('hSet', function ($fd, $data) use ($server) {
    if (count($data) < 3) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hSet' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }
    $field = $data[1];
    $value = $data[2];
    $count = !isset($server->data[$key][$field]) ? 1 : 0;
    $server->data[$key][$field] = $value;
    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('hGetAll', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hGetAll' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::MAP, $server->data[$key]));
});

$server->on('WorkerStart', function ($server) {
    $server->tick(10000, function () use ($server) {
        file_put_contents(DB_FILE, serialize($server->data));
    });
});

$server->start();
```
### クライアント

```shell
$ redis-cli -h 127.0.0.1 -p 9501
127.0.0.1:9501> set name swoole
OK
127.0.0.1:9501> get name
"swoole"
127.0.0.1:9501> sadd swooler rango
(integer) 1
127.0.0.1:9501> sadd swooler twosee guoxinhua
(integer) 2
127.0.0.1:9501> smembers swooler
1) "rango"
2) "twosee"
3) "guoxinhua"
127.0.0.1:9501> hset website swoole "www.swoole.com"
(integer) 1
127.0.0.1:9501> hset website swoole "swoole.com"
(integer) 0
127.0.0.1:9501> hgetall website
1) "swoole"
2) "swoole.com"
127.0.0.1:9501> test
(error) ERR unknown command 'test'
127.0.0.1:9501>
```
