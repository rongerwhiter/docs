# 協力TCP/UDPクライアント

`Coroutine\Client`は、`TCP`、`UDP`、[unixSocket](/learn?id=IPCとは)伝達プロトコルの[Socketクライアント](/coroutine_client/socket)のラッパーコードを提供し、使用時には`new Swoole\Coroutine\Client`だけで利用可能です。

* **実装原理**

    * `Coroutine\Client`がネットワークリクエストに関与するすべてのメソッドは、`Swoole`が[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を行い、ビジネス層は関知する必要がありません
    * 使用法は[Client](/client)の同期モードメソッドと完全に同じです
    * `connect`タイムアウト設定は`Connect`、`Recv`、`Send`のいずれにも同じく適用されます

* **継承関係**

    * `Coroutine\Client`と[Client](/client)は継承関係にはありませんが、`Client`が提供するメソッドはすべて`Coroutine\Client`で利用可能です。詳細は [Swoole\Client](/client?id=メソッド) を参照してください。ここでは繰り返し説明しません。
    * `Coroutine\Client`では`set`メソッドを使用して[設定オプション](/client?id=設定)を設定することができ、その使用法は`Client->set`と完全に同じであり、異なる関数に対しては、`set()`関数のセクションで個別に説明します。

* **使用例**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```

* **プロトコル処理**

コルーチンクライアントは、長さと`EOF`プロトコル処理もサポートしており、設定方法は [Swoole\Client](/client?id=設定) と完全に同じです。

```php
$client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
$client->set(array(
    'open_length_check'     => true,
    'package_length_type'   => 'N',
    'package_length_offset' => 0, //パッケージの長さ値が何バイト目にあるか
    'package_body_offset'   => 4, //長さの計算が始まるバイト数
    'package_max_length'    => 2000000, //プロトコルの最大長
));
```
### connect()

リモートサーバーに接続します。

```php
Swoole\Coroutine\Client->connect(string $host, int $port, float $timeout = 0.5): bool
```

  * **パラメータ** 

    * **`string $host`**
      * **機能**：リモートサーバーのアドレス【内部で自動的にコルーチン切り替えが行われ、ドメインをIPアドレスに解決します】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **機能**：リモートサーバーのポート
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：ネットワークI/Oのタイムアウト時間；`connect/send/recv` を含み、タイムアウトが発生した場合、接続は自動的に`close` されます。詳細は[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照してください。
      * **値の単位**：秒【浮動小数点数をサポートし、たとえば`1.5` は`1s` + `500ms` を表します】
      * **デフォルト値**：`0.5s`
      * **その他の値**：なし

* **ヒント**

    * 接続に失敗すると`false` が返されます
    * タイムアウト後に戻ると、`$cli->errCode` が `110` であることを確認してください

* **再試行**

!> `connect` が失敗した場合、直接再接続はできません。既存の`ソケット`を`close` してから再び`connect` を行う必要があります。

```php
//接続失敗
if ($cli->connect('127.0.0.1', 9501) == false) {
    //既存のソケットを閉じる
    $cli->close();
    //再試行
    $cli->connect('127.0.0.1', 9501);
}
```

* **例**

```php
if ($cli->connect('127.0.0.1', 9501)) {
    $cli->send('data');
} else {
    echo 'connect failed.';
}

if ($cli->connect('/tmp/rpc.sock')) {
    $cli->send('data');
} else {
    echo 'connect failed.';
}
```
### isConnected()

Clientの接続状態を返します

```php
Swoole\Coroutine\Client->isConnected(): bool
```

  * **返り値**

    * `false`を返すと、現在サーバーに接続されていないことを示します
    * `true`を返すと、現在サーバーに接続されていることを示します
    
!> `isConnected`メソッドはアプリケーションレベルの状態を返します。これは、`Client`が`connect`を実行し、`Server`に接続に成功したことを示し、`close`を実行して接続を閉じていないことを意味します。`Client`は`send`、`recv`、`close`などを実行できますが、再度`connect`を実行することはできません。  
これは接続が常に利用可能であることを意味するわけではありません。`send`または`recv`を実行する際にエラーが発生する可能性があります。アプリケーションレベルはTCP接続の状態を取得できず、実際の接続の利用可能な状態を取得するためには、`send`や`recv`を実行するとアプリケーションレベルとカーネルがやり取りし、その後、接続が利用可能かどうかが確認されます。
### send()

データを送信します。

```php
Swoole\Coroutine\Client->send(string $data): int|bool
```

  * **パラメータ** 

    * **`string $data`**
    
      * **機能**：送信するデータで、文字列型である必要があり、バイナリデータをサポートします
      * **デフォルト値**：なし
      * **その他の値**：なし

  * 送信に成功すると、`Socket`バッファに書き込まれたバイト数が返され、データをできるだけ送信します。返されたバイト数が入力の`$data`の長さと異なる場合、`Socket`が対向先で閉じられている可能性があります。次に`send`または`recv`を呼び出すと、対応するエラーコードが返されます。

  * 送信に失敗するとfalseが返され、`$client->errCode`を使用してエラー原因を取得できます。
### recv()

recvメソッドはサーバーからデータを受信するために使用されます。

```php
Swoole\Coroutine\Client->recv(float $timeout = 0): string|bool
```

  * **引数** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点数がサポートされています。たとえば`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超時规则)を参照
      * **他の値**：なし

    !> タイムアウトを設定する場合、指定された引数が優先されます。次に、`set`メソッドで渡す`timeout`構成が使用されます。タイムアウトエラーコードは`ETIMEDOUT`です。

  * **戻り値**

    * [通信プロトコル](/client?id=协议解析)が設定されている場合、`recv`は完全なデータを返します。長さは[package_max_length](/server/setting?id=package_max_length)に制限されます。
    * 通信プロトコルが設定されていない場合、`recv`は最大で`64K`のデータを返します。
    * 通信プロトコルが設定されていない場合、元のデータを返し、PHPコードでネットワークプロトコルを自分で処理する必要があります。
    * `recv`が空の文字列を返す場合、サーバーが切断されたことを示し、`close`が必要です。
    * `recv`が失敗した場合、`false`を返し、`$client->errCode`でエラーの理由を確認し、処理方法は後述の[完全な例](/coroutine_client/client?id=完整示例)を参照してください。
