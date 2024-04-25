# 코루틴 디버깅

`Swoole` 코루틴을 사용할 때 디버깅할 수 있는 방법은 다음과 같습니다.

## GDB 디버깅

### GDB 실행 <!-- {docsify-ignore} -->

```shell
gdb php test.php
```

### gdbinit <!-- {docsify-ignore} -->

```shell
(gdb) source /path/to/swoole-src/gdbinit
```

### 브레이크포인트 설정 <!-- {docsify-ignore} -->

예시: `co::sleep` 함수

```shell
(gdb) b zim_swoole_coroutine_util_sleep
```

### 현재 프로세스의 모든 코루틴과 상태 출력 <!-- {docsify-ignore} -->

```shell
(gdb) co_list 
coroutine 1 SW_CORO_YIELD
coroutine 2 SW_CORO_RUNNING
```

### 현재 실행 중인 코루틴의 콜 스택 출력 <!-- {docsify-ignore} -->

```shell
(gdb) co_bt 
coroutine cid:[2]
[0x7ffff148a100] Swoole\Coroutine->sleep(0.500000) [internal function]
[0x7ffff148a0a0] {closure}() /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:7 
[0x7ffff141e0c0] go(object[0x7ffff141e110]) [internal function]
[0x7ffff141e030] (main) /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:10
```

### 지정된 코루틴 id의 콜 스택 출력 <!-- {docsify-ignore} -->

``` shell
(gdb) co_bt 1
[0x7ffff1487100] Swoole\Coroutine->sleep(0.500000) [internal function]
[0x7ffff14870a0] {closure}() /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:3 
[0x7ffff141e0c0] go(object[0x7ffff141e110]) [internal function]
[0x7ffff141e030] (main) /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:10 
```

### 전역 코루틴 상태 출력 <!-- {docsify-ignore} -->

```shell
(gdb) co_status 
	 stack_size: 2097152
	 call_stack_size: 1
	 active: 1
	 coro_num: 2
	 max_coro_num: 3000
	 peak_coro_num: 2
```

## PHP 코드 디버깅

현재 프로세스 내의 모든 코루틴을 순회하고 호출 스택을 출력합니다.

```php
Swoole\Coroutine::listCoroutines(): Swoole\Coroitine\Iterator
```

!> `4.1.0` 이상 버전이 필요합니다.

* 이터레이터를 반환하며, `foreach`를 사용하여 순회하거나 `iterator_to_array`로 배열로 변환할 수 있습니다.

```php
use Swoole\Coroutine;
$coros = Coroutine::listCoroutines();
foreach($coros as $cid)
{
	var_dump(Coroutine::getBackTrace($cid));
}
```

[Swoole 마이크로 코스의 비디오 튜토리얼](https://course.swoole-cloud.com/course-video/66)를 참조하십시오.
