# Swoole\Thread

`6.0` 버전부터 다중 스레드 지원이 제공되었습니다. 스레드 API를 사용하여 다중 프로세스 대신 사용할 수 있습니다. 다중 프로세스 대비, `Thread`는 더 풍부한 동시성 데이터 구조를 제공하므로 게임 서버, 통신 서버를 개발하는 데 더 편리합니다.

## Compile
- `PHP`는 `ZTS` 모드여야하며 `PHP`를 컴파일 할 때 `--enable-zts`를 추가해야 합니다.
- `Swoole`을 컴파일 할 때 `--enable-swoole-thread` 컴파일 옵션을 추가해야 합니다.

`--enable-swoole-thread` 컴파일 매개변수는 비-스레드 모드인 `Server`, `Atomic`, `Lock` 모듈과 상호 배타적이며, `thread` 기능을 활성화하면 다중 프로세스 모드에서 사용할 수 없습니다.

## Information

```shell
php -v
PHP 8.1.23 (cli) (built: Mar 20 2024 19:48:19) (ZTS)
Copyright (c) The PHP Group
Zend Engine v4.1.23, (c) Zend Technologies
```

`(ZTS)`는 스레드 안전을 활성화했음을 나타냅니다.

```shell
php --ri swoole
php --ri swoole

swoole

Swoole => enabled
...
thread => enabled
...
```

`thread => enabled`는 다중 스레드 지원이 활성화되었음을 나타냅니다.

## Create Thread

```php
Thread::exec(string $script_file, array ...$argv);
```

참고로 생성된 자식 스레드는 부모 스레드로부터 어떤 리소스도 상속받지 않으므로, 자식 스레드에서 다음 내용은 모두 초기화되어야 합니다:
- 이미로드된 `PHP` 파일, 다시 `include/require`로 로드해야 함
- `autoload`
- 클래스, 함수, 상수는 초기화되며, 다시 `PHP` 파일을 로드해야 함
- 전역 변수, 예: `$GLOBALS`, `$_GET/$_POST` 등은 초기화됨
- 클래스의 정적 속성, 함수의 정적 변수는 초기값으로 재설정됨
- `php.ini` 옵션, 예: `error_reporting()`은 자식 스레드에서 다시 설정해야 함

데이터를 자식 스레드에 전달하려면 스레드 매개변수를 사용해야 합니다. 자식 스레드에서는 여전히 새로운 스레드를 생성할 수 있습니다.

### Parameters
- `$script_file`: 스레드가 시작한 후 실행할 스크립트
- `...$argv`: 스레드 매개변수를 전달하며, 직렬화 가능한 변수여야 하며 `resource` 리소스 핸들을 전달할 수 없고, 자식 스레드에서는 `Thread::getArguments()`를 사용할 수 있음

### Return Value
`Thread` 객체를 반환하며, 부모 스레드에서 자식 스레드에 대해 `join()` 등의 작업을 수행할 수 있습니다.

스레드 객체가 소멸될 때 자동으로 `join()` 등 대기하여 자식 스레드가 종료됩니다. 이는 블로킹을 유발할 수 있으므로 `$thread->detach()` 메서드를 사용하여 자식 스레드를 부모 스레드로부터 분리하여 독립적으로 실행할 수 있습니다.


### Example
```php
use Swoole\Thread;

$args = Thread::getArguments();
$c = 4;

if (empty($args)) {
    # 부모 스레드
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
} else {
    # 자식 스레드
    echo "Thread #" . $args[0] . "\n";
    while (1) {
        sleep(1);
        file_get_contents('https://www.baidu.com/');
    }
}
```


## Constants
- `Thread::HARDWARE_CONCURRENCY`는 시스템에서 지원하는 하드웨어 동시성 스레드 수, 즉 `CPU` 코어 수를 가져옵니다.

## Server
- 모든 작업 프로세스는 스레드를 사용하여 실행됩니다. `Worker`, `Task Worker`, `User Process`를 포함합니다.
- `bootstrap` 및 `init_arguments` 두 가지 구성을 추가하여 작업 스레드의 진입 스크립트 파일, 스레드 초기화 매개변수를 설정합니다.

```php
$http = new Swoole\Http\Server("0.0.0.0", 9503);
$http->set([
    'worker_num' => 2,
    'task_worker_num' => 3,
    'bootstrap' => __FILE__,
    'init_arguments' => function () use ($http) {
        $map = new Swoole\Thread\Map;
        return [$map];
    }
]);

$http->on('Request', function ($req, $resp) use ($http) {
    $resp->end('hello world');
});

$http->on('pipeMessage', function ($http, $srcWorkerId, $msg) {
    echo "[worker#" . $http->getWorkerId() . "]\treceived pipe message[$msg] from " . $srcWorkerId . "\n";
});

$http->addProcess(new \Swoole\Process(function () {
   echo "user process, id=" . \Swoole\Thread::getId();
   sleep(2000);
}));

$http->on('Task', function ($server, $taskId, $srcWorkerId, $data) {
    var_dump($taskId, $srcWorkerId, $data);
    return ['result' => uniqid()];
});

$http->on('Finish', function ($server, $taskId, $data) {
    var_dump($taskId, $data);
});

$http->on('WorkerStart', function ($serv, $wid) {
    var_dump(\Swoole\Thread::getArguments(), $wid);
});

$http->on('WorkerStop', function ($serv, $wid) {
    var_dump('stop: T' . \Swoole\Thread::getId());
});

$http->start();
```
