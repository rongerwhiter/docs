# Coroutine\Http2\Client

協力Http2クライアント
```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $domain = 'www.zhihu.com';
    $cli = new Client($domain, 443, true);
    $cli->set([
        'timeout' => -1,
        'ssl_host_name' => $domain
    ]);
    $cli->connect();
    $req = new Request();
    $req->method = 'POST';
    $req->path = '/api/v4/answers/300000000/voters';
    $req->headers = [
        'host' => $domain,
        'user-agent' => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip'
    ];
    $req->data = '{"type":"up"}';
    $cli->send($req);
    $response = $cli->recv();
    var_dump(assert(json_decode($response->data)->error->code === 10002));
});
```
## 方法

Please let me know if you need any further assistance.
### __construct()

构造方法。

```php
Swoole\Coroutine\Http2\Client::__construct(string $host, int $port, bool $open_ssl = false): void
```

  * **参数** 

    * **`string $host`**
      * **功能**：目标主机的IP地址【`$host`如果为域名底层需要进行一次`DNS`查询】
      * **默认值**：无
      * **其它值**：无

    * **`int $port`**
      * **功能**：目标端口【`Http`一般为`80`端口，`Https`一般为`443`端口】
      * **默认值**：无
      * **其它值**：无

    * **`bool $open_ssl`**
      * **功能**：是否开启`TLS/SSL`隧道加密 【`https`网站必须设置为`true`】
      * **默认值**：`false`
      * **其它值**：`true`

  * **注意**

    !> -如果你需要请求外网URL请修改`timeout`为更大的数值，参考[客户端超时规则](/coroutine_client/init?id=超时规则)  
    -`$ssl`需要依赖`openssl`，必须在编译`Swoole`时启用[--enable-openssl](/environment?id=编译选项)
```php
Swoole\Coroutine\Http2\Client->set(array $options): void
```
### connect()

サーバーに接続します。このメソッドには引数はありません。

!> `connect`を呼び出すと、[coroutine scheduling](/coroutine?id=coroutine-scheduling) が自動的に行われ、接続が成功するか失敗するかに関わらず、`connect` が返ります。接続が確立した後は、`send` メソッドを使用してサーバーにリクエストを送信できます。

```php
Swoole\Coroutine\Http2\Client->connect(): bool
```

  * **Return Value**

    * 接続に成功した場合は `true` が返されます。
    * 接続に失敗した場合は `false` が返され、`errCode` プロパティをチェックしてエラーコードを取得してください。
### stats()

流の状態を取得します。

```php
Swoole\Coroutine\Http2\Client->stats([$key]): array|bool
```

  * **例**

```php
var_dump($client->stats(), $client->stats()['local_settings'], $client->stats('local_settings'));
```
### isStreamExist()

指定されたストリームが存在するかどうかを判断します。

```php
Swoole\Coroutine\Http2\Client->isStreamExist(int $stream_id): bool
```
### send()

サーバーにリクエストを送信し、内部では`Http2`の`stream`が自動的に確立されます。複数のリクエストを同時に送信できます。

```php
Swoole\Coroutine\Http2\Client->send(Swoole\Http2\Request $request): int|false
```

  * **Arguments** 

    * **`Swoole\Http2\Request $request`**
      * **Description**：Swoole\Http2\Requestオブジェクトを送信します
      * **Default**：なし
      * **Other**：なし

  * **Return Value**

    * 成功：ストリームの番号が返されます。番号は`1`から始まり、奇数で増加します。
    * 失敗：`false`が返されます。

  * **Note**

    * **Requestオブジェクト**

      !> `Swoole\Http2\Request`オブジェクトには、情報を書き込むためのプロパティしかありません。

      * `headers`：HTTPヘッダーの配列
      * `method`：文字列、リクエストメソッドを設定します（例えば、`GET`、`POST`）
      * `path`：文字列、URLパスを設定します（例：`/index.php?a=1&b=2`、必ず`/`で始める必要があります）
      * `cookies`：配列、`COOKIES`を設定します
      * `data`：リクエストの本文を設定します。文字列の場合、そのまま`RAW form-data`として送信されます
      * `data`が配列の場合、自動的に`x-www-form-urlencoded`形式の`POST`コンテンツにパッケージ化され、`Content-Type`が`application/x-www-form-urlencoded`に設定されます
      * `pipeline`：ブール値、`true`に設定すると、`$request`を送信した後、`stream`を閉じずにデータコンテンツを継続して書き込むことができます。

    * **pipeline**

      * デフォルトでは`send`メソッドはリクエストを送信した後、現在の`Http2 Stream`を終了します。`pipeline`を有効にすると、内部では`stream`が維持され、`write`メソッドを複数回呼び出してデータフレームをサーバーに送信できます。`write`メソッドを参照してください。
