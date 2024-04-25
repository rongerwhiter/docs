# 동일하지 않은 하향 변경
## v5.0.0
* `Server`의 기본 실행 모드가 `SWOOLE_BASE`로 변경되었습니다.
* 최소 `PHP` 버전이 `8.0`으로 상향 조정되었습니다.
* 모든 클래스 메소드와 함수에 타입 힌트가 추가되어 강력한 타입 모드로 변경되었습니다.
* 밑줄(`PSR-0`) 카테고리 별명이 제거되었으며 네임스페이스 스타일 클래스 이름만 유지되었습니다. 예를 들어 `swoole_server`는 `Swoole\Server`로 변경해야 합니다.
* `Swoole\Coroutine\Redis` 및 `Swoole\Coroutine\MySQL`은 더 이상 사용되지 않으며 `Runtime Hook`과 원시 `Redis`/`MySQL` 클라이언트를 사용해야 합니다.
## v4.8.0

- `BASE` 모드에서는 `onStart` 콜백이 항상 첫 번째 작업 프로세스인 (`workerId`가 `0`인) 시작할 때 호출되며,`onWorkerStart`보다 먼저 실행됩니다. `onStart` 함수에서 항상 코루틴 `API`를 사용할 수 있으며, `Worker-0`에서 치명적인 오류로 다시 시작될 때 다시 `onStart`이 호출됩니다. 
이전 버전에서는 하나의 작업 프로세스만 있는 경우 `Worker-0`에서 `onStart`이 호출되었습니다. 여러 작업 프로세스가 있는 경우 `Manager` 프로세스에서 실행됩니다.
## v4.7.0

- `Table\Row`이 제거되었습니다. `Table`은 더 이상 배열 형식으로 읽기/쓰기를 지원하지 않습니다.
## v4.6.0

- `session id`의 최대 제한이 제거되었으며 더 이상 중복되지 않습니다.
- Coroutine을 사용하는 경우 `pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`와 같은 안전하지 않은 기능이 비활성화되었습니다.
- 기본적으로 coroutine hook을 활성화합니다.
- PHP7.1을 더 이상 지원하지 않습니다.
- `Event::rshutdown()`을 더 이상 사용하지 않도록 표시하고 Coroutine\run을 사용하도록 변경했습니다.
## v4.5.4

- `SWOOLE_HOOK_ALL`에 `SWOOLE_HOOK_CURL`이 포함되었습니다.
- `ssl_method`가 제거되었으며, `ssl_protocols`를 지원합니다.
## v4.4.12

- 이 버전에서는 WebSocket 프레임 압축을 지원하며, push 메서드의 세 번째 매개변수를 flags로 변경했습니다. strict_types가 설정되지 않은 경우 코드 호환성에 영향을주지 않지만, 그렇지 않으면 bool을 int로 암시 적으로 변환 할 수 없는 유형 오류가 발생합니다. 이 문제는 v4.4.13에서 수정될 것입니다.
## v4.4.1

- 등록 된 신호는 더 이상 이벤트 루프를 유지하는 조건으로 간주되지 않습니다. **프로그램이 신호를 등록했지만 다른 작업을 수행하지 않은 경우 곧바로 종료됩니다** (이 경우 프로세스를 종료하지 않고 타이머를 등록하여 방지할 수 있음)
## v4.4.0

- 공식 `PHP`와 호환을 유지하기 위해 `PHP7.0`을 더 이상 지원하지 않습니다. (@matyhtf)
- `Serialize` 모듈이 제거되었으며, 별도의 [ext-serialize](https://github.com/swoole/ext-serialize) 확장으로 유지됩니다.
- `PostgreSQL` 모듈이 제거되었으며, 별도의 [ext-postgresql](https://github.com/swoole/ext-postgresql) 확장으로 유지됩니다.
- `Runtime::enableCoroutine`은 더 이상 코루틴 내부 및 외부 환경을 자동으로 호환하지 않으며, 한 번 활성화되면 모든 차단 작업은 코루틴 내에서 호출해야 합니다. (@matyhtf)
- 새로운 코루틴 `MySQL` 클라이언트 드라이버가 도입되어 하부 설계가 더 규격화되었지만, 몇 가지 미세한 하위 호환성 변경 사항이 있습니다. (자세한 내용은 [4.4.0 업데이트 로그](https://wiki.swoole.com/wiki/page/p-4.4.0.html) 참조)
## v4.3.0

- 모든 비동기 모듈이 제거되었습니다. 자세한 내용은 [독립 비동기 확장](https://wiki.swoole.com/wiki/page/p-async_ext.html) 또는 [4.3.0 업데이트 로그](https://wiki.swoole.com/wiki/page/p-4.3.0.html)을 참조하십시오.
## v4.2.13

> 由于历史API设计存在问题导致的不可避免的不兼容变更

* 코루틴 Redis 클라이언트 구독 모드 동작 변경, 자세한 내용은 [구독 모드](https://wiki.swoole.com/#/coroutine_client/redis?id=%e8%ae%a2%e9%98%85%e6%a8%a1%e5%bc%8f)를 참조하세요.
## v4.2.12

> 실험적인 기능 + 역사적 API 설계의 문제로 인한 피할 수 없는 호환성 변경 사항

- `task_async` 설정이 제거되었으며, [task_enable_coroutine](https://wiki.swoole.com/#/server/setting?id=task_enable_coroutine)로 대체되었습니다.
## v4.2.5

- `onReceive` 및 `Server::getClientInfo`가 `UDP` 클라이언트 지원에서 제거되었습니다.
## v4.2.0

- `swoole_http2_client` 비동기 기능이 완전히 제거되었습니다. 코루틴 기반 HTTP2 클라이언트를 사용해주세요.
## v4.0.4

이 버전부터, 비동기 `Http2\Client`는 `E_DEPRECATED` 알림을 발생시키고 다음 버전에서 삭제될 예정입니다. 대신 `Coroutine\Http2\Client`를 사용하십시오.

`Http2\Response`의 `body` 속성이 `data`로 변경되었습니다. 이 변경은 `request`와 `response` 두 요소를 일치시키고 HTTP2 프레임 유형 이름과 더 일치하도록 하기 위한 것입니다.

이 버전부터 `Coroutine\Http2\Client`는 상당히 완벽한 HTTP2 프로토콜 지원을 제공하여 기업 환경의 애플리케이션 요구 사항을 충족할 수 있습니다. 이에 따라 HTTP2에 대한 일련의 변경 사항이 매우 필요하게 되었습니다.
## v4.0.3

`swoole_http2_response` 및 `swoole_http2_request`를 일관되게 유지하도록하십시오. 모든 속성 이름은 복수형으로 변경되었습니다. 다음 속성이 관련됩니다.

- `headers`
- `cookies`
## v4.0.2

> 기반 구현이 지나치게 복잡하여 유지보수하기 어려우며 사용자가 종종 오해를 일으키므로 아래 API를 일시적으로 삭제했습니다:

- `Coroutine\Channel::select`

그러나 동시에 개발 요구사항을 충족시키기 위해 `Coroutine\Channel->pop` 메서드에 `timeout`을 설정할 수 있는 두 번째 매개변수가 추가되었습니다.
## v4.0

> 코루틴 커널이 업그레이드되어 언제든지 어디서든지 코루틴을 호출할 수 있으며 특별한 처리가 필요하지 않으므로 다음 API가 삭제되었습니다.

- `Coroutine::call_user_func`
- `Coroutine::call_user_func_array`
