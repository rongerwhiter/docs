# Swoole\Server\Task

`Swoole\Server\Task`에 대해 자세히 설명합니다. 이 클래스는 매우 간단하지만, `new Swoole\Server\Task()`를 통해 `Task` 객체를 얻을 수 없으며, 이 객체는 서버 정보를 전혀 포함하지 않습니다. 또한 `Swoole\Server\Task`의 임의의 메서드를 실행하면 치명적인 오류가 발생합니다.

```shell
Invalid instance of Swoole\Server\Task in /home/task.php on line 3
```


## 속성

### $data
`worker` 프로세스에서 `task` 프로세스로 전송된 데이터 `data`로, 이 속성은 `string` 유형의 문자열입니다.

```php
Swoole\Server\Task->data
```

### $dispatch_time
해당 데이터가 `task` 프로세스에 도착한 시간인 `dispatch_time`을 반환하며, 이 속성은 `double` 유형입니다.

```php
Swoole\Server\Task->dispatch_time
```

### $id
해당 데이터가 `task` 프로세스에 도착한 시간인 `dispatch_time`을 반환하며, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Task->id
```

### $worker_id
해당 데이터가 어느 `worker` 프로세스에서 왔는지를 반환하며, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Task->worker_id
```

### $flags
이 비동기 작업의 일부 플래그 정보인 `flags`로, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Task->flags
```

?> `flags`의 반환 결과는 다음과 같은 유형이 있습니다:  
  - SWOOLE_TASK_NOREPLY | SWOOLE_TASK_NONBLOCK은 이것이 `Worker` 프로세스에서 `task` 프로세스로 전송되는 것이 아님을 나타냅니다. 이때 `onTask` 이벤트에서 `Swoole\Server::finish()`를 호출하면 경고가 발생합니다.  
  - SWOOLE_TASK_CALLBACK | SWOOLE_TASK_NONBLOCK은 `Swoole\Server::finish()`에서 마지막 콜백 함수가 null이 아니라는 것을 나타내며, `onFinish` 이벤트가 실행되지 않고 이 콜백 함수만 실행됩니다. 
  - SWOOLE_TASK_COROUTINE | SWOOLE_TASK_NONBLOCK는 작업을 코루틴 형태로 처리할 것을 나타냅니다. 
  - SW_TASK_NONBLOCK는 기본값으로, 위의 세 가지 상황 모두에 해당되지 않을 때입니다.

## 메서드

### finish()

[Task 프로세스](/learn?id=taskworker프로세스)에서`Worker` 프로세스에게 작업이 완료되었음을 알리는 데 사용됩니다. 이 함수는 결과 데이터를 `Worker` 프로세스에 전달할 수 있습니다.

```php
Swoole\Server\Task->finish(mixed $data): bool
```

  * **매개변수**

    * `mixed $data`

      * 기능: 작업 처리 결과 내용
      * 기본값: 없음
      * 다른 값: 없음

  * **팁**
    * `finish` 메서드는 여러 번 호출할 수 있으며, `Worker` 프로세스는 여러 번 [onFinish](/server/events?id=onfinish) 이벤트를 트리거합니다.
    * `onTask` 콜백 함수에서 `finish` 메서드를 호출한 후에도 `return` 데이터는 여전히 [onFinish](/server/events?id=onfinish) 이벤트를 트리거합니다.
    * `Swoole\Server\Task->finish`는 선택 사항입니다. `Worker` 프로세스가 작업 실행 결과에 관심이 없다면 이 함수를 호출할 필요가 없습니다.
    * `onTask` 콜백 함수에서 문자열을 `return`하는 것은 `finish`를 호출하는 것과 동일합니다.

  * **주의사항**

  !> `Swoole\Server\Task->finish` 함수를 사용하려면 `Server`에 [onFinish](/server/events?id=onfinish) 콜백 함수를 설정해야 합니다. 이 함수는 [Task 프로세스](/learn?id=taskworker프로세스)의 [onTask](/server/events?id=ontask) 콜백에서만 사용할 수 있습니다.


### pack()

주어진 데이터를 직렬화합니다.

```php
Swoole\Server\Task->pack(mixed $data): string|false
```

  * **매개변수**

    * `mixed $data`

      * 기능: 작업 처리 결과 내용
      * 기본값: 없음
      * 다른 값: 없음

  * **반환값**
    * 성공적으로 호출되면 직렬화된 결과가 반환됩니다. 

### unpack()

주어진 데이터를 역직렬화합니다.

```php
Swoole\Server\Task->unpack(string $data): mixed
```

  * **매개변수**

    * `string $data`

      * 기능: 역직렬화해야 하는 데이터
      * 기본값: 없음
      * 다른 값: 없음

  * **반환값**
    * 성공적으로 호출되면 역직렬화된 결과가 반환됩니다. 

## 예시
```php
<?php
$server->on('task', function(Swoole\Server $serv, Swoole\Server\Task $task) {
    $task->finish(['result' => true]);
});
```
