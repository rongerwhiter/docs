# 동시 대기열

동시에 접근 가능한 `Queue` 구조를 생성하여 스레드에 전달할 수 있는 구조체입니다. 다른 스레드에서 읽거나 쓸 때 가시적입니다.
자세한 기능은 [동시 Map](thread/map.md)를 참고하세요.


## 사용법
`Thread\Queue`는 선입선출 데이터 구조로, 두 가지 핵심 작업이 있습니다:
- `Queue::push($value)` 데이터를 대기열에 쓰기
- `Queue::pop()` 대기열 헤드부터 데이터를 가져오기

## 유의사항
- `ArrayList`는 요소를 추가할 수만 있고 무작위로 삭제하거나 할당할 수는 없습니다.

## 예시 코드

```php
use Swoole\Thread;
use Swoole\Thread\Queue;

$args = Thread::getArguments();
$c = 4;
$n = 128;

if (empty($args)) {
    $threads = [];
    $queue = new Queue;
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i, $queue);
    }
    while ($n--) {
        $queue->push(base64_encode(random_bytes(16)), Queue::NOTIFY_ONE);
        usleep(random_int(10000, 100000));
    }
    $n = 4;
    while ($n--) {
        $queue->push('', Queue::NOTIFY_ONE);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
    var_dump($queue->count());
} else {
    $queue = $args[1];
    while (1) {
        $job = $queue->pop(-1);
        if (!$job) {
            break;
        }
        var_dump($job);
    }
}
```

## 메소드 목록

### Queue::push()

대기열에 데이터를 씁니다.

```php
function Queue::push(mixed $value, int $notify = 0);
```

- `$value` : 쓸 데이터 내용
- `$notify` : 데이터를 읽는 대기 중인 스레드를 알리는지 여부, `Queue::NOTIFY_ONE`은 하나의 스레드를 깨우고, `Queue::NOTIFY_ALL`은 모든 스레드를 깨웁니다.


### Queue::pop()

대기열 헤드부터 데이터를 가져옵니다.

```php
function Queue::pop(double $timeout = 0);
```

- `$timeout=0` : 기본값으로, 기다리지 않고 대기열이 비었을 때 즉시 `NULL`을 반환합니다.
- `$timeout!=0` : 대기열이 비었을 때 생산자 `push()` 데이터를 기다립니다. `$timeout`이 음수인 경우는 절대 타임아웃이 없음을 의미합니다.

### Queue::count()
대기열 요소 수를 가져옵니다.

### Queue::clean()
모든 요소를 지웁니다.
