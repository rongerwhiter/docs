# Coroutine\Barrier

[Swoole Library](https://github.com/swoole/library)에는 더 편리한 코루틴 동시성 관리 도구 인 `Coroutine\Barrier` 코루틴 장벽 또는 코루틴 바리어가 제공됩니다. 이는 `PHP` 레퍼런스 카운팅과 `Coroutine API`를 기반으로 구현되었습니다.

[Coroutine\WaitGroup](/coroutine/wait_group)에 비해, `Coroutine\Barrier`는 더 간단하게 사용할 수 있으며, 매개변수 전달이나 `use` 구문을 통해 서브 코루틴 함수를 도입할 수 있습니다.

!> Swoole 버전 >= v4.5.5에서 사용 가능합니다.

## 사용 예

```php
use Swoole\Coroutine\Barrier;
use Swoole\Coroutine\System;
use function Swoole\Coroutine\run;
use Swoole\Coroutine;

run(function () {
    $barrier = Barrier::make();

    $count = 0;
    $N = 4;

    foreach (range(1, $N) as $i) {
        Coroutine::create(function () use ($barrier, &$count) {
            System::sleep(0.5);
            $count++;
        });
    }

    Barrier::wait($barrier);
    
    assert($count == $N);
});
```

## 실행 흐름

* 먼저 `Barrier::make()`로 새로운 코루틴 바리어를 생성합니다.
* 서브 코루틴에서 `use` 구문을 사용하여 바리어를 전달하고 레퍼런스 카운팅을 증가시킵니다.
* 대기해야 하는 위치에 `Barrier::wait($barrier)`를 추가하면 현재 코루틴이 일시 중단되어 해당 코루틴 바리어를 참조하는 서브 코루틴이 종료될 때까지 대기합니다.
* 서브 코루틴이 종료될 때 `$barrier` 객체의 레퍼런스 카운트가 감소하며, `0`이 될 때까지 계속 됩니다.
* 모든 서브 코루틴이 작업을 완료하고 종료되면 `$barrier` 객체의 레퍼런스 카운트가 `0`이되며, 바리어가 해제됩니다. 이때 바닥에있는 소멸자는 자동으로 일시 중단된 코루틴을 다시 활성화하고 `Barrier::wait($barrier)` 함수에서 반환합니다.

`Coroutine\Barrier`는 [WaitGroup](/coroutine/wait_group) 및 [Channel](/coroutine/channel)보다 사용하기 쉬운 동시성 제어기로, `PHP` 동시성 프로그래밍의 사용자 경험을 크게 향상시킵니다.
