# Coroutine HTTP/WebSocket client

The coroutine version of the `HTTP` client is written in pure `C` at the lower level and does not depend on any third-party extension libraries, boasting extremely high performance.

* Supports `Http-Chunk`, `Keep-Alive` features, supports `form-data` format
* The `HTTP` protocol version is `HTTP/1.1`
* Supports upgrading to a `WebSocket` client
* `gzip` compression format support requires the `zlib` library
* The client only implements core functionality; it is recommended to use [Saber](https://github.com/swlib/saber) in actual projects.
This text does not need translation as it is a code block.
### errCode

エラーステータスコード。`connect/send/recv/close`が失敗した場合、またはタイムアウトした場合、`Swoole\Coroutine\Http\Client->errCode`の値が自動的に設定されます。

```php
Swoole\Coroutine\Http\Client->errCode: int
```

`errCode`の値は`Linux errno`と等しいです。`socket_strerror`を使用してエラーコードをエラーメッセージに変換できます。

```php
// 接続拒否の場合、エラーコードは111
// タイムアウトした場合、エラーコードは110
echo socket_strerror($client->errCode);
```

!> 参考：[Linux エラーコード一覧](/other/errno?id=linux)
### body

上記は前回のリクエストのレスポンスボディを保存します。

```php
Swoole\Coroutine\Http\Client->body: string
```

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->get('/get');
    echo $cli->body;
    $cli->close();
});
```
### statusCode

HTTPステータスコード、例えば200、404などです。ステータスコードが負数の場合、接続に問題があることを示します。
[詳細を見る](/coroutine_client/http_client?id=getstatuscode)

```php
Swoole\Coroutine\Http\Client->statusCode: int
```
```python
def greet():
    print("Hello, how can I help you today?")
