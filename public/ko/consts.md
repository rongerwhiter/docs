# 상수

!> 여기에는 모든 상수가 포함되어 있지 않습니다. 모든 상수를 확인하려면 [ide-helper](https://github.com/swoole/ide-helper/blob/master/output/swoole/constants.php)를 방문하거나 설치하세요.

## Swoole

상수 | 기능
---|---
SWOOLE_VERSION | 현재 Swoole의 버전 번호, 문자열 형태, 예: 1.6.0

## 생성자 매개변수

상수 | 기능
---|---
[SWOOLE_BASE](/learn?id=swoole_base) | 베이스 모드를 사용하여, 비즈니스 코드가 Reactor 프로세스에서 직접 실행됩니다.
[SWOOLE_PROCESS](/learn?id=swoole_process) | 프로세스 모드를 사용하여, 비즈니스 코드가 Worker 프로세스에서 실행됩니다.

## 소켓 타입

상수 | 기능
---|---
SWOOLE_SOCK_TCP | TCP 소켓을 생성합니다.
SWOOLE_SOCK_TCP6 | TCP IPv6 소켓을 생성합니다.
SWOOLE_SOCK_UDP | UDP 소켓을 생성합니다.
SWOOLE_SOCK_UDP6 | UDP IPv6 소켓을 생성합니다.
SWOOLE_SOCK_UNIX_DGRAM | Unix Dgram 소켓을 생성합니다.
SWOOLE_SOCK_UNIX_STREAM | Unix Stream 소켓을 생성합니다.
SWOOLE_SOCK_SYNC | 동기 클라이언트를 의미합니다.

## SSL 암호화 방식

상수 | 기능
---|---
SWOOLE_SSLv3_METHOD | -
SWOOLE_SSLv3_SERVER_METHOD | -
SWOOLE_SSLv3_CLIENT_METHOD | -
SWOOLE_SSLv23_METHOD (기본 암호화 방식) | -
SWOOLE_SSLv23_SERVER_METHOD | -
SWOOLE_SSLv23_CLIENT_METHOD | -
SWOOLE_TLSv1_METHOD | -
SWOOLE_TLSv1_SERVER_METHOD | -
SWOOLE_TLSv1_CLIENT_METHOD | -
SWOOLE_TLSv1_1_METHOD | -
SWOOLE_TLSv1_1_SERVER_METHOD | -
SWOOLE_TLSv1_1_CLIENT_METHOD | -
SWOOLE_TLSv1_2_METHOD | -
SWOOLE_TLSv1_2_SERVER_METHOD | -
SWOOLE_TLSv1_2_CLIENT_METHOD | -
SWOOLE_DTLSv1_METHOD | -
SWOOLE_DTLSv1_SERVER_METHOD | -
SWOOLE_DTLSv1_CLIENT_METHOD | -
SWOOLE_DTLS_SERVER_METHOD | -
SWOOLE_DTLS_CLIENT_METHOD | -

!> `SWOOLE_DTLSv1_METHOD`, `SWOOLE_DTLSv1_SERVER_METHOD`, `SWOOLE_DTLSv1_CLIENT_METHOD`는 `v4.5.0` 이상의 Swoole 버전에서 삭제되었습니다.

## SSL 프로토콜

상수 | 기능
---|---
SWOOLE_SSL_TLSv1 | -
SWOOLE_SSL_TLSv1_1 | -
SWOOLE_SSL_TLSv1_2 | -
SWOOLE_SSL_TLSv1_3 | -
SWOOLE_SSL_SSLv2 | -
SWOOLE_SSL_SSLv3 | -

!> Swoole 버전 `v4.5.4` 이상에서 사용 가능

## 로그 레벨

상수 | 기능
---|---
SWOOLE_LOG_DEBUG | 디버그 로그, 내부 커널 개발 디버깅에만 사용
SWOOLE_LOG_TRACE | 추적 로그, 시스템 문제 추적에 사용 가능, 디버그 로그는 핵심 정보가 있습니다
SWOOLE_LOG_INFO | 일반 정보, 정보 표시 용도
SWOOLE_LOG_NOTICE | 알림 정보, 시스템에 일부 동작이 있을 수 있음, 예: 재시작, 종료
SWOOLE_LOG_WARNING | 경고 정보, 시스템에 문제가 발생할 수 있음
SWOOLE_LOG_ERROR | 오류 정보, 시스템에 중요한 오류가 발생했으며 즉시 처리해야 함
SWOOLE_LOG_NONE | 로그 정보를 비활성화, 로그 정보가 표시되지 않음

!> `SWOOLE_LOG_DEBUG` 및 `SWOOLE_LOG_TRACE` 두 종류의 로그는 Swoole 확장을 컴파일할 때 [--enable-debug-log](/environment?id=debug-parameter) 또는 [--enable-trace-log](/environment?id=debug-parameter)를 사용해야 합니다. 일반 버전에서도 `log_level = SWOOLE_LOG_TRACE`로 설정해도 이러한 유형의 로그를 인쇄할 수 없습니다.

## 추적 태그

운영 중인 서비스는 처리 중인 많은 요청이 있으며, 하위 수준의 로그 수는 매우 많습니다. `trace_flags`를 사용하여 추적 로그의 태그를 설정하여 일부 추적 로그만 인쇄할 수 있습니다. `trace_flags`는 `|` 또는 연산자를 사용하여 여러 추적 항목을 설정할 수 있습니다.

```php
$serv->set([
	'log_level' => SWOOLE_LOG_TRACE,
	'trace_flags' => SWOOLE_TRACE_SERVER | SWOOLE_TRACE_HTTP2,
]);
```

다음과 같은 추적 항목을 지원합니다.

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`
