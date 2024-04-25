# Http\Server

?> `Http\Server`は[Server](/server/init)から継承しているため、`Server`が提供するすべての`API`や設定が使用できます。プロセスモデルも同じです。[Server](/server/init)セクションを参照してください。

組み込みの`HTTP`サーバーのサポートを利用して、数行のコードで高並列、高パフォーマンス、[非同期I/O](/learn?id=同期io异步io)を持つマルチプロセス`HTTP`サーバーを作成できます。

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->start();
```

`Apache bench`ツールを使用してストレステストを行った結果、通常のPCマシン（`Inter Core-I5 4-core + 8GB`メモリ）で、`Http\Server`は約`11万QPS`に達することができます。

これは`PHP-FPM`、`Golang`、`Node.js`のデフォルトの`HTTP`サーバーをはるかに上回ります。性能はほぼ`Nginx`の静的ファイル処理と同等です。

```shell
ab -c 200 -n 200000 -k http://127.0.0.1:9501/
```

* **HTTP2 プロトコルの使用**

  * `SSL`を使用した`HTTP2`プロトコルを使用するには、`openssl`をインストールする必要があり、高いバージョンの`openssl`が`TLS1.2`、`ALPN`、`NPN`をサポートしている必要があります。
  * コンパイル時には[--enable-http2](/environment?id=编译选项)を有効にする必要があります。
  * Swoole5からは、デフォルトでhttp2プロトコルが有効になっています。

```shell
./configure --enable-openssl --enable-http2
```

`HTTP`サーバーの[open_http2_protocol](/http_server?id=open_http2_protocol)を`true`に設定します。

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_cert_file' => $ssl_dir . '/ssl.crt',
    'ssl_key_file' => $ssl_dir . '/ssl.key',
    'open_http2_protocol' => true,
]);
```

* **Nginx + Swoole の設定**

!> `Http\Server`は`HTTP`プロトコルのサポートが完全ではないため、動的リクエストを処理するためのアプリケーションサーバーとしてのみ使用し、前段に`Nginx`を逆プロキシとして追加することをお勧めします。

```nginx
server {
    listen 80;
    server_name swoole.test;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:9501;
    }
}
```

?> クライアントの真の`IP`を取得するには`$request->header['x-real-ip']`を読み取ることができます。
## 方法

This text does not require translation.
### on()

?> **Register event callback functions.**

?> Similar to [Server callbacks](/server/events), but with the following differences:

  - `Http\Server->on` does not accept [onConnect](/server/events?id=onconnect)/[onReceive](/server/events?id=onreceive) callback settings
  - `Http\Server->on` accepts an additional event type called `onRequest`, which is triggered when a client request is received

```php
$http_server->on('request', function(\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
     $response->end("<h1>hello swoole</h1>");
});
```

This function will be called when a complete HTTP request is received. The callback function takes `2` parameters:

* [Swoole\Http\Request](/http_server?id=httpRequest): HTTP request information object, which includes headers, GET/POST data, cookies, etc.
* [Swoole\Http\Response](/http_server?id=httpResponse): HTTP response object, enabling HTTP operations such as setting cookies, headers, statuses, etc.

!> When the [onRequest](/http_server?id=on) callback function returns, the `$request` and `$response` objects will be destroyed.
```php
Swoole\Http\Server->start();
```
## Swoole\Http\Request

`HTTP`リクエストオブジェクトは、GET、POST、COOKIE、Headerなどの関連する情報を保持します。

!> `Http\Request`オブジェクトを参照するために`&`記号を使用しないでください。
### ヘッダー

?> **`HTTP`リクエストのヘッダー情報。配列のタイプであり、すべての`key`は小文字です。**

```php
Swoole\Http\Request->header: array
```

* **例**

```php
echo $request->header['host'];
echo $request->header['accept-language'];
```
### サーバー

?> **HTTPリクエストに関連するサーバー情報。**

?> PHPの`$_SERVER`配列に相当します。HTTPリクエストのメソッド、URLパス、クライアントのIPアドレスなどが含まれています。

```php
Swoole\Http\Request->server: array
```

配列の`key`はすべて小文字であり、`PHP`の`$_SERVER`配列と合致しています。

* **例**

```php
echo $request->server['request_time'];
```

key | 説明
---|---
query_string | `GET`パラメーターのリクエスト、例：`id=1&cid=2` GETパラメータがない場合、この項目は存在しません
request_method | リクエストメソッド、`GET/POST`など
request_uri | GETパラメータのないアクセス先アドレス、例：`/favicon.ico`
path_info | `request_uri`と同じ
request_time | `request_time`は`Worker`で設定されており、[SWOOLE_PROCESS](/learn?id=swoole_process)モードでは`dispatch`プロセスが存在するため、実際のパケット受信時間と大きく異なる場合があります。特に、サーバーの処理能力を超えるリクエストがある場合、`request_time`は実際のパケット受信時間よりもかなり遅れる可能性があります。`$server->getClientInfo`メソッドを使用して`last_time`を取得することで、正確な受信時間を取得できます。
request_time_float | リクエストが開始される時間のタイムスタンプ、マイクロ秒単位、`float`型、例：`1576220199.2725`
server_protocol | サーバープロトコルのバージョン番号、`HTTP`の場合：`HTTP/1.0`または`HTTP/1.1`、`HTTP2`の場合：`HTTP/2`
server_port | サーバーがリッスンしているポート
remote_port | クライアントのポート
remote_addr | クライアントのIPアドレス
master_time | 前回通信時刻
### get