### close()

接続を閉じます。

!> `close` はブロッキングされず、すぐに返されます。閉じる操作にはコルーチンの切り替えはありません。

```php
Swoole\Coroutine\Client->close(): bool
```
### peek()

データを覗き見ます。

!> `peek`メソッドは`socket`を直接操作するため、[coroutine scheduling（協調スケジューリング）](/coroutine?id=協調スケジューリング)を引き起こしません。

```php
Swoole\Coroutine\Client->peek(int $length = 65535): string
```

  * **ヒント**

    * `peek`メソッドはカーネルの`socket`バッファ内のデータを覗き見るだけであり、オフセットは行いません。`peek`した後に`recv`を呼び出すと、この部分のデータを読み取ることができます。
    * `peek`メソッドはノンブロッキングです。すぐに返されます。`socket`バッファにデータがある場合、データ内容が返されます。バッファが空の場合、`false`が返され、`$client->errCode`が設定されます。
    * 接続が閉じられている場合、`peek`は空の文字列を返します。
```php
Swoole\Coroutine\Client->set(array $settings): bool
```

  * **設定パラメータ**

    * [Swoole\Client](/client?id=set) を参照してください。

* **[Swoole\Client](/client?id=set)との違い**
    
    コルーチンクライアントは、より細かいタイムアウト制御を提供します。以下を設定できます：

    * `timeout`：接続、送信、受信の全てのタイムアウトを含む総合的なタイムアウト
    * `connect_timeout`：接続のタイムアウト
    * `read_timeout`：受信のタイムアウト
    * `write_timeout`：送信のタイムアウト
    * [クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照してください

* **例**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    $client->set(array(
        'timeout' => 0.5,
        'connect_timeout' => 1.0,
        'write_timeout' => 10.0,
        'read_timeout' => 0.5,
    ));

    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```
```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5)) {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    while (true) {
        $data = $client->recv();
        if (strlen($data) > 0) {
            echo $data;
            $client->send(time() . PHP_EOL);
        } else {
            if ($data === '') {
                // 全等于空 直接关闭连接
                $client->close();
                break;
            } else {
                if ($data === false) {
                    // 可以自行根据业务逻辑和错误码进行处理，例如：
                    // 如果超时时则不关闭连接，其他情况直接关闭连接
                    if ($client->errCode !== SOCKET_ETIMEDOUT) {
                        $client->close();
                        break;
                    }
                } else {
                    $client->close();
                    break;
                }
            }
        }
        \Co::sleep(1);
    }
});
```
