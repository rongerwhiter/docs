# 함수 목록

Swoole은 네트워크 통신 관련 함수뿐만 아니라 PHP 프로그램이 시스템 정보를 받아오는 몇 가지 함수를 제공합니다.
## swoole_set_process_name()

프로세스 이름을 설정하는 데 사용됩니다. 프로세스 이름을 변경하면 `php your_file.php` 대신 설정한 문자열이 표시됩니다.

이 함수는 문자열 매개변수를 받습니다.

이 함수는 PHP 5.2 이상의 모든 버전에서 사용할 수 있으며 PHP 5.5에서 제공하는 [cli_set_process_title](https://www.php.net/manual/zh/function.cli-set-process-title.php) 기능과 동일합니다. 그러나 `swoole_set_process_name`은 `cli_set_process_title`보다 호환성이 낮습니다. `cli_set_process_title` 함수가 있다면 `cli_set_process_title`을 우선 사용해야 합니다.

```php
function swoole_set_process_name(string $name): void
```

사용 예시:

```php
swoole_set_process_name("swoole server");
```
### Swoole Server의 각 프로세스 이름 변경하는 방법 <!-- {docsify-ignore} -->

* [onStart](/server/events?id=onstart)이 호출될 때 주 프로세스 이름 변경
* [onManagerStart](/server/events?id=onmanagerstart)이 호출될 때 관리 프로세스(`manager`)의 이름 변경
* [onWorkerStart](/server/events?id=onworkerstart)이 호출될 때 worker 프로세스 이름 변경
 
!> 낮은 버전의 Linux 커널과 Mac OSX는 프로세스 이름 변경을 지원하지 않습니다.
## swoole_strerror()

오류 코드를 오류 정보로 변환합니다.

함수 원형:

```php
function swoole_strerror(int $errno, int $error_type = 1): string
```

오류 유형:

* `1` : 표준 `Unix Errno`, 시스템 콜 오류로 인해 발생하며, `EAGAIN`, `ETIMEDOUT` 등이 있습니다.
* `2` : `getaddrinfo` 오류 코드, `DNS` 조작으로 발생합니다.
* `9` : `Swoole` 하위 수준 오류 코드, `swoole_last_error()`를 사용하여 획득합니다.

사용 예:

```php
var_dump(swoole_strerror(swoole_last_error(), 9));
```
```php
function swoole_version(): string
```

사용 예시:

```php
var_dump(SWOOLE_VERSION); // SWOOLE_VERSION 전역 변수는 swoole 확장의 버전을 나타냄
var_dump(swoole_version());
/**
반환 값:
string(6) "1.9.23"
string(6) "1.9.23"
**/
```  
## swoole_errno()

최근 시스템 호출의 오류 코드를 가져 옵니다. `C/C++`의 `errno` 변수와 동일합니다.

```php
function swoole_errno(): int
```

오류 코드의 값은 운영 체제에 따라 다릅니다. 오류를 오류 메시지로 변환하기 위해 `swoole_strerror`를 사용할 수 있습니다.
## swoole_get_local_ip()

이 함수는 현재 컴퓨터의 모든 네트워크 인터페이스의 IP 주소를 가져오는 데 사용됩니다.

```php
function swoole_get_local_ip(): array
```

사용 예:

```php
// 현재 컴퓨터의 모든 네트워크 인터페이스의 IP 주소 가져오기
$list = swoole_get_local_ip();
print_r($list);
/**
Return Value
Array
(
      [eno1] => 10.10.28.228
      [br-1e72ecd47449] => 172.20.0.1
      [docker0] => 172.17.0.1
)
**/
```

!> 주의사항
* 현재 IPv4 주소만 반환되며, 반환된 결과에서 로컬 루프 주소 127.0.0.1은 필터링됩니다.
* 결과 배열은 인터페이스 이름을 키로 하는 연관 배열입니다. 예를 들어 `array("eth0" => "192.168.1.100")`
* 이 함수는 실시간으로 `ioctl` 시스템 콜을 사용하여 인터페이스 정보를 가져오며, 하위 시스템에 캐시가 없습니다.
```php
function swoole_clear_dns_cache()
```
## swoole_get_local_mac()

`Mac` 주소를 가져오는 함수입니다.

```php
function swoole_get_local_mac(): array
```

* 성공적으로 호출하면 모든 네트워크 카드의 `Mac` 주소가 반환됩니다.

```php
array(4) {
  ["lo"]=>
  string(17) "00:00:00:00:00:00"
  ["eno1"]=>
  string(17) "64:00:6A:65:51:32"
  ["docker0"]=>
  string(17) "02:42:21:9B:12:05"
  ["vboxnet0"]=>
  string(17) "0A:00:27:00:00:00"
}
```
## swoole_cpu_num()

로컬 머신의 CPU 코어 수를 가져옵니다.

```php
function swoole_cpu_num(): int
```

* 성공적으로 호출하면 CPU 코어 수가 반환됩니다. 예:

```shell
php -r "echo swoole_cpu_num();"
```  
## swoole_last_error()

Swoole 하위 레벨의 최근 오류 코드를 반환합니다.

```php
function swoole_last_error(): int
```

You can use `swoole_strerror(swoole_last_error(), 9)` to convert the error to an error message. For a full list of error messages, refer to the [Swoole error code list](/other/errno?id=swoole)
```php
function swoole_mime_type_add(string $suffix, string $mime_type): bool
```
```php
function swoole_mime_type_set(string $suffix, string $mime_type): bool
```  
```php
function swoole_mime_type_delete(string $suffix): bool
```
```php
function swoole_mime_type_get(string $filename): string
```  
## swoole_mime_type_exists()

확장자에 해당하는 MIME 유형이 존재하는지 확인합니다.

```php
function swoole_mime_type_exists(string $suffix): bool
```
## swoole_substr_json_decode()

제로 카피 JSON 역직렬화입니다. `$offset` 및 `$length`를 제외한 다른 매개변수는 [json_decode](https://www.php.net/manual/en/function.json-decode.php)와 동일합니다.

!> Swoole 버전 >= `v4.5.6`에서 사용할 수 있으며, `v4.5.7` 버전부터는 컴파일 시[--enable-swoole-json](/environment?id=通用参数) 매개변수를 추가하여 활성화해야 합니다. 사용 사례는 [Swoole 4.5.6 지원 제로 카피 JSON 또는 PHP 역직렬화](https://wenda.swoole.com/detail/107587)를 참조하세요.

```php
function swoole_substr_json_decode(string $packet, int $offset, int $length, bool $assoc = false, int $depth = 512, int $options = 0)
```

  * **예시**

```php
$val = json_encode(['hello' => 'swoole']);
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(json_decode(substr($str, 4, $l), true));
var_dump(swoole_substr_json_decode($str, 4, $l, true));
```
## swoole_substr_unserialize()

零拷贝 PHP 역직렬화 함수로, `$offset`과 `$length`를 제외한 다른 매개변수들은 [unserialize](https://www.php.net/manual/en/function.unserialize.php) 함수와 동일합니다.

!> Swoole 버전 >= `v4.5.6`에서 사용 가능합니다. 사용 사례는 [Swoole 4.5.6 지원 Zero-Copy JSON 또는 PHP 역직렬화](https://wenda.swoole.com/detail/107587)를 참고하세요.

```php
function swoole_substr_unserialize(string $packet, int $offset, int $length, array $options= [])
```

  * **예시**

```php
$val = serialize('hello');
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(unserialize(substr($str, 4, $l)));
var_dump(swoole_substr_unserialize($str, 4, $l));
```
```php
function swoole_error_log(int $level, string $msg)
```
## swoole_clear_error()

소켓의 오류 또는 마지막 오류 코드에서 오류를 지웁니다.

!> Swoole 버전 >= `v4.6.0`에서 사용 가능합니다

```php
function swoole_clear_error()
```
```php
function swoole_coroutine_socketpair(int $domain , int $type , int $protocol): array|bool
```
## swoole_async_set

이 함수는 비동기 I/O 관련 옵션을 설정할 수 있습니다.

```php
function swoole_async_set(array $settings)
```

- enable_signalfd `signalfd` 기능 사용의 활성화/비활성화
- enable_coroutine 내장 코루틴을 전환합니다. 자세한 내용은 [여기](/server/setting?id=enable_coroutine)를 참조하세요.
- aio_core_worker_num AIO 최소 프로세스 수를 설정합니다.
- aio_worker_num AIO 최대 프로세스 수를 설정합니다.
## swoole_error_log_ex()

지정된 수준과 오류 코드의 로그를 작성합니다.

```php
function swoole_error_log_ex(int $level, int $error, string $msg)
```

!> Swoole 버전 >= `v4.8.1`에서 사용 가능
## swoole_ignore_error()

특정 오류 코드의 오류 로그를 무시합니다.

```php
function swoole_ignore_error(int $error)
```

!> Swoole 버전 >= `v4.8.1`에서 사용 가능
