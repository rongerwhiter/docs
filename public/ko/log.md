# 버전 업데이트 기록

엄격한 버전 업데이트 기록이 `v1.5` 버전부터 구축되었습니다. 현재 평균 반년마다 큰 버전이 출시되며, 작은 버전은 매주 `2-4`회 릴리스됩니다.
```php
<?php

echo "제안되는 PHP 버전은 다음과 같습니다:";
echo "
- 7.4
- 8.0
- 8.1
- 8.2
";

?>
```
## 권장되는 Swoole 버전
`Swoole5.x`와 `Swoole4.8.x`

두 버전 간의 차이는 `v5.x`는 주요 변경사항이 있는 반면, `v4.8.x`는 **주요** 변경사항이 없고 `BUG`만 수정하는 브랜치입니다.

!> `v4.x` 이상 버전은 [enable_coroutine](/server/setting?id=enable_coroutine)을 설정하여 코루틴 기능을 비활성화할 수 있어 코루틴 버전이 아닌 버전으로 변경할 수 있습니다.
## 버전 유형

* `alpha` - 특징 미리보기 버전입니다. 개발 일정에 따라 작업이 완료되어 공개 미리보기가 시작되었으며, 더 많은 `BUG`가 존재할 수 있습니다.
* `beta` - 테스트 버전으로, 이미 개발 환경 테스트에 사용할 수 있으며 `BUG`가 존재할 수 있습니다.
* `rc[1-n]` - 리브릿 출시 버전으로, 출시 주기에 진입하여 대규모 테스트를 수행 중이며, 이 기간 동안에도 `BUG`를 발견할 수 있습니다.
* 접미사가 없는 경우 안정 버전을 나타냅니다. 이는 해당 버전이 개발이 완료되어 공식적으로 사용할 수 있는 것을 의미합니다.
## 현재 버전 정보 확인

```shell
php --ri swoole
```
## v6.0.0
- 다중 스레드 모드 지원 추가
- `Coroutine\Redis`, `Coroutine\MySQL`, `Coroutine\PostgreSQL` 클라이언트 제거, `ext-redis`, `mysqli`, `pdo_mysql`, `pdo_pgsql`로 대체
## v5.1.0  
### 새로운 기능
- `pdo_pgsql`에 대한 코루틴 지원 추가
- `pdo_odbc`에 대한 코루틴 지원 추가
- `pdo_oci`에 대한 코루틴 지원 추가
- `pdo_sqlite`에 대한 코루틴 지원 추가
- `pdo_pgsql`, `pdo_odbc`, `pdo_oci`, `pdo_sqlite`의 연결 풀 구성 추가
### 향상
- `Http\Server`의 성능이 향상되어 극한 상황에서 `60%` 향상될 수 있습니다.
### 수리
- `WebSocket` 서버 클라이언트의 요청으로 인한 메모리 누수 수정
- `http` 서버의 정상 종료로 인해 클라이언트가 종료되지 않는 문제 수정
- 컴파일 시 `--enable-thread-context` 옵션을 사용했을 때 `Process::signal()` 기능이 작동하지 않는 문제 수정
- `SWOOLE_BASE` 모드에서 비정상 종료 시 연결 수가 잘못 통계되는 문제 수정
- `stream_select()` 함수 서명 오류 수정
- 파일 MIME 정보의 대소문자 구분 수정
- `Http2\Request::$usePipelineRead`의 오타 수정으로 인해 PHP 8.2 환경에서 경고가 발생하는 문제 수정
- `SWOOLE_BASE` 모드에서의 메모리 누수 문제 수정
- `Http\Response::cookie()` 메서드에서 쿠키 만료 시간 설정으로 인한 메모리 누수 문제 수정
- `SWOOLE_BASE` 모드에서의 연결 누수 문제 수정
### 내부
- PHP 8.3에서 swoole의 php_url_encode 함수 서명 문제를 수정합니다.
- 유닛 테스트 옵션 문제를 수정합니다.
- 코드를 최적화 및 재구성합니다.
- PHP 8.3과 호환됩니다.
- 32비트 운영 체제에서의 컴파일은 지원되지 않습니다.
##  v5.0.3
### 향상
- `--with-nghttp2_dir` 옵션 추가, 시스템의 `nghttp2` 라이브러리 사용 가능
- 바이트 길이 또는 크기 관련 옵션 지원
- `Process\Pool::sendMessage()` 함수 추가
- `Http\Response:cookie()`에서 `max-age` 지원
### 수정
- `Server task/pipemessage/finish` 이벤트로 인한 메모리 누수 수정
### 내부
- `http` 응답 헤더 충돌 시 오류가 발생하지 않습니다.
- `Server` 연결 종료 시 오류가 발생하지 않습니다.
## v5.0.2
### 강화
- `http2`의 기본 설정을 구성할 수 있도록 지원
- 8.1 또는 그 이상의 `xdebug`를 지원
- 여러 소켓을 가진 curl 핸들을 지원하도록 기본 curl을 리팩토링하였습니다. 예를 들어 curl ftp 프로토콜
- `Process::setPriority/getPriority`에 `who` 매개변수를 추가했습니다
- `Coroutine\Socket::getBoundCid()` 메소드를 추가했습니다
- `Coroutine\Socket::recvLine/recvWithBuffer` 메소드의 `length` 매개변수의 기본값을 `65536`으로 조정했습니다
- 교차 코루틴 종료 기능을 리팩토링하여 메모리 해제를 더 안전하게 만들었으며 치명적인 오류 발생시 크래시 문제를 해결했습니다
- `Coroutine\Client`、`Coroutine\Http\Client`、`Coroutine\Http2\Client`에 `socket` 속성을 추가하여 `socket` 리소스를 직접 조작할 수 있습니다
- `Http\Server`에서 `http2` 클라이언트로 빈 파일을 전송합니다
- `Coroutine\Http\Server`의 우아한 다시 시작을 지원합니다. 서버를 닫을 때 클라이언트 연결이 강제로 끊어지지 않고 새 요청을 수신만 중단합니다
- `pcntl_rfork`와 `pcntl_sigwaitinfo`를 안전하지 않은 함수 목록에 추가했으며, 코루틴 컨테이너를 시작할 때 닫힐 것입니다
- `SWOOLE_BASE` 모드 프로세스 관리자를 리팩토링하여 종료 및 다시로드 동작을 `SWOOLE_PROCESS`와 일관되도록 했습니다
## v5.0.1
- `PHP-8.2`를 지원하며, 코루틴 예외 처리를 개선하였으며, `ext-soap`과 호환됩니다.
- `pgsql` 코루틴 클라이언트의 `LOB` 지원이 추가되었습니다.
- WebSocket 클라이언트가 개선되어 헤더가 `=` 대신에 `websocket`를 포함하여 업그레이드되었습니다.
- 서버가 `connection close`를 보낼 때 `http 클라이언트`가 최적화되어 `keep-alive`를 비활성화합니다.
- 압축 라이브러리가 없는 경우 `Accept-Encoding` 헤더를 추가하지 않도록 `http 클라이언트`를 최적화하였습니다.
- 디버그 정보가 개선되었으며, `PHP-8.2`에서 암호를 민감한 매개변수로 설정합니다.
- `Server::taskWaitMulti()`를 강화하여, 코루틴 환경에서 더 이상 블로킹되지 않습니다.
- 로그 함수가 최적화되어, 로그 파일 쓰기 실패 시 화면에 더 이상 출력되지 않습니다.
### 수리
- `Coroutine::printBackTrace()` 및 `debug_print_backtrace()`의 인수 호환성 문제를 수정했습니다.
- 소켓 리소스 지원이 포함된 `Event::add()`를 수정했습니다.
- `zlib`가없을 때의 컴파일 오류를 수정했습니다.
- 예상치 못한 문자열로 구문 분석 될 때 Unpack 서버 작업의 충돌 문제를 수정했습니다.
- `1ms` 이하의 타이머를 추가할 때 `0`으로 강제 설정되는 문제를 수정했습니다.
- 열을 추가하기 전에 `Table::getMemorySize()`를 사용하여 충돌이 발생하는 문제를 수정했습니다.
- `Http\Response::setCookie()` 메소드의 만료 매개변수 이름을 `expires`로 수정했습니다.
## V5.0.0
### 새로운 특징
- `Server`에 `max_concurrency` 옵션 추가됨
- `Coroutine\Http\Client`에 `max_retries` 옵션 추가됨
- `name_resolver` 전역 옵션 추가됨. `Server`에 `upload_max_filesize` 옵션 추가됨
- `Coroutine::getExecuteTime()` 메소드 추가됨
- `Server`에 `SWOOLE_DISPATCH_CONCURRENT_LB`의 `dispatch_mode` 추가됨
- 모든 함수의 매개변수와 반환 값을위한 타입 시스템을 강화함
- 모든 생성자가 실패할 때 예외를 throw하도록 오류 처리 개선
- `Server`의 기본 모드를 `SWOOLE_BASE` 모드로 조정함
- `pgsql` 코루틴 클라이언트를 코어 라이브러리로 이전함. `4.8.x` 브랜치에서의 모든 `bug` 수정 포함
### 제거
- `PSR-0` 스타일의 클래스 이름이 제거되었습니다.
- 닫힌 함수에 자동으로 `Event::wait()`를 추가하는 기능이 제거되었습니다.
- `Server::tick/after/clearTimer/defer`의 별칭이 제거되었습니다.
- `--enable-http2/--enable-swoole-json`이 제거되었고, 기본적으로 활성화되도록 조정되었습니다.
### 폐기 처리
- 코루틴 클라이언트 `Coroutine\Redis`와 `Coroutine\MySQL`는 기본적으로 폐기 처리됩니다.
## v4.8.13
- 네이티브 curl을 리팩토링하여 다중 소켓을 지원하도록 업그레이드하였습니다. 예를 들어 curl FTP 프로토콜을 지원합니다.
- `http2` 설정을 수동으로 설정할 수 있도록 지원합니다.
- WebSocket 클라이언트를 개선했으며, 헤더에 `equal` 대신 `websocket`을 포함시켰습니다.
- HTTP 클라이언트를 최적화하여 서버가 연결을 닫을 때 `keep-alive`를 비활성화했습니다.
- 디버그 정보를 향상시켰으며, PHP-8.2에서 비밀번호를 민감한 매개변수로 설정하였습니다.
- `HTTP Range Requests`를 지원합니다.
### 수정
- `Coroutine::printBackTrace()` 및 `debug_print_backtrace()`의 인수 호환성 문제를 수정했습니다.
- `WebSocket` 서버에서 동시에 `HTTP2`와 `WebSocket` 프로토콜을 활성화할 때 길이를 잘못 분석하는 문제를 수정했습니다.
- `Server::send()`, `Http\Response::end()`, `Http\Response::write()` 및 `WebSocket/Server::push()`에서 `send_yield`시 메모리 누수 문제를 수정했습니다.
- `Table::getMemorySize()`를 사용하여 열을 추가하기 전에 충돌을 일으키는 문제를 수정했습니다.
## v4.8.12
### 강화
- PHP8.2 지원
- `Event::add()` 함수는 `sockets 리소스`를 지원합니다.
- `Http\Client::sendfile()`은 4G 이상의 파일을 지원합니다.
- `Server::taskWaitMulti()`는 코루틴 환경을 지원합니다.
### 수정
- 잘못된 `multipart body`를 수신하면 오류 메시지가 표시되는 문제를 수정했습니다.
- 타이머의 시간이 `1ms` 미만인 경우 발생하는 오류를 수정했습니다.
- 디스크 공간이 가득 차서 발생하는 데드락 문제를 수정했습니다.
## v4.8.11
### 향상된 사항
- `Intel CET` 보안 방어 메커니즘 지원
- `Server::$ssl` 속성 추가
- `pecl`을 사용하여 `swoole`을 컴파일할 때 `enable-cares` 속성 추가
- `multipart_parser` 해석기 재구성
### 수정
- `pdo` 지속 연결로 인한 예외로 인한 세그멘테이션 오류 수정
- 코루틴을 사용한 소멸자로 인한 세그멘테이션 오류 수정
- `Server::close()`의 잘못된 오류 메시지 수정
## v4.8.10
### 수정사항

