# HTTP サーバー

## プログラムコード

以下のコードをhttpServer.phpに書き込んでください。

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('Request', function ($request, $response) {
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});

$http->start();
```

`HTTP`サーバーはリクエストとレスポンスに焦点を当てる必要があるため、 `onRequest` イベントを1つだけ監視すれば十分です。新しい`HTTP`リクエストが発生するとこのイベントがトリガーされます。イベントコールバック関数には `2` つのパラメータがあります。1 つは`$request`オブジェクトで、リクエストに関連する情報を含みます。

もう 1 つは`response`オブジェクトで、`response`オブジェクトを操作してリクエストに対する応答を完了できます。`$response->end()`メソッドは、HTMLコンテンツを出力し、このリクエストを終了することを意味します。

* `0.0.0.0` はすべての`IP`アドレスをリッスンすることを表します。1 台のサーバーには複数の`IP`がある場合があり、例えば `127.0.0.1` ループバックIP、`192.168.1.100` ローカルネットワークIP、`210.127.20.2` インターネットIPがあります。ここで、1 つのIPを指定することもできます。
* `9501` はリッスンするポート番号で、使用中の場合、プログラムは致命的なエラーをスローして実行を中断します。

## サービスの起動

```shell
php httpServer.php
```
* ブラウザを開いて`http://127.0.0.1:9501`にアクセスしてプログラムの結果を表示することができます。
* Apache `ab`ツールを使用してサーバーに負荷テストを行うこともできます。

## Chrome による二重リクエストの問題

`Chrome` ブラウザでサーバーにアクセスすると、追加のリクエスト `/favicon.ico` が生成されますが、コード内で `404` エラーを返すことができます。

```php
$http->on('Request', function ($request, $response) {
    if ($request->server['path_info'] == '/favicon.ico' || $request->server['request_uri'] == '/favicon.ico') {
        $response->end();
        return;
    }
    var_dump($request->get, $request->post);
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});
```

## URL ルーティング

アプリケーションは `$request->server['request_uri']` に基づいてルーティングを実装できます。例えば：`http://127.0.0.1:9501/test/index/?a=1`という場合、コード内で次のようにURLルーティングを実装できます。

```php
$http->on('Request', function ($request, $response) {
    list($controller, $action) = explode('/', trim($request->server['request_uri'], '/'));
    // $controller、$actionに基づいて異なるコントローラークラスやメソッドにマップする。
    (new $controller)->$action($request, $response);
});
```