?> **`HTTP`リクエストの`GET`パラメータは、`PHP`の`$_GET`に相当し、配列形式です。**

```php
Swoole\Http\Request->get: array
```

* **例**

```php
// 例：index.php?hello=123
echo $request->get['hello'];
// すべてのGETパラメータを取得
var_dump($request->get);
```

* **注意**

!> `HASH`攻撃を防ぐため、`GET`パラメータの最大許容数は`128`を超えてはいけません。
### 投稿

?> **`HTTP`リクエストの`POST`パラメーターは、配列の形式です。**

```php
Swoole\Http\Request->post: array
```

* **例**

```php
echo $request->post['hello'];
```

* **注意**

!> - `POST`および`Header`の合計サイズは[package_max_length](/server/setting?id=package_max_length)の設定を超えてはなりません。超えると悪意のあるリクエストと見なされます  
- `POST`パラメーターの最大数は128個を超えてはいけません
### クッキー

?> **HTTPリクエストに含まれるCOOKIE情報は、キーと値の配列形式です。**

```php
Swoole\Http\Request->cookie: array
```

* **例**

```php
echo $request->cookie['username'];
```
### ファイル

?> **ファイルのアップロード情報。**

?> 'form'名が'key'である二次元配列です。PHPの`$_FILES`と同様です。ファイルサイズは[package_max_length](/server/setting?id=package_max_length)で設定された値を超えてはいけません。Swooleはメッセージを解析する際にメモリを占有するため、メッセージが大きいほど、メモリの使用量も大きくなります。そのため、`Swoole\Http\Server`を使用して大きなファイルのアップロードやユーザーが自分で断点再開機能を設計することは避けてください。

```php
Swoole\Http\Request->files: array
```

* **例**

```php
Array
(
    [name] => facepalm.jpg // ブラウザからのアップロード時に渡されるファイル名
    [type] => image/jpeg // MIMEタイプ
    [tmp_name] => /tmp/swoole.upfile.n3FmFr // アップロードされた一時ファイル、ファイル名は/tmp/swoole.upfileで始まります
    [error] => 0
    [size] => 15476 // ファイルサイズ
)
```

* **注意**

!> `Swoole\Http\Request`オブジェクトが破棄されると、アップロードされた一時ファイルは自動的に削除されます
### getContent()

!> Swoole version >= `v4.5.0` が利用可能で、低いバージョンでは`rawContent`の別名を使用できます（この別名は常に保持され、下位互換性が保たれます）

?> **`POST`リクエストの元のボディを取得します。**

?> `application/x-www-form-urlencoded`形式以外のHTTP `POST`リクエストに使用されます。元の`POST`データを返します。この関数は`PHP`の`fopen('php://input')`と同等です。

```php
Swoole\Http\Request->getContent(): string|false
```

* **Return Values**

  * コンテキスト接続が存在しない場合は`false`を返します。

!> 一部のケースでは、サーバーはHTTP `POST`リクエストのパラメータを解析する必要がない場合があります。[http_parse_post](/http_server?id=http_parse_post)構成により、`POST`データの解析を無効にすることができます。
### getData()

?> **HTTP2を除いて、完全な元の`Http`リクエストメッセージを取得します。`Http Header`と`Http Body`が含まれます**

```php
Swoole\Http\Request->getData(): string|false
```

  * **Return Value**

    * 成功した場合はメッセージが返され、コンテキスト接続が存在しないか、`Http2`モードである場合には`false`が返されます
### create()

?> **Create a `Swoole\Http\Request` object.**

!> Available in Swoole version >= `v4.6.0`

```php
Swoole\Http\Request->create(array $options): Swoole\Http\Request
```

  * **Parameters**

    * **`array $options`**
      * **Description**: Optional parameters to configure the `Request` object

| Parameter                                         | Default Value | Description                                           |
| ------------------------------------------------- | ------------- | ----------------------------------------------------- |
| [parse_cookie](/http_server?id=http_parse_cookie) | true          | Set whether to parse `Cookie`                         |
| [parse_body](/http_server?id=http_parse_post)      | true          | Set whether to parse `Http Body`                      |
| [parse_files](/http_server?id=http_parse_files)   | true          | Set the switch to parse uploaded files                |
| enable_compression                                | true, false if compression unsupported by the server | Set whether to enable compression                      |
| compression_level                                 | 1             | Set compression level, range is 1-9, higher level results in smaller compressed size but higher CPU consumption |
| upload_tmp_dir                                    | /tmp          | Temporary file storage location for file uploads      |

  * **Return Value**

    * Returns a `Swoole\Http\Request` object

