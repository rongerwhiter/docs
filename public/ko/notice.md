# 코루틴 프로그래밍에 필요한 지식

Swoole의 [코루틴](/coroutine) 기능을 사용하려면이 장의 프로그래밍 지식을 신중히 읽어보시기 바랍니다.
## 프로그래밍 패러다임

* 코루틴 내부에서는 전역 변수를 사용하지 말아야 합니다.
* 코루틴은 `use` 키워드를 사용하여 외부 변수를 현재 범위에 가져오는 것을 금지하며, 참조를 사용하지 말아야 합니다.
* 코루틴 간의 통신은 반드시[채널](/coroutine/channel)을 사용해야 합니다.

!> 다시 말해, 코루틴 간의 통신에는 전역 변수나 외부 변수를 현재 범위에 가져오지 말아야 하며, `Channel`을 사용해야 합니다.

* 만약 프로젝트에서 `zend_execute_ex` 또는 `zend_execute_internal`을 후킹한 확장이 있는 경우, C 스택에 특히 주의해야 합니다. [Co::set](/coroutine/coroutine?id=set)을 사용하여 C 스택 크기를 다시 설정할 수 있습니다.

!> 이 두 개의 진입점 함수를 후킹하면 대부분의 경우 플랫한 PHP 명령을 `C` 함수 호출로 바꾸어 C 스택의 소비를 증가시킵니다.
## 코루틴 종료

Swoole의 이전 버전에서 `exit`를 사용하여 강제로 스크립트를 종료하면 메모리 오류가 발생하여 예상치 못한 결과나 코어 덤프가 발생할 수 있습니다. Swoole 서비스에서 `exit`를 사용하면 전체 서비스 프로세스가 종료되며 내부의 모든 코루틴이 비정상 종료되어 심각한 문제가 발생합니다. Swoole은 오랫동안 개발자가 `exit`를 사용하는 것을 금지했지만, 예외를 던지는 비정상적인 방식인 `catch`를 사용하여 최상위에서 `exit`와 동일한 종료 논리를 구현할 수 있습니다.

!> **v4.2.2** 버전 이상에서는 스크립트( `http_server`를 만들지 않은 경우)가 현재 코루틴만 있는 상황에서 `exit`로 종료할 수 있습니다.

Swoole **v4.1.0** 버전 이상에서는 `코루틴`, `서비스 이벤트 루프`에서 PHP의 `exit`를 직접적으로 지원합니다. 이 경우에는 하위 수준에서 캐치할 수 있는 `Swoole\ExitException`을 자동으로 throw합니다. 개발자는 필요한 위치에서 예외를 캐치하고 원래의 PHP와 동일한 종료 논리를 구현할 수 있습니다.
### Swoole\ExitException

`Swoole\ExitException`는 `Exception`을 상속하며 `getStatus`와 `getFlags`라는 두 개의 메소드가 추가되었습니다:

```php
namespace Swoole;

class ExitException extends \Exception
{
	public function getStatus(): mixed
	public function getFlags(): int
}
```
```php
public function getStatus(): mixed
```

`getStatus()` 메서드는 `exit($status)`가 호출될 때 전달된 `status` 매개변수를 가져옵니다. 이 메서드는 모든 유형의 변수를 지원합니다.
```
#### getFlags()

`exit`할 때의 환경 정보 마스크를 얻습니다.

```php
public function getFlags(): int
```

다음 마스크를 사용할 수 있습니다:

| 상수 | 설명 |
| -- | -- |
| SWOOLE_EXIT_IN_COROUTINE | 코루틴 안에서 종료 |
| SWOOLE_EXIT_IN_SERVER | 서버에서 종료 |
### 사용 방법
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

function route()
{
    controller();
}

function controller()
{
    your_code();
}

function your_code()
{
    Coroutine::sleep(.001);
    exit(1);
}

run(function () {
    try {
        route();
    } catch (\Swoole\ExitException $e) {
        var_dump($e->getMessage());
        var_dump($e->getStatus() === 1);
        var_dump($e->getFlags() === SWOOLE_EXIT_IN_COROUTINE);
    }
});
```
```php
use function Swoole\Coroutine\run;

$exit_status = 0;
run(function () {
    try {
        exit(123);
    } catch (\Swoole\ExitException $e) {
        global $exit_status;
        $exit_status = $e->getStatus();
    }
});
var_dump($exit_status);
```
## 예외 처리

