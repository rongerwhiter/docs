# 並行呼び出し

[//]: # (
このセットディファー機能は削除しました。というのは、セットディファーをサポートするクライアントはすべて、一括コルーチン化を推奨しています。
)

`サブコルーチン（go）` + `チャネル（channel）`を使用して並行リクエストを実装します。

!> まず[概要](/coroutine)をご覧いただくことをお勧めします。協調基本概念を理解した後、このセクションをご覧ください。

### 実装原理

* `onRequest`内で2つの`HTTP`リクエストを並行して行う必要がある場合、`go`関数を使用して`2`つのサブコルーチンを作成し、複数の`URL`に対して並行リクエストを行います。
* `channel`を作成し、`use`クロージャ参照構文を使用して、子コルーチンに渡します。
* メインコルーチンは`chan->pop`を繰り返し呼び出し、子コルーチンがタスクを完了するのを待ちます。`yield`して一時停止状態に入ります。
* 並行して実行される2つのサブコルーチンのうちいずれかがリクエストを完了すると、`chan->push`を呼び出してデータをメインコルーチンにプッシュします。
* 子コルーチンが`URL`のリクエストを完了した後、終了し、メインコルーチンは一時停止状態から復帰し、続行して`$resp->end`を呼び出してレスポンス結果を送信します。

### 使用例

```php
$serv = new Swoole\Http\Server("127.0.0.1", 9503, SWOOLE_BASE);

$serv->on('request', function ($req, $resp) {
	$chan = new Channel(2);
	go(function () use ($chan) {
		$cli = new Swoole\Coroutine\Http\Client('www.qq.com', 80);
			$cli->set(['timeout' => 10]);
			$cli->setHeaders([
			'Host' => "www.qq.com",
			"User-Agent" => 'Chrome/49.0.2587.3',
			'Accept' => 'text/html,application/xhtml+xml,application/xml',
			'Accept-Encoding' => 'gzip',
		]);
		$ret = $cli->get('/');
		$chan->push(['www.qq.com' => $cli->body]);
	});

	go(function () use ($chan) {
		$cli = new Swoole\Coroutine\Http\Client('www.163.com', 80);
		$cli->set(['timeout' => 10]);
		$cli->setHeaders([
			'Host' => "www.163.com",
			"User-Agent" => 'Chrome/49.0.2587.3',
			'Accept' => 'text/html,application/xhtml+xml,application/xml',
			'Accept-Encoding' => 'gzip',
		]);
		$ret = $cli->get('/');
		$chan->push(['www.163.com' => $cli->body]);
	});
	
	$result = [];
	for ($i = 0; $i < 2; $i++)
	{
		$result += $chan->pop();
	}
	$resp->end(json_encode($result));
});
$serv->start();
```

!> `Swoole`が提供する[WaitGroup](/coroutine/wait_group)機能を使用すると、もう少し簡単になります。