* **Example**
```php
Swoole\Http\Request::create([
    'parse_cookie' => true,
    'parse_body' => true,
    'parse_files' => true,
    'enable_compression' => true,
    'compression_level' => 1,
    'upload_tmp_dir' => '/tmp',
]);
```
### parse()

?> **HTTPリクエストデータパケットを解析し、正常に解析されたデータの長さを返します。**

!> Swoole version >= `v4.6.0` で使用できます

```php
Swoole\Http\Request->parse(string $data): int|false
```

  * **Parameters**

    * **`string $data`**
      * 解析するパケットデータ

  * **Return Value**

    * 解析に成功すると解析されたデータの長さが返され、接続コンテキストが存在しないか、コンテキストが既に終了している場合は`false`が返されます
### isCompleted()

?> **Get whether the current `HTTP` request data packet has reached the end.**

!> Available in Swoole version >= `v4.6.0`

```php
Swoole\Http\Request->isCompleted(): bool
```

  * **Return Value**

    * `true` if it is the end, `false` means the connection context has ended or not reached the end

* **Example**

```php
use Swoole\Http\Request;

$data = "GET /index.html?hello=world&test=2123 HTTP/1.1\r\n";
$data .= "Host: 127.0.0.1\r\n";
$data .= "Connection: keep-alive\r\n";
$data .= "Pragma: no-cache\r\n";
$data .= "Cache-Control: no-cache\r\n";
$data .= "Upgrade-Insecure-Requests: \r\n";
$data .= "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36\r\n";
$data .= "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n";
$data .= "Accept-Encoding: gzip, deflate, br\r\n";
$data .= "Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,ja;q=0.6\r\n";
$data .= "Cookie: env=pretest; phpsessid=fcccs2af8673a2f343a61a96551c8523d79ea; username=hantianfeng\r\n";

/** @var Request $req */
$req = Request::create(['parse_cookie' => false]);
var_dump($req);

var_dump($req->isCompleted());
var_dump($req->parse($data));

var_dump($req->parse("\r\n"));
var_dump($req->isCompleted());

var_dump($req);
// Since cookie parsing is disabled, it will be null
var_dump($req->cookie);
```
### getMethod()

?> **Get the current `HTTP` request method.**

!> Available since Swoole version >= `v4.6.2`

```php
Swoole\Http\Request->getMethod(): string|false
```
  * **Return Value**

    * Returns the uppercase request method if successful, `false` indicates that the connection context does not exist

```php
var_dump($request->server['request_method']);
var_dump($request->getMethod());
```
## Swoole\Http\Response

`HTTP`レスポンスオブジェクトは、このオブジェクトのメソッドを呼び出すことで`HTTP`レスポンスの送信が可能です。

?> `Response`オブジェクトが破棄されるとき、`[end](/http_server?id=end)`を呼び出して`HTTP`レスポンスを送信しなかった場合、自動的に`end("")`が実行されます。

!> `Http\Response`オブジェクトを参照する際に`&`記号を使用しないでください。
### header() :id=setheader

?> **HTTPレスポンスのヘッダー情報を設定する**【エイリアス：`setHeader`】

```php
Swoole\Http\Response->header(string $key, string $value, bool $format = true): bool;
```

* **パラメータ** 

  * **`string $key`**
    * **機能**：`HTTP`ヘッダーの`Key`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`string $value`**
    * **機能**：`HTTP`ヘッダーの`value`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`bool $format`**
    * **機能**：`Key`を`HTTP`の規約に従ってフォーマットする必要があるか【デフォルトは`true`で自動フォーマットされる】
    * **デフォルト値**：`true`
    * **その他の値**：なし

* **戻り値** 

  * 設定に失敗すると`false`を返す
  * 設定に成功すると`true`を返す
* **注意**

    -`header`を設定する際は`end`メソッドの前に設定する必要がある  
    -`$key`は完全に`HTTP`の規約に準拠している必要があり、各語の最初の文字が大文字で、中国語、アンダースコア、または他の特殊文字を含んではいけない  
    -`$value`は必ず記入する必要がある  
    -`$ucwords`を`true`に設定すると、下位レイヤーが自動的に`$key`を規約に従ってフォーマットする  
    -同じ`$key`の`HTTP`ヘッダーを複数回設定すると、最後の一回だけが適用される  
    -クライアントが`Accept-Encoding`を設定している場合、サーバーは`Content-Length`応答を設定できない、この状況を検知すると`Swoole`は`Content-Length`の値を無視し、警告を出す  
    -`Content-Length`応答を設定した場合、`Swoole\Http\Response::write()`を呼び出すことはできない、`Swoole`はこの状況を検知すると`Content-Length`の値を無視し、警告を出す

