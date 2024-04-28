# 協力FastCGIクライアント

PHP-FPMでは効率的なバイナリプロトコルである `FastCGI protocol` を使用して通信を行います。 FastCGIクライアントを使用すると、HTTPリバースプロキシを介さずに直接PHP-FPMサービスとやり取りできます。

[PHPソースディレクトリ](https://github.com/swoole/library/blob/master/src/core/Coroutine/FastCGI)
## 簡単な使用例

[別のサンプルコード](https://github.com/swoole/library/tree/master/examples/fastcgi)

!> 以下のサンプルコードは、コルーチンで呼び出す必要があります。
### クイックコール

```php
#greeter.php
echo 'Hello ' . ($_POST['who'] ?? 'World');
```

```php
echo \Swoole\Coroutine\FastCGI\Client::call(
    '127.0.0.1:9000', // FPMのリスンアドレス、unixsocketアドレスのような形式も許可されます
    '/tmp/greeter.php', // 実行するエントリーポイントファイル
    ['who' => 'Swoole'] // 付随するPOST情報
);
```
```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1:9000', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withScriptFilename(__DIR__ . '/greeter.php')
        ->withMethod('POST')
        ->withBody(['who' => 'Swoole']);
    $response = $client->execute($request);
    echo "Result: {$response->getBody()}\n";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "Error: {$exception->getMessage()}\n";
}
``` 

このコードはPSRスタイルに従っています。
```php
#var.php
var_dump($_SERVER);
var_dump($_GET);
var_dump($_POST);
```

```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withDocumentRoot(__DIR__)
        ->withScriptFilename(__DIR__ . '/var.php')
        ->withScriptName('var.php')
        ->withMethod('POST')
        ->withUri('/var?foo=bar&bar=char')
        ->withHeader('X-Foo', 'bar')
        ->withHeader('X-Bar', 'char')
        ->withBody(['foo' => 'bar', 'bar' => 'char']);
    $response = $client->execute($request);
    echo "Result: \n{$response->getBody()}";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "Error: {$exception->getMessage()}\n";
}
```
### WordPressのワンクリックプロキシ

!> この方法は生産性がありません。本番環境では、古いAPIエンドポイントの一部を古いFPMサービスにプロキシするためにプロキシを使用することができます（サイト全体をプロキシするのではなく）。

```php
use Swoole\Constant;
use Swoole\Coroutine\FastCGI\Proxy;
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\Http\Server;

$documentRoot = '/var/www/html'; # WordPressプロジェクトのルートディレクトリ
$server = new Server('0.0.0.0', 80, SWOOLE_BASE); # ここでポートはWordPressの設定と一致する必要があります。通常、特定のポートを指定する必要はなく、通常80ポートです
$server->set([
    Constant::OPTION_WORKER_NUM => swoole_cpu_num() * 2,
    Constant::OPTION_HTTP_PARSE_COOKIE => false,
    Constant::OPTION_HTTP_PARSE_POST => false,
    Constant::OPTION_DOCUMENT_ROOT => $documentRoot,
    Constant::OPTION_ENABLE_STATIC_HANDLER => true,
    Constant::OPTION_STATIC_HANDLER_LOCATIONS => ['/wp-admin', '/wp-content', '/wp-includes'], # 静的リソースのパス
]);
$proxy = new Proxy('127.0.0.1:9000', $documentRoot); # プロキシオブジェクトを作成
$server->on('request', function (Request $request, Response $response) use ($proxy) {
    $proxy->pass($request, $response); # リクエストをワンクリックでプロキシ
});
$server->start();
```
## Methods
### コール

This is the description of a static method that directly creates a new client connection, sends a request to the FPM server, and receives the response body.

!> FPM only supports short connections, so in most cases, creating persistent objects doesn't make much sense.

```php
Swoole\Coroutine\FastCGI\Client::call(string $url, string $path, $data = '', float $timeout = -1): string
```

  * **Parameters** 

    * **`string $url`**
      * **Description**: FPM listening address [e.g., `127.0.0.1:9000`, `unix:/tmp/php-cgi.sock`, etc.]
      * **Default value**: N/A
      * **Other values**: N/A

    * **`string $path`**
      * **Description**: The entry file to execute
      * **Default value**: N/A
      * **Other values**: N/A

    * **`$data`**
      * **Description**: Additional request data
      * **Default value**: N/A
      * **Other values**: N/A

    * **`float $timeout`**
      * **Description**: Set timeout period [default is -1 for no timeout]
      * **Unit**: Seconds [supports floating-point numbers, e.g., 1.5 = 1s + 500ms]
      * **Default value**: `-1`
      * **Other values**: N/A

  * **Return Value** 

    * Returns the body of the server's response.
    * Throws `Swoole\Coroutine\FastCGI\Client\Exception` in case of errors.
### __construct

クライアントオブジェクトのコンストラクタ、ターゲットのFPMサーバーを指定します

```php
Swoole\Coroutine\FastCGI\Client::__construct(string $host, int $port = 0)
```

  * **Parameters**

    * **`string $host`**
      * **Description**: アドレスを指定します【例：`127.0.0.1`、`unix://tmp/php-fpm.sock`など】
      * **Default**: なし
      * **Other values**: なし

    * **`int $port`**
      * **Description**: サーバーポートを指定します【UNIXソケットの場合、必要ありません】
      * **Default**: なし
      * **Other values**: なし
### execute

リクエストを実行し、レスポンスを返す

```php
Swoole\Coroutine\FastCGI\Client->execute(Request $request, float $timeout = -1): Response
```

  * **パラメータ** 

    * **`Swoole\FastCGI\Request|Swoole\FastCGI\HttpRequest $request`**
      * **機能**：リクエスト情報を含むオブジェクトで、通常はHTTPリクエストをシミュレートするために`Swoole\FastCGI\HttpRequest`を使用し、特別な要件がある場合にのみFPMプロトコルの元のリクエストクラス`Swoole\FastCGI\Request`を使用します
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します【デフォルトは`-1`で、タイムアウトしないことを表します】
      * **単位**：秒【浮動小数点数がサポートされ、たとえば`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：`-1`
      * **その他の値**：なし

  * **戻り値** 

    * リクエストオブジェクトの型と対応するResponseオブジェクトが返され、`Swoole\FastCGI\HttpRequest`の場合は`Swoole\FastCGI\HttpResponseオブジェクト`が返され、FPMサーバーのレスポンス情報が含まれます
    * エラーが発生した場合は`Swoole\Coroutine\FastCGI\Client\Exception`例外がスローされます
```php
class HttpRequest
{
    // Implementation for HttpRequest class
}

class HttpResponse
{
    // Implementation for HttpResponse class
}
```

FastCGIモジュール用のHTTPリクエストおよびレスポンスのクラスについてのソースコードは、以下の通りです。非常にシンプルで、コード自体がドキュメントとなっています:

[Swoole\FastCGI\HttpRequest](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpRequest.php)
[Swoole\FastCGI\HttpResponse](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpResponse.php)
```