- `stream_select`의 타임아웃 매개변수가 `1ms` 미만이면 `0`으로 재설정합니다.
- 컴파일시 `-Werror=format-security` 추가시 컴파일이 실패하는 문제를 수정했습니다.
- `curl`을 사용하여 `Swoole\Coroutine\Http\Server`에서 세그멘테이션 오류가 발생하는 문제를 수정했습니다.
## v4.8.9
### 강화

- `Http2` 서버에서 `http_auto_index` 옵션을 지원합니다.
### 수정

- `Cookie` 파서 최적화, `HttpOnly` 옵션 지원
- #4657 수정, `socket_create` 메소드 반환 타입 문제 수정
- `stream_select` 메모리 누수 수정
### CLI 업데이트

- `CygWin`에 SSL 인증서 체인이 포함되어 SSL 인증 오류가 해결되었습니다.
- `PHP-8.1.5`로 업데이트되었습니다.
## v4.8.8
### 최적화

- SW_IPC_BUFFER_MAX_SIZE를 64k로 줄입니다.
- http2의 header_table_size를 최적화합니다.
### 수정

- `enable_static_handler`를 사용하여 정적 파일을 다운로드할 때 소켓 오류가 발생하는 문제를 수정했습니다.
- http2 서버 NPN 오류를 수정했습니다.
## v4.8.7
### 강화

- curl_share 지원 추가
### 수정

- arm32 아키텍처에서 정의되지 않은 심볼 오류 수정
- `clock_gettime()` 호환성 수정
- 커널이 대규모 메모리를 부족하게 할 경우, PROCESS 모드 서버가 전송 실패하는 문제 수정
## v4.8.6
### 수정

- boost/context API 이름에 접두사 추가
- 구성 옵션 최적화
## v4.8.5
### 수리

- 테이블의 매개변수 유형을 복원합니다.
- 잘못된 데이터를 받을 때 웹소켓 프로토콜을 사용하여 크래시되는 문제를 수정합니다.
## v4.8.4
### 수정 사항

- PHP-8.1과의 호환성을 갖기 위해 소켓 후크 수정
- PHP-8.1과의 호환성을 갖기 위해 Table 수정
- 일부 상황에서 코루틴 스타일의 HTTP 서버가 "Content-Type"을 "application/x-www-form-urlencoded"로 잘못 구문 분석하는 것을 수정
## v4.8.3
### 새로운 API

- `Coroutine\Socket::isClosed()` 메소드 추가
### 수정사항

- PHP 8.1 버전에서 curl native hook의 호환성 문제를 수정했습니다.
- PHP 8 버전에서 sockets hook의 호환성 문제를 수정했습니다.
- sockets hook 함수의 반환 값을 수정했습니다.
- Http2Server sendfile에서 content-type을 설정할 수 없는 문제를 수정했습니다.
- HttpServer date 헤더의 성능을 최적화하고 캐시를 추가했습니다.
## v4.8.2
### 수정

- `proc_open` 후킹에서의 메모리 누수 문제 수정
- `curl native` 후킹과 PHP-8.0, PHP-8.1의 호환성 문제 수정
- Manager 프로세스에서 연결을 정상적으로 닫을 수 없는 문제 수정
- Manager 프로세스에서 `sendMessage`를 사용할 수 없는 문제 수정
- `Coroutine\Http\Server`가 매우 큰 POST 데이터를 받을 때 발생하는 구문 분석 오류 문제 수정
- PHP8 환경에서 치명적인 오류가 발생할 때 직접 종료되지 않는 문제 수정
- `coroutine max_concurrency` 구성 항목 조정, `Co::set()`에서만 사용할 수 있도록 함
- `Coroutine::join()`에서 존재하지 않는 코루틴을 무시하도록 조정
## v4.8.1
### 새로운 API