!> Swoole バージョン >= `v4.6.0`では、同じ`$key`の`HTTP`ヘッダーを複数回設定でき、`$value`は`array`、`object`、`int`、`float`など複数のタイプをサポートし、下位レイヤーは`toString`変換を行い、末尾の空白と改行を削除します。

* **例**

```php
$response->header('content-type', 'image/jpeg', true);

$response->header('Content-Length', '100002 ');
$response->header('Test-Value', [
    "a\r\n",
    'd5678',
    "e  \n ",
    null,
    5678,
    3.1415926,
]);
$response->header('Foo', new SplFileInfo('bar'));
```
### trailer()

?> **`Header`情報を`HTTP`レスポンスの末尾に追加し、`HTTP2`でのみ利用可能で、メッセージ完全性のチェック、デジタル署名などに使用されます。**

```php
Swoole\Http\Response->trailer(string $key, string $value): bool;
```

* **パラメータ** 

  * **`string $key`**
    * **機能**：`HTTP`ヘッダーの`Key`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`string $value`**
    * **機能**：`HTTP`ヘッダーの`value`
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値** 

  * 設定に失敗した場合は`false`を返す
  * 設定に成功した場合は`true`を返す

* **注意**

  !> 同じ`$key`を持つ`Http`ヘッダーを繰り返し設定すると、上書きされます。最後のものが取り出されます。

* **例**

```php
$response->trailer('grpc-status', 0);
$response->trailer('grpc-message', '');
```
### cookie()

?> **HTTPレスポンスのcookie情報を設定します。エイリアスは`setCookie`です。このメソッドの引数は、PHPの`setcookie`と同じです。**

```php
Swoole\Http\Response->cookie(string $key, string $value = '', int $expire = 0 , string $path = '/', string $domain  = '', bool $secure = false , bool $httponly = false, string $samesite = '', string $priority = ''): bool;
```

  * **パラメーター** 

    * **`string $key`**
      * **機能**：CookieのKey
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $value`**
      * **機能**：Cookieのvalue
      * **デフォルト値**：なし
      * **その他の値**：なし
  
    * **`int $expire`**
      * **機能**：Cookieの有効期限
      * **デフォルト値**：0、期限なし
      * **その他の値**：なし

    * **`string $path`**
      * **機能**：Cookieのサーバーパスを指定します。
      * **デフォルト値**：/
      * **その他の値**：なし

    * **`string $domain`**
      * **機能**：Cookieのドメインを指定します
      * **デフォルト値**：''
      * **その他の値**：なし

    * **`bool $secure`**
      * **機能**：安全なHTTPS接続を介してCookieを送信するかどうかを指定します
      * **デフォルト値**：''
      * **その他の値**：なし

    * **`bool $httponly`**
      * **機能**：HttpOnly属性を持つCookieにブラウザのJavaScriptからアクセスを許可するかどうか、`true`は許可しないことを示し、`false`は許可することを示します
      * **デフォルト値**：false
      * **その他の値**：なし

    * **`string $samesite`**
      * **機能**：サードパーティCookieを制限し、セキュリティリスクを軽減します。 `Strict`、`Lax`、`None`のいずれかが選択できます
      * **デフォルト値**：''
      * **その他の値**：なし

    * **`string $priority`**
      * **機能**：Cookieの優先度。Cookieの数量が規定を超えると、低優先度のものが先に削除されます。 `Low`、`Medium`、`High`のいずれかが選択できます
      * **デフォルト値**：''
      * **その他の値**：なし
  
  * **戻り値** 

    * 失敗した場合は`false`を返します
    * 成功した場合は`true`を返します 

* **注意**

  !> - `cookie`の設定は[end](/http_server?id=end)メソッドの前に行う必要があります
  - `samesite` パラメータは `v4.4.6` からサポートされるようになりました。 `priority` パラメータは `v4.5.8` からサポートされるようになりました
  - `Swoole`は`$value`を自動的に`urlencode`エンコードしますが、`rawCookie()`メソッドを使用して`$value`のエンコード処理を無効にできます
  - `Swoole`は、同じ`$key`の複数の`COOKIE`を設定することを許可します
### rawCookie()

?> **HTTPレスポンスのcookie情報を設定します**

!> `rawCookie()`関数は、`cookie()`関数と同じパラメータを取りますが、エンコード処理を行いません。
### status()

?> **Sending `Http` status code. Aliased as `setStatusCode()`**

```php
Swoole\Http\Response->status(int $http_status_code, string $reason = ''): bool
```

* **Parameters** 

  * **`int $http_status_code`**
    * **Description**：Set `HttpCode`
    * **Default value**：None
    * **Other values**：None

  * **`string $reason`**
    * **Description**：Reason for the status code
    * **Default value**：''
    * **Other values**：None

  * **Return Value** 

    * Returns `false` if setting fails
    * Returns `true` if setting is successful

* **Note**

  * If only the first parameter `$http_status_code` is passed in, it must be a valid `HttpCode`, such as `200`, `502`, `301`, `404`, etc. Otherwise, it will be set to `200` status code.
  * If the second parameter `$reason` is set, `$http_status_code` can be any value, including undefined `HttpCode`, such as `499`.
  * The `status` method must be executed before calling [$response->end()](/http_server?id=end).
### gzip()

!> `gzip()` method has been deprecated since `4.1.0`, please refer to [http_compression](/http_server?id=http_compression)；In the new version, the `http_compression` configuration option is used instead of the `gzip` method.  
The main reason is that the `gzip()` method did not check the `Accept-Encoding` header passed by the client browser, which could cause the client to fail to decompress if it does not support `gzip` compression when forced to use it.  
The brand new `http_compression` configuration option will automatically choose whether to compress and select the best compression algorithm based on the client's `Accept-Encoding` header.

?> **Enable `Http GZIP` compression. Compression can reduce the size of `HTML` content, effectively save network bandwidth, and improve response time. `gzip` must be executed before sending content in `write/end`, or an error will be thrown.**
```php
Swoole\Http\Response->gzip(int $level = 1);
```

* **Parameters** 
   
     * **`int $level`**
       * **Description**：Compression level, the higher the level, the smaller the size after compression, but more `CPU` consumption.
       * **Default Value**：1
       * **Other Values**：`1-9`

!> After calling the `gzip` method, the underlying system will automatically add `Http` encoding headers, so relevant `Http` headers should not be set in PHP code; Images in `jpg/png/gif` formats are already compressed and do not need to be compressed again.

!> `gzip` function depends on the `zlib` library. During the compilation of Swoole, the system will check if `zlib` exists. If not, the `gzip` method will not be available. You can install the `zlib` library using `yum` or `apt-get`:

```shell
sudo apt-get install libz-dev
```
### redirect()

?> **`Http`リダイレクトを送信します。このメソッドを呼び出すと、自動的に`end`が送信されてレスポンスが終了します。**

```php
Swoole\Http\Response->redirect(string $url, int $http_code = 302): bool
```

* **パラメータ**
  * **`string $url`**
    * **機能**：リダイレクト先の新しいアドレス、`Location`ヘッダーとして送信されます
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $http_code`**
    * **機能**：ステータスコード【デフォルトは`302`一時的リダイレクトで、`301`を渡すことで永続的リダイレクトを示します】
    * **デフォルト値**：`302`
    * **その他の値**：なし

