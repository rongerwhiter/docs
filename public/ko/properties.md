# 속성
### $setting

[Server->set()](/server/methods?id=set) 함수로 설정된 매개변수는 `Server->$setting` 속성에 저장됩니다. 콜백 함수에서 실행 매개변수의 값을 액세스할 수 있습니다. 이 속성은 `array` 유형의 배열입니다.

```php
Swoole\Server->setting
```

  * **예제**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(array('worker_num' => 4));

echo $server->setting['worker_num'];
```
### $connections

`TCP` 연결 반복자는 `foreach`를 사용하여 현재 서버의 모든 연결을 반복할 수 있습니다. 이 속성의 기능은 [Server->getClientList](/server/methods?id=getclientlist)와 동일하지만 더 사용하기 쉽습니다.

반복되는 요소는 개별 연결의 `fd`입니다.

```php
Swoole\Server->connections
```

!> `$connections` 속성은 반복자 객체이며 PHP 배열이 아니기 때문에 `var_dump` 또는 배열 첨자를 사용하여 액세스할 수 없습니다. 대신 `foreach`를 통해 반복 작업을 수행해야 합니다.

  * **기본 모드**

    * [SWOOLE_BASE](/learn?id=swoole_base) 모드에서 `TCP` 연결에 대한 프로세스 간 조작이 지원되지 않으므로 `BASE` 모드에서는 현재 프로세스에서만 `$connections` 반복자를 사용할 수 있습니다.

  * **예시**

```php
foreach ($server->connections as $fd) {
  var_dump($fd);
}
echo "현재 서버에는 " . count($server->connections) . "개의 연결이 있습니다.\n";
```
### $host

`host` 속성은 현재 서버가 수신 대기 중인 호스트 주소를 나타내는 `string` 타입의 문자열입니다.

```php
Swoole\Server->host
```
### $port

`port` 속성은 현재 서버가 수신 대기하는 포트를 나타내며 `int` 유형의 정수입니다.

```php
Swoole\Server->port
```
### $type

현재 Server의 유형을 나타내는 `type` 속성입니다. 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server->type
```
!> 이 속성은 다음 값 중 하나를 반환합니다.
- `SWOOLE_SOCK_TCP` TCP IPv4 소켓
- `SWOOLE_SOCK_TCP6` TCP IPv6 소켓
- `SWOOLE_SOCK_UDP` UDP IPv4 소켓
- `SWOOLE_SOCK_UDP6` UDP IPv6 소켓
- `SWOOLE_SOCK_UNIX_DGRAM` Unix 소켓 DGRAM
- `SWOOLE_SOCK_UNIX_STREAM` Unix 소켓 스트림
### $ssl

현재 서버가 `ssl`을 사용하는지 여부를 반환합니다. 이 속성은 `bool` 유형입니다.

```php
Swoole\Server->ssl
```
### $mode

현재 서버의 프로세스 모드 `mode`를 반환합니다. 이 속성은 `int` 타입의 정수입니다.

```php
Swoole\Server->mode
```

!> 이 속성은 다음 값 중 하나를 반환합니다.
- `SWOOLE_BASE` 단일 프로세스 모드
- `SWOOLE_PROCESS` 다중 프로세스 모드
### $포트

`$ports`는 포트 배열로, 서버가 여러 포트를 수신할 경우 `Server::$ports`를 반복하여 모든 `Swoole\Server\Port` 객체를 얻을 수 있습니다.

여기서 `swoole_server::$ports[0]`은 생성자에 설정된 주 서버 포트입니다.

  * **예시**

```php
$ports = $server->ports;
$ports[0]->set($settings);
$ports[1]->on('Receive', function () {
    //콜백
});
```
### $master_pid

현재 서버의 주 프로세스 `PID`를 반환합니다.

```php
Swoole\Server->master_pid
```

!> `onStart/onWorkerStart` 이후에만 얻을 수 있습니다.

  * **예시**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->master_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $manager_pid

현재 서버 관리 프로세스의 `PID`를 반환하는 속성입니다. 이 속성은 `int` 형식의 정수입니다.

```php
Swoole\Server->manager_pid
```

!> `onStart/onWorkerStart` 이후에만 액세스할 수 있습니다.

  * **예시**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->manager_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $worker_id

현재 `Worker` 프로세스의 ID를 가져옵니다. [Task worker 프로세스](/learn?id=taskworker프로세스)도 포함됩니다. 이 속성은 `int`형 정수입니다.

```php
Swoole\Server->worker_id
```
  * **예시**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num' => 8,
    'task_worker_num' => 4,
]);
$server->on('WorkerStart', function ($server, int $workerId) {
    if ($server->taskworker) {
        echo "task workerId：{$workerId}\n";
        echo "task worker_id：{$server->worker_id}\n";
    } else {
        echo "workerId：{$workerId}\n";
        echo "worker_id：{$server->worker_id}\n";
    }
});
$server->on('Receive', function ($server, $fd, $reactor_id, $data) {
});
$server->on('Task', function ($serv, $task_id, $reactor_id, $data) {
});
$server->start();
```

  * **팁**

    * 이 속성은 [onWorkerStart](/server/events?id=onworkerstart)의 `$workerId`와 동일합니다.
    * `Worker` 프로세스의 ID 범위는 `[0, $server->setting['worker_num'] - 1]`입니다.
    * [Task worker 프로세스](/learn?id=taskworker프로세스)의 ID 범위는 `[$server->setting['worker_num'], $server->setting['worker_num'] + $server->setting['task_worker_num'] - 1]` 입니다.

!> 작업 프로세스가 다시 시작된 후 `worker_id` 값은 변경되지 않습니다.
### $taskworker

현재 프로세스가 `Task` 프로세스인지 여부를 나타냅니다. 이 속성은 `bool` 타입입니다.

```php
Swoole\Server->taskworker
```

  * **Return Value**

    * `true`: 현재 프로세스가 `Task` 작업 프로세스인 경우
    * `false`: 현재 프로세스가 `Worker` 프로세스인 경우
### $worker_pid

현재 `Worker` 프로세스의 운영 체제 프로세스 ID를 가져옵니다. `posix_getpid()`의 반환 값과 동일한 값이며, 이 속성은 정수형 `int`입니다.

```php
Swoole\Server->worker_pid
```
