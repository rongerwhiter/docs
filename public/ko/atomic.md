`Atomic`는 `Swoole`의 하위 수준에서 제공되는 원자 계수 연산 클래스로, 정수의 락이 걸리지 않는 원자 증감을 쉽게 수행할 수 있습니다.

* 공유 메모리를 사용하여 다른 프로세스 간에 계수 조작이 가능합니다.
* `gcc/clang`에서 제공하는 `CPU` 원자 명령에 기반하여 락을 걸 필요가 없습니다.
* 서버 프로그램에서는 `Server->start` 이전에 생성해야 `Worker` 프로세스에서 사용할 수 있습니다.
* 기본적으로 `32`비트 부호 없는 타입을 사용하며, `64`비트 부호 있는 정수가 필요한 경우 `Swoole\Atomic\Long`을 사용할 수 있습니다.

!> [onReceive](/server/events?id=onreceive) 등의 콜백 함수에서 계수기를 생성하지 마십시오. 그렇게 하면 메모리가 계속 증가하여 메모리 누수가 발생할 수 있습니다.

!> `64`비트 부호 있는 long 정수 원자 계수를 지원하며, `new Swoole\Atomic\Long`을 사용하여 생성해야 합니다. `Atomic\Long`은 `wait`와 `wakeup` 메소드를 지원하지 않습니다.
```php
$atomic = new Swoole\Atomic();

$serv = new Swoole\Server('127.0.0.1', '9501');
$serv->set([
    'worker_num' => 1,
    'log_file' => '/dev/null'
]);
$serv->on("start", function ($serv) use ($atomic) {
    if ($atomic->add() == 2) {
        $serv->shutdown();
    }
});
$serv->on("ManagerStart", function ($serv) use ($atomic) {
    if ($atomic->add() == 2) {
        $serv->shutdown();
    }
});
$serv->on("ManagerStop", function ($serv) {
    echo "shutdown\n";
});
$serv->on("Receive", function () {
    
});
$serv->start();
```
해당 카테고리는 지원되지 않습니다.
### __construct()

생성자. 원자적인 카운터 객체를 생성합니다.

```php
Swoole\Atomic::__construct(int $init_value = 0);
```

  * **매개변수** 

    * **`int $init_value`**
      * **기능**：초기화할 값 지정
      * **기본값**：`0`
      * **다른 값**：없음

!> -`Atomic`는 `32`비트 부호 없는 정수만 처리 가능하며, 최대 `42`억까지 지원하며 음수는 지원되지 않습니다;  
-`Server`에서 원자적 카운터를 사용하려면 `Server->start` 이전에 만들어야 합니다;  
-프로세스에서 원자적 카운터를 사용하려면 `Process->start` 이전에 만들어야 합니다.
### add()

카운트를 증가시킵니다.

```php
Swoole\Atomic->add(int $add_value = 1): int
```

  * **매개변수** 

    * **`int $add_value`**
      * **기능**：증가시킬 값【양의 정수여야 함】
      * **기본값**：`1`
      * **다른 값**：없음

  * **반환 값**

    * `add` 메서드가 성공적으로 실행된 후 결과 값 반환

!> 원래 값에 추가되면서 `42`억을 초과하면 오버플로우가 발생하며, 상위 비트 값이 삭제됩니다.
### sub()

카운트를 감소시킵니다.

```php
Swoole\Atomic->sub(int $sub_value = 1): int
```

  * **매개변수** 

    * **`int $sub_value`**
      * **기능**：감소시킬 값【양의 정수여야 함】
      * **기본값**：`1`
      * **다른 값**：없음

  * **반환 값**

    * `sub` 메소드 실행 성공 후 결과 값 반환

!> 원래 값에서 빼기 작업을 수행하면 0 이하가 될 경우 오버플로우가 발생하며, 상위 비트 값이 삭제됩니다.
### get()

현재 카운터의 값을 가져옵니다.

```php
Swoole\Atomic->get(): int
```

  * **반환 값**

    * 현재 숫자값을 반환합니다.
### set()

현재 값을 지정된 숫자로 설정합니다.

```php
Swoole\Atomic->set(int $value): void
```

  * **매개변수** 

    * **`int $value`**
      * **기능**：설정할 대상 숫자를 지정합니다.
      * **기본값**：없음
      * **기타 값**：없음