* **戻り値**
  * 呼び出しに成功すると`true`が返り、呼び出しに失敗するか接続コンテキストが存在しない場合は`false`が返ります

* **例**

```php
$http = new Swoole\Http\Server("0.0.0.0", 9501, SWOOLE_BASE);

$http->on('request', function ($req, Swoole\Http\Response $resp) {
    $resp->redirect("http://www.baidu.com/", 301);
});

$http->start();
```
### write()

?> **Enable `Http Chunk` to send response content to the browser in segments.**

?> You can refer to the `Http` protocol standard documentation for more information about `Http Chunk`.

```php
Swoole\Http\Response->write(string $data): bool
```

  * **Parameters** 

    * **`string $data`**
      * **Functionality**: The data content to be sent 【Maximum length must not exceed `2M`, controlled by [buffer_output_size](/server/setting?id=buffer_output_size) configuration】
      * **Default Value**: None
      * **Other Values**: None

  * **Return Value** 
  
    * Returns `true` upon successful call, returns `false` if the call fails or connection context does not exist

* **Tips**

  * After using `write` to send data in segments, the [end](/http_server?id=end) method will not accept any parameters. Calling `end` will simply send a `Chunk` of length `0` to indicate that data transmission is complete
  * If `Content-Length` is set via the Swoole\Http\Response::header() method, and then this method is called, `Swoole` will ignore the `Content-Length` setting and throw a warning
  * `Http2` cannot use this function, otherwise a warning will be thrown
  * If the client supports response compression, `Swoole\Http\Response::write()` will forcibly disable compression
### sendfile()

?> **ファイルをブラウザに送信します。**

