# 프로세스 간의 잠금 Lock

`PHP` 코드에서는 데이터 동기화를 위해 쉽게 잠금을 만들 수 있습니다. `Lock` 클래스는 `5`가지 종류의 잠금을 지원합니다.

| 잠금 종류 | 설명 |
| --- | --- |
| SWOOLE_MUTEX | 상호 배제 잠금 |
| SWOOLE_RWLOCK | 읽기/쓰기 잠금 |
| SWOOLE_SPINLOCK | 스핀락 |
| SWOOLE_FILELOCK | 파일 잠금 (폐기됨) |
| SWOOLE_SEM | 세마포어 (폐기됨) |

!> [onReceive](/server/events?id=onreceive) 등의 콜백 함수에서 잠금을 만들지 마십시오. 그렇지 않으면 메모리가 지속적으로 증가하여 메모리 누수가 발생할 수 있습니다.

## 사용 예시

```php
$lock = new Swoole\Lock(SWOOLE_MUTEX);
echo "[Master]create lock\n";
$lock->lock();
if (pcntl_fork() > 0)
{
  sleep(1);
  $lock->unlock();
} 
else
{
  echo "[Child] Lock 대기 중\n";
  $lock->lock();
  echo "[Child] Lock 획득\n";
  $lock->unlock();
  exit("[Child] 종료\n");
}
echo "[Master]lock 해제\n";
unset($lock);
sleep(1);
echo "[Master]종료\n";
```

## 경고

!> 코루틴에서는 잠금을 사용할 수 없으니 주의하세요. `lock` 및 `unlock` 작업 사이에서 코루틴 전환을 유발할 수 있는 `API`를 사용하지 마세요.

### 잘못된 예시

!> 이 코드는 코루틴 모드에서 `100%` 데드락을 발생시킵니다. [이 게시물](https://course.swoole-cloud.com/article/2)을 참조하세요.

```php
$lock = new Swoole\Lock();
$c = 2;

while ($c--) {
  go(function () use ($lock) {
      $lock->lock();
      Co::sleep(1);
      $lock->unlock();
  });
}
```

## 메소드

### __construct()

생성자.

```php
Swoole\Lock::__construct(int $type = SWOOLE_MUTEX, string $lockfile = '');
```

!> 잠금 객체를 반복해서 생성/파괴하지 마세요. 그렇지 않으면 메모리 누수가 발생합니다.

  * **매개변수** 

    * **`int $type`**
      * **기능**：잠금 종류
      * **기본값**：`SWOOLE_MUTEX`【상호 배제 잠금】
      * **다른 값**：없음

    * **`string $lockfile`**
      * **기능**：파일 잠금의 경로를 지정합니다【`SWOOLE_FILELOCK` 유형의 경우 필수로 전달해야 함】
      * **기본값**：없음
      * **다른 값**：없음

!> 각 종류의 잠금은 서로 다른 메소드를 지원합니다. 예를 들어, 읽기/쓰기 잠금, 파일 잠금은 `$lock->lock_read()`를 지원할 수 있습니다. 또한 파일 잠금을 제외한 다른 유형의 잠금은 반드시 부모 프로세스 내에서 생성되어야 합니다. 그래야 `fork`된 자식 프로세스가 서로 잠금을 경쟁할 수 있습니다.

### lock()

잠금 작업을 수행합니다. 다른 프로세스가 잠금을 보유하고 있는 경우 여기서 블록됩니다. 보유한 프로세스가 `unlock()`으로 잠금을 해제할 때까지 기다립니다.

```php
Swoole\Lock->lock(): bool
```

### trylock()

잠금 작업을 수행합니다. `lock` 메소드와 달리, `trylock()`는 차단되지 않고 즉시 반환합니다.

```php
Swoole\Lock->trylock(): bool
```

  * **반환값**

    * 잠금 성공 시 `true`를 반환하며, 이때 공유 변수를 수정할 수 있습니다.
    * 잠금 실패 시 `false`를 반환하며, 다른 프로세스가 잠금을 보유한다는 것을 나타냅니다.

!> `SWOOlE_SEM` 세마포어는 `trylock` 메소드를 지원하지 않습니다.

### unlock()

잠금을 해제합니다.

```php
Swoole\Lock->unlock(): bool
```

### lock_read()

읽기 전용 잠금을 얻습니다.

```php
Swoole\Lock->lock_read(): bool
```

* 읽기 잠금을 보유하는 동안 다른 프로세스도 읽기 잠금을 얻고 읽기 작업을 계속할 수 있습니다.
* 그러나 `$lock->lock()` 또는 `$lock->trylock()`을 사용할 수 없습니다. 이 두 메소드는 배타적 잠금을 얻기 위한 것이며, 배타적 잠금이 있는 동안 다른 프로세스는 어떠한 잠금 작업도 수행할 수 없습니다.
* 다른 프로세스가 배타적 잠금(호출 `$lock->lock()`/`$lock->trylock()`)을 획득하면, `$lock->lock_read()`는 차단됩니다. 이 때 배타적 잠금을 보유한 프로세스가 잠금을 해제할 때까지 기다립니다.

!> `SWOOLE_RWLOCK` 및 `SWOOLE_FILELOCK` 유형의 잠금만 읽기 전용 잠금을 지원합니다.

### trylock_read()

잠금을 얻습니다. 이 메소드는 `lock_read()`와 동일하지만 차단되지 않습니다.

```php
Swoole\Lock->trylock_read(): bool
```

!> 호출은 즉시 반환되므로 잠금을 얻었는지 확인해야 합니다.

### lockwait()

잠금을 얻습니다. `lock()` 메소드와 유사하게 작용하지만 `lockwait()`은 타임아웃을 설정할 수 있습니다.

```php
Swoole\Lock->lockwait(float $timeout = 1.0): bool
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：지정된 타임아웃 시간
      * **단위**：초【부동소수점 형식 지원, 예: `1.5`는 `1초 + 500ms`를 나타냄】
      * **기본값**：`1`
      * **다른 값**：없음

  * **반환값**

    * 지정된 시간 내에 잠금을 얻지 못하면 `false`를 반환합니다.
    * 잠금 성공 시 `true`를 반환합니다.

!> `Mutex` 유형의 잠금만 `lockwait`를 지원합니다.
