# Process\Manager

프로세스 관리자는 [Process\Pool](/process/process_pool)을 기반으로 합니다. 여러 개의 프로세스를 관리할 수 있습니다. `Process\Pool`과 비교하면 서로 다른 작업을 수행하는 여러 프로세스를 쉽게 만들 수 있으며 각 프로세스가 코루틴 환경에 있는 지를 제어할 수 있습니다.

## 호환되는 버전

| 버전 | 클래스명 | 업데이트 내역 |
| ------ | ----------------------------- | ---------------------------------------- |
| v4.5.3 | Swoole\Process\ProcessManager | -                                        |
| v4.5.5 | Swoole\Process\Manager        | 이름 변경, ProcessManager를 Manager의 별칭으로 사용 |

!> `v4.5.3` 이상의 버전에서 사용 가능합니다.

## 사용 예제

```php
use Swoole\Process\Manager;
use Swoole\Process\Pool;

$pm = new Manager();

for ($i = 0; $i < 2; $i++) {
    $pm->add(function (Pool $pool, int $workerId) {
    });
}

$pm->start();
```

## 메소드

### __construct()

생성자 함수입니다.

```php
Swoole\Process\Manager::__construct(int $ipcType = SWOOLE_IPC_NONE, int $msgQueueKey = 0);
```

* **파라미터**

  * **`int $ipcType`**
    * **기능** : 프로세스 간 통신 모드, `Process\Pool`의 `$ipc_type`과 동일합니다. 【기본값은`0`이며 아무런 프로세스 간 통신 기능을 사용하지 않음을 의미합니다.】
    * **기본값** : `0`
    * **다른 값** : 없음

  * **`int $msgQueueKey`**
      * **기능** : 메시지 큐의 `key`, `Process\Pool`의 `$msgqueue_key` 와 동일
      * **기본값** : 없음
      * **다른 값**: 없음

### setIPCType()

작업 프로세스 간 통신 방법을 설정합니다.

```php
Swoole\Process\Manager->setIPCType(int $ipcType): self;
```

* **파라미터**

  * **`int $ipcType`**
      * **기능** : 프로세스 간 통신 모드를 설정합니다.
      * **기본값** : 없음
      * **다른 값** : 없음

### getIPCType()

작업 프로세스 간 통신 방법을 가져옵니다.

```php
Swoole\Process\Manager->getIPCType(): int;
```

### setMsgQueueKey()

메시지 큐의 `key`를 설정합니다.

```php
Swoole\Process\Manager->setMsgQueueKey(int $msgQueueKey): self;
```

* **파라미터**

  * **`int $msgQueueKey`**
      * **기능** : 메시지 큐의 `key`를 설정합니다.
      * **기본값** : 없음
      * **다른 값** : 없음

### getMsgQueueKey()

메시지 큐의 `key`를 가져옵니다.

```php
Swoole\Process\Manager->getMsgQueueKey(): int;
```

### add()

작업 프로세스를 추가합니다.

```php
Swoole\Process\Manager->add(callable $func, bool $enableCoroutine = false): self;
```

* **파라미터**

  * **`callable $func`**
      * **기능** : 현재 프로세스가 실행할 콜백 함수
      * **기본값** : 없음
      * **다른 값** : 없음

  * **`bool $enableCoroutine`**
      * **기능** : 이 프로세스에 대해 콜백 함수를 실행하기 위해 코루틴을 생성할 지 여부
      * **기본값** : false
      * **다른 값** : 없음

### addBatch()

일괄적으로 작업 프로세스를 추가합니다.

```php
Swoole\Process\Manager->addBatch(int $workerNum, callable $func, bool $enableCoroutine = false): self
```

* **파라미터**

  * **`int $workerNum`**
      * **기능** : 프로세스 수량을 일괄적으로 추가합니다.
      * **기본값** : 없음
      * **다른 값** : 없음

  * **`callable $func`**
      * **기능** : 이러한 프로세스가 실행할 콜백 함수
      * **기본값** : 없음
      * **다른 값** : 없음

  * **`bool $enableCoroutine`**
      * **기능** : 이러한 프로세스에 대해 콜백 함수를 실행하기 위해 코루틴을 생성할 지 여부
      * **기본값** : 없음
      * **다른 값** : 없음

### start()

작업 프로세스를 시작합니다.

```php
Swoole\Process\Manager->start(): void
```