```

### __construct()

构造方法。

```php
Swoole\Coroutine\Http\Client::__construct(string $host, int $port, bool $ssl = false);
```

  * **参数** 

    * **`string $host`**
      * **功能**：目标服务器主机地址【可以为IP或域名，底层自动进行域名解析，若是本地UNIXSocket则应以形如`unix://tmp/your_file.sock`的格式填写；若是域名不需要填写协议头`http://`或`https://`】
      * **默认值**：无
      * **其它值**：无

    * **`int $port`**
      * **功能**：目标服务器主机端口
      * **默认值**：无
      * **其它值**：无

    * **`bool $ssl`**
      * **功能**：是否启用`SSL/TLS`隧道加密，如果目标服务器是https必须设置`$ssl`参数为`true`
      * **默认值**：`false`
      * **其它值**：无

  * **示例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->set(['timeout' => 1]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```
### set()

クライアントパラメータを設定します。

```php
Swoole\Coroutine\Http\Client->set(array $options);
```

このメソッドは、`Swoole\Client->set` が受け入れるオプションと完全に同じです。詳細は [Swoole\Client->set](/client?id=set) のドキュメントを参照してください。

`Swoole\Coroutine\Http\Client` には、`HTTP`および`WebSocket`クライアントを制御するための追加オプションがあります。
#### 额外选项

Translate the text inside code block:
```plaintext
#### 额外选项
```
##### タイムアウト制御

`timeout`オプションを設定し、HTTPリクエストのタイムアウト検出を有効にします。単位は秒であり、最小の粒度はミリ秒をサポートしています。

```php
$http->set(['timeout' => 3.0]);
```

- 接続がタイムアウトするか、サーバーが接続を閉じた場合、`statusCode`は`-1`に設定されます
- 指定された時間内にサーバーが応答を返さない場合、リクエストがタイムアウトし、`statusCode`は`-2`に設定されます
- リクエストがタイムアウトした後、バックエンドは自動的に接続を切断します
- [クライアントのタイムアウト規則](/coroutine_client/init?id=超时规则)を参照
##### keep_alive

`keep_alive`オプションを設定して、HTTP長い接続を有効または無効にします。

```php
$http->set(['keep_alive' => false]);
```
##### websocket_mask

> Due to RFC regulations, this configuration is enabled by default after v4.4.0, but it may cause performance loss. If the server does not have a mandatory requirement, you can set it to false to disable.

`WebSocket`クライアントでマスクを有効または無効にします。デフォルトは有効です。有効にすると、WebSocketクライアントが送信するデータに対してマスクを使用してデータ変換を行います。

```php
$http->set(['websocket_mask' => false]);
```
##### websocket_compression

> Requires `v4.4.12` or higher

When set to `true`, **allows** frames to be compressed with zlib; whether compression is actually applied depends on the server's ability to handle compression (determined by handshake information, refer to `RFC-7692`)

To actually compress a specific frame, you need to use the `SWOOLE_WEBSOCKET_FLAG_COMPRESS` flag parameter. For detailed usage, refer to [this section](/websocket_server?id=websocket-frames-compression-(rfc-7692))

```php
$http->set(['websocket_compression' => true]);
```
### setMethod()

リクエストメソッドを設定します。この設定は現在のリクエストでのみ有効であり、リクエストを送信した後にメソッドの設定はすぐにクリアされます。

```php
Swoole\Coroutine\Http\Client->setMethod(string $method): void
```

  * **パラメーター** 

    * **`string $method`**
      * **機能**：メソッドを設定します 
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `HTTP`標準に準拠したメソッド名である必要があります。`$method`が誤って設定されると、`HTTP`サーバーによってリクエストが拒否される可能性があります。

  * **例**

```php
$http->setMethod("PUT");
```  
### setHeaders()

HTTPリクエストヘッダを設定します。

```php
Swoole\Coroutine\Http\Client->setHeaders(array $headers): void
```

  * **パラメータ** 

    * **`array $headers`**
      * **目的**：リクエストヘッダを設定します 【キーと値のペアの配列でなければなりません。下層では、`$key`:`$value`形式の標準の`HTTP`ヘッダ形式に自動マッピングされます】
      * **デフォルト値**：なし
      * **他の値**：なし

!> `setHeaders`で設定された`HTTP`ヘッダは、`Coroutine\Http\Client`オブジェクトの寿命中、毎回のリクエストで永続的に有効です。`setHeaders`を再度呼び出すと、前回の設定が上書きされます。
### setCookies()

`Cookie`を設定し、値は`urlencode`でエンコードされ、元の情報を維持する場合は、`setHeaders`を使用して`Cookie`という名前の`header`を設定してください。

```php
Swoole\Coroutine\Http\Client->setCookies(array $cookies): void
```

  * **Parameters** 

    * **`array $cookies`**
      * **Description**: `COOKIE`を設定します【キーと値が対応する配列である必要があります】
      * **Default Value**: なし
      * **Other Values**: なし

!> -`COOKIE`を設定すると、クライアントオブジェクトが有効な間は継続して保存されます  
-サーバーが設定した`COOKIE`は`cookies`配列にマージされ、現在の`HTTP`クライアントの`COOKIE`情報を取得するには`$client->cookies`プロパティを使用できます  
-`setCookies`メソッドを繰り返し呼び出すと、現在の`Cookies`状態が上書きされ、以前にサーバーが発行した`COOKIE`および以前に設定した`COOKIE`が破棄されます
### setData()

HTTPリクエストのボディを設定します。

```php
Swoole\Coroutine\Http\Client->setData(string|array $data): void
```

  * **パラメータ** 

    * **`string|array $data`**
      * **説明**：リクエストのボディを設定します
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * `$data`を設定した場合、かつ`$method`が設定されていない場合、自動的にPOSTに設定されます
    * `$data`が配列であり`Content-Type`が`application/x-www-form-urlencoded`形式の場合、内部では`http_build_query`が自動的に行われます
    * `addFile`や`addData`を使用して`form-data`形式が有効になっている場合、`$data`が文字列の場合は無視されます(形式が異なるため)が、配列の場合は、内部で`form-data`形式で配列のフィールドが追加されます
### addFile()

ファイルをPOSTします。

!> `addFile`を使用すると、`POST`の`Content-Type`が自動的に`form-data`に変更されます。`addFile`は`sendfile`に基づいており、非同期で非常に大きなファイルの送信をサポートできます。

```php
Swoole\Coroutine\Http\Client->addFile(string $path, string $name, string $mimeType = null, string $filename = null, int $offset = 0, int $length = 0): void
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：ファイルのパス【必須パラメータで、存在しないファイルや空のファイルは指定できません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $name`**
      * **機能**：フォームの名前【必須パラメータで、`FILES`パラメータ内のキーです】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $mimeType`**
      * **機能**：ファイルの`MIME`タイプ【オプションのパラメータで、ファイルの拡張子に基づいて自動的に推測されます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $filename`**
      * **機能**：ファイル名【オプションのパラメータ】
      * **デフォルト値**：`basename($path)`
      * **その他の値**：なし

    * **`int $offset`**
      * **機能**：ファイルのオフセットをアップロード【オプションのパラメータで、データをファイルの中間部分から転送することができます。この機能はレジューム機能のサポートに使用できます。】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $length`**
      * **機能**：データのサイズを送信【オプションのパラメータ】
      * **デフォルト値**：ファイル全体のサイズがデフォルトです
      * **その他の値**：なし

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $cli->set(['timeout' => -1]);
    $cli->addFile(__FILE__, 'file1', 'text/plain');
    $cli->post('/post', ['foo' => 'bar']);
    echo $cli->body;
    $cli->close();
});
```
### addData()

Uploads file content using a string.

!> `addData` is available in version `v4.1.0` and above.

```php
Swoole\Coroutine\Http\Client->addData(string $data, string $name, string $mimeType = null, string $filename = null): void
```

  * **Parameters** 

    * **`string $data`**
      * **Function**: Data content 【Required parameter, maximum length not to exceed [buffer_output_size](/server/setting?id=buffer_output_size)】
      * **Default value**: None
      * **Other values**: None

    * **`string $name`**
      * **Function**: Name of the form 【Required parameter, `key` in `$_FILES` parameter】
      * **Default value**: None
      * **Other values**: None

    * **`string $mimeType`**
      * **Function**: MIME format of the file 【Optional parameter, default is `application/octet-stream`】
      * **Default value**: None
      * **Other values**: None

    * **`string $filename`**
      * **Function**: File name 【Optional parameter, default is `$name`】
      * **Default value**: None
      * **Other values**: None

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('httpbin.org', 80);
    $client->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $client->set(['timeout' => -1]);
    $client->addData(Co::readFile(__FILE__), 'file1', 'text/plain');
    $client->post('/post', ['foo' => 'bar']);
    echo $client->body;
    $client->close();
});
```
### get()