코루틴 프로그래밍에서는 예외를 처리할 때 `try/catch`를 직접 사용할 수 있습니다. **그러나 예외를 코루틴 내에서 처리해야 하며, 코루틴 간 예외 전파는 허용되지 않습니다**.

!> 응용 프로그램에서 `throw`된 `Exception`뿐만 아니라, 하위 수준의 오류도 처리할 수 있습니다. 함수나 클래스, 메서드가 존재하지 않는 경우도 이에 포함됩니다.  
### 오류 예시

다음 코드에서 `try/catch`와 `throw`는 서로 다른 코루틴에 있기 때문에 해당 예외를 코루틴 내에서 잡을 수 없습니다. 코루틴이 종료될 때 잡히지 않은 예외가 발생하면 치명적인 오류가 발생합니다.

```bash
PHP Fatal error:  Uncaught RuntimeException
```

```php
try {
	Swoole\Coroutine::create(function () {
		throw new \RuntimeException(__FILE__, __LINE__);
	});
}
catch (\Throwable $e) {
	echo $e;
}
```
### 올바른 예시

코루틴 내에서 예외를 잡습니다.

```php
function test() {
	throw new \RuntimeException(__FILE__, __LINE__);
}

Swoole\Coroutine::create(function () {
	try {
		test();
	}
	catch (\Throwable $e) {
		echo $e;
	}
});
```
## `get` / `set` 매직 메소드 안에서 코루틴 전환을 발생시킬 수 없습니다