```php
Swoole\Http\Response->sendfile(string $filename, int $offset = 0, int $length = 0): bool
```

  * **パラメータ** 

    * **`string $filename`**
      * **機能**：送信するファイルの名前【ファイルが存在しないかアクセス権がない場合、`sendfile`は失敗します】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $offset`**
      * **機能**：ファイルのオフセット【ファイルの中間部分からデータを転送することができます。これは断片転送をサポートするために使用できます】
      * **デフォルト値**：`0`
      * **その他の値**：なし

    * **`int $length`**
      * **機能**：送信するデータのサイズ
      * **デフォルト値**：ファイルのサイズ
      * **その他の値**：なし

  * **戻り値** 

      * 成功すると`true`を返し、失敗した場合や接続コンテキストが存在しない場合は`false`を返します

* **ヒント**

  * ファイルのMIME形式を下位レイヤーで推測することはできませんので、アプリケーションコードで`Content-Type`を指定する必要があります
  * `sendfile`を呼び出す前に`write`メソッドで`Http-Chunk`を送信してはいけません
  * `sendfile`を呼び出した後、下位レイヤーは自動的に`end`を実行します
  * `sendfile`は`gzip`圧縮をサポートしていません

* **例**

```php
$response->header('Content-Type', 'image/jpeg');
$response->sendfile(__DIR__.$request->server['request_uri']);
```
### end()

?> **Send `Http` response body and end request processing.**

```php
Swoole\Http\Response->end(string $html): bool
```

  * **Parameters**
  
    * **`string $html`**
      * **Function**：Content to send
      * **Default value**：None
      * **Other values**：None

  * **Return Value**
  
    * If successful, `true` is returned, if failed or connection context does not exist, `false` is returned.

* **Tips**

  * `end` can only be called once. If you need to send data to the client multiple times, use the [write](/http_server?id=write) method
  * If the client has enabled [KeepAlive](/coroutine_client/http_client?id=keep_alive), the connection will be kept, and the server will wait for the next request
  * If the client has not enabled `KeepAlive`, the server will disconnect
  * Due to the limit of [output_buffer_size](/server/setting?id=buffer_output_size), the content to be sent by `end` is limited by default to `2M`. If it exceeds this limit, the response will fail, and an error like the following will be thrown:

!> The solution is to use [sendfile](/http_server?id=sendfile), [write](/http_server?id=write), or adjust [output_buffer_size](/server/setting?id=buffer_output_size)

```bash
WARNING finish (ERRNO 1203): The length of data [262144] exceeds the output buffer size[131072], please use the sendfile, chunked transfer mode or adjust the output_buffer_size
```
### detach()

?> **レスポンスオブジェクトを分離します。** このメソッドを使用すると、`$response`オブジェクトが破棄される際に自動的に[end](/http_server?id=httpresponse)されなくなります。[Http\Response::create](/http_server?id=create)と[Server->send](/server/methods?id=send)と一緒に使用します。

```php
Swoole\Http\Response->detach(): bool
```

  * **返り値** 

    * 成功した場合は`true`が返されます。失敗した場合や接続コンテキストが存在しない場合は`false`が返されます。

* **例** 

  * **異プロセスへのレスポンス**

  ?> 特定の場合、[Taskプロセス](/learn?id=taskworkerプロセス)でクライアントにレスポンスを返す必要があります。このような場合、`detach`を使用して`$response`オブジェクトを独立させることができます。[Taskプロセス](/learn?id=taskworkerプロセス)で`$response`を再構築し、HTTPリクエストに応答を行うことができます。

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->set(['task_worker_num' => 1, 'worker_num' => 1]);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->task(strval($resp->fd));
  });

  $http->on('finish', function () {
      echo "task finish";
  });

  $http->on('task', function ($serv, $task_id, $worker_id, $data) {
      var_dump($data);
      $resp = Swoole\Http\Response::create($data);
      $resp->end("in task");
      echo "async task\n";
  });

  $http->start();
  ```

  * **任意のコンテンツを送信**

  ?> 特定のシナリオでは、クライアントに特別なレスポンス内容を送信する必要があります。`Http\Response`オブジェクトに付属の`end`メソッドでは要件を満たすことができない場合、`detach`を使用してレスポンスオブジェクトを分離し、それからHTTPプロトコルのレスポンスデータを独自で組み立て、`Server->send`を使用してデータを送信できます。

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->send($resp->fd, "HTTP/1.1 200 OK\r\nServer: server\r\n\r\nHello World\n");
  });

  $http->start();
  ```
### create()

?> **Create a new `Swoole\Http\Response` object.**

!> Before using this method, be sure to call the `detach` method to detach the old `$response` object, as failing to do so may result in sending two response contents for the same request.

```php
Swoole\Http\Response::create(object|array|int $server = -1, int $fd = -1): Swoole\Http\Response
```

  * **Parameters** 

    * **`int $server`**
      * **Description**: `Swoole\Server` or `Swoole\Coroutine\Socket` object, an array (with two parameters - the first one being the `Swoole\Server` object, and the second one being the `Swoole\Http\Request` object), or a file descriptor
      * **Default Value**: -1
      * **Other Values**: N/A

    * **`int $fd`**
      * **Description**: File descriptor. Required if the `$server` parameter is a `Swoole\Server` object
      * **Default Value**: -1
      * **Other Values**: N/A

  * **Return Value** 

    * Returns a new `Swoole\Http\Response` object if the call is successful, `false` if failed

* **Example**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
    $resp->detach();
    // Example 1
    $resp2 = Swoole\Http\Response::create($req->fd);
    // Example 2
    $resp2 = Swoole\Http\Response::create($http, $req->fd);
    // Example 3
    $resp2 = Swoole\Http\Response::create([$http, $req]);
    // Example 4
    $socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
    $socket->connect('127.0.0.1', 9501)
    $resp2 = Swoole\Http\Response::create($socket);
    $resp2->end("hello world");
});

$http->start();
```
### isWritable()