GETリクエストを送信します。

```php
Swoole\Coroutine\Http\Client->get(string $path): void
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：`URL`のパスを設定します【例：`/index.html`、ここには`http://domain`を渡すことはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```

!> `get`を使用すると、`setMethod`で設定されたリクエストメソッドが無視され、強制的に`GET`が使用されます。
### post()

POSTリクエストを発行します。

```php
Swoole\Coroutine\Http\Client->post(string $path, mixed $data): void
```

  * **パラメーター** 

    * **`string $path`**
      * **機能**：`URL`パスを設定します【例：`/index.html`、ここでは`http://domain`を渡すことはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`mixed $data`**
      * **機能**：リクエストデータ
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `$data`が配列の場合、自動的に`x-www-form-urlencoded`形式の`POST`コンテンツにパッケージ化され、`Content-Type`が`application/x-www-form-urlencoded`に設定されます

  * **注意**

    !> `post`を使用すると、`setMethod`で設定されたリクエストメソッドが無視され、強制的に`POST`が使用されます

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->post('/post.php', array('a' => '123', 'b' => '456'));
    echo $client->body;
    $client->close();
});
```
### upgrade()

`WebSocket`接続にアップグレードします。

```php
Swoole\Coroutine\Http\Client->upgrade(string $path): bool
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：`URL`パスを設定します【例：`/`、ここで`http://domain`を渡すことはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * 一部の場合、リクエストは成功して`upgrade`は`true`を返しますが、サーバーが`HTTP`ステータスコードを`101`ではなく`200`または`403`に設定している場合、これはサーバーがハンドシェイクリクエストを拒否したことを意味します
    * `WebSocket`ハンドシェイクが成功した後、`push`メソッドを使用してサーバーにメッセージを送信したり、`recv`を呼び出してメッセージを受信したりできます
    * `upgrade`は一度の[協力的なスケジューリング](/coroutine?id=協力的なスケジューリング)を引き起こします

  * **例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```  
### push()

`WebSocket`サーバーにメッセージをプッシュします。

!> `push`メソッドは`upgrade`成功後にのみ実行できます  
`push`メソッドは[コルーチンスケジューリング](/coroutine?id=协程调度)を生成しません。書き込み送信バッファに書き込んだ後、すぐにリターンします

```php
Swoole\Coroutine\Http\Client->push(mixed $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

  * **Parameters** 

    * **`mixed $data`**
      * **Description**：送信するデータの内容【デフォルトは`UTF-8`のテキスト形式です。他のエンコーディングやバイナリデータの場合は`WEBSOCKET_OPCODE_BINARY`を使用してください】
      * **Default**：None
      * **Other values**：None

      !> Swoole version >= v4.2.0 `$data`は [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトを使用可能で、さまざまなフレームタイプを送信できます。

    * **`int $opcode`**
      * **Description**：操作タイプ
      * **Default**：`WEBSOCKET_OPCODE_TEXT`
      * **Other values**：None

      !> `$opcode`は有効な`WebSocket OPCode`である必要があります。そうでない場合、失敗し、エラー情報`opcode max 10`が出力されます。

    * **`int|bool $finish`**
      * **Description**：操作タイプ
      * **Default**：`SWOOLE_WEBSOCKET_FLAG_FIN`
      * **Other values**：None

      !> `v4.4.12`バージョン以降、`finish`パラメータ(`bool`型)は`flags`(`int`型)に変更され、`WebSocket`圧縮をサポートするため、`finish`は`SWOOLE_WEBSOCKET_FLAG_FIN`値である`1`に対応しています。以前の`bool`型の値は暗黙的に`int`型に変換され、この変更により下位互換性があります。 また、圧縮`flag`は`SWOOLE_WEBSOCKET_FLAG_COMPRESS`です。

  * **Return Value**

    * 送信に成功した場合、`true`を返します。
    * 接続が存在しない、閉じられている、`WebSocket`が完了していない場合、送信に失敗した場合は`false`を返します。

  * **Error Codes**

Error Code | Description
---|---
8502 | 不正な OPCode
8503 | サーバーに接続されていないか接続がすでに閉じられています
8504 | ハンドシェイクが失敗しました
### recv()

メッセージを受信します。`WebSocket`のみに使用され、`upgrade()`と一緒に使用する必要があります。例を参照してください。

```php
Swoole\Coroutine\Http\Client->recv(float $timeout = 0)
```

  * **Arguments** 

    * **`float $timeout`**
      * **Description**：`upgrade()`を呼び出して`WebSocket`接続にアップグレードする場合にのみこのパラメータが有効です
      * **Unit**：seconds【浮動小数点型がサポートされており、`1.5`は`1秒`+`500ミリ秒`を示します】
      * **Default**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照
      * **Other values**：無し

      !> タイムアウトを設定し、指定されたパラメータを優先して使用します。次に、`set`メソッドで渡された`timeout`構成を使用します。
  
  * **Return Value**

    * 正常に実行すると、frameオブジェクトを返します
    * 失敗した場合は`false`を返し、`Swoole\Coroutine\Http\Client`の`errCode`属性をチェックしてください。コルーチンクライアントには`onClose`コールバックがなく、接続が閉じられた場合、recvは`false`を返し、errCode=0になります。
 
  * **Example**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```
### download()

ファイルをHTTP経由でダウンロードします。

!> downloadメソッドは、データを受信した後、それをディスクに書き込みます。HTTP Bodyをメモリ内で結合するのではなく、ダウンロードメソッドを使用することで、少量のメモリで非常に大きなファイルのダウンロードが可能です。

```php
Swoole\Coroutine\Http\Client->download(string $path, string $filename, int $offset = 0): bool
```

  * **パラメーター** 

    * **`string $path`**
      * **機能**：`URL`パスを設定します
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $filename`**
      * **機能**：ダウンロードした内容を書き込むファイルのパスを指定します【`downloadFile`プロパティに自動的に書き込まれます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $offset`**
      * **機能**：ファイルへの書き込みオフセットを指定します【これは一時停止と再開をサポートするために使用でき、`HTTP`ヘッダー`Range:bytes=$offset`と組み合わせて使用できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `$offset`が`0`の場合、ファイルが既に存在する場合、ファイルが自動的にクリアされます

  * **戻り値**

    * 正常に実行された場合は`true`を返します
    * ファイルのオープンに失敗した場合や`fseek()`ファイルが失敗した場合は`false`を返します

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $host = 'cdn.jsdelivr.net';
    $client = new Client($host, 443, true);
    $client->set(['timeout' => -1]);
    $client->setHeaders([
        'Host' => $host,
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => '*',
        'Accept-Encoding' => 'gzip'
    ]);
    $client->download('/gh/swoole/swoole-src/mascot.png', __DIR__ . '/logo.png');
});
```
### getCookies()

`HTTP`応答の`cookie`コンテンツを取得します。

```php
Swoole\Coroutine\Http\Client->getCookies(): array|false
```

!> Cookie情報はurldecodeデコードされます。元のCookie情報を取得するには、以下の手順に従って解析してください。
```php
var_dump($client->set_cookie_headers);
````

```php
var_dump($client->headers);
```
```php
Swoole\Coroutine\Http\Client->getHeaders(): array|false
```
### getStatusCode()

`HTTP`レスポンスのステータスコードを取得します。

```php
Swoole\Coroutine\Http\Client->getStatusCode(): int|false
```

  * **ヒント**

    * **ステータスコードが負数の場合、接続に問題があることを示します。**

ステータスコード | v4.2.10 以上のバージョンに対応する定数 | 説明
---|---|---
-1 | SWOOLE_HTTP_CLIENT_ESTATUS_CONNECT_FAILED | 接続タイムアウト、サーバーがポートを監視していないかネットワーク接続が失われた場合、具体的なネットワークエラーコードを$errCodeで取得できます
-2 | SWOOLE_HTTP_CLIENT_ESTATUS_REQUEST_TIMEOUT | リクエストがタイムアウトしました。サーバーは指定されたtimeout時間内に応答を返していません
-3 | SWOOLE_HTTP_CLIENT_ESTATUS_SERVER_RESET | クライアントのリクエスト送信後、サーバーが強制的に接続を切断しました
-4 | SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED | クライアントが送信に失敗しました（この定数はSwooleバージョン>=`v4.5.9`で使用可能で、それより前のバージョンではステータスコードを使用してください）
### getBody()

`HTTP`レスポンスのボディコンテンツを取得します。

```php
Swoole\Coroutine\Http\Client->getBody(): string|false
```
### close()

接続を閉じます。

```php
Swoole\Coroutine\Http\Client->close(): bool
```

!> `close`を呼び出した後、`get`、`post`などのメソッドを再度リクエストすると、Swooleがサーバーに再接続します。
### execute()

`execute()`は、より低レベルのHTTPリクエストメソッドであり、[setMethod](/coroutine_client/http_client?id=setmethod)や[setData](/coroutine_client/http_client?id=setdata)などのインターフェースを使用して、リクエストのメソッドやデータを設定する必要があります。

```php
Swoole\Coroutine\Http\Client->execute(string $path): bool
```

* **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $httpClient = new Client('httpbin.org', 80);
    $httpClient->setMethod('POST');
    $httpClient->setData('swoole');
    $status = $httpClient->execute('/post');
    var_dump($status);
    var_dump($httpClient->getBody());
});
```
## Functions

To facilitate the use of `Coroutine\Http\Client`, three functions have been added:

!> Available in Swoole version >= `v4.6.4`
```php
function request(string $url, string $method, $data = null, array $options = null, array $headers = null, array $cookies = null)
```  
```php
function post(string $url, $data, array $options = null, array $headers = null, array $cookies = null)
```
### get()

`GET`リクエストを送信するために使用されます。

```php
function get(string $url, array $options = null, array $headers = null, array $cookies = null)
```
```php
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\Http\get;
use function Swoole\Coroutine\Http\post;
use function Swoole\Coroutine\Http\request;

run(function () {
    go(function () {
        $data = get('http://httpbin.org/get?hello=world');
        $body = json_decode($data->getBody());
        assert($body->headers->Host === 'httpbin.org');
        assert($body->args->hello === 'world');
    });
    go(function () {
        $random_data = base64_encode(random_bytes(128));
        $data = post('http://httpbin.org/post?hello=world', ['random_data' => $random_data]);
        $body = json_decode($data->getBody());
        assert($body->headers->Host === 'httpbin.org');
        assert($body->args->hello === 'world');
        assert($body->form->random_data === $random_data);
    });
});
```
