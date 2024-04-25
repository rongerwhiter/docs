# 병행 맵

병행 Map 구조를 생성하여 스레드에게 전달할 수 있습니다. 다른 스레드에서 읽고 쓸 때 가시적입니다.

## 특징
- `Map`과 `ArrayList`, `Queue`는 자동으로 메모리를 할당하며, `Table`과 같이 고정 할당할 필요가 없습니다.
- 내부적으로 자동으로 잠금이 걸리므로 스레드 안전합니다.
- 오직 `null/bool/int/float/string` 유형만 지원하며, 다른 유형은 쓸 때 자동으로 직렬화되고 읽을 때 역직렬화됩니다.
- 반복자를 지원하지 않으며, 반복자에서 요소를 삭제하면 메모리 오류가 발생할 수 있습니다.
- 스레드를 생성하기 전에 `Map`, `ArrayList`, `Queue` 객체를 스레드의 인수로 전달해야 합니다.

## 사용법
`Thread\Map`은 `ArrayAccess` 인터페이스를 구현하여 배열 작업으로 직접 사용할 수 있습니다.

## 예시

```php
use Swoole\Thread;
use Swoole\Thread\Map;

$args = Thread::getArguments();
if (empty($args)) {
    $map = new Map;
    $thread = Thread::exec(__FILE__, $i, $map);
    sleep(1);
    $map['test'] = unique();
    $thread->join();
} else {
    $map = $args[1];
    sleep(2);
    var_dump($map['test']);
}
```

## 메서드

### Map::count()
요소 수를 가져옵니다.

### Map::keys()
모든 `key`를 반환합니다.

### Map::clean()
모든 요소를 지웁니다.