이유: [PHP7 내부 분석 참고](https://github.com/pangudashu/php7-internal/blob/40645cfe087b373c80738881911ae3b178818f11/3/zend_object.md)

> **참고:** 클래스에 __get() 메소드가 존재하는 경우, 인스턴스 객체의 속성 메모리(즉, properties_table) 할당 시에 HashTable이 여분으로 할당됩니다. 이 HashTable은 입력된 $var 이름을 각 호출 때마다 저장하여 무한 재귀 호출을 방지합니다. 예를 들어:
>
> ***public function __get($var) { return $this->$var; }***
>
> 이 경우, __get() 호출 시 존재하지 않는 속성에 다시 접근하는데, 이는 __get() 메소드에서 재귀 호출을 일으킵니다. 따라서 $var 요청을 확인하지 않으면 계속해서 재귀 호출이 진행됩니다. 이에, __get() 호출 전에 현재 $var가 이미 __get() 안에 있는지 확인하고, 있다면 __get()을 다시 호출하지 않고 $var을 그 HashTable에 키로 삽입한 뒤, 해시 값을 *guard |= IN_ISSET으로 설정하고, __get()을 완료한 후 해시 값을 *guard &= ~IN_ISSET로 재설정합니다.
>
> 이 HashTable은 단순히 __get()만 사용하는 것이 아니며, 다른 매직 메소드도 사용합니다. 따라서 해시 값의 유형은 zend_long이며, 다른 매직 메소드는 다른 bit 위치를 사용합니다. 둘째로, 모든 객체가 이 HashTable을 추가로 할당하는 것은 아니며, 객체 생성 시 ***zend_class_entry.ce_flags***에 ***ZEND_ACC_USE_GUARDS***가 포함되어 있는지에 따라 할당 여부가 결정되며, 클래스 컴파일 시 __get()과 같은 메소드가 정의되어 있는지 확인하고, 이를 발견하면 ce_flags에 이 마스크를 설정합니다.

코루틴을 전환한 후, 다음 호출은 무한 재귀 호출로 간주되어 문제가 되며, 이 문제는 PHP의 **특성**에 기인하며, PHP 개발팀과의 커뮤니케이션을 통해 현재까지 해결책이 없음을 확인했습니다.

주의: 매직 메소드 안에 코루틴 전환을 유발하는 코드는 없지만, 코루틴 강제 스케줄링을 활성화한 경우에는 매직 메소드가 강제로 코루틴 전환될 수 있습니다.

권장사항: 직접 `get`/`set` 메소드를 구현하여 명시적으로 호출하십시오.

원문 링크: [#2625](https://github.com/swoole/swoole-src/issues/2625)
## 심각한 오류

다음 행동은 심각한 오류를 초래할 수 있습니다.
### 여러 개의 코루틴 간에 연결 공유

동기식 블로킹 프로그램과는 달리 코루틴은 병렬적으로 요청을 처리하기 때문에 동시에 여러 요청이 병렬로 처리될 수 있습니다. 클라이언트 연결을 공유하면 서로 다른 코루틴 간에 데이터가 혼선을 일으킬 수 있습니다. 참고: [/question/use?id=client-has-already-been-bound-to-another-coroutine](/question/use?id=client-has-already-been-bound-to-another-coroutine)
### 클래스 정적 변수/전역 변수를 사용하여 컨텍스트 저장

여러 개의 코루틴은 동시에 실행되기 때문에 코루틴 컨텍스트 내용을 저장하는 데 클래스 정적 변수나 전역 변수를 사용할 수 없습니다. 지역 변수를 사용하는 것은 안전합니다. 왜냐하면 지역 변수의 값은 자동으로 코루틴 스택에 저장되기 때문에 다른 코루틴에서 코루틴의 지역 변수에 액세스할 수 없습니다.
#### 오류 예시

```php
$server = new Swoole\Http\Server('127.0.0.1', 9501);

$_array = [];
$server->on('request', function ($request, $response) {
    global $_array;
    // /a 요청 (코루틴 1)
    if ($request->server['request_uri'] == '/a') {
        $_array['name'] = 'a';
        co::sleep(1.0);
        echo $_array['name'];
        $response->end($_array['name']);
    }
    // /b 요청 (코루틴 2)
    else {
        $_array['name'] = 'b';
        $response->end();
    }
});
$server->start();
```

`2`개의 동시 요청을 보냅니다.

```shell
curl http://127.0.0.1:9501/a
curl http://127.0.0.1:9501/b
```

* 코루틴 `1`에서 전역 변수 `$_array['name']`의 값이 `a`로 설정됨
* 코루틴 `1`이 `co::sleep`을 호출하여 일시 중지됨
* 코루틴 `2`가 실행되어 `$_array['name']`의 값을 `b`로 변경하고 종료
* 이제 타이머가 반환되어 백그라운드에서 코루틴 `1`을 다시 실행함. 코루틴 `1`의 로직에는 상황에 따른 의존성이 있습니다. `$_array['name']`의 값을 다시 출력할 때, 프로그램은 `a`를 예상하지만, 이 값은 이미 코루틴 `2`에 의해 수정되어 `b`가 출력됩니다. 이렇게 되면 논리적 오류가 발생합니다.
* 마찬가지로, 클래스 정적 변수 `Class::$array`, 전역 객체 속성 `$object->array`, 기타 슈퍼글로벌 변수 `$GLOBALS` 등을 사용하여 코루틴 프로그램에서 상황을 저장하는 것은 매우 위험합니다. 예상치 못한 동작이 발생할 수 있습니다.

![](../_images/coroutine/notice-1.png)
#### 올바른 예시: Context를 사용하여 컨텍스트 관리

코루틴 컨텍스트를 관리하기 위해 `Context` 클래스를 사용할 수 있습니다. `Context` 클래스 내에서 `Coroutine::getuid`를 사용하여 코루틴 `ID`를 가져와 다른 코루틴 간에 전역 변수를 격리하며, 코루틴이 종료될 때 컨텍스트 데이터를 정리합니다.

```php
use Swoole\Coroutine;

class Context
{
    protected static $pool = [];

    static function get($key)
    {
        $cid = Coroutine::getuid();
        if ($cid < 0)
        {
            return null;
        }
        if(isset(self::$pool[$cid][$key])){
            return self::$pool[$cid][$key];
        }
        return null;
    }

    static function put($key, $item)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            self::$pool[$cid][$key] = $item;
        }

    }

    static function delete($key = null)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            if($key){
                unset(self::$pool[$cid][$key]);
            }else{
                unset(self::$pool[$cid]);
            }
        }
    }
}
```

사용:

```php
use Swoole\Coroutine\Context;

$server = new Swoole\Http\Server('127.0.0.1', 9501);

$server->on('request', function ($request, $response) {
    if ($request->server['request_uri'] == '/a') {
        Context::put('name', 'a');
        co::sleep(1.0);
        echo Context::get('name');
        $response->end(Context::get('name'));
        // 코루틴 종료시 정리
        Context::delete('name');
    } else {
        Context::put('name', 'b');
        $response->end();
        // 코루틴 종료시 정리
        Context::delete();
    }
});
$server->start();
```
