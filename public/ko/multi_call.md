# 동시 호출

[//]: # (
여기에는 setDefer 기능이 삭제되었습니다. 왜냐하면 setDefer를 지원하는 클라이언트는 모두 "한 번에 코루틴 변환"을 권장합니다.
)

`go` + `channel`을 사용하여 동시 요청을 구현합니다.

!>[개요](/coroutine)를 먼저 읽고 코루틴의 기본 개념을 파악한 후 이 섹션을 봐주시기 바랍니다.

### 구현 원리

* `onRequest` 함수에서 두 개의 `HTTP` 요청을 동시에 보내려면 `go` 함수를 사용하여 2개의 자식 코루틴을 생성하고 여러 `URL`에 동시에 요청합니다.
* 또한 `channel`을 만들고 `use` 클로저 참조 문법을 사용하여 이를 자식 코루틴에 전달합니다.
* 주 코루틴은 `chan->pop`을 반복 호출하여 자식 코루틴이 작업을 완료할 때까지 기다리며 `yield`를 사용하여 일시 중단 상태로 들어갑니다.
* 두 개의 동시 자식 코루틴 중 하나가 요청을 완료하면 `chan->push`를 호출하여 데이터를 주 코루틴으로 푸시합니다.
* 자식 코루틴이 `URL` 요청을 완료하면 종료되고, 주 코루틴은 중단된 상태에서 복구되어 계속 진행하며 `$resp->end`를 호출하여 응답 결과를 보냅니다.

### 사용 예시

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

!> 더 간단하게 하려면 `Swoole`이 제공하는[WaitGroup](/coroutine/wait_group) 기능을 사용하면 됩니다.