### write()

More data frames are sent to the server, and write can be called multiple times to write data frames to the same stream.

```php
Swoole\Coroutine\Http2\Client->write(int $streamId, mixed $data, bool $end = false): bool
```

  * **Parameters** 

    * **`int $streamId`**
      * **Function**: Stream number, returned by the `send` method
      * **Default**: None
      * **Other values**: None

    * **`mixed $data`**
      * **Function**: Content of the data frame, can be a string or an array
      * **Default**: None
      * **Other values**: None

    * **`bool $end`**
      * **Function**: Whether to close the stream
      * **Default**: `false`
      * **Other values**: `true`

  * **Usage Example**

```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9518);
    $cli->set(['timeout' => 1]);
    var_dump($cli->connect());

    $req3 = new Request();
    $req3->path = "/index.php";
    $req3->headers = [
        'host' => "localhost",
        "user-agent" => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip',
    ];
    $req3->pipeline = true;
    $req3->method = "POST";
    $streamId = $cli->send($req3);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    //end stream
    $cli->write($streamId, ['int' => rand(1000, 9999), 'end' => true], true);
    var_dump($cli->recv());
    $cli->close();
});
```

!> If you want to use `write` to send data frames in segments, you must set `$request->pipeline` to `true` when making the `send` request.  
Once a data frame with `end` set to `true` is sent, the stream will close, and you cannot call `write` to send data to this stream anymore.
### recv()

リクエストを受信します。

!> このメソッドを呼び出すと、[コルーチンスケジューリング](/coroutine?id=协程调度)が発生します。

```php
Swoole\Coroutine\Http2\Client->recv(float $timeout): Swoole\Http2\Response;
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します。[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照
      * **値の単位**：秒【浮動小数点数がサポートされており、例えば`1.5`は`1s` + `500ms`を表します】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

成功した場合、Swoole\Http2\Response オブジェクトが返されます。

```php
/**@var $resp Swoole\Http2\Response */
var_dump($resp->statusCode); // サーバーから送信されたHTTPステータスコード、例：200、502など
var_dump($resp->headers); // サーバーから送信されたヘッダ情報
var_dump($resp->cookies); // サーバーが設定したCOOKIE情報
var_dump($resp->set_cookie_headers); // サーバーが返す元のCOOKIE情報、domainやpathを含む
var_dump($resp->data); // サーバーから送信されたレスポンスデータ
```

!> Swooleバージョン < [v4.0.4](/version/bc?id=_404) の場合、`data`属性は`body`属性です；Swooleバージョン < [v4.0.3](/version/bc?id=_403) の場合、`headers` および `cookies` は単数形式です。
### read()

`recv()`関数とほぼ同様ですが、`pipeline`型のレスポンスについて、`read`関数は部分的に複数回読み取ることができます。これにより、メモリの使用を節約したり、プッシュされた情報をできるだけ早く受信できるようになります。一方、`recv`関数は常に全てのフレームを1つの完全なレスポンスに組み立ててから返します。

!> このメソッドを呼び出すと、[コルーチンスケジューリング](/coroutine?id=协程调度)が発生します。

```php
Swoole\Coroutine\Http2\Client->read(float $timeout): Swoole\Http2\Response;
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します。[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照してください。
      * **単位**：秒【浮動小数点数をサポートしており、例：`1.5`は`1秒`+ `500ミリ秒`を示します】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **返り値**

    成功した場合は Swoole\Http2\Response オブジェクトが返されます。
### goaway()

GOAWAYフレームは、接続の終了を開始するか、重大なエラー状態のシグナルを送信するために使用されます。

```php
Swoole\Coroutine\Http2\Client->goaway(int $error_code = SWOOLE_HTTP2_ERROR_NO_ERROR, string $debug_data): bool
```
### ping()

PINGフレームは、送信元からの最小ラウンドトリップ時間を計測し、アイドル接続がまだ有効かどうかを確認するためのメカニズムです。

```php
Swoole\Coroutine\Http2\Client->ping(): bool
```
### close()

閉じる。

```php
Swoole\Coroutine\Http2\Client->close(): bool
```