### cmpset()

만약 현재 값이 매개변수 `1`과 같다면 현재 값을 매개변수 `2`로 설정합니다.

```php
Swoole\Atomic->cmpset(int $cmp_value, int $set_value): bool
```

  * **매개변수**

    * **`int $cmp_value`**
      * **기능**: 현재 값이 `$cmp_value`와 같으면 `true`를 반환하고 현재 값을 `$set_value`로 설정합니다. 값이 다르면 `false`를 반환합니다【42억 미만의 정수여야 함】
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $set_value`**
      * **기능**: 현재 값이 `$cmp_value`와 같으면 `true`를 반환하고 현재 값을 `$set_value`로 설정합니다. 값이 다르면 `false`를 반환합니다【42억 미만의 정수여야 함】
      * **기본값**: 없음
      * **기타 값**: 없음
### wait()

대기 상태로 설정됩니다.

!> 원자 카운터 값이 0일 때 프로그램은 대기 상태로 들어갑니다. 다른 프로세스가 `wakeup`을 호출하여 프로그램을 다시 깨울 수 있습니다. 이 기능은 `Linux Futex`를 기반으로 하며, 이 기능을 사용하면 대기, 알림, 잠금 기능을 4바이트의 메모리로 구현할 수 있습니다. `Futex`를 지원하지 않는 플랫폼에서는 밑바닥부터 `usleep(1000)` 루프를 통해 모의 구현됩니다.

```php
Swoole\Atomic->wait(float $timeout = 1.0): bool
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：지정된 시간이 초과되면 대기를 끝냅니다【`-1`로 설정하면 영원히 기다립니다. 다른 프로세스가 깨울 때까지 계속 대기합니다】
      * **시간 단위**：초【부동 소수점을 지원합니다. 예: `1.5`는 `1초` + `500ms`를 나타냅니다】
      * **기본 값**：`1`
      * **기타 값**：없음

  * **반환 값** 

    * 시간 초과 시 `false`가 반환되고, 오류 코드는 `EAGAIN`이며 `swoole_errno` 함수로 확인할 수 있습니다.
    * 성공 시 `true`가 반환되고, 다른 프로세스가 `wakeup`을 통해 현재 잠금을 성공적으로 깨운 것을 나타냅니다.

  * **코루틴 환경**

  `wait`는 코루틴이 아닌 전체 프로세스를 차단하므로, 프로세스가 멈추지 않도록 하기 위해 코루틴 환경에서 `Atomic->wait()`를 사용하는 것을 피해야 합니다.

!> - `wait/wakeup` 기능을 사용할 때 원자 카운터의 값은 `0` 또는 `1` 여야 하며, 그렇지 않으면 정상적으로 사용할 수 없습니다.  
- 물론 원자 카운터 값이 `1` 인 경우는 대기 상태로 들어갈 필요가 없으므로 리소스가 현재 사용 가능한 상태입니다. `wait` 함수는 즉시 `true`를 반환합니다.

  * **사용 예시**

    ```php
    $n = new Swoole\Atomic;
    if (pcntl_fork() > 0) {
        echo "master start\n";
        $n->wait(1.5);
        echo "master end\n";
    } else {
        echo "child start\n";
        sleep(1);
        $n->wakeup();
        echo "child end\n";
    }
    ```
### wakeup()

唤醒处于wait状态的其他进程。

```php
Swoole\Atomic->wakeup(int $n = 1): bool
```

  * **参数** 

    * **`int $n`**
      * **功能**：唤醒的进程数量
      * **默认值**：无
      * **其它值**：无

* 현재 원자 카운트가 `0`이면, 대기 중인 프로세스가 없음을 의미하며, `wakeup`은 즉시 `true`를 반환합니다.
* 현재 원자 카운트가 `1`이면, 현재 대기 중인 프로세스가 있음을 의미하며, `wakeup`은 대기 중인 프로세스를 깨우고 `true`를 반환합니다.
* 깨어난 프로세스는 반환된 후 원자 카운트가 `0`으로 설정되며, 이제 `wait` 중인 다른 프로세스를 다시 깨울 수 있습니다.
