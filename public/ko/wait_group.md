# Coroutine\WaitGroup

`Swoole4`에서는 [Channel](/coroutine/channel)을 사용하여 코루틴 간의 통신, 의존 관리, 코루틴 동기화를 구현할 수 있습니다. [Channel](/coroutine/channel)을 기반으로 하여 `Golang`의 `sync.WaitGroup` 기능을 쉽게 구현할 수 있습니다.

## 구현 코드

> 이 기능은 PHP로 작성된 기능이며, C/C++ 코드가 아닙니다. 구현된 소스 코드는 [Library](https://github.com/swoole/library/blob/master/src/core/Coroutine/WaitGroup.php) 안에 있습니다.

* `add` 메소드는 카운트를 증가시킵니다.
* `done`은 작업이 완료되었음을 나타냅니다.
* `wait`는 모든 작업이 완료될 때까지 대기하고 현재 코루틴 실행을 복원합니다.
* `WaitGroup` 객체는 재사용할 수 있으며, `add`, `done`, `wait` 이후에 다시 사용할 수 있습니다.

## 사용 예시

```php
<?php
use Swoole\Coroutine;
use Swoole\Coroutine\WaitGroup;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $wg = new WaitGroup();
    $result = [];

    $wg->add();
    // 첫 번째 코루틴 시작
    Coroutine::create(function () use ($wg, &$result) {
        // 타오바오 홈페이지를 요청하는 코루틴 클라이언트 시작
        $cli = new Client('www.taobao.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.taobao.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['taobao'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    $wg->add();
    // 두 번째 코루틴 시작
    Coroutine::create(function () use ($wg, &$result) {
        // 바이두 홈페이지를 요청하는 코루틴 클라이언트 시작
        $cli = new Client('www.baidu.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.baidu.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['baidu'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    // 모든 작업이 완료될 때까지 현재 코루틴을 일시 중지하고 기다린 뒤 다시 실행
    $wg->wait();
    // 여기에는 2개의 작업 실행 결과가 포함된 $result가 있습니다.
    var_dump($result);
});
```