?> **`Swoole\Http\Response`オブジェクトが終了(`end`)または分離(`detach`)されているかどうかを判断します。**

```php
Swoole\Http\Response->isWritable(): bool
```

  * **返り値** 

    * `Swoole\Http\Response`オブジェクトが終了していない場合または分離されていない場合は`true`を返し、それ以外の場合は`false`を返します。


!> Swooleのバージョン >= `v4.6.0` で使用可能

* **例**

```php
use Swoole\Http\Server;
use Swoole\Http\Request;
use Swoole\Http\Response;

$http = new Server('0.0.0.0', 9501);

$http->on('request', function (Request $req, Response $resp) {
    var_dump($resp->isWritable()); // true
    $resp->end('hello');
    var_dump($resp->isWritable()); // false
    $resp->setStatusCode(403); // http response is unavailable (maybe it has been ended or detached)
});

$http->start();
```
## 配置选项
### http_parse_cookie

?> **Configuration for the `Swoole\Http\Request` object, disable cookie parsing, keeping the unprocessed original `Cookies` information in the `header`. Enabled by default**

```php
$server->set([
    'http_parse_cookie' => false,
]);
```  

### http_parse_cookie

?> **`Swoole\Http\Request`オブジェクトの構成で、Cookieの解析を無効にし、`header`に未処理の元の`Cookies`情報を保持します。デフォルトで有効**  

```php
$server->set([
    'http_parse_cookie' => false,
]);
```  
### http_parse_post

?> **Configuration for parsing POST messages related to `Swoole\Http\Request` objects, defaults to enabled**

* When set to `true`, it automatically parses the request body with `Content-Type x-www-form-urlencoded` into the `POST` array.
* When set to `false`, POST parsing will be disabled.

```php
$server->set([
    'http_parse_post' => false,
]);
```
### http_parse_files

?> **Set the upload file parsing switch for the `Swoole\Http\Request` object. Enabled by default.**

```php
$server->set([
    'http_parse_files' => false,
]);
```
### http_compression

?> **`Swoole\Http\Response`オブジェクト向けの設定で、圧縮を有効にします。デフォルトは有効です。**

!> - `http-chunk`はセグメントごとの個別の圧縮をサポートしていません。[write](/http_server?id=write)メソッドを使用すると、圧縮が強制的に無効になります。  
- `http_compression`は`v4.1.0`またはそれ以降で使用可能です。

```php
$server->set([
    'http_compression' => false,
]);
```

現在、`gzip`、`br`、`deflate`の3種類の圧縮形式がサポートされており、基盤となる動作はブラウザクライアントからの`Accept-Encoding`ヘッダに基づいて自動的に圧縮方法を選択します（圧縮アルゴリズムの優先度：`br` > `gzip` > `deflate`）。

**依存関係：**

`gzip`および`deflate`は`zlib`ライブラリに依存しており、`Swoole`のコンパイル時にシステムに`zlib`が存在するかどうかがチェックされます。

`yum`または`apt-get`を使用して`zlib`ライブラリをインストールできます：

```shell
sudo apt-get install libz-dev
```

`br`圧縮形式は`google`の`brotli`ライブラリに依存しており、インストール方法については自分で検索してください。`Swoole`のコンパイル時に、システムに`brotli`が存在するかどうかがチェックされます。
### http_compression_level / compression_level / http_gzip_level

?> **`Swoole\Http\Response`オブジェクト向けの圧縮レベルの設定**
  
!> `$level` 圧縮レベルは`1-9`の範囲です。レベルが高いほど圧縮後のサイズは小さくなりますが、`CPU`の消費量が増えます。デフォルトは`1`で、最大は`9`です。
### http_compression_min_length / compression_min_length

?> **`Swoole\Http\Response`オブジェクトに対する設定で、圧縮を有効にする最小バイト数を設定します。デフォルトは20バイトです。**

!> 利用可能なのはSwooleバージョン >= `v4.6.3`

```php
$server->set([
    'compression_min_length' => 128,
]);
```
### upload_tmp_dir

?> **Set the temporary directory for uploaded files. The directory must not exceed a maximum length of `220` bytes.**

```php
$server->set([
    'upload_tmp_dir' => '/data/uploadfiles/',
]);
```
### upload_max_filesize

?> **Upload file max size configuration**

```php
$server->set([
    'upload_max_filesize' => 5 * 1024,
]);
```
### enable_static_handler

静的ファイルリクエスト処理機能を有効にします。`document_root`と一緒に使用する必要があります。デフォルトは`false`です。
### http_autoindex

`http autoindex`機能を有効にします。デフォルトでは無効です。
### http_index_files

`http_autoindex`と一緒に使用すると、索引化されるファイルのリストを指定します。

