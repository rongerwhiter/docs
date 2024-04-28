# 쓰레드 관리

## Thread::join()

자식 쓰레드가 종료될 때까지 기다립니다. 자식 쓰레드가 여전히 실행 중인 경우, `join()`은 차단됩니다.

```php
$thread = Thread::exec(__FILE__, $i);
$thread->join();
```

## Thread::joinable()

자식 쓰레드가 이미 종료되었는지 확인합니다.

### 반환 값
- `true` : 자식 쓰레드가 이미 종료됨. 이 경우 `join()` 호출이 차단되지 않습니다.
- `false` : 종료되지 않았음을 나타냄.

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->joinable());
```

## Thread::detach()

자식 쓰레드를 부모 쓰레드의 제어에서 분리하여, 더 이상 `join()`을 사용하여 쓰레드가 종료될 때까지 대기하거나 자원을 회수할 필요가 없습니다.

```php
$thread = Thread::exec(__FILE__, $i);
$thread->detach();
unset($thread);
```

## Thread::getId()

현재 쓰레드의 `ID`를 가져오는 정적 메서드로, 자식 쓰레드에서 호출됩니다.

```php
var_dump(Thread::getId());
```

## Thread::getId()

현재 쓰레드의 `ID`를 가져오는 정적 메서드로, 자식 쓰레드에서 호출됩니다.

```php
var_dump(Thread::getId());
```

## Thread::getArguments()

현재 쓰레드의 인수를 가져오는 정적 메서드로, 자식 쓰레드에서 호출되며 부모 쓰레드가 `Thread::exec()` 시 전달한 것입니다.

```php
var_dump(Thread::getArguments());
```

## Thread::$id

이 객체 속성을 사용하여 자식 쓰레드의 `ID`를 가져옵니다.

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->id);
```
