# Coroutine\Channel

> [개요](/coroutine)를 먼저 확인한 후 기본적인 코루틴 개념을 이해한 후 이 섹션을 읽는 것이 좋습니다.

채널은 코루틴 간 통신에 사용되며 다중 생산자 코루틴 및 다중 소비자 코루틴을 지원합니다. 하위 레벨에서 코루틴의 전환 및 스케줄링을 자동으로 처리합니다.
## 구현 원리

  * 채널은 `PHP`의 `Array`와 유사하며, 메모리만 사용하고 다른 추가 리소스를 요구하지 않습니다. 모든 작업은 메모리 작업으로, `IO` 소비 없이 이루어집니다.
  * 하위 수준에서는 `PHP` 참조 계산을 사용하여, 메모리 복사가 없습니다. 거대한 문자열이나 배열을 전달하더라도 추가 성능 소비가 발생하지 않습니다.
  * `channel`은 참조 계산을 기반으로 하며, 영구 복사가 없습니다. 
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

run(function(){
    $channel = new Channel(1);
    Coroutine::create(function () use ($channel) {
        for($i = 0; $i < 10; $i++) {
            Coroutine::sleep(1.0);
            $channel->push(['rand' => rand(1000, 9999), 'index' => $i]);
            echo "{$i}\n";
        }
    });
    Coroutine::create(function () use ($channel) {
        while(1) {
            $data = $channel->pop(2.0);
            if ($data) {
                var_dump($data);
            } else {
                assert($channel->errCode === SWOOLE_CHANNEL_TIMEOUT);
                break;
            }
        }
    });
});
```  
해당 문서는 현재 수리 중 입니다. 곧 돌아올게요!
### __construct()

채널 생성자 메서드.

```php
Swoole\Coroutine\Channel::__construct(int $capacity = 1)
```

  * **매개변수** 

    * **`int $capacity`**
      * **기능**: 용량 설정 【`1` 이상의 정수여야 함】
      * **기본값**: `1`
      * **다른 값들**: 없음

!> PHP 참조 카운팅을 사용하여 변수를 저장하기 때문에, 캐시 영역은 `$capacity * sizeof(zval)` 바이트만큼의 메모리를 차지합니다. `PHP7` 버전에서 `zval`의 크기는 `16`바이트이며, 예를 들어, `$capacity = 1024`인 경우, `Channel`은 최대`16K`의 메모리를 차지할 것입니다.

!> `Server`에서 사용하는 경우 [onWorkerStart](/server/events?id=onworkerstart) 이후에 생성해야 합니다.
### push()

데이터를 채널에 기록합니다.

```php
Swoole\Coroutine\Channel->push(mixed $data, float $timeout = -1): bool
```

  * **매개변수** 

    * **`mixed $data`**
      * **기능**：데이터를 push합니다 【익명 함수와 리소스를 포함한 모든 유형의 PHP 변수가 될 수 있습니다】
      * **기본값**：없음
      * **다른 값**：없음

      !> 모호성을 피하기 위해 `0`, `false`, `빈 문자열`, `null`과 같은 빈 데이터를 채널에 기록하지 마십시오.

    * **`float $timeout`**
      * **기능**：타임아웃 시간을 설정합니다
      * **단위**：초【부동 소수점 지원, 예: `1.5`는 `1초` + `500ms`를 의미합니다】
      * **기본값**：`-1`
      * **다른 값**：없음
      * **버전 영향**：Swoole 버전 >= v4.2.12

      !> 채널이 가득 찬 경우, `push`는 현재 코루틴을 일시 중단시키고, 지정된 시간 동안 소비자가 데이터를 소비하지 않으면 시간 초과가 발생합니다. 이 경우, 밑부분에서 현재 코루틴을 복원하고, `push` 호출이 즉시 `false`를 반환하여 실패합니다.

  * **반환 값**

    * 실행이 성공하면`true`를 반환합니다
    * 채널이 닫힌 경우 실패 시 `false`를 반환하며, `$channel->errCode`를 사용하여 오류 코드를 얻을 수 있습니다

  * **확장**

    * **채널이 가득 찬 경우**

      * 자동으로 현재 코루틴을 `yield`하고, 다른 소비자 코루틴이 데이터를 `pop` 소비하면 채널이 다시 기록 가능하고, 현재 코루틴을 다시 `resume`합니다
      * 여러 프로듀서 코루틴이 동시에 `push`하는 경우, 내부에서 자동으로 대기열을 만들며, 순서대로 이러한 프로듀서 코루틴을 하나씩 `resume`합니다

    * **채널이 비어 있는 경우**

      * 소비자 코루틴 하나를 자동으로 깨웁니다
      * 여러 소비자 코루틴이 동시에 `pop`하는 경우, 내부에서 자동으로 대기열을 만들어, 이러한 소비자 코루틴을 하나씩 순서대로 `resume`합니다

!> `Coroutine\Channel`은 로컬 메모리를 사용하며, 다른 프로세스 간에는 메모리가 격리됩니다. `push` 및 `pop` 작업은 동일한 프로세스 내의 서로 다른 코루틴에서만 수행할 수 있습니다.
### pop()

데이터를 채널에서 읽습니다.

```php
Swoole\Coroutine\Channel->pop(float $timeout = -1): mixed
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：타임아웃 시간 설정
      * **값의 단위**：초【부동 소수점 사용 가능, 예: `1.5`는 `1초` + `500밀리초`를 의미함】
      * **기본값**：`-1`【무한대로 타임아웃하지 않음을 의미함】
      * **다른 값**：없음
      * **버전 영향**：Swoole 버전 >= v4.0.3

  * **반환값**

    * 반환값은 익명 함수 및 리소스를 포함한 모든 종류의 PHP 변수일 수 있습니다.
    * 채널이 닫힌 경우, 작업은 실패하여 `false`를 반환합니다.

  * **확장**

    * **채널이 꽉 차있을 때**

      * 데이터를 소비한 후 `pop`은 자동으로 하나의 생산자 코루틴을 깨우고, 새 데이터를 쓸 수 있도록 합니다.
      * 여러 생산자 코루틴이 동시에 `push`할 때, 하위 시스템은 자동으로 대기열을 처리하며, 순서대로 이러한 생산자 코루틴을 `resume`합니다.

    * **채널이 비어 있을 때**

      * 현재 코루틴을 자동으로 `yield`하고, 다른 생산자 코루틴이 데이터를 `push`하면 채널이 읽을 수 있으므로 현재 코루틴을 다시 `resume`합니다.
      * 여러 소비자 코루틴이 동시에 `pop`할 때, 하위 시스템은 자동으로 대기열을 처리하며, 순서대로 이러한 소비자 코루틴을 `resume`합니다.