```php
$server->set([
    'document_root' => '/data/webroot/example.com',
    'enable_static_handler' => true,
    'http_autoindex' => true,
    'http_index_files' => ['indesx.html', 'index.txt'],
]);
```
### http_compression_types / compression_types

?> **`Swoole\Http\Response`オブジェクトに対する設定で、圧縮するレスポンスタイプを設定します**

```php
$server->set([
        'http_compression_types' => [
            'text/html',
            'application/json'
        ],
    ]);
```

!> Swoole バージョン >= `v4.8.12` で利用可能
### static_handler_locations

?> **静的ハンドラのパスを設定します。タイプは配列で、デフォルトでは無効です。**

!> Swooleのバージョン >= `v4.4.0` で利用可能

```php
$server->set([
    'static_handler_locations' => ['/static', '/app/images'],
]);
```

* `Nginx`の`location`ディレクティブに似ており、1つまたは複数のパスを静的パスとして指定できます。指定されたパスのみが`URL`内にある場合に静的ファイルハンドラーが有効になります。それ以外の場合は動的リクエストと見なされます。
* `location`項目は/で始まる必要があります。
* `/app/images`のように複数階層のパスをサポートしています。
* `static_handler_locations`を有効にした場合、リクエストに対応するファイルが存在しない場合、直接404エラーが返されます。
### open_http2_protocol

?> **Enable the `HTTP2` protocol parsing** [default: `false`]

!> You need to enable the [--enable-http2](/environment?id=compile-options) option at compile time. `Swoole5` compiles with http2 enabled by default.
### document_root

?> **静的なファイルのルートディレクトリを配置し、`enable_static_handler`と一緒に使用します。**

!> この機能はかなり簡単ですので、公共ネットワーク環境で直接使用しないでください

```php
$server->set([
    'document_root' => '/data/webroot/example.com', // v4.4.0以下のバージョンでは、ここには絶対パスを指定する必要があります
    'enable_static_handler' => true,
]);
```

* `document_root`を設定し、`enable_static_handler`を`true`に設定した後、下位層は`Http`リクエストを受信すると、まずdocument_rootのパスにリクエストされたファイルが存在するかどうかを判断し、存在する場合はファイルの内容を直接クライアントに送信し、もはや[onRequest](/http_server?id=on)コールバックをトリガーしません。
* 静的ファイル処理機能を使用する場合、動的PHPコードと静的ファイルを分離するため、静的ファイルを特定のディレクトリに配置する必要があります
### max_concurrency

?> **可限制 `HTTP1/2` 服务的最大并发请求数量，超过之后返回 `503` 错误，默认值为4294967295，即为无符号int的最大值**

```php
$server->set([
    'max_concurrency' => 1000,
]);
```
### worker_max_concurrency

一括並行化を有効にした後、`worker`プロセスはリクエストを絶え間なく受け取ります。過度な負荷を避けるために、`worker`プロセスのリクエスト実行数を制限するために`worker_max_concurrency`を設定できます。リクエスト数がこの値を超えると、`worker`プロセスは余分なリクエストをキューに一時保持します。デフォルト値は4294967295で、これは符号なし整数の最大値です。`worker_max_concurrency`を設定しない場合は、`max_concurrency`を設定している場合、Swooleは自動的に`worker_max_concurrency`を`max_concurrency`と同じ値に設定します。

```php
$server->set([
    'worker_max_concurrency' => 1000,
]);
```

Swooleバージョン >= `v5.0.0` で利用可能
### http2_header_table_size

?> Define the maximum size of the `header table` for HTTP/2 network connections.

```php
$server->set([
  'http2_header_table_size' => 0x1
])
``` 

定義HTTP/2ネットワーク接続の最大`header table`サイズ。
### http2_enable_push

?> この設定は、HTTP2プッシュを有効または無効にするために使用されます。

```php
$server->set([
  'http2_enable_push' => 0x2
])
```
### http2_max_concurrent_streams

?> Setting the maximum number of multiplexed streams to be received per HTTP/2 connection.

```php
$server->set([
  'http2_max_concurrent_streams' => 0x3
])
``` 

### http2_max_concurrent_streams

?> HTTP/2接続毎に受信する多重化されたストリームの最大数を設定します。

```php
$server->set([
  'http2_max_concurrent_streams' => 0x3
])
```
### http2_init_window_size

HTTP/2ストリーム制御ウィンドウの初期サイズを設定します。

```php
$server->set([
  'http2_init_window_size' => 0x4
])
```  
### http2_max_frame_size

?> HTTP/2ネットワーク接続を介して送信される単一のHTTP/2プロトコルフレームの本文の最大サイズを設定します。

```php
$server->set([
  'http2_max_frame_size' => 0x5
])
```
### http2_max_header_list_size

?> HTTP/2ストリーム上で送信できるリクエストヘッダーの最大サイズを設定します。

```php
$server->set([
  'http2_max_header_list_size' => 0x6
])
```  