- `swoole_error_log_ex()` 및 `swoole_ignore_error()` 함수 추가 (#4440) (@matyhtf)
# 버전 업데이트 기록

엄격한 버전 업데이트 기록이 `v1.5` 버전부터 시작되었습니다. 현재 평균 반년마다 대규모 업데이트 및 2주에서 4주마다 소규모 업데이트가 이루어지고 있습니다.
## 권장하는 PHP 버전

* 7.4
* 8.0
* 8.1
* 8.2
## 권장되는 Swoole 버전
`Swoole5.x` 및 `Swoole4.8.x`

두 버전의 차이는 다음과 같습니다: `v5.x`는 주요 이터레이션 브랜치이며, `v4.8.x`는 주요 이터레이션 브랜치가 아닌 버그만 수정하는 브랜치입니다.

!> 버전 `v4.x` 이상은 [enable_coroutine](/server/setting?id=enable_coroutine)을 설정하여 코루틴 기능을 비활성화하여 코루틴 버전이 아니게 할 수 있습니다.
## 버전 유형

* `alpha` : 기능 미리보기 버전으로, 개발 일정에있는 작업이 완료되어 공개 미리보기를 진행하며, 많은 `버그`가 있을 수 있습니다.
* `beta` : 테스트 버전으로, 개발 환경 테스트에 사용할 수 있으며, `버그`가 있을 수 있습니다.
* `rc[1-n]` : 릴리스 후보 버전으로, 출시 주기에 진입하여 광범위한 테스트를 진행하며, 이 기간 동안 `버그`가 발견 될 수 있습니다.
* 접미사가 없는 경우 안정된 버전을 의미하며, 이 버전은 완전히 개발되어 공식적으로 사용할 수 있음을 나타냅니다.
## 현재 버전 정보 확인

```shell
php --ri swoole
```
## v6.0.0
- 다중 스레드 모드 지원 추가
- `Coroutine\Redis`、`Coroutine\MySQL` 및 `Coroutine\PostgreSQL` 클라이언트 제거되었으며, `ext-redis`、`mysqli`、`pdo_mysql` 및 `pdo_pgsql`로 대체되었습니다
## v5.1.0
### 새로운 기능
- `pdo_pgsql`의 코루틴 지원 추가
- `pdo_odbc`의 코루틴 지원 추가
- `pdo_oci`의 코루틴 지원 추가
- `pdo_sqlite`의 코루틴 지원 추가
- `pdo_pgsql` , `pdo_odbc` , `pdo_oci` , `pdo_sqlite`의 연결 풀 구성
### 향상
- `Http\Server`의 성능을 개선했으며, 극한 상황에서 최대 `60%` 향상될 수 있습니다.
### 수정 사항
- `WebSocket` 코루틴 클라이언트가 각 요청마다 발생하는 메모리 누수 수정
- `http 코루틴 서버` 우아한 종료로 인한 클라이언트 종료 문제 수정
- `--enable-thread-context` 옵션을 사용하여 컴파일 할 때 `Process::signal()`이 작동하지 않는 문제 수정
- 비정상 종료 시 `SWOOLE_BASE` 모드에서 연결 수 통계 오류 수정
- `stream_select()` 함수 서명 오류 수정
- 파일 MIME 정보 대소문자 구분 수정
- `Http2\Request::$usePipelineRead`의 철자 오류 수정으로 PHP 8.2 환경에서 경고가 발생하는 문제 수정
- `SWOOLE_BASE` 모드에서의 메모리 누수 문제 수정
- `Http\Response::cookie()`에서 쿠키 만료 시간을 설정할 때 발생하는 메모리 누수 문제 수정
- `SWOOLE_BASE` 모드에서의 연결 누출 문제 수정
### 커널
- PHP 8.3에서 swoole의 `php_url_encode` 함수 서명 문제를 수정했습니다.
- 단위 테스트 옵션 문제를 수정했습니다.
- 코드를 최적화하고 재구성했습니다.
- PHP 8.3과 호환되도록 했습니다.
- 32비트 운영 체제에서의 컴파일은 지원하지 않습니다.
##  v5.0.3
### 강화
- `--with-nghttp2_dir` 옵션 추가되었으며, 시스템의 `nghttp2` 라이브러리 사용 가능
- 바이트 길이 또는 크기 관련 옵션 지원
- `Process\Pool::sendMessage()` 함수 추가
- `Http\Response:cookie()`에서 `max-age` 지원
### 수정
- `Server task/pipemessage/finish` 이벤트로 인한 메모리 누수 수정
### 내부 핵심
- `http` 응닑 헤더 충돌시 오류가 발생하지 않습니다
- `Server` 연결 종료시 오류가 발생하지 않습니다
## v5.0.2
### 강화
- `http2`의 기본 설정을 구성할 수 있도록 지원합니다.
- 8.1 버전 이상의 `xdebug`를 지원합니다.
- 여러 소켓을 가진 curl 핸들러를 지원하기 위해 기본 curl을 재구성했습니다. 예를 들어 curl ftp 프로토콜입니다.
- `Process::setPriority/getPriority`에 `who` 매개변수를 추가했습니다.
- `Coroutine\Socket::getBoundCid()` 메서드를 추가했습니다.
- `Coroutine\Socket::recvLine/recvWithBuffer` 메서드의 `length` 매개변수의 기본값을 `65536`으로 조정했습니다.
- 교차 코루틴 종료 기능을 재구성하여 메모리 해제를 더 안전하게 하고 치명적인 오류 발생 시 충돌 문제를 해결했습니다.
- `Coroutine\Client`, `Coroutine\Http\Client`, `Coroutine\Http2\Client`에 `socket` 속성을 추가하여 `socket` 리소스를 직접 조작할 수 있습니다.
- `Http\Server`가 `http2` 클라이언트로 빈 파일을 보낼 수 있도록 지원합니다.
- `Coroutine\Http\Server`의 우아한 재시작을 지원합니다. 서버가 종료될 때 클라이언트 연결은 더 이상 강제로 종료되지 않고 새 요청 수신만 중지됩니다.
- `pcntl_rfork`와 `pcntl_sigwaitinfo`를 안전하지 않은 함수 목록에 추가했고, 코루틴 컨테이너가 시작될 때 닫힐 것입니다.
- `SWOOLE_BASE` 모드의 프로세스 관리자를 재구성하여 종료 및 다시로드 동작을 `SWOOLE_PROCESS`와 일관되게 합니다.
##  v5.0.1 
- `PHP-8.2`을 지원하여, 코루틴 예외 처리를 개선하고 `ext-soap`과 호환되도록 함
- `pgsql` 코루틴 클라이언트의 `LOB` 지원 추가
- 웹소켓 클라이언트를 개선하여, 헤더에 `websocket`을 사용하고 `=` 대신 사용
- 서버에서 `connection close`을 보낼 때 `http 클라이언트`를 최적화하여 `keep-alive` 비활성화
- 압축 라이브러리가 없을 때 `http 클라이언트`를 최적화하여 `Accept-Encoding` 헤더 추가를 금지
- 디버그 정보를 개선하여, `PHP-8.2`에서는 비밀번호를 민감한 매개변수로 설정
- `Server::taskWaitMulti()`를 강화하여, 코루틴 환경에서 블로킹이 발생하지 않도록 함
- 로그 기능을 최적화하여, 로그 파일 작성 실패 시 화면에 더 이상 출력하지 않음
### 수정
- `Coroutine::printBackTrace()` 및 `debug_print_backtrace()`의 매개변수 호환성 문제가 수정되었습니다.
- `Event::add()`에서 소켓 자원 지원이 수정되었습니다.
- `zlib`가 없는 상태에서의 컴파일 오류가 수정되었습니다.
- 잘못된 문자열로 구문 분석된 경우 Unpack 서버 작업의 충돌 문제가 수정되었습니다.
- `1ms`보다 작은 타이머를 추가하면 강제로 `0`으로 설정되는 문제가 수정되었습니다.
- `Table::getMemorySize()`를 사용하여 열을 추가하기 전에 발생하는 충돌 문제가 수정되었습니다.
- `Http\Response::setCookie()` 메서드의 만료 매개변수 이름이 `expires`로 변경되었습니다.
## V5.0.0
### 새로운 기능
- `Server`의 `max_concurrency` 옵션을 추가했습니다.
- `Coroutine\Http\Client`의 `max_retries` 옵션을 추가했습니다.
- `name_resolver` 전역 옵션을 추가했습니다. `Server`에 `upload_max_filesize` 옵션을 추가했습니다.
- `Coroutine::getExecuteTime()` 메소드를 추가했습니다.
- `Server`에 `SWOOLE_DISPATCH_CONCURRENT_LB`의 `dispatch_mode`를 추가했습니다.
- 모든 함수의 매개변수와 반환값에 타입을 추가하여 유형 시스템을 강화했습니다.
- 오류 처리를 최적화했으며, 모든 생성자는 실패할 경우 예외를 throw합니다.
- `Server`의 기본 모드를 조정하여 기본값이 `SWOOLE_BASE` 모드로 변경되었습니다.
- `pgsql` 코루틴 클라이언트를 코어 라이브러리로 이동했습니다. `4.8.x` 브랜치의 모든 `bug` 수정이 포함되었습니다.
### 삭제
- `PSR-0` 스타일의 클래스 이름이 제거되었습니다.
- 콜백 함수에서 자동으로 `Event::wait()`를 추가하는 기능이 제거되었습니다.
- `Server::tick/after/clearTimer/defer`의 별칭이 제거되었습니다.
- `--enable-http2/--enable-swoole-json`이 제거되었고, 기본적으로 활성화되어 있습니다.
### 폐기

- 코루틴 클라이언트 `Coroutine\Redis` 및 `Coroutine\MySQL`는 기본적으로 폐기됩니다.
## v4.8.13
- 오리지널 curl을 리팩토링하여 여러 소켓을 가진 curl 핸들을 지원하도록 함, 예를들어 curl FTP 프로토콜
- 수동으로 `http2` 설정 설정 가능
- `WebSocket 클라이언트` 개선, 헤더 업그레이드로 `웹소켓` 추가而不是 `equal`
- 서버가 연결을 종료할 때 `keep-alive`를 비활성화하도록 HTTP 클라이언트 최적화
- 디버깅 정보를 개선하여 PHP-8.2에서 비밀번호를 민감한 매개변수로 설정
- `HTTP 범위 요청` 지원
### 수정
- `Coroutine::printBackTrace()` 및 `debug_print_backtrace()`의 매개변수 호환성 문제가 수정되었습니다.
- `WebSocket` 서버에서 `HTTP2` 및 `WebSocket` 프로토콜을 동시에 활성화할 때 길이를 잘못 구문 분석하는 문제가 수정되었습니다.
- `Server::send()`, `Http\Response::end()`, `Http\Response::write()` 및 `WebSocket/Server::push()`에서 `send_yield` 시 메모리 누수 문제가 수정되었습니다.
- `Table::getMemorySize()`를 사용하기 전에 열을 추가하면 충돌이 발생하는 문제가 수정되었습니다.
## 4.8.12
### 향상된 내용
- PHP8.2를 지원합니다.
- `Event::add()` 함수가 `sockets resources`를 지원합니다.
- `Http\Client::sendfile()`이 4GB 이상의 파일을 지원합니다.
- `Server::taskWaitMulti()`가 코루틴 환경을 지원합니다.
### 수정 사항
- 잘못된 `multipart body`를 수신하면 오류 메시지가 발생하는 문제를 수정했습니다.
- 타이머의 시간이 `1ms` 미만으로 설정된 경우 오류가 발생하는 문제를 수정했습니다.
- 디스크 공간이 가득 차서 발생하는 데드락 문제를 수정했습니다.
## v4.8.11
### 강화
- `Intel CET` 보안 방어 메커니즘 지원
- `Server::$ssl` 속성 추가
- `pecl`로 `swoole` 컴파일 시 `enable-cares` 속성 추가
- `multipart_parser` 해석기 재구성
### 수정
- `pdo` 지속 연결로 인한 예외로 인한 세그멘테이션 오류 수정
- 코루틴을 사용하여 소멸자가 세그멘테이션 오류를 일으키는 문제 수정
- `Server::close()`의 잘못된 오류 메시지 수정
## v4.8.10
### 修复

- `stream_select`의 타임아웃 매개변수가 `1ms` 미만일 때, 이를 `0`으로 재설정합니다.
- 컴파일 시 ` -Werror=format-security` 추가 시 컴파일 실패 문제를 수정하였습니다.
- `curl`을 사용하여 `Swoole\Coroutine\Http\Server`가 세그멘테이션 오류를 일으키는 문제가 수정되었습니다.
## v4.8.9
### 향상

- `Http2` 서버에서 `http_auto_index` 옵션을 지원합니다.
### 수정

- `Cookie` 파서 최적화, `HttpOnly` 옵션 지원
- #4657 수정, `socket_create` 메서드 반환 타입 문제 수정
- `stream_select` 메모리 누수 수정
### CLI 업데이트

- `CygWin`은 SSL 인증서 체인을 함께 제공하여 SSL 인증 오류를 해결했습니다.
- `PHP-8.1.5`로 업데이트되었습니다.
## v4.8.8
### 최적화

- SW_IPC_BUFFER_MAX_SIZE를 64k로 줄이기
- http2의 header_table_size를 최적화하십시오.
### 수정사항

- enable_static_handler를 사용하여 정적 파일을 다운로드할 때 소켓 오류가 발생하는 문제를 수정했습니다.
- http2 서버 NPN 오류를 수정했습니다.
## v4.8.7
### 강화

- curl_share 지원 추가
### 수정사항

- arm32 아키텍처에서 정의되지 않은 기호 오류 수정
- `clock_gettime()` 호환성 수정
- 커널이 대량 메모리를 부족하게 하여 발생하는 경우, PROCESS 모드 서버를 보내지 못하는 문제 수정
## v4.8.6
### 修复

- 为 boost/context API 名称添加了前缀
- 优化配置选项
## v4.8.5
### 수정

- 테이블의 매개변수 유형 복원
- 잘못된 데이터를 수신할 때 웹소켓 프로토콜 사용 시 충돌 수정
## v4.8.4
### 수정

- sockets hook와 PHP-8.1의 호환성 문제 해결
- Table과 PHP-8.1의 호환성 문제 해결
- 일부 경우에 코루틴 스타일의 HTTP 서버에서 `Content-Type`이 `application/x-www-form-urlencoded`로 표시된 `POST` 매개변수가 예상과 일치하지 않는 문제를 해결
## v4.8.3
### 새 API

- `Coroutine\Socket::isClosed()` 메서드 추가
### 수정

- PHP8.1 버전에서 curl native hook의 호환성 문제 수정
- PHP8에서 sockets hook의 호환성 문제 수정
- 올바른 값을 반환하지 않는 sockets hook 함수 수정
- Http2Server sendfile이 content-type을 설정할 수 없는 문제 수정
- HttpServer의 date header 성능을 최적화하여 캐시를 추가함
## v4.8.2  
- `proc_open` 후크 메모리 누수 문제 수정
- curl 네이티브 후크와 PHP-8.0 및 PHP-8.1의 호환성 문제 수정
- Manager 프로세스에서 연결을 정상적으로 닫을 수 없는 문제 수정
- Manager 프로세스에서 `sendMessage`를 사용할 수 없는 문제 수정
- `Coroutine\Http\Server`에서 초대형 POST 데이터 구문 분석 예외 처리 문제 수정
- PHP8 환경에서 치명적인 오류 발생 시 직접 종료되지 않는 문제 수정
- coroutine `max_concurrency` 구성 항목 조정, `Co::set()`에서만 사용 허용
- `Coroutine::join()`에서 없는 코루틴을 무시하도록 조정
## v4.8.1
### 새로운 API

- `swoole_error_log_ex()` 및 `swoole_ignore_error()` 함수가 추가되었습니다 (#4440) (@matyhtf)
### 강화

- ext-swoole_plus의 admin api를 ext-swoole로 이전(#4441) (@matyhtf)
- admin server에 get_composer_packages 명령어 추가 (swoole/library@07763f46) (swoole/library@8805dc05) (swoole/library@175f1797) (@sy-records) (@yunbaoi)
- 쓰기 작업에 대한 POST 방법 요청 제한이 추가되었습니다 (swoole/library@ac16927c) (@yunbaoi)
- admin server가 클래스 메소드 정보를 가져오는 것을 지원합니다 (swoole/library@690a1952) (@djw1028769140) (@sy-records)
- admin server 코드를 최적화했습니다 (swoole/library#128) (swoole/library#131) (@sy-records)
- admin server가 여러 대상 및 다중 API에 대한 병렬 요청을 지원합니다 (swoole/library#124) (@sy-records)
- admin server가 인터페이스 정보를 가져오는 것을 지원합니다 (swoole/library#130) (@sy-records)
- SWOOLE_HOOK_CURL이 CURLOPT_HTTPPROXYTUNNEL을 지원합니다 (swoole/library#126) (@sy-records)
### 修复

- join 方法禁止并发调用同一个协程 (#4442) (@matyhtf)
- 修复 Table 原子锁意外释放的问题 (#4446) (@Txhua) (@matyhtf)
- 修复丢失的 helper options (swoole/library#123) (@sy-records)
- 修复 get_static_property_value 命令参数错误 (swoole/library#129) (@sy-records)
## v4.8.0
### 하위 호환성 변경

- 기본 모드에서 onStart 콜백은 항상 첫 번째 작업 프로세스(worker id 0) 실행 시에 호출되며 onWorkerStart보다 먼저 실행됩니다. (#4389) (@matyhtf)
### 새로운 API

- `Co::getStackUsage()` 메서드 추가 (#4398) (@matyhtf) (@twose)
- `Coroutine\Redis`에 몇 가지 API 추가 (#4390) (@chrysanthemum)
- `Table::stats()` 메서드 추가 (#4405) (@matyhtf)
- `Coroutine::join()` 메서드 추가 (#4406) (@matyhtf)
### 새로운 기능

- 서버 명령 지원 (#4389) (@matyhtf)
- `Server::onBeforeShutdown` 이벤트 콜백 지원 (#4415) (@matyhtf)
- 웹 소켓 패킷이 실패할 때 오류 코드를 설정합니다 (swoole/swoole-src@d27c5a5) (@matyhtf)
- `Timer::exec_count` 필드를 추가했습니다 (#4402) (@matyhtf)
- `mkdir` 후킹이 open_basedir ini 설정을 지원합니다 (#4407) (@NathanFreeman)
- 라이브러리에 vendor_init.php 스크립트를 추가했습니다 (swoole/library@6c40b02) (@matyhtf)
- SWOOLE_HOOK_CURL이 CURLOPT_UNIX_SOCKET_PATH를 지원합니다 (swoole/library#121) (@sy-records)
- 클라이언트가 ssl_ciphers 구성을 설정할 수 있습니다 (#4432) (@amuluowin)
- `Server::stats()`에 일부 새로운 정보를 추가했습니다 (#4410) (#4412) (@matyhtf)
### 修复

- 修复文件上传时，对文件名字进行不必要的 URL decode (swoole/swoole-src@a73780e) (@matyhtf)
- 修复 HTTP2 max_frame_size 问题 (#4394) (@twose)
- 修复 curl_multi_select bug #4393 (#4418) (@matyhtf)
- 修复丢失的 coroutine options (#4425) (@sy-records)
- 修复当发送缓冲区满的时候，连接无法被 close 的问题 (swoole/swoole-src@2198378) (@matyhtf)
## 버전 4.7.1
- `System::dnsLookup`는 `/etc/hosts`를 쿼리할 수 있습니다. (#4341) (#4349) (@zmyWL) (@NathanFreeman)
- mips64의 boost context 지원이 추가되었습니다. (#4358) (@dixyes)
- `SWOOLE_HOOK_CURL`이 `CURLOPT_RESOLVE` 옵션을 지원합니다. (swoole/library#107) (@sy-records)
- `SWOOLE_HOOK_CURL`이 `CURLOPT_NOPROGRESS` 옵션을 지원합니다. (swoole/library#117) (@sy-records)
- riscv64의 boost context 지원이 추가되었습니다. (#4375) (@dixyes)
### 수정

- PHP-8.1이 종료될 때 발생하는 메모리 오류를 수정했습니다. (#4325) (@twose)
- 8.1.0beta1에서 직렬화할 수 없는 클래스를 수정했습니다. (#4335) (@remicollet)
- 여러 개의 코루틴이 재귀적으로 디렉토리를 만들지 못하는 문제를 수정했습니다. (#4337) (@NathanFreeman)
- 외부로 큰 파일을 전송할 때 가끔 발생하는 native curl의 시간 초과 문제와 CURL WRITEFUNCTION에서 코루틴 파일 API를 사용하면 충돌하는 문제를 수정했습니다. (#4360) (@matyhtf)
- `PDOStatement::bindParam()`에서 첫 번째 매개변수가 문자열을 예상했던 문제를 수정했습니다. (swoole/library#116) (@sy-records)
## v4.7.0
### 새로운 API

- `Process\Pool::detach()` 메소드 추가 (#4221) (@matyhtf)
- `Server`가 `onDisconnect` 콜백 함수를 지원합니다. (#4230) (@matyhtf)
- `Coroutine::cancel()` 및 `Coroutine::isCanceled()` 메소드 추가 (#4247) (#4249) (@matyhtf)
- `Http\Client`가 `http_compression` 및 `body_decompression` 옵션을 지원합니다. (#4299) (@matyhtf)
- `prepare`할 때 엄격한 유형의 필드가있는 코루틴 MySQL 클라이언트 지원 (#4238) (@Yurunsoft)
- DNS는 `c-ares` 라이브러리를 지원합니다. (#4275) (@matyhtf)
- `Server`는 다른 포트에 대해 다른 포트에 대한 하트 비트 검사 시간을 구성 할 수 있습니다. (#4290) (@matyhtf)
- `Server`의 `dispatch_mode`는 `SWOOLE_DISPATCH_CO_CONN_LB` 및 `SWOOLE_DISPATCH_CO_REQ_LB` 모드를 지원합니다. (#4318) (@matyhtf)
- `ConnectionPool::get()`은 `timeout` 매개 변수를 지원합니다. (swoole/library#108) (@leocavalcante)
- 후크 Curl은 `CURLOPT_PRIVATE` 옵션을 지원합니다. (swoole/library#112) (@sy-records)
- `PDOStatementProxy::setFetchMode()` 메소드의 함수 선언을 최적화했습니다. (swoole/library#109) (@yespire)
- 스레드 컨텍스트를 사용할 때 대량의 코루틴을 생성하면 스레드를 생성할 수 없는 예외가 발생하는 문제를 수정했습니다. (@matyhtf)
- Swoole을 설치할 때 php_swoole.h 헤더 파일이 누락되는 문제를 수정했습니다. (@sy-records)
- EVENT_HANDSHAKE의 하위 호환성 문제를 수정했습니다. (@sy-records)
- SW_LOCK_CHECK_RETURN 매크로가 함수를 두 번 호출할 수 있는 문제를 수정했습니다. (@zmyWL)
- M1 칩에서의 `Atomic\Long` 문제를 수정했습니다. (@matyhtf)
- `Coroutine\go()`에서 반환 값이 누락되는 문제를 수정했습니다. (@matyhtf)
- `StringObject`의 반환 값 유형 문제를 수정했습니다. (@leocavalcante) (@sy-records)
### 내커널

- PHP에서 이미 비활성화된 함수를 후킹하는 것이 금지되었습니다. (#4283) (@twose)
### 테스트

- `Cygwin` 환경 빌드 추가 (#4222) (@sy-records)
- `alpine 3.13` 및 `3.14`에서의 컴파일 테스트 추가 (#4309) (@limingxinleo)
## v4.6.7
### 강화

- 관리자 프로세스와 작업 동기화 프로세스에 `Process::signal()` 함수 호출 지원 (#4190) (@matyhtf)
### 修复

- 修复信号不能被重复注册的问题 (#4170) (@matyhtf)
- 修复在 OpenBSD/NetBSD 上编译失败的问题 (#4188) (#4194) (@devnexen)
- 修复监听可写事件时特殊情况 onClose 事件丢失 (#4204) (@matyhtf)
- 修复 Symfony HttpClient 使用 native curl 的问题 (#4204) (@matyhtf)
- 修复`Http\Response::end()`方法总是返回 true 的问题 (swoole/swoole-src@66fcc35) (@matyhtf)
- 修复 PDOStatementProxy 产生的 PDOException (swoole/library#104) (@twose)
### 내부 커널

- Worker 버퍼를 다시 구성하여 이벤트 데이터에 메시지 ID 플래그를 추가합니다 #4163 (@matyhtf)
- 요청 엔터티가 너무 크다는 로그 수준을 경고 수준으로 변경합니다 (#4175) (@sy-records)
- inet_ntoa 및 inet_aton 함수를 대체합니다. (#4199) (@remicollet)
- output_buffer_size의 기본값을 UINT_MAX로 변경합니다 (swoole/swoole-src@46ab345) (@matyhtf)
## v4.6.6
### 강화

- FreeBSD에서 Master 프로세스가 종료된 후 Manager 프로세스로 SIGTERM 신호를 보낼 수 있도록 지원 (#4150) (@devnexen)
- Swoole을 PHP에 정적으로 컴파일할 수 있도록 지원 (#4153) (@matyhtf)
- SNI이 HTTP 프록시를 사용하도록 지원 (#4158) (@matyhtf)
### 수정 사항

- 동기 클라이언트 비동기 연결 오류 수정 (#4152) (@matyhtf)
- Hook으로 인한 원본 curl multi로 인한 메모리 누수 수정 (swoole/swoole-src@91bf243) (@matyhtf)
## v4.6.5
### 신규 API

- WaitGroup에 count 메서드 추가 (swoole/library#100) (@sy-records) (@deminy)
### 향상

- 기본 curl multi 지원 (#4093) (#4099) (#4101) (#4105) (#4113) (#4121) (#4147) (swoole/swoole-src@cd7f51c) (@matyhtf) (@sy-records) (@huanghantao)
- HTTP/2 Response에 배열을 사용하여 헤더를 설정할 수 있음
### 수정

- NetBSD 빌드 수정(#4080) (@devnexen)
- OpenBSD 빌드 수정(#4108) (@devnexen)
- illumos/solaris 빌드 수정, 멤버 별칭만(#4109) (@devnexen)
- 핸드셰이크가 미완료일 때 SSL 연결의 하트비트 감지 수정(#4114) (@matyhtf)
- 프록시 사용 중 일부 오류 발생 수정`host`에 `host:port`가 있는 경우 Http\Client(#4124) (@Yurunsoft)
- Swoole\Coroutine\Http::request에서 header 및 cookie 설정 수정(swoole/library#103) (@leocavalcante) (@deminy)
### 내부

- BSD에서 asm context 지원 (#4082) (@devnexen)
- FreeBSD에서 arc4random_buf를 사용하여 getrandom 구현 (#4096) (@devnexen)
- darwin arm64 context 최적화: label workaround 삭제 (#4127) (@devnexen)
### 테스트

- 알파인 빌드 스크립트 추가 (#4104) (@limingxinleo)
## v4.6.4
### 새로운 API

- Coroutine\Http::request, Coroutine\Http::post, Coroutine\Http::get 함수 추가 (swoole/library#97) (@matyhtf)
### 강화

- ARM 64 빌드 지원 (#4057) (@devnexen)
- Swoole TCP 서버에서 open_http_protocol 설정 지원 (#4063) (@matyhtf)
- ssl 클라이언트 인증서만 설정하는 기능 추가 (91704ac) (@matyhtf)
- FreeBSD의 tcp_defer_accept 옵션 지원 (#4049) (@devnexen)
- Coroutine\Http\Client를 사용할 때 프록시 권한이 누락된 문제를 수정했습니다 (edc0552) (@matyhtf)
- Swoole\Table의 메모리 할당 문제를 수정했습니다 (3e7770f) (@matyhtf)
- Coroutine\Http2\Client의 동시 연결 중 충돌을 수정했습니다 (630536d) (@matyhtf)
- DTLS의 enable_ssl_encrypt 문제를 수정했습니다 (842733b) (@matyhtf)
- Coroutine\Barrier의 메모리 누수를 수정했습니다 (swoole/library#94) (@Appla) (@FMiS)
- CURLOPT_PORT와 CURLOPT_URL 순서로 인한 오프셋 오류를 수정했습니다 (swoole/library#96) (@sy-records)
- `Table::get($key, $field)`에서 필드 유형이 float일 때의 오류를 수정했습니다 (08ea20c) (@matyhtf)
- Swoole\Table의 메모리 누수를 수정했습니다 (d78ca8c) (@matyhtf)
## v4.4.24
### 수리

- http2 클라이언트의 동시 연결 시 crash 수정 (#4079)
## v4.6.3
### 새 API 추가

- Swoole\Coroutine\go 함수 추가 (swoole/library@82f63be) (@matyhtf)
- Swoole\Coroutine\defer 함수 추가 (swoole/library@92fd0de) (@matyhtf)
### 강화

- HTTP 서버에 compression_min_length 옵션 추가 (#4033) (@matyhtf)
- 응용 계층에서 Content-Length HTTP 헤더를 설정할 수 있도록 함 (#4041) (@doubaokun)
### 수정

- 파일 오픈 제한에 도달할 때 coredump를 방지하는 프로그램 수정 (swoole/swoole-src@709813f) (@matyhtf)
- JIT 비활성화 문제 수정 (#4029) (@twose)
- `Response::create()` 매개변수 오류 수정 (swoole/swoole-src@a630b5b) (@matyhtf)
- ARM 플랫폼에서 task_worker_id가 잘못 보고되는 문제 수정 (#4040) (@doubaokun)
- 네이티브 curl 후킹이 활성화된 PHP8에서 coredump를 방지하는 수정 (#4042)(#4045) (@Yurunsoft) (@matyhtf)
- 치명적인 오류 시 shutdown 단계에서의 메모리 오버런 오류 수정 (#4050) (@matyhtf)
### 내커널

- ssl_connect/ssl_shutdown 최적화 (#4030) (@matyhtf)
- 치명적인 오류 발생 시 프로세스를 직접 종료합니다. (#4053) (@matyhtf)
## v4.6.2
### 새로운 API

- `Http\Request\getMethod()` 메소드를 추가했습니다 (#3987) (@luolaifa000)
- `Coroutine\Socket->recvLine()` 메소드를 추가했습니다 (#4014) (@matyhtf)
- `Coroutine\Socket->readWithBuffer()` 메소드를 추가했습니다 (#4017) (@matyhtf)
### 강화

- `Response\create()` 메서드를 강화하여 Server와 독립적으로 사용할 수 있도록 함 (#3998) (@matyhtf)
- `Coroutine\Redis->hExists`에서 compatibility_mode를 설정한 후 bool 타입을 반환하도록 지원 (swoole/swoole-src@b8cce7c) (@matyhtf)
- `socket_read`에서 PHP_NORMAL_READ 옵션을 설정하는 기능을 지원 (swoole/swoole-src@b1a0dcc) (@matyhtf)
### 수정사항

- PHP8에서 `Coroutine::defer`가 coredump되는 문제를 수정했습니다 (#3997) (@huanghantao)
- 스레드 컨텍스트를 사용할 때 `Coroutine\Socket::errCode`를 잘못 설정하는 문제를 수정했습니다 (swoole/swoole-src@004d08a) (@matyhtf)
- 최신 macOS에서 Swoole이 컴파일되지 않는 문제를 수정했습니다 (#4007) (@matyhtf)
- md5_file 매개변수로 url을 전달할 때 php stream context가 null 포인터가 되는 문제를 수정했습니다 (#4016) (@ZhiyangLeeCN)
### 내부 커널

- AIO 스레드 풀을 사용하여 stdio 훅 설정(stdio를 소켓으로 인식하여 발생한 여러 코루틴 읽기/쓰기 문제 해결) (#4002) (@matyhtf)
- HttpContext를 재구성하였습니다. (#3998) (@matyhtf)
- `Process::wait()`를 재구성하였습니다. (#4019) (@matyhtf)
## v4.6.1
### 향상

- `--enable-thread-context` 컴파일 옵션 추가 (#3970) (@matyhtf)
- 세션 ID 조작 시 연결의 존재 여부 확인 (#3993) (@matyhtf)
- CURLOPT_PROXY 강화 (swoole/library#87) (@sy-records)
### 修复

- 修复 pecl 安装中的最小 PHP 版本 (#3979) (@remicollet)
- 修复 pecl 安装时没有 `--enable-swoole-json` 和 `--enable-swoole-curl` 选项 (#3980) (@sy-records)
- 修复 openssl 线程安全问题 (b516d69f) (@matyhtf)
- 修复 enableSSL coredump (#3990) (@huanghantao)
### 내커널

- 이벤트 데이터가 비어 있을 때 코어 덤프가 발생하는 것을 피하기 위해 ipc writev를 최적화했습니다. (9647678) (@matyhtf)
## v4.5.11
### 강화

- Swoole\Table 최적화 (#3959) (@matyhtf)
- CURLOPT_PROXY 강화 (swoole/library#87) (@sy-records)
### 수정 사항

- 테이블 증가 및 감소시 모든 열을 지울 수 없는 문제 수정 (#3956) (@matyhtf) (@sy-records)
- 컴파일시 발생하는 'clock_id_t' 오류 수정 (49fea171) (@matyhtf)
- fread 버그 수정 (#3972) (@matyhtf)
- ssl 다중 스레드 충돌 수정 (7ee2c1a0) (@matyhtf)
- 잘못된 uri 형식으로 인해 발생하는 foreach에 대한 제공된 인수가 잘못된 오류 수정 (swoole/library#80) (@sy-records)
- trigger_error 매개변수 오류 수정 (swoole/library#86) (@sy-records)
## v4.6.0
### 하위 호환성이 없는 변경 사항

- `세션 id`의 최대 제한이 제거되어 더 이상 새로운 값이 반복되지 않습니다. (#3879) (@matyhtf)
- 코루틴 사용 시 `pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`을 비활성화하십시오. (#3880) (@matyhtf)
- 기본적으로 코루틴 후크가 활성화됩니다. (#3903) (@matyhtf)
### 제거

- PHP7.1을 더 이상 지원하지 않습니다. (4a963df) (9de8d9e) (@matyhtf)
### 폐기됨

-`Event::rshutdown()`가 deprecated로 표시되었습니다. 대신 `Coroutine\run`을 사용하세요. (#3881) (@matyhtf)
- 새로운 API 추가

- setPriority/getPriority 기능 지원 (#3876) (@matyhtf)
- native-curl hook 지원 (#3863) (@matyhtf) (@huanghantao)
- 서버 이벤트 콜백 함수가 객체 스타일의 매개변수 전달 지원, 기본적으로 객체 스타일의 매개변수 전달 하지 않음 (#3888) (@matyhtf)
- hook sockets 확장 지원 (#3898) (@matyhtf)
- 중복 헤더 지원 (#3905) (@matyhtf)
- SSL sni 지원 (#3908) (@matyhtf)
- hook stdio 지원 (#3924) (@matyhtf)
- stream_socket의 capture_peer_cert 옵션 지원 (#3930) (@matyhtf)
- Http\Request::create/parse/isCompleted 추가 (#3938) (@matyhtf)
- Http\Response::isWritable 추가 (db56827) (@matyhtf)
### 강화

- Server의 모든 시간 정밀도를 int에서 double로 변경 (#3882) (@matyhtf)
- swoole_client_select 함수에서 poll 함수의 EINTR 상황을 확인합니다. (#3909) (@shiguangqi)
- 코루틴 데드락 감지 기능 추가 (#3911) (@matyhtf)
- 다른 프로세스에서 SWOOLE_BASE 모드로 연결을 닫을 수 있도록 지원 (#3916) (@matyhtf)
- Server 마스터 프로세스와 워커 프로세스 간 통신 성능을 최적화하여 메모리 복사를 줄임 (#3910) (@huanghantao) (@matyhtf)
### 수정

- Coroutine\Channel이 닫혔을 때 내부 데이터를 모두 pop합니다 (960431d) (@matyhtf)
- JIT 사용 시 메모리 오류를 수정했습니다 (#3907) (@twose)
- `port->set()` dtls 컴파일 오류를 수정했습니다 (#3947) (@Yurunsoft)
- connection_list 오류를 수정했습니다 (#3948) (@sy-records)
- SSL 확인을 수정했습니다 (#3954) (@matyhtf)
- Table 증가 및 감소 시 모든 열을 지울 수 없는 문제를 수정했습니다 (#3956) (@matyhtf) (@sy-records)
- LibreSSL 2.7.5로 빌드 실패를 수정했습니다 (#3962) (@matyhtf)
- 정의되지 않은 상수 CURLOPT_HEADEROPT 및 CURLOPT_PROXYHEADER를 수정했습니다 (swoole/library#77) (@sy-records)
### 내부

- 기본적으로 SIGPIPE 신호를 무시합니다 (9647678) (@matyhtf)
- PHP 코루틴 및 C 코루틴을 동시에 실행하는 경우 지원됩니다 (c94bfd8) (@matyhtf)
- get_elapsed 테스트 추가 (#3961) (@luolaifa000)
- get_init_msec 테스트 추가 (#3964) (@luffluo)
## v4.5.10
### 修复

- 修复使用 Event::cycle 时产生的 coredump (93901dc) (@matyhtf)
- 兼容 PHP8 (f0dc6d3) (@matyhtf)
- 修复 connection_list 错误 (#3948) (@sy-records)
## v4.4.23
### 수정

- Swoole\Table 감소 시 데이터 오류 수정 (bcd4f60d)(0d5e72e7) (@matyhtf)
- 동기 클라이언트 오류 메시지 수정 (#3784)
- 폼 데이터 경계 해석시 발생하는 메모리 오버플로우 문제 수정 (#3858)
- 채널 버그 수정, 닫힌 후에 기존 데이터를 pop 할 수 없는 문제 수정
## v4.5.9
### 강화

- Coroutine\Http\Client에 SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED 상수를 추가했습니다 (#3873) (@sy-records)
### 수정

- PHP8와 호환성을 유지하도록 수정 (#3868) (#3869) (#3872) (@twose) (@huanghantao) (@doubaokun)
- 정의되지 않은 상수 CURLOPT_HEADEROPT와 CURLOPT_PROXYHEADER를 수정 (swoole/library#77) (@sy-records)
- CURLOPT_USERPWD를 수정함 (swoole/library@7952a7b) (@twose)
## v4.5.8
### 새로운 API

- swoole_error_log 함수 추가, log_rotation 최적화 (swoole/swoole-src@67d2bff) (@matyhtf)
- readVector 및 writeVector SSL 지원 (#3857) (@huanghantao)
### 강화

- 자식 프로세스가 종료되면 System::wait가 블로킹을 종료하도록 함 (#3832) (@matyhtf)
- DTLS는 16K의 패킷을 지원합니다. (#3849) (@matyhtf)
- Response::cookie 메서드는 priority 매개변수를 지원합니다. (#3854) (@matyhtf)
- 더 많은 CURL 옵션을 지원합니다. (swoole/library#71) (@sy-records)
- CURL HTTP 헤더를 처리할 때 대소문자 구분이 없어서 덮어쓰기 문제를 해결했습니다. (swoole/library#76) (@filakhtov) (@twose) (@sy-records)
### 수정

- readv_all 및 writev_all에서 EAGAIN 오류 처리 문제 수정 (#3830) (@huanghantao)
- PHP8 컴파일 경고 수정 (swoole/swoole-src@03f3fb0) (@matyhtf)
- Swoole\Table 이진 안전 문제 수정 (#3842) (@twose)
- MacOS에서 System::writeFile이 파일 추가 모드 오버라이드 문제 수정 (swoole/swoole-src@a71956d) (@matyhtf)
- CURL의 CURLOPT_WRITEFUNCTION 수정 (swoole/library#74) (swoole/library#75) (@sy-records)
- HTTP form-data 구문 분석시 메모리 오버플로우 문제 수정 (#3858) (@twose)
- PHP8에서 'is_callable()'이 클래스의 비공개 메서드에 액세스할 수 없는 문제 수정 (#3859) (@twose)
### 커널

- SwooleG.std_allocator를 사용하여 메모리 할당 함수를 다시 구현했습니다 (#3853) (@matyhtf)
- 파이프를 다시 구현했습니다 (#3841) (@matyhtf)
## v4.5.7
### 새 API 추가

- Coroutine\Socket 클라이언트에 writeVector, writeVectorAll, readVector, readVectorAll 메서드를 추가했습니다 (#3764) (@huanghantao)
- server->stats에 task_worker_num 및 dispatch_count를 추가했습니다 (#3771) (#3806) (@sy-records) (@matyhtf)
- json, mysqlnd, sockets와 같은 확장 종속성이 추가되었습니다 (#3789) (@remicollet)
- server->bind의 uid 최소값을 INT32_MIN으로 제한했습니다 (#3785) (@sy-records)
- swoole_substr_json_decode에 컴파일 옵션을 추가하여 음의 오프셋을 지원했습니다 (#3809) (@matyhtf)
- CURL의 CURLOPT_TCP_NODELAY 옵션을 지원합니다 (swoole/library#65) (@sy-records) (@deminy)
### 수정사항

- 동기 클라이언트 연결 정보 오류 수정 (#3784) (@twose)
- hook scandir 함수의 문제 수정 (#3793) (@twose)
- 코루틴 장벽 barrier 내의 오류 수정 (swoole/library#68) (@sy-records)
### 커널

- boost.stacktrace를 사용하여 print-backtrace를 최적화했습니다. (#3788) (@matyhtf)
## v4.5.6
### 새로운 API

- [swoole_substr_unserialize](/functions?id=swoole_substr_unserialize)와 [swoole_substr_json_decode](/functions?id=swoole_substr_json_decode)가 추가되었습니다. (#3762) (@matyhtf)
### 강화

- `Coroutine\Http\Server`의 `onAccept` 메서드를 비공개로 변경(dfcc83b) (@matyhtf)
### 수정 사항

- Coverity 문제를 수정했습니다 (#3737) (#3740) (@matyhtf)
- Alpine 환경에서 발생한 몇 가지 문제를 수정했습니다 (#3738) (@matyhtf)
- swMutex_lockwait를 수정했습니다 (0fc5665) (@matyhtf)
- PHP-8.1 설치 실패를 수정했습니다 (#3757) (@twose)
### 내커널

- `Socket::read/write/shutdown`에 활성 검사를 추가했습니다 (#3735) (@matyhtf)
- session_id와 task_id의 유형을 int64로 변경했습니다 (#3756) (@matyhtf)
## v4.5.5

!> 이 버전은 [설정 항목](/server/setting) 검사 기능을 추가했습니다. Swoole이 제공하는 옵션이 아닌 것이 설정되면 경고가 발생합니다.

```shell
PHP Warning:  unsupported option [foo] in @swoole-src/library/core/Server/Helper.php 
```

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->set(['foo' => 'bar']);

$http->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```
### 새로운 API

- Process\Manager를 추가하고 Process\ProcessManager를 별칭으로 수정합니다 (swoole/library#eac1ac5) (@matyhtf)
- HTTP2 서버 GOAWAY를 지원합니다 (#3710) (@doubaokun)
- `Co\map()` 함수를 추가합니다 (swoole/library#57) (@leocavalcante)
- http2 unix 소켓 클라이언트 지원 (#3668) (@sy-records)
- worker 프로세스가 종료된 후 worker 프로세스 상태를 SW_WORKER_EXIT로 설정합니다. (#3724) (@matyhtf)
- `Server::getClientInfo()`의 반환 값에 send_queued_bytes와 recv_queued_bytes를 추가합니다. (#3721) (#3731) (@matyhtf) (@Yurunsoft)
- Server가 stats_file 구성 옵션을 지원합니다. (#3725) (@matyhtf) (@Yurunsoft)
- PHP8에서의 컴파일 문제 (zend_compile_string 변경)를 수정했습니다. (#3670) (@twose)
- PHP8에서의 컴파일 문제 (ext/sockets 호환성)를 수정했습니다. (#3684) (@twose)
- PHP8에서의 컴파일 문제 (php_url_encode_hash_ex 변경)를 수정했습니다. (#3713) (@remicollet)
- 'const char*'에서 'char*'로의 잘못된 유형 변환을 수정했습니다. (#3686) (@remicollet)
- HTTP 프록시 하에서 작동하지 않는 HTTP2 클라이언트의 문제를 수정했습니다. (#3677) (@matyhtf) (@twose)
- PDO 연결 끊김 시 데이터가 혼합되는 문제를 수정했습니다. (swoole/library#54) (@sy-records)
- IPv6를 사용할 때 UDP 서버의 포트 해석 오류를 수정했습니다.
- Lock::lockwait의 타임아웃이 무효화되는 문제를 수정했습니다.
## v4.5.4
### 하위 호환성이 없는 변경 사항

- SWOOLE_HOOK_ALL에 SWOOlE_HOOK_CURL이 포함되었습니다. (#3606) (@matyhtf)
- ssl_method를 제거하고 ssl_protocols를 추가했습니다. (#3639) (@Yurunsoft)
### 새 API 추가

- 배열의 firstKey 및 lastKey 메서드 추가 (swoole/library#51) (@sy-records)
### 강화

- 웹 소켓 서버의 open_websocket_ping_frame, open_websocket_pong_frame 구성 옵션을 추가하였습니다. (#3600) (@Yurunsoft)
### 수정 사항

- 파일 크기가 2GB를 초과할 때 fseek ftell이 올바르지 않은 문제를 수정했습니다 (#3619) (@Yurunsoft)
- 소켓 장벽의 문제를 수정했습니다 (#3627) (@matyhtf)
- HTTP 프록시 핸드셰이크의 문제를 수정했습니다 (#3630) (@matyhtf)
- 상대방이 청크 데이터를 전송할 때 HTTP 헤더를 잘못 해석하는 문제를 수정했습니다 (#3633) (@matyhtf)
- zend_hash_clean이 실패하는 문제를 수정했습니다 (#3634) (@twose)
- 이벤트 루프에서 손상된 fd를 제거할 수 없는 문제를 수정했습니다 (#3650) (@matyhtf)
- 잘못된 패킷을 수신할 때 코어 덤프가 발생하는 문제를 수정했습니다 (#3653) (@matyhtf)
- array_key_last의 버그를 수정했습니다 (swoole/library#46) (@sy-records)
### 커널

- 코드 최적화 (#3615) (#3617) (#3622) (#3635) (#3640) (#3641) (#3642) (#3645) (#3658) (@matyhtf)
- Swoole Table에 데이터를 쓸 때 불필요한 메모리 작업을 줄임 (#3620) (@matyhtf)
- AIO 재구성 (#3624) (@Yurunsoft)
- readlink/opendir/readdir/closedir 후크 지원 (#3628) (@matyhtf)
- swMutex_create 최적화, SW_MUTEX_ROBUST 지원 (#3646) (@matyhtf)
## v4.5.3
### 새로운 API 추가

- `Swoole\Process\ProcessManager` 추가 (swoole/library#88f147b) (@huanghantao)
- ArrayObject::append, StringObject::equals 추가 (swoole/library#f28556f) (@matyhtf)
- [Coroutine::parallel](/coroutine/coroutine?id=parallel) 추가 (swoole/library#6aa89a9) (@matyhtf)
- [Coroutine\Barrier](/coroutine/barrier) 추가 (swoole/library#2988b2a) (@matyhtf)
- `usePipelineRead`를 추가하여 http2 클라이언트 스트리밍을 지원합니다 (#3354) (@twose)
- 파일을 다운로드하는 동안 http 클라이언트는 데이터를 수신하기 전에 파일을 만들지 않습니다 (#3381) (@twose)
- http 클라이언트는 `bind_address`와 `bind_port` 설정을 지원합니다 (#3390) (@huanghantao)
- http 클라이언트는 `lowercase_header` 설정을 지원합니다 (#3399) (@matyhtf)
- `Swoole\Server`는 `tcp_user_timeout` 설정을 지원합니다 (#3404) (@huanghantao)
- `Coroutine\Socket`은 이벤트 장벽을 추가하여 코루틴 전환을 줄입니다 (#3409) (@matyhtf)
- 특정한 swString에 `메모리 할당기`를 추가합니다 (#3418) (@matyhtf)
- cURL은 `__toString`을 지원합니다 (swoole/library#38) (@twose)
- `WaitGroup` 생성자에서 `wait count`를 직접 설정할 수 있습니다 (swoole/library#2fb228b8) (@matyhtf)
- `CURLOPT_REDIR_PROTOCOLS`를 추가합니다 (swoole/library#46) (@sy-records)
- http1.1 서버는 trailer를 지원합니다 (#3485) (@huanghantao)
- 코루틴 sleep 시간이 1ms 미만이면 현재 코루틴을 양보합니다 (#3487) (@Yurunsoft)
- http 정적 핸들러는 심볼릭 링크된 파일을 지원합니다 (#3569) (@LeiZhang-Hunter)
- Server에서 close 메소드를 호출한 후 즉시 WebSocket 연결을 닫습니다 (#3570) (@matyhtf)
- `stream_set_blocking` 후크를 지원합니다 (#3585) (@Yurunsoft)
- 비동기 HTTP2 서버는 흐름 제어를 지원합니다 (#3486) (@huanghantao) (@matyhtf)
- onPackage 콜백 함수 실행 후 소켓 버퍼를 해제합니다 (#3551) (@huanghantao) (@matyhtf)
### 修复

- 修复 WebSocket coredump, 处理协议错误的状态 (#3359) (@twose)
- 修复 swSignalfd_setup 函数以及 wait_signal 函数里的空指针错误 (#3360) (@twose)
- 修复在设置了 dispatch_func 时候，调用`Swoole\Server::close`会报错的问题 (#3365) (@twose)
- 修复`Swoole\Redis\Server::format`函数中 format_buffer 初始化问题 (#3369) (@matyhtf) (@twose)
- 修复 MacOS 上无法获取 mac 地址的问题 (#3372) (@twose)
- 修复 MySQL 测试用例 (#3374) (@qiqizjl)
- 修复多处 PHP8 兼容性问题 (#3384) (#3458) (#3578) (#3598) (@twose)
- 修复 hook 的 socket write 中丢失了 php_error_docref, timeout_event 和返回值问题 (#3383) (@twose)
- 修复异步 Server 无法在`WorkerStart`回调函数中关闭 Server 的问题 (#3382) (@huanghantao)
- 修复心跳线程在操作 conn->socket 的时候，可能会发生 coredump 的问题 (#3396) (@huanghantao)
- 修复 send_yield 的逻辑问题 (#3397) (@twose) (@matyhtf)
- 修复 Cygwin64 上的编译问题 (#3400) (@twose)
- 修复 WebSocket finish 属性无效的问题 (#3410) (@matyhtf)
- 修复遗漏的 MySQL transaction 错误状态 (#3429) (@twose)
- 修复 hook 后的`stream_select`与 hook 之前返回值行为不一致的问题 (#3440) (@Yurunsoft)
- 修复使用`Coroutine\System`来创建子进程时丢失`SIGCHLD`信号的问题 (#3446) (@huanghantao)
- 修复`sendwait`不支持 SSL 的问题 (#3459) (@huanghantao)
- 修复`ArrayObject`和`StringObject`的若干问题 (swoole/library#44) (@matyhtf)
- 修复 mysqli 异常信息错误 (swoole/library#45) (@sy-records)
- 修复当设置`open_eof_check`后，`Swoole\Client`无法获取正确的`errCode`的问题 (#3478) (@huanghantao)
- 修复 MacOS 上 `atomic->wait()`/`wakeup()`的若干问题 (#3476) (@Yurunsoft)
- 修复`Client::connect`连接拒绝的时候，返回成功状态的问题 (#3484) (@matyhtf)
- 修复 alpine 环境下 nullptr_t 没有被声明的问题 (#3488) (@limingxinleo)
- 修复 HTTP Client 下载文件的时候，double-free 的问题 (#3489) (@Yurunsoft)
- 修复`Server`被销毁时候，`Server\Port`没释放导致的内存泄漏问题 (#3507) (@twose)
- 修复 MQTT 协议解析问题 (318e33a) (84d8214) (80327b3) (efe6c63) (@GXhua) (@sy-records)
- 修复`Coroutine\Http\Client->getHeaderOut`方法导致的 coredump 问题 (#3534) (@matyhtf)
- 修复 SSL 验证失败后，丢失了错误信息的问题 (#3535) (@twose)
- 修复 README 中，`Swoole benchmark`链接错误的问题 (#3536) (@sy-records) (@santalex)
- 修复在`HTTP header/cookie`中使用`CRLF`后导致的`header`注入问题 (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 修复 issue #3463 中提到的变量错误的问题 (#3547) (chromium1337) (@huanghantao)
- 修复 pr #3463 中提到的错别字问题 (#3547) (@deminy)
- 修复协程 WebSocket 服务器 frame->fd 为空的问题 (#3549) (@huanghantao)
- 修复心跳线程错误判断连接状态导致的连接泄漏问题 (#3534) (@matyhtf)
- 修复`Process\Pool`中阻塞了信号的问题 (#3582) (@huanghantao) (@matyhtf)
- 修复`SAPI`中使用 send headers 的问题 (#3571) (@twose) (@sshymko)
- 修复`CURL`执行失败的时候，未设置`errCode`和`errMsg`的问题 (swoole/library#1b6c65e) (@sy-records)
- 修复当调用了`setProtocol`方法后，`swoole_socket_coro`accept coredump 的问题 (#3591)
- C++ 스타일 사용 (#3349) (#3351) (#3454) (#3479) (#3490) (@huanghantao) (@matyhtf)
- `Swoole` 알려진 문자열을 추가하여 `PHP` 객체 속성 읽기 성능 향상 (#3363) (@huanghantao)
- 다양한 코드 최적화 (#3350) (#3356) (#3357) (#3423) (#3426) (#3461) (#3463) (#3472) (#3557) (#3583) (@huanghantao) (@twose) (@matyhtf)
- 다양한 테스트 코드 최적화 (#3416) (#3481) (#3558) (@matyhtf)
- `Swoole\Table`의 `int` 유형 간소화 (#3407) (@matyhtf)
- `sw_memset_zero` 추가 및 `bzero` 함수 대체 (#3419) (@CismonX)
- 로깅 모듈 최적화 (#3432) (@matyhtf)
- 많은 부분의 libswoole 재구성 (#3448) (#3473) (#3475) (#3492) (#3494) (#3497) (#3498) (#3526) (@matyhtf)
- 많은 헤더 파일 포함 재구성 (#3457) (@matyhtf) (@huanghantao)
- `Channel::count()`와 `Channel::get_bytes()` 추가 (f001581) (@matyhtf)
- `scope guard` 추가 (#3504) (@huanghantao)
- libswoole 커버리지 테스트 추가 (#3431) (@huanghantao)
- lib-swoole/ext-swoole MacOS 환경 테스트 추가 (#3521) (@huanghantao)
- lib-swoole/ext-swoole Alpine 환경 테스트 추가 (#3537) (@limingxinleo)
## v4.5.2

[v4.5.2](https://github.com/swoole/swoole-src/releases/tag/v4.5.2)은 버그 수정 버전입니다. 하위 호환성 변경 사항은 없습니다.
- `Server->set(['log_rotation' => SWOOLE_LOG_ROTATION_DAILY])`를 지원하여 로그를 날짜별로 생성할 수 있습니다 (#3311) (@matyhtf)
- `swoole_async_set(['wait_signal' => true])`을 지원하여 시그널 리스너가 있는 경우 리액터가 종료되지 않습니다 (#3314) (@matyhtf)
- `Server->sendfile`로 빈 파일을 전송할 수 있습니다 (#3318) (@twose)
- 워커의 바쁨 및 여유 알림 메시지를 최적화했습니다 (#3328) (@huanghantao)
- HTTPS 프록시에서 Host 헤더에 대한 구성을 최적화했습니다 (ssl_host_name을 사용하여 구성) (#3343) (@twose)
- SSL은 기본적으로 ecdh auto 모드를 사용합니다 (#3316) (@matyhtf)
- SSL 클라이언트는 연결이 끊겼을 때 조용히 종료됩니다 (#3342) (@huanghantao)
### 수정

- OSX 플랫폼에서 `Server->taskWait` 문제 수정 (#3330) (@matyhtf)
- MQTT 프로토콜 구문 분석 버그 수정 (8dbf506b) (@guoxinhua) (2ae8eb32) (@twose)
- Content-Length int 타입 오버플로우 문제 수정 (#3346) (@twose)
- PRI 패킷 길이 확인 부족 문제 수정 (#3348) (@twose)
- CURLOPT_POSTFIELDS를 비울 수 없는 문제 수정 (swoole/library@ed192f64) (@twose)
- 새로운 연결 객체가 다음 연결을 받기 전에 해제되지 않는 문제 수정 (swoole/library@1ef79339) (@twose)
### 내부 커널

- 소켓 쓰기 제로 카피 기능 (#3327) (@twose)
- 글로벌 변수 읽기/쓰기를 대체하기 위해 swoole_get_last_error/swoole_set_last_error 두 개 사용 (e25f262a) (@matyhtf) (#3315) (@huanghantao)
## v4.5.1

[v4.5.1](https://github.com/swoole/swoole-src/releases/tag/v4.5.1)는 버그 수정 버전으로, `v4.5.0`에서 소개된 System 파일 함수의 폐기 표시를 보완했습니다.
- hook 하위의 socket_context의 bindto 구성 지원 (#3275) (#3278) (@codinghuang)
- client::sendto에서 주소 자동 DNS 해석 지원 (#3292) (@codinghuang)
- Process->exit(0)은 프로세스를 직접 종료시키며, shutdown_functions를 실행한 후 종료하려면 PHP가 제공하는 exit를 사용하십시오 (a732fe56) (@matyhtf)
- `log_date_format` 구성 지원으로 로그 날짜 형식을 변경할 수 있습니다. `log_date_with_microseconds`는 로그에서 마이크로초 시간을 표시합니다 (baf895bc) (@matyhtf)
- CURLOPT_CAINFO 및 CURLOPT_CAPATH 지원 (swoole/library#32) (@sy-records)
- CURLOPT_FORBID_REUSE 지원 (swoole/library#33) (@sy-records)
### 수리

- 32 비트 빌드 실패 수정 (#3276) (#3277) (@remicollet) (@twose)
- 코루틴 클라이언트가 중복 연결 시 EISCONN 오류 메시지가 표시되지 않는 문제 수정 (#3280) (@codinghuang)
- Table 모듈에서 잠재적 버그 수정 (d7b87b65) (@matyhtf)
- 정의되지 않은 동작으로 인한 서버의 null 포인터 수정 (방어적 프로그래밍) (#3304) (#3305) (@twose)
- 하트비트 구성을 활성화했을 때 발생하는 null 포인터 오류 수정 (#3307) (@twose)
- mysqli 구성이 적용되지 않는 문제 수정 (swoole/library#35)
- 헤더에서 비표준적인 공백이 누락된 경우 응답을 구문 분석하는 문제 수정 (swoole/library#27) (@Yurunsoft)
### 폐기됨

- Coroutine\System의 (fread/fgets/fwrite) 등 메소드를 폐기 처리하십시오. (hook 기능을 사용하여 PHP에서 제공하는 파일 함수를 직접 사용하십시오) (c7c9bb40) (@twose)
### 내핵

- 사용자 정의 객체에 메모리를 할당하기 위해 zend_object_alloc을 사용하십시오 (cf1afb25) (@twose)
- 로깅 모듈에 더 많은 구성 옵션을 추가하기 위한 최적화 (#3296) (@matyhtf)
- 많은 코드 최적화 및 단위 테스트 추가 (swoole/library) (@deminy)
## v4.5.0

[v4.5.0](https://github.com/swoole/swoole-src/releases/tag/v4.5.0)는 주요 업데이트입니다. 이 업데이트에서는 v4.4.x에서 이미 폐기로 표시된 모듈을 제거했습니다.
### 새로운 API 추가

- DTLS 지원, 이제 이 기능을 사용하여 WebRTC 애플리케이션을 작성할 수 있습니다. (#3188) (@matyhtf)
- 내장된 `FastCGI` 클라이언트로, FPM으로 요청을 프록시하거나 FPM 애플리케이션을 호출할 수 있습니다. (swoole/library#17) (@twose)
- `Co::wait`, `Co::waitPid` (자식 프로세스 회수용), `Co::waitSignal` (신호 대기용) (#3158) (@twose)
- `Co::waitEvent` (소켓에서 지정된 이벤트가 발생될 때까지 기다림) (#3197) (@twose)
- `Co::set(['exit_condition' => $callable])` (프로그램 종료 조건을 사용자 정의하는 데 사용) (#2918) (#3012) (@twose)
- `Co::getElapsed` (코루틴 실행 시간을 얻어 분석 및 통계 또는 좀비 코루틴 발견에 사용) (#3162) (@doubaokun)
- `Socket::checkLiveness` (시스템 호출을 통해 연결이 활성 상태인지 확인), `Socket::peek` (읽기 버퍼를 엿봄) (#3057) (@twose)
- `Socket->setProtocol(['open_fastcgi_protocol' => $bool])` (내장 FastCGI 언팩 지원) (#3103) (@twose)
- `Server::get(Master|Manager|Worker)Pid`, `Server::getWorkerId` (비동기 서버 싱글톤 및 정보 가져오기) (#2793) (#3019) (@matyhtf)
- `Server::getWorkerStatus` (워커 프로세스 상태 가져오기, SWOOLE_WORKER_BUSY, SWOOLE_WORKER_IDLE과 같은 상수 반환하여 바쁨 여부 표시) (#3225) (@matyhtf)
- `Server->on('beforeReload', $callable)` 및 `Server->on('afterReload', $callable)` (서버 재로드 이벤트, 매니저 프로세스에서 발생) (#3130) (@hantaohuang)
- `Http\Server` 정적 파일 처리기는 이제 `http_index_files`와 `http_autoindex` 구성도 지원합니다. (#3171) (@hantaohuang)
- `Http2\Client->read(float $timeout = -1)` 메소드가 스트리밍 응답을 읽을 수 있도록 지원합니다. (#3011) (#3117) (@twose)
- `Http\Request->getContent` (rawContent 메소드의 별칭) (#3128) (@hantaohuang)
- `swoole_mime_type_(add|set|delete|get|exists)()` (mime 관련 API, 내장 mime 유형을 추가, 삭제, 조회, 변경할 수 있습니다) (#3134) (@twose)
### 강화

- 마스터 및 워커 프로세스 간 메모리 복사 최적화(극한으로 성능 향상, 4배 증가) (#3075) (#3087) (@hantaohuang)
- WebSocket 디스패치 로직 최적화 (#3076) (@matyhtf)
- WebSocket 프레임 구성 시 한 번의 메모리 복사 최적화 (#3097) (@matyhtf)
- SSL 인증 모듈 최적화 (#3226) (@matyhtf)
- SSL accept 및 SSL 핸드셰이크 분리, 느린 SSL 클라이언트로 인한 코루틴 서버의 봉쇄 문제 해결 (#3214) (@twose)
- MIPS 아키텍처 지원 (#3196) (@ekongyun)
- UDP 클라이언트가 이제 수신된 도메인을 자동으로 분석할 수 있습니다 (#3236) (#3239) (@huanghantao)
- Coroutine\Http\Server에서 일부 일반 옵션을 지원하도록 확장 (#3257) (@twose)
- WebSocket 핸드셰이크 시 쿠키 설정 지원 추가 (#3270) (#3272) (@twose)
- CURLOPT_FAILONERROR 지원 (@sy-records)
- CURLOPT_SSLCERTTYPE, CURLOPT_SSLCERT, CURLOPT_SSLKEYTYPE, CURLOPT_SSLKEY 지원 (@sy-records)
- CURLOPT_HTTPGET 지원 (swoole/library@d730bd08) (@shiguangqi)
### 삭제

- `Runtime::enableStrictMode` 메서드 삭제 (b45838e3) (@twose)
- `Buffer` 클래스 삭제 (559a49a8) (@twose)
- 새로운 C++ API: coroutine::async 함수를 이용하여 람다를 전달하면 비동기 스레드 작업을 시작할 수 있습니다 (#3127) (@matyhtf)
- 하위 이벤트-API의 정수형 fd를 swSocket 객체로 재구성했습니다 (#3030) (@matyhtf)
- 모든 핵심 C 파일이 C++ 파일로 변환되었습니다 (#3030) (71f987f3) (@matyhtf)
- 일련의 코드 최적화를 수행했습니다 (#3063) (#3067) (#3115) (#3135) (#3138) (#3139) (#3151) (#3168) (@hantaohuang)
- 헤더 파일의 표준화를 위한 최적화를 수행했습니다 (#3051) (@matyhtf)
- `enable_reuse_port` 구성 항목을 보다 표준화하도록 재구성했습니다 (#3192) (@matyhtf)
- 소켓 관련 API를 보다 표준화하도록 재구성했습니다 (#3193) (@matyhtf)
- 시스템 호출 한 번을 줄이기 위해 버퍼 예측을 통해 수행했습니다 (3b5aa85d) (@matyhtf)
- 하위 새로고침 타이머 swServerGS::now를 제거하고 직접 시간 함수를 사용하여 시간을 가져옵니다 (#3152) (@hantaohuang)
- 프로토콜 구성자를 최적화했습니다 (#3108) (@twose)
- 더 나은 호환성을 갖는 C 구조 초기화 방법을 지원합니다 (#3069) (@twose)
- bit 필드를 모두 uchar 유형으로 통합했습니다 (#3071) (@twose)
- 병렬 테스트를 지원하여 더 빠른 속도를 제공합니다 (#3215) (@twose)
### 수정사항

- enable_delay_receive를 활성화한 후 onConnect가 작동하지 않는 문제를 수정했습니다 (#3221) (#3224) (@matyhtf)
- 다른 모든 버그 수정은 v4.4.x 브랜치에 병합되었으며 업데이트 로그에 포함되어 있습니다. 따라서 여기에 더 이상 언급하지 않겠습니다.
## v4.4.22
### 수정사항

- HTTP2 클라이언트가 HTTP 프록시에서 작동하지 않는 문제를 수정하였습니다 (#3677) (@matyhtf) (@twose)
- PDO 연결 끊김 시 데이터가 혼재되는 문제를 수정하였습니다 (swoole/library#54) (@sy-records)
- swMutex_lockwait을 수정하였습니다 (0fc5665) (@matyhtf)
- UDP 서버가 IPv6를 사용할 때 포트 해석 오류를 수정하였습니다
- systemd file descriptors의 문제를 수정하였습니다
## v4.4.20

[v4.4.20](https://github.com/swoole/swoole-src/releases/tag/v4.4.20)은 버그 수정 버전으로, 하위 호환성 변경 사항은 없습니다.
### 수정

- `Swoole\Server::close` 호출 시 `dispatch_func`이 설정된 경우 오류가 발생하는 문제를 수정했습니다. (#3365) (@twose)
- `Swoole\Redis\Server::format` 함수 내에서 `format_buffer` 초기화 문제를 수정했습니다. (#3369) (@matyhtf) (@twose)
- MacOS에서 MAC 주소를 가져올 수 없는 문제를 수정했습니다. (#3372) (@twose)
- MySQL 테스트 케이스를 수정했습니다. (#3374) (@qiqizjl)
- 비동기 서버에서 `WorkerStart` 콜백 함수에서 서버를 닫을 수 없는 문제를 수정했습니다. (#3382) (@huanghantao)
- 누락된 MySQL 트랜잭션 오류 상태를 수정했습니다. (#3429) (@twose)
- HTTP 클라이언트로 파일을 다운로드할 때 발생하는 double-free 문제를 수정했습니다. (#3489) (@Yurunsoft)
- `Coroutine\Http\Client->getHeaderOut` 메서드로 발생하는 coredump 문제를 수정했습니다. (#3534) (@matyhtf)
- `HTTP header/cookie`에서 `CRLF` 사용으로 인한 `header` 주입 문제를 수정했습니다. (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 코루틴 WebSocket 서버에서 frame->fd가 비어 있는 문제를 수정했습니다. (#3549) (@huanghantao)
- phpredis를 후킹하여 발생하는 `read error on connection` 문제를 수정했습니다. (#3579) (@twose)
- MQTT 프로토콜 구문 분석 문제를 수정했습니다. (#3573) (#3517) (9ad2b455) (@GXhua) (@sy-records)
## v4.4.19

[v4.4.19](https://github.com/swoole/swoole-src/releases/tag/v4.4.19)는 버그 수정 버전으로, 하위 호환성에 영향을 주지 않습니다.

!> 주의: v4.4.x는 더 이상 주요 유지 보수 버전이 아니며 필요시에만 버그를 수정합니다.
### 修复

- 从 v4.5.2 合并了所有 bug 修复补丁
## v4.4.18

[v4.4.18](https://github.com/swoole/swoole-src/releases/tag/v4.4.18)는 버그 수정 버전으로, 하위 호환성 변경 사항은 없습니다.
- UDP 클라이언트는 이제 들어오는 도메인을 자동으로 해석할 수 있습니다. (#3236) (#3239) (@huanghantao)
- CLI 모드에서 더 이상 stdout와 stderr를 닫지 않습니다 (종료 후에 발생한 오류 로그를 표시합니다). (#3249) (@twose)
- Coroutine\Http\Server에 일부 일반적으로 사용되는 옵션이 추가되었습니다. (#3257) (@twose)
- WebSocket 핸드셰이크 중에 쿠키 설정을 지원합니다. (#3270) (#3272) (@twose)
- CURLOPT_FAILONERROR를 지원합니다 (swoole/library#20) (@sy-records)
- CURLOPT_SSLCERTTYPE, CURLOPT_SSLCERT, CURLOPT_SSLKEYTYPE, CURLOPT_SSLKEY를 지원합니다 (swoole/library#22) (@sy-records)
- CURLOPT_HTTPGET을 지원합니다 (swoole/library@d730bd08) (@shiguangqi)
- 가능한 모든 PHP-Redis 확장의 버전과 호환되도록 노력했습니다 (다른 버전의 생성자는 다른 매개변수를 전달합니다) (swoole/library#24) (@twose)
- 연결 객체의 복제를 금지합니다 (swoole/library#23) (@deminy)
- SSL 핸드셰이크 실패 문제를 해결하였습니다 (dc5ac29a) (@twose)
- 오류 메시지 생성 시 발생하는 메모리 오류를 수정하였습니다 (#3229) (@twose)
- 빈 프록시 인증 정보를 수정하였습니다 (#3243) (@twose)
- 채널 메모리 누수 문제를 수정하였습니다 (실제로는 메모리 누수가 아님) (#3260) (@twose)
- Co\Http\Server에서 순환 참조 시 발생하는 일회용 메모리 누수를 수정하였습니다 (#3271) (@twose)
- `ConnectionPool->fill`에서의 오타를 수정하였습니다 (swoole/library#18) (@NHZEX)
- curl 클라이언트가 리다이렉션을 만날 때 연결을 업데이트하지 않는 문제를 수정하였습니다 (swoole/library#21) (@doubaokun)
- ioException이 발생할 때 널 포인터 문제를 해결하였습니다 (swoole/library@4d15a4c3) (@twose)
- ConnectionPool@put에 널을 전달할 때 새 연결을 반환하지 않아 데드락이 발생하는 문제를 수정하였습니다 (swoole/library#25) (@Sinute)
- mysqli 프록시 구현으로 인한 write_property 오류를 수정하였습니다 (swoole/library#26) (@twose)
## v4.4.17

[v4.4.17](https://github.com/swoole/swoole-src/releases/tag/v4.4.17)는 버그 수정 버전으로, 하위 호환성 변경 사항이 없습니다.
### 강화

- SSL 서버의 성능 향상 (#3077) (85a9a595) (@matyhtf)
- HTTP 헤더 크기 제한 제거 (#3187) (@twose)
- MIPS 지원 (#3196) (@ekongyun)
- CURLOPT_HTTPAUTH 지원 (swoole/library@570318be) (@twose)
### 수정 사항

- package_length_func의 동작 및 잠재적인 일회성 메모리 누수 수정 (#3111) (@twose)
- HTTP 상태 코드 304에서의 오류 동작 수정 (#3118) (#3120) (@twose)
- Trace 로그 오류로 인한 메모리 오류 수정 (#3142) (@twose)
- OpenSSL 함수 서명 수정 (#3154) (#3155) (@twose)
- SSL 오류 메시지 수정 (#3172) (@matyhtf) (@twose)
- PHP-7.4 호환성 수정 (@twose) (@matyhtf)
- HTTP-chunk 길이 구문 분석 오류 수정 (19a1c712) (@twose)
- 청크된 모드에서 멀티파트 요청 구문 분석기 동작 수정 (3692d9de) (@twose)
- PHP-Debug 모드에서 ZEND_ASSUME 어설션 실패 수정 (fc0982be) (@twose)
- Socket 오류 주소 수정 (d72c5e3a) (@twose)
- Socket getname 수정 (#3177) (#3179) (@matyhtf)
- 빈 파일에 대한 정적 파일 처리기 오류 수정 (#3182) (@twose)
- Coroutine\Http\Server 파일 업로드 문제 수정 (#3189) (#3191) (@twose)
- shutdown 중에 발생할 수 있는 메모리 오류 수정 (44aef60a) (@matyhtf)
- Server->heartbeat 수정 (#3203) (@matyhtf)
- CPU 스케쥴러가 무한 루프를 스케줄링하지 못하는 문제 수정 (#3207) (@twose)
- 불변 배열에 대한 유효하지 않은 쓰기 작업 수정 (#3212) (@twose)
- WaitGroup에서 여러 번의 wait 문제 수정 (swoole/library@537a82e1) (@twose)
- 빈 헤더 처리 수정 (cURL과 일관성 유지) (swoole/library@7c92ed5a) (@twose)
- IO 메소드가 false를 반환했을 때 예외 처리하는 문제 수정 (swoole/library@f6997394) (@twose)
- cURL-hook에서 proxy 포트 번호가 헤더에 여러 번 추가되는 문제 수정 (swoole/library@5e94e5da) (@twose)
## v4.4.16

[v4.4.16](https://github.com/swoole/swoole-src/releases/tag/v4.4.16)는 버그 수정 버전으로, 하위 호환성 변경 사항은 없습니다.
### 강화

- 이제 [Swoole 버전 지원 정보](https://github.com/swoole/swoole-src/blob/master/SUPPORTED.md)를 얻을 수 있습니다.
- 더 사용자 친화적인 오류 메시지 (0412f442) (09a48835) (@twose)
- 특정 시스템에서 시스템 호출 루프에 빠지는 것을 방지합니다. (069a0092) (@matyhtf)
- PDOConfig에 드라이버 옵션을 추가했습니다. (swoole/library#8) (@jcheron)
### 수정

- http2_session.default_ctx 메모리 오류 수정 (bddbb9b1) (@twose)
- 초기화되지 않은 http_context 수정 (ce77c641) (@twose)
- Table 모듈에서의 오타 수정 (메모리 오류 발생 가능성) (db4eec17) (@twose)
- Server의 task-reload의 잠재적 문제 수정 (e4378278) (@GXhua)
- 불완전한 코루틴 HTTP 서버 요청 수정 (#3079) (#3085) (@hantaohuang)
- static 핸들러 수정 (파일이 비어 있을 때 404 응답을 반환하지 않아야 함) (#3084) (@Yurunsoft)
- http_compression_level 설정이 제대로 작동하지 않는 문제 수정 (16f9274e) (@twose)
- Coroutine HTTP2 서버에서 핸들이 등록되지 않아서 발생하는 널 포인터 오류 수정 (ed680989) (@twose)
- socket_dontwait 구성이 작동하지 않는 문제 수정 (27589376) (@matyhtf)
- zend::eval이 여러 번 실행될 수 있는 문제 수정 (#3099) (@GXhua)
- 연결 종료 후에 응답하는 HTTP2 서버에서 발생하는 널 포인터 오류 수정 (#3110) (@twose)
- PDOStatementProxy::setFetchMode의 부적절한 적응 수정 (swoole/library#13) (@jcheron)
