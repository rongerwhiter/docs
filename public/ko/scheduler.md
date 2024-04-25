# Coroutine\Scheduler

모든 [코루틴](/coroutine)은 `코루틴 컨테이너` 안에서 [생성](/coroutine/coroutine?id=create)되어야 합니다. `Swoole` 프로그램을 시작할 때 대부분의 경우 자동으로 `코루틴 컨테이너`가 생성됩니다. `Swoole`을 사용하여 프로그램을 시작하는 방법은 총 세 가지입니다:

- [비동기 스타일](/server/init)의 서버 프로그램의 [start](/server/methods?id=start) 메소드를 호출합니다. 이 방식으로 시작하면 이벤트 콜백 내에서 `코루틴 컨테이너`가 생성됩니다. [enable_coroutine](/server/setting?id=enable_coroutine)을 참고하세요.
- `Swoole`이 제공하는 두 가지 프로세스 관리 모듈인 [Process](/process/process)와 [Process\Pool](/process/process_pool)의 [start](/process/process_pool?id=start) 메소드를 호출합니다. 이 방식으로 시작하면 프로세스가 시작될 때 `코루틴 컨테이너`가 생성됩니다. 이 두 모듈의 생성자의 `enable_coroutine` 매개변수를 참고하세요.
- 직접 코루틴을 작성하여 프로그램을 시작하는 다른 방법은 먼저 코루틴 컨테이너를 생성해야 합니다 (`Coroutine\run()` 함수는 `main` 함수의 역할로 이해할 수 있습니다), 예를 들어:

* **전체 코루틴`HTTP` 서비스 시작**

```php
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/', function ($request, $response) {
        $response->end("<h1>Index</h1>");
    });
    $server->handle('/test', function ($request, $response) {
        $response->end("<h1>Test</h1>");
    });
    $server->handle('/stop', function ($request, $response) use ($server) {
        $response->end("<h1>Stop</h1>");
        $server->shutdown();
    });
    $server->start();
});
echo 1; // 실행되지 않음
```

* **2개의 코루틴을 추가하여 병행 작업 수행**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    Coroutine::create(function() {
        var_dump(file_get_contents("http://www.xinhuanet.com/"));
    });

    Coroutine::create(function() {
        Coroutine::sleep(1);
        echo "done\n";
    });
});
echo 1; // 실행 가능
```

!> `Swoole v4.4+` 버전에서 사용 가능합니다.

!> `Coroutine\run()`을 중첩할 수 없습니다.  
`Coroutine\run()` 내부의 로직에서 `Coroutine\run()` 이후 처리하지 않은 이벤트가 있는 경우 [이벤트 루프](learn?id=what-is-eventloop)를 수행하므로 후속 코드가 실행되지 않습니다. 반대로 이벤트가 더 이상 없으면 아래 코드를 계속 실행하며 다시 `Coroutine\run()`을 호출할 수 있습니다.

앞서 언급한 `Coroutine\run()` 함수는 사실 `Swoole\Coroutine\Scheduler` 클래스(코루틴 스케줄러 클래스)를 래핑한 것인데, 자세한 내용을 알고 싶은 분들은 `Swoole\Coroutine\Scheduler`의 메서드를 참조하시기 바랍니다.
### set()

?> **코루틴 실행 시간 매개변수를 설정합니다.**

?> `Coroutine::set` 메서드의 별칭입니다. [Coroutine::set](/coroutine/coroutine?id=set) 문서를 참조하세요.

```php
Swoole\Coroutine\Scheduler->set(array $options): bool
```

  * **예시**

```php
$sch = new Swoole\Coroutine\Scheduler;
$sch->set(['max_coroutine' => 100]);
```
### getOptions()

?> **설정된 코루틴 실행 시 옵션을 가져옵니다.** Swoole 버전 >= `v4.6.0`에서 사용 가능합니다.

?> `Coroutine::getOptions` 메서드의 별칭입니다. [Coroutine::getOptions](/coroutine/coroutine?id=getoptions) 문서를 참조하십시오.

```php
Swoole\Coroutine\Scheduler->getOptions(): null|array
```
### add()

?> **작업을 추가합니다.** 

```php
Swoole\Coroutine\Scheduler->add(callable $fn, ... $args): bool
```

  * **매개변수** 

    * **`callable $fn`**
      * **기능**：콜백 함수
      * **기본값**：없음
      * **다른 값**：없음

    * **`... $args`**
      * **기능**：옵션 매개변수, 코루틴에 전달됩니다
      * **기본값**：없음
      * **다른 값**：없음

  * **예시**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;
$scheduler->add(function ($a, $b) {
    Coroutine::sleep(1);
    echo assert($a == 'hello') . PHP_EOL;
    echo assert($b == 12345) . PHP_EOL;
    echo "완료.\n";
}, "hello", 12345);

$scheduler->start();
```
  
  * **주의사항**

    !> `go` 함수와 다르게, 여기에 추가된 코루틴은 즉시 실행되지 않고, `start` 메소드가 호출될 때 함께 시작되고 실행됩니다. 프로그램에서 코루틴만 추가하고 `start`를 호출하여 시작하지 않으면, 코루틴 함수인 `$fn`은 실행되지 않습니다.
### parallel()

?> **병렬 작업 추가.** 

?> `add` 메서드와 다르게, `parallel` 메서드는 병렬 코루틴을 생성합니다. `start`할 때 `$num`개의 `$fn` 코루틴을 동시에 시작하여 병렬로 실행합니다.

```php
Swoole\Coroutine\Scheduler->parallel(int $num, callable $fn, ... $args): bool
```

  * **매개변수** 

    * **`int $num`**
      * **기능**：시작하는 코루틴의 개수
      * **기본값**：없음
      * **기타**：없음

    * **`callable $fn`**
      * **기능**：콜백 함수
      * **기본값**：없음
      * **기타**：없음

    * **`... $args`**
      * **기능**：선택적 매개변수, 코루틴에 전달됨
      * **기본값**：없음
      * **기타**：없음

  * **예시**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;

$scheduler->parallel(10, function ($t, $n) {
    Coroutine::sleep($t);
    echo "Co ".Coroutine::getCid()."\n";
}, 0.05, 'A');

$scheduler->start();
```
### start()

?> **프로그램을 시작합니다.**

?> `add` 및 `parallel` 메소드로 추가된 코루틴 작업을 반복하고 실행합니다.

```php
Swoole\Coroutine\Scheduler->start(): bool
```

  * **반환값**

    * 성공적으로 시작되면 모든 추가된 작업을 실행하며, 모든 코루틴이 종료되면 `start`는 `true`를 반환합니다.
    * 실패하면 `false`가 반환되며, 이미 시작됐거나 다른 스케줄러가 생성되어 다시 생성할 수 없는 이유일 수 있습니다.
