# 定时기 Timer

밀리초 정밀도의 타이머입니다. `epoll_wait` 및 `setitimer`를 기반으로 하며 데이터 구조는 `최소 힙`을 사용하여 대규모 타이머를 지원할 수 있습니다.

* `Manager` 및 `TaskWorker` 프로세스와 같은 동기 I/O 프로세스에서는 `setitimer`와 신호를 사용합니다.
* 비동기 I/O 프로세스에서는 `epoll_wait`/`kevent`/`poll`/`select` 타임아웃을 사용합니다.
## 성능

최소 힙 자료 구조를 사용하여 타이머를 구현하므로, 타이머의 추가 및 삭제는 모두 메모리 조작이므로 성능이 매우 우수합니다.

> 공식 벤치마크 스크립트 [timer.php](https://github.com/swoole/benchmark/blob/master/timer.php)에서 무작위 시간의 타이머 10만 개를 추가하거나 삭제하는 데 약 `0.08초`가 소요됩니다.

```shell
~/workspace/swoole/benchmark$ php timer.php
add 100000 timer :0.091133117675781s
del 100000 timer :0.084658145904541s
```

!> 타이머는 메모리 조작이므로 `IO` 비용이 없습니다.
## 차이

`Timer`와 `PHP` 자체의 `pcntl_alarm`은 다릅니다. `pcntl_alarm`은 `시계 신호 + tick` 함수에 기반을 두어 일부 결함이 있습니다:

  * 최대로 지원하는 단위가 초까지이며, `Timer`는 밀리초까지 사용할 수 있습니다
  * 동시에 여러 타이머 프로그램을 설정할 수 없음
  * `pcntl_alarm`은 `declare(ticks = 1)`에 의존하며, 성능이 매우 떨어집니다
## 제로 밀리초 타이머

하위 레벨에서는 `0`으로 설정된 시간 매개변수를 지원하지 않습니다. 이는 `Node.js` 등과 다릅니다. `Swoole`에서는 [Swoole\Event::defer](/event?id=defer)를 사용하여 유사한 기능을 구현할 수 있습니다.

```php
Swoole\Event::defer(function () {
  echo "hello\n";
});
```

!> 위 코드는 `JS`의 `setTimeout(func, 0)`과 완전히 동일한 효과를 제공합니다.
## 별칭

`tick()`、`after()`、`clear()` 모두 함수 스타일의 별칭을 갖습니다.

클래스 정적 메서드 | 함수 스타일 별칭
---|---
`Swoole\Timer::tick()` | `swoole_timer_tick()`
`Swoole\Timer::after()` | `swoole_timer_after()`
`Swoole\Timer::clear()` | `swoole_timer_clear()`
해당 메소드에 대한 설명을 입력하세요.
### tick()

정기적으로 발생하는 인터벌 타이머를 설정합니다.

`after` 타이머와는 달리 `tick` 타이머는 [Timer::clear](/timer?id=clear)를 호출할 때까지 계속해서 발생합니다.

```php
Swoole\Timer::tick(int $msec, callable $callback_function, ...$params): int
```

!> 1. 타이머는 현재 프로세스 공간에서만 유효합니다  
   2. 타이머는 순수한 비동기 방식으로 작동하며, [동기 I/O](/learn?id=동기io비동기io) 함수와 함께 사용할 수 없습니다. 그렇게 하면 타이머가 잘못된 시간에 실행될 수 있습니다  
   3. 타이머 실행 중에는 일정한 오차가 발생할 수 있습니다

  * **매개변수** 

    * **`int $msec`**
      * **기능**：지정된 시간
      * **단위**：밀리초【예: `1000`은 `1`초를 의미하며 `v4.2.10` 미만 버전에서 최대 값은 `86400000`】
      * **기본값**：없음
      * **기타**：없음

    * **`callable $callback_function`**
      * **기능**：시간 초과 후 실행될 함수, 호출 가능해야 합니다
      * **기본값**：없음
      * **기타**：없음

    * **`...$params`**
      * **기능**：실행 함수에 전달되는 데이터【이 매개변수는 선택적입니다】
      * **기본값**：없음
      * **기타**：없음
      
      !> `use` 문법을 사용하여 익명 함수에서 콜백 함수로 매개변수를 전달할 수 있습니다

  * **$callback_function 콜백 함수** 

    ```php
    callbackFunction(int $timer_id, ...$params);
    ```

      * **`int $timer_id`**
        * **기능**：타이머의`ID`【[Timer::clear](/timer?id=clear)로 이 타이머를 제거할 수 있음】
        * **기본값**：없음
        * **기타**：없음

      * **`...$params`**
        * **기능**：`Timer::tick`로 전달된 세 번째 매개변수`$param`
        * **기본값**：없음
        * **기타**：없음

  * **확장**

    * **타이머 보정**

      타이머 콜백 함수의 실행 시간이 다음 타이머 실행 시간에 영향을 주지 않습니다. 예 : `0.002초`에 `10ms`의 `tick` 타이머를 설정하면 첫 번째는 `0.012초`에 콜백 함수를 실행하고, 콜백 함수가 `5ms`를 실행하면 다음 타이머는 여전히 `0.022초`에 트리거됩니다. 즉 `0.027초`가 아닙니다.
      
      그러나 만약 타이머 콜백 함수가 실행 시간이 너무 길어서 다음 타이머 실행 시간을 덮어썼다면, 로우 레벨에서 시간 보정이 이루어지고 만료된 동작은 폐기됩니다. 다음 시간 콜백입니다. 위의 예에서 `0.012초`에 실행된 콜백 함수가 `15ms`를 실행하면 정상적으로 `0.022초`에 한 번 타이머 콜백을 호출해야합니다. 실제로 이 타이머는 `0.027초`에 실패하므로 시간은 이미 만료되었습니다. 로우 레벨에서는 `0.032초`에 다시 타이머 콜백이 트리거됩니다.
    
    * **코루틴 모드**

      코루틴 환경에서 `Timer::tick` 콜백은 자동으로 코루틴을 생성하며 코루틴 관련 `API`를 직접 사용할 수 있습니다. 코루틴을 생성하기 위해 `go`를 호출할 필요가 없습니다.
      
      !> [enable_coroutine](/timer?id=close-timer-co)를 설정하여 자동 코루틴 생성을 끌 수 있습니다

  * **사용 예제**

    ```php
    Swoole\Timer::tick(1000, function(){
        echo "timeout\n";
    });
    ```

    * **올바른 예제**

    ```php
    Swoole\Timer::tick(3000, function (int $timer_id, $param1, $param2) {
        echo "timer_id #$timer_id, after 3000ms.\n";
        echo "param1 is $param1, param2 is $param2.\n";

        Swoole\Timer::tick(14000, function ($timer_id) {
            echo "timer_id #$timer_id, after 14000ms.\n";
        });
    }, "A", "B");
    ```

    * **잘못된 예제**

    ```php
    Swoole\Timer::tick(3000, function () {
        echo "after 3000ms.\n";
        sleep(14);
        echo "after 14000ms.\n";
    });
    ```
### after()

특정 시간 후에 함수를 실행합니다. `Swoole\Timer::after` 함수는 일회성 타이머로, 실행 완료 후에는 파기됩니다.

이 함수는 `PHP` 표준 라이브러리의 `sleep` 함수와 다릅니다. `after`는 논블로킹(non-blocking)이며, `sleep`은 호출되면 현재 프로세스가 차단되어 새 요청을 처리할 수 없게 됩니다.

```php
Swoole\Timer::after(int $msec, callable $callback_function, ...$params): int
```

  * **매개변수** 

    * **`int $msec`**
      * **기능**：지정된 시간
      * **값의 단위**：밀리초【`1000`은 `1`초를 의미하며, `v4.2.10` 미만 버전에서 최대값은 `86400000`을 초과할 수 없음】
      * **기본값**：없음
      * **다른 값**：없음

    * **`callable $callback_function`**
      * **기능**：시간이 지난 후에 실행될 함수로, 호출 가능해야 합니다.
      * **기본값**：없음
      * **다른 값**：없음
    
    * **`...$params`**
      * **기능**：실행 함수로 데이터를 전달합니다【이 매개변수는 선택 사항입니다】
      * **기본값**：없음
      * **다른 값**：없음

      !> 콜백 함수로 매개변수를 전달하기 위해 익명 함수의 "use" 구문을 사용할 수 있습니다.

  * **반환 값**

    * 성공 시 타이머`ID`가 반환되며, 타이머를 취소하려면 [Swoole\Timer::clear](/timer?id=clear)를 호출할 수 있습니다.

  * **확장**

    * **코루틴 모드**

      코루틴 환경에서 [Swoole\Timer::after](/timer?id=after) 콜백 내에서 자동으로 코루틴이 생성되어 코루틴 관련 `API`를 직접 사용할 수 있으므로 `go`를 통해 코루틴을 만들 필요가 없습니다.

      !> [enable_coroutine](/timer?id=close-timer-co)를 설정하여 자동 코루틴 생성을 중지할 수 있습니다.

  * **사용 예시**

```php
$str = "Swoole";
Swoole\Timer::after(1000, function() use ($str) {
    echo "Hello, $str\n";
});
```
### clear()

타이머 `ID`를 사용하여 타이머를 제거합니다.

```php
Swoole\Timer::clear(int $timer_id): bool
```

  * **매개변수** 

    * **`int $timer_id`**
      * **기능**: 타이머 `ID`【[Timer::tick](/timer?id=tick) 또는 [Timer::after](/timer?id=after)를 호출한 후에 반환된 정수 ID】
      * **기본값**: 없음
      * **다른 값들**: 없음

!> `Swoole\Timer::clear`는 다른 프로세스의 타이머를 지우는 데 사용할 수 없으며, 현재 프로세스에서만 작동합니다.

  * **사용 예시**

```php
$timer = Swoole\Timer::after(1000, function () {
    echo "timeout\n";
});

var_dump(Swoole\Timer::clear($timer));
var_dump($timer);

// 출력: bool(true) int(1)
// 출력 안 함: timeout
```
### clearAll()

현재 Worker 프로세스 내의 모든 타이머를 지웁니다.

!> Swoole 버전 >= `v4.4.0`에서 사용 가능

```php
Swoole\Timer::clearAll(): bool
```
```php
Swoole\Timer::info(int $timer_id): array
```

  * **반환 값**

```php
array(5) {
  ["exec_msec"]=>
  int(6000)
  ["exec_count"]=>
  int(5)
  ["interval"]=>
  int(1000)
  ["round"]=>
  int(0)
  ["removed"]=>
  bool(false)
}
```
### list()

반환 타이머 이터레이터, 현재 Worker 프로세스 내의 모든 `timer`의 id를 `foreach`를 통해 순회할 수 있습니다.

！> Swoole 버전 >=  `v4.4.0`에서 사용 가능

```php
Swoole\Timer::list(): Swoole\Timer\Iterator
```

  * **사용 예시**

```php
foreach (Swoole\Timer::list() as $timer_id) {
    var_dump(Swoole\Timer::info($timer_id));
}
```
### stats()

타이머 상태를 확인합니다.

!> Swoole 버전 >= `v4.4.0`에서 사용 가능

```php
Swoole\Timer::stats(): array
```

  * **반환 값**

```php
array(3) {
  ["initialized"]=>
  bool(true)
  ["num"]=>
  int(1000)
  ["round"]=>
  int(1)
}
```
### set()

타이머 관련 매개변수를 설정합니다.

```php
Swoole\Timer::set(array $array): void
```

!> 이 메서드는 `v4.6.0` 버전부터 폐기되었습니다.
## 코루틴 종료 :id=close-timer-co

기본적으로 타이머는 콜백 함수를 실행할 때 자동으로 코루틴을 생성하며, 타이머의 코루틴을 닫을 수 있습니다.

```php
swoole_async_set([
  'enable_coroutine' => false,
]);
```
