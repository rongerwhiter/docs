# 서버 측(코루틴 스타일) <!-- {docsify-ignore-all} -->

`Swoole\Coroutine\Server`와 [비동기 스타일](/server/init)의 서버와의 차이점은 `Swoole\Coroutine\Server`가 완전히 코루틴 기반으로 구현된 서버라는 것입니다. [전체 예제](/coroutine/server?id=완전한-예시)를 참조하세요.

## 장점:

- 이벤트 콜백 함수를 설정할 필요가 없습니다. 연결 설정, 데이터 수신, 데이터 송신, 연결 종료는 모두 순차적으로 이루어지며 [비동기 스타일](/server/init)의 동시성 문제가 없습니다. 예를 들어:

```php
$serv = new Swoole\Server("127.0.0.1", 9501);

// 연결 이벤트를 수신 대기
$serv->on('Connect', function ($serv, $fd) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);// 여기서 OnConnect의 코루틴이 중단됩니다.
    Co::sleep(5);// 여기서 connect가 비교적 느린 상황을 모방하는 sleep
    $redis->set($fd,"fd $fd connected");
});

// 데이터 수신 이벤트를 수신 대기
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);// 여기서 onReceive의 코루틴이 중단됩니다.
    var_dump($redis->get($fd));// onReceive의 코루틴의 redis 연결이 먼저 생성될 수도 있으므로, 위의 set이 아직 실행되지 않은 상태에서 get이 false로 나오는 논리 오류가 발생할 수 있습니다.
});

// 연결 닫힘 이벤트를 수신 대기
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

// 서버 시작
$serv->start();
```

상기 `비동기 스타일`의 서버는 이벤트 순서를 보장할 수 없으며, 즉 `onConnect`이 끝난 후에 `onReceive`로 진입해야 한다는 것을 보장할 수 없습니다. 왜냐하면 코루틴화를 시작하면, `onConnect`와 `onReceive` 콜백이 모두 자동적으로 코루틴을 생성하며, IO를 만날 경우 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 발생하며, 비동기 스타일에서는 스케줄링 순서를 보장할 수 없지만, 코루틴 스타일의 서버는 이 문제가 없습니다.

- 서비스를 동적으로 시작 및 중지할 수 있습니다. 비동기 스타일의 서비스는 `start()`가 호출된 이후에는 아무 작업도 수행할 수 없지만, 코루틴 스타일은 동적으로 서비스를 시작하거나 중지할 수 있습니다.

## 단점:

- 코루틴 스타일의 서비스는 자동으로 여러 프로세스를 생성하지 않으며, [Process\Pool](/process/process_pool) 모듈을 사용하여 다중 코어를 활용해야 합니다.
- 코루틴 스타일 서비스는 사실상 [Co\Socket](/coroutine_client/socket) 모듈의 래핑이므로, 코루틴 스타일을 사용하려면 일정한 소켓 프로그래밍 경험이 필요합니다.
- 현재 래핑 수준이 비동기 스타일 서버보다 높지 않으며, 일부 기능은 수동으로 구현해야 합니다. 예를 들어, `reload` 기능은 신호를 수신하여 로직 실행 등을 수동으로 처리해야 합니다.