### stats()

채널의 상태를 가져옵니다.

```php
Swoole\Coroutine\Channel->stats(): array
```

  * **반환값**

    배열을 반환하며, 버퍼 채널은 `4`개의 정보를 포함하고, 버퍼가 없는 채널은 `2`개의 정보를 반환합니다.
    
    - `consumer_num` 소비자 수, 현재 채널이 비어 있음을 나타내며, 다른 코루틴이 `push` 메소드를 호출하여 데이터를 생성하기를 기다리는 `N`개의 코루틴이 있음
    - `producer_num` 생산자 수, 현재 채널이 가득 차 있음을 나타내며, 다른 코루틴이 `pop` 메소드를 호출하여 데이터를 소비하기를 기다리는 `N`개의 코루틴이 있음
    - `queue_num` 채널 내 요소 수

```php
array(
  "consumer_num" => 0,
  "producer_num" => 1,
  "queue_num" => 10
);
```
### close()

채널을 닫습니다. 대기 중인 모든 읽기 및 쓰기 코루틴을 깨웁니다.

```php
Swoole\Coroutine\Channel->close(): bool
```

!> 모든 프로듀서 코루틴을 깨우고, `push` 메소드는 `false`를 반환합니다. 모든 컨슈머 코루틴을 깨우고, `pop` 메소드는 `false`를 반환합니다.
```php
Swoole\Coroutine\Channel->length(): int
```

채널 내의 요소 수를 가져옵니다.
```
### isEmpty()

현재 채널이 비어 있는지 확인합니다.

```php
Swoole\Coroutine\Channel->isEmpty(): bool
```
### isFull()

현재 채널이 꽉 찼는지 여부를 판별합니다.

```php
Swoole\Coroutine\Channel->isFull(): bool
```
## 속성
### 용량

채널 버퍼 용량입니다.

[생성자](/coroutine/channel?id=__construct)에서 설정한 용량이 여기에 저장되지만 **설정한 용량이 1보다 작을 경우** 이 변수는 1이 됩니다.

```php
Swoole\Coroutine\Channel->capacity: int
```
### errCode

에러 코드를 가져옵니다.

```php
Swoole\Coroutine\Channel->errCode: int
```

  * **반환값**

값 | 상수 | 설명
---|---|---
0 | SWOOLE_CHANNEL_OK | 기본 성공
-1 | SWOOLE_CHANNEL_TIMEOUT | 타임아웃 발생 시 pop 실패(타임아웃)
-2 | SWOOLE_CHANNEL_CLOSED | 채널이 이미 닫힘, 채널 조작 계속하기
