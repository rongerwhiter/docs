# 병행 List

병행적인 `List` 구조를 생성하고, 이를 자식 스레드에게 전달할 수 있습니다. 다른 스레드에서 읽고 쓸 때 보입니다.
자세한 특징은 [병행 Map](thread/map.md) 에서 확인할 수 있습니다.


## 사용법
`Thread\AraryList`는 `ArrayAccess` 인터페이스를 구현했기 때문에 배열처럼 직접 조작할 수 있습니다.

## 주의사항
- `ArrayList`는 요소를 추가할 수만 있고, 순차적으로 삭제하거나 할당할 수 없습니다.

## 예시

```php
use Swoole\Thread;
use Swoole\Thread\AraryList;

$args = Thread::getArguments();
if (empty($args)) {
    $list = new AraryList;
    $thread = Thread::exec(__FILE__, $i, $list);
    sleep(1);
    $list[] = unique();
    $thread->join();
} else {
    $list = $args[1];
    sleep(2);
    var_dump($list[0]);
}
```
