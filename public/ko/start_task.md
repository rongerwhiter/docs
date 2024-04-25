# 비동기 작업 실행(Task)

서버 프로그램에서 시간이 많이 소요되는 작업을 수행해야 할 때(예: 채팅 서버에서 브로드캐스트 전송, 웹 서버에서 이메일 전송), 이러한 함수를 직접 실행하면 현재 프로세스를 차단하여 서버 응답이 느려질 수 있습니다.

Swoole은 비동기 작업 처리 기능을 제공하여 비동기 작업을 TaskWorker 프로세스 풀에 전달하여 현재 요청의 처리 속도에 영향을주지 않습니다.

## 프로그램 코드

첫 번째 TCP 서버를 기반으로하며 [onTask](/server/events?id=ontask)와 [onFinish](/server/events?id=onfinish) 2개의 이벤트 콜백 함수를 추가하기만 하면됩니다. 또한 작업 프로세스 수를 설정해야하며 작업의 시간 및 양에 따라 적절한 작업 프로세스를 구성할 수 있습니다.

다음 코드를 task.php에 작성하십시오.

```php
$serv = new Swoole\Server('127.0.0.1', 9501);

// 설정하는 비동기 작업의 작업 프로세스 수.
$serv->set([
    'task_worker_num' => 4
]);

// 이 콜백 함수는 worker 프로세스에서 실행됩니다.
$serv->on('Receive', function($serv, $fd, $reactor_id, $data) {
    // 비동기 작업 전달
    $task_id = $serv->task($data);
    echo "Dispatch AsyncTask: id={$task_id}\n";
});

// 비동기 작업 처리(Task 이벤트 콜백 함수는 task 프로세스에서 실행됨).
$serv->on('Task', function ($serv, $task_id, $reactor_id, $data) {
    echo "New AsyncTask[id={$task_id}]".PHP_EOL;
    // 작업 실행 결과 반환
    $serv->finish("{$data} -> OK");
});

// 비동기 작업 결과 처리(이 콜백 함수는 worker 프로세스에서 실행됩니다).
$serv->on('Finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[{$task_id}] Finish: {$data}".PHP_EOL;
});

$serv->start();
```

`$serv->task()`를 호출한 후 프로그램이 즉시 반환되어 코드 계속 실행됩니다. onTask 콜백 함수는 Task 프로세스 풀에서 비동기로 실행됩니다. 실행이 완료되면 `$serv->finish()`를 호출하여 결과를 반환합니다.

!> finish 작업은 선택 사항이며 결과를 반환하지 않을 수도 있습니다. `onTask` 이벤트에서 결과를 반환하려면 `return`을 통해 반환하는 것과 동일하게 `Swoole\Server::finish()` 작업을 호출하는 것입니다.
