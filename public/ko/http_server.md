# Http\Server

?> `Http\Server`는 [Server](/server/init)에서 상속되기 때문에 `Server`에서 제공하는 모든 `API` 및 설정을 사용할 수 있으며 프로세스 모델도 동일합니다. [Server](/server/init) 섹션을 참조하십시오.

내장된 `HTTP` 서버를 지원하여 몇 줄의 코드로 고성능, 비동기 입출력(Asynchronous I/O) 다중 프로세스 `HTTP` 서버를 작성할 수 있습니다.

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->start();
```

`Apache ab` 도구를 사용하여 부하 테스트를 수행하면 일반 PC 기계인 `인텔 Core i5 4코어 + 8GB 메모리`에서 `Http\Server`의 대략 `11만 QPS`에 도달할 수 있습니다.

`PHP-FPM`, `Golang`, `Node.js` 기본 `Http` 서버보다 월등히 뛰어납니다. 성능은 거의 `Nginx`의 정적 파일 처리에 근접합니다.

```shell
ab -c 200 -n 200000 -k http://127.0.0.1:9501/
```

* **HTTP2 프로토콜 사용**

  * `SSL`을 사용하는 `HTTP2` 프로토콜은 `openssl`을 설치해야 하며, 고급 버전의 `openssl`은 `TLS1.2`, `ALPN`, `NPN`을 지원해야 합니다.
  * 컴파일시 [--enable-http2](/environment?id=compile-options)를 사용하여 활성화해야 합니다.
  * Swoole5부터는 기본적으로 http2 프로토콜을 사용합니다.

```shell
./configure --enable-openssl --enable-http2
```

`HTTP` 서버의 [open_http2_protocol](/http_server?id=open_http2_protocol)를 `true`로 설정합니다.

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_cert_file' => $ssl_dir . '/ssl.crt',
    'ssl_key_file' => $ssl_dir . '/ssl.key',
    'open_http2_protocol' => true,
]);
```

* **Nginx + Swoole 구성**

!> `Http\Server`는 `HTTP` 프로토콜을 완전히 지원하지 않으므로 동적 요청 처리용으로만 애플리케이션 서버로 사용하는 것이 좋으며, 프론트엔드에 `Nginx`를 프록시로 사용하는 것이 좋습니다.

```nginx
server {
    listen 80;
    server_name swoole.test;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:9501;
    }
}
```

?> 클라이언트의 실제 `IP`를 가져 오려면 `$request->header['x-real-ip']`를 읽을 수 있습니다.
해당 문서에서는 다양한 방법에 대해 설명합니다.
### on()

?> **이벤트 콜백 함수를 등록합니다.**

?> [서버의 이벤트](/server/events)와 동일하지만 다음과 같은 차이가 있습니다:

   - `Http\Server->on`은 [onConnect](/server/events?id=onconnect)/[onReceive](/server/events?id=onreceive) 콜백을 설정하지 않습니다.
   - `Http\Server->on`은 새로운 이벤트 유형인 `onRequest`을 추가로 받습니다. 클라이언트에서 전송된 요청은 `Request` 이벤트에서 실행됩니다.

```php
$http_server->on('request', function(\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
     $response->end("<h1>hello swoole</h1>");
});
```

완전한 HTTP 요청을 수신한 후에이 함수를 콜백합니다. 콜백 함수에는 `2`개의 매개변수가 있습니다:

* [Swoole\Http\Request](/http_server?id=httpRequest), `HTTP` 요청 정보 객체로 `header/get/post/cookie` 등과 관련된 정보를 포함합니다.
* [Swoole\Http\Response](/http_server?id=httpResponse), `HTTP` 응답 객체로 `cookie/header/status` 등의 `HTTP` 작업을 지원합니다.

!> [onRequest](/http_server?id=on) 콜백 함수가 반환될 때 `$request`와 `$response` 객체가 소멸됩니다.
### start()

?> **HTTP 서버 시작**

?> 시작하면 포트를 청취하고 새로운 `HTTP` 요청을 받습니다.

```php
Swoole\Http\Server->start();
```
## Swoole\Http\Request

`HTTP` 요청 객체는 `HTTP` 클라이언트 요청과 관련된 정보를 저장하며 `GET`、`POST`、`COOKIE`、`Header` 등을 포함합니다.

!> `Http\Request` 객체를 참조할 때 `&` 기호를 사용하지 마십시오.
### 헤더

?> **`HTTP` 요청의 헤더 정보입니다. 배열 형식이며 모든 `key`는 소문자입니다.**

```php
Swoole\Http\Request->header: array
```

* **예시**

```php
echo $request->header['host'];
echo $request->header['accept-language'];
```
### 서버

?> **`HTTP` 요청과 관련된 서버 정보입니다.**

?> `PHP`의 `$_SERVER` 배열과 유사합니다. 요청 방법, `URL` 경로, 클라이언트 `IP` 등의 정보를 포함합니다.

```php
Swoole\Http\Request->server: array
```

배열의 키는 모두 소문자이며 `PHP`의 `$_SERVER` 배열과 일치합니다.

* **예시**

```php
echo $request->server['request_time'];
```

키 | 설명
---|---
query_string | `GET` 매개변수, 예: `id=1&cid=2`. `GET` 매개변수가 없는 경우에는 해당 항목이 존재하지 않습니다.
request_method | 요청 방법, `GET/POST` 등
request_uri | `GET` 매개변수가 없는 요청 주소, 예: `/favicon.ico`
path_info | `request_uri`와 동일
request_time | `request_time`은 `Worker`에서 설정됩니다. [SWOOLE_PROCESS](/learn?id=swoole_process) 모드에서는 `dispatch` 과정이 존재하기 때문에 실제 패킷 수신 시간과 차이가 있을 수 있습니다. 특히 서버 처리 능력을 초과하는 요청이 있을 때, `request_time`은 실제 패킷 수신 시간보다 오래 늦을 수 있습니다. 정확한 패킷 수신 시간을 얻으려면 `$server->getClientInfo` 메소드를 사용하여 `last_time`을 얻을 수 있습니다.
request_time_float | 요청이 시작된 시간을 마이크로초 단위의 `float` 유형으로 나타낸 타임스탬프, 예: `1576220199.2725`
server_protocol | 서버 프로토콜 버전 번호, `HTTP`는 `HTTP/1.0` 또는 `HTTP/1.1`이고, `HTTP2`는 `HTTP/2`입니다.
server_port | 서버가 수신 대기 중인 포트
remote_port | 클라이언트의 포트
remote_addr | 클라이언트의 `IP` 주소
master_time | 마지막 통신 시간부터의 경과 시간
### get

?> **`HTTP` 요청의 `GET` 매개변수는 `PHP`의 `$_GET`과 비슷하며, 배열 형식입니다.**

```php
Swoole\Http\Request->get: array
```

* **예시**

```php
// 예: index.php?hello=123
echo $request->get['hello'];
// 모든 GET 매개변수 가져오기
var_dump($request->get);
```

* **주의**

!> `해시` 공격을 방지하기 위해, `GET` 매개변수는 최대 `128`개를 초과할 수 없습니다.
### 게시물

?> **`HTTP` 요청의 `POST` 매개변수는 배열 형식입니다**

```php
Swoole\Http\Request->post: array
```

* **예시**

```php
echo $request->post['hello'];
```

* **주의**

!> - `POST` 및 `Header`의 크기가 [package_max_length](/server/setting?id=package_max_length) 설정을 초과해서는 안됩니다. 그렇지 않으면 악의적인 요청으로 간주될 수 있습니다  
- `POST` 매개변수의 최대 개수는 `128`개를 초과할 수 없습니다
### 쿠키

?> **`HTTP` 요청에서 전달되는 `쿠키` 정보는 키-값 쌍의 배열 형식입니다.**

```php
Swoole\Http\Request->cookie: array
```

* **예시**

```php
echo $request->cookie['username'];
```
### 파일

?> **파일 업로드 정보입니다.**

?> `form` 이름에 따른 이차원 배열입니다. `PHP`의 `$_FILES`와 동일합니다. 최대 파일 크기는 [package_max_length](/server/setting?id=package_max_length)에서 설정한 값 이하여야 합니다. Swoole이 메시지를 해석하는 동안 메모리를 사용하기 때문에 메시지가 크면 메모리 사용량도 커집니다. 따라서 `Swoole\Http\Server`로 대용량 파일 업로드를 처리하거나 사용자가 직접 중단된 업로드 기능을 설계하지 않도록 주의하시기 바랍니다.

```php
Swoole\Http\Request->files: array
```

* **예시**

```php
Array
(
    [name] => facepalm.jpg // 브라우저에서 업로드한 파일 이름
    [type] => image/jpeg // MIME 타입
    [tmp_name] => /tmp/swoole.upfile.n3FmFr // 업로드된 임시 파일, 파일 이름은 /tmp/swoole.upfile로 시작함
    [error] => 0
    [size] => 15476 // 파일 크기
)
```

* **주의**

!> `Swoole\Http\Request` 객체가 파괴될 때, 업로드된 임시 파일이 자동으로 삭제됩니다.
### getContent()

!> Swoole version >= `v4.5.0`에서 사용 가능하며, 이전 버전에서는 `rawContent` 별칭을 사용할 수 있습니다. (이 별칭은 계속 지원되며 하위 호환성이 유지됩니다)

?> **원시 `POST` 본문을 가져옵니다.**

?> `application/x-www-form-urlencoded` 형식이 아닌 HTTP `POST` 요청에 사용됩니다. 오리지널 `POST` 데이터를 반환하며, 이 함수는 `PHP`의 `fopen('php://input')`과 동일한 역할을 합니다.

```php
Swoole\Http\Request->getContent(): string|false
```

  * **Return Values**

    * 성공적으로 실행되면 메시지가 반환되며, 연결이 없을 시에는 `false`가 반환됩니다

!> 일부 경우에는 서버가 HTTP `POST` 요청 매개변수를 구문 분석할 필요가 없을 수 있습니다. [http_parse_post](/http_server?id=http_parse_post)를 통해 설정하여 `POST` 데이터 해석을 비활성화할 수 있습니다.
### getData()

?> **`Http2`에서는 사용할 수 없음에 주의하십시오. `Http Header` 및 `Http Body`를 포함한 완전한 원시 `Http` 요청 메시지를 가져옵니다.**

```php
Swoole\Http\Request->getData(): string|false
```

  * **Return Value**

    * 성공적으로 실행되면 메시지를 반환하고, 연결이 존재하지 않거나 `Http2` 모드에서는 `false`를 반환합니다.
### create()

?> **`Swoole\Http\Request` 객체를 생성합니다.**

!> Swoole 버전 >= `v4.6.0`에서 사용 가능

```php
Swoole\Http\Request->create(array $options): Swoole\Http\Request
```

  * **매개변수**

    * **`array $options`**
      * **기능**：선택적 매개변수로, `Request` 객체의 구성을 설정하는 데 사용됩니다.

| 매개변수                                              | 기본값 | 설명                                                                |
| ------------------------------------------------- | ------ | ----------------------------------------------------------------- |
| [parse_cookie](/http_server?id=http_parse_cookie) | true   | `Cookie`를 구문 분석할지 여부 설정                                               |
| [parse_body](/http_server?id=http_parse_post)      | true   | `Http Body`를 구문 분석할지 여부 설정                                    |
| [parse_files](/http_server?id=http_parse_files)   | true   | 파일 업로드 구문 분석을 설정합니다                                                   |
| enable_compression                                | true (서버가 압축을 지원하지 않는 경우 기본값은 false)   | 압축을 사용할지 여부 설정                                                   |
| compression_level                                 | 1      | 압축 수준을 설정합니다. 1-9 사이의 범위이며, 수준이 높을수록 압축 후 크기는 작아지지만 CPU 소비가 더 많아집니다        |
| upload_tmp_dir                                 | /tmp      | 임시 파일 저장 위치, 파일 업로드에 사용됩니다        |

  * **반환값**

    * `Swoole\Http\Request` 객체를 반환합니다.

* **예시**
```php
Swoole\Http\Request::create([
    'parse_cookie' => true,
    'parse_body' => true,
    'parse_files' => true,
    'enable_compression' => true,
    'compression_level' => 1,
    'upload_tmp_dir' => '/tmp',
]);
```
### parse()

?> **`HTTP` 요청 데이터를 해석하여 성공적으로 해석된 데이터 패킷의 길이를 반환합니다.**

!> Swoole 버전 >= `v4.6.0`에서 사용 가능

```php
Swoole\Http\Request->parse(string $data): int|false
```

  * **파라미터**

    * **`string $data`**
      * 해석하려는 데이터 패킷

  * **반환값**

    * 성공적으로 해석된 데이터 패킷 길이를 반환하며, 연결 컨텍스트가 없거나 컨텍스트가 이미 종료된 경우 `false`를 반환합니다.
### isCompleted()

?> **현재 `HTTP` 요청 데이터 패킷이 완료되었는지 확인합니다.**

!> Swoole 버전 >= `v4.6.0`에서 사용 가능

```php
Swoole\Http\Request->isCompleted(): bool
```

  * **반환 값**

    * `true`는 이미 완료된 것을 나타내며, `false`는 연결 컨텍스트가 종료되었거나 아직 끝에 도달하지 않은 것을 나타냅니다.

* **예시**

```php
use Swoole\Http\Request;

$data = "GET /index.html?hello=world&test=2123 HTTP/1.1\r\n";
$data .= "Host: 127.0.0.1\r\n";
$data .= "Connection: keep-alive\r\n";
$data .= "Pragma: no-cache\r\n";
$data .= "Cache-Control: no-cache\r\n";
$data .= "Upgrade-Insecure-Requests: \r\n";
$data .= "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36\r\n";
$data .= "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n";
$data .= "Accept-Encoding: gzip, deflate, br\r\n";
$data .= "Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,ja;q=0.6\r\n";
$data .= "Cookie: env=pretest; phpsessid=fcccs2af8673a2f343a61a96551c8523d79ea; username=hantianfeng\r\n";

/** @var Request $req */
$req = Request::create(['parse_cookie' => false]);
var_dump($req);

var_dump($req->isCompleted());
var_dump($req->parse($data));

var_dump($req->parse("\r\n"));
var_dump($req->isCompleted());

var_dump($req);
// parse_cookie가 닫혔기 때문에 null이 될 것입니다.
var_dump($req->cookie);
```
### getMethod()

?> **현재 `HTTP` 요청의 요청 방식을 가져옵니다.**

!> Swoole 버전 >= `v4.6.2`에서 사용 가능

```php
Swoole\Http\Request->getMethod(): string|false
```
  * **반환 값**

    * 요청 방식의 대문자를 반환하며, `false`는 연결 컨텍스트가 없음을 의미합니다.

```php
var_dump($request->server['request_method']);
var_dump($request->getMethod());
```
## Swoole\Http\Response

`HTTP` 응답 객체는 이 객체의 메소드를 호출하여 `HTTP` 응답을 보낼 수 있습니다.

?> `Response` 객체가 소멸될 때, `end` 메소드를 호출하지 않고 `HTTP` 응답을 보내지 않았다면, 자동으로 `end("")`가 실행됩니다.

!> `Http\Response` 객체를 참조할 때 `&` 기호를 사용하지 마십시오.
### header() :id=setheader

?> **HTTP 응답의 헤더 정보를 설정합니다**【setHeader 별칭】

```php
Swoole\Http\Response->header(string $key, string $value, bool $format = true): bool;
```

* **매개변수**

  * **`string $key`**
    * **기능**：`HTTP` 헤더의 `Key`
    * **기본값**：없음
    * **다른 값**：없음

  * **`string $value`**
    * **기능**：`HTTP` 헤더의 `value`
    * **기본값**：없음
    * **다른 값**：없음

  * **`bool $format`**
    * **기능**：`Key`를 `HTTP` 약속에 맞게 포맷팅해야 하는지 여부【기본값은 `true`로 자동 포맷팅됨】
    * **기본값**：`true`
    * **다른 값**：없음

* **반환 값**

  * 설정 실패 시 `false` 반환
  * 설정 성공 시 `true` 반환
* **주의 사항**

   - `header` 설정은 `end` 메서드 이전에 이루어져야 합니다.
   - `$key`는 완전히 `HTTP` 규정을 준수해야 하며, 각 단어의 첫 글자는 대문자여야 하며, 중국어, 밑줄 또는 다른 특수 문자를 포함해서는 안 됩니다.
   - `$value`를 작성해야 합니다.
   - `$ucwords`를 `true`로 설정하면 하위 레벨에서 `$key`를 자동으로 규정 포맷합니다.
   - 동일한 `$key`의 `HTTP` 헤더를 중복 설정하면 가장 최근 설정으로 덮어씁니다.
   - 클라이언트가 `Accept-Encoding`을 설정하면 서버는 `Content-Length` 응답을 설정할 수 없으며, 이 경우 `Swoole`은 `Content-Length`의 값을 무시하고 경고를 표시합니다.
   - `Content-Length`를 설정하면 응답에서 `Swoole\Http\Response::write()`를 호출할 수 없습니다. 이 경우 `Swoole`은 `Content-Length`의 값을 무시하고 경고를 표시합니다.

!> Swoole 버전 `v4.6.0` 이상에서 같은 `$key`의 `HTTP` 헤더를 중복으로 설정하고, `$value`가 `array`, `object`, `int`, `float`와 같은 다양한 유형을 지원하며, 하위에서 `toString` 변환을 수행하고 뒤에 공백과 개행을 제거합니다.

* **예시**

```php
$response->header('content-type', 'image/jpeg', true);

$response->header('Content-Length', '100002 ');
$response->header('Test-Value', [
    "a\r\n",
    'd5678',
    "e  \n ",
    null,
    5678,
    3.1415926,
]);
$response->header('Foo', new SplFileInfo('bar'));
```
### trailer()

?> **`Header` 정보를 `HTTP` 응답 끝에 첨부하는데 사용됩니다. `HTTP2`에서만 사용할 수 있으며, 메시지 무결성 검사, 디지털 서명 등에 사용됩니다.**

```php
Swoole\Http\Response->trailer(string $key, string $value): bool;
```

* **Parameter** 

  * **`string $key`**
    * **기능**：`HTTP` 헤더의 키
    * **기본값**：없음
    * **기타 값**：없음

  * **`string $value`**
    * **기능**：`HTTP` 헤더의 값
    * **기본값**：없음
    * **기타 값**：없음

* **Return** 

  * 설정 실패 시 `false` 반환
  * 설정 성공 시 `true` 반환

* **주의사항**

  !> 동일한 `$key`의 `Http` 헤더를 중복으로 설정하면 덮어쓰기되어 마지막 값만 유효합니다.

* **예시**

```php
$response->trailer('grpc-status', 0);
$response->trailer('grpc-message', '');
```
### cookie()

?> **`HTTP` 응답의 `cookie` 정보를 설정합니다. `setCookie`의 별칭입니다. 이 메서드의 매개변수는 `PHP`의 `setcookie`와 동일합니다.**

```php
Swoole\Http\Response->cookie(string $key, string $value = '', int $expire = 0 , string $path = '/', string $domain  = '', bool $secure = false , bool $httponly = false, string $samesite = '', string $priority = ''): bool;
```

  * **매개변수** 

    * **`string $key`**
      * **기능**：`Cookie`의 `Key`
      * **기본값**：없음
      * **다른 값**：없음

    * **`string $value`**
      * **기능**：`Cookie`의 `value`
      * **기본값**：없음
      * **다른 값**：없음
  
    * **`int $expire`**
      * **기능**：`Cookie`의 `만료 시간`
      * **기본값**：0, 만료 안 됨
      * **다른 값**：없음

    * **`string $path`**
      * **기능**：`Cookie의 서버 경로를 지정`
      * **기본값**：/
      * **다른 값**：없음

    * **`string $domain`**
      * **기능**：`Cookie의 도메인을 지정`
      * **기본값**：''
      * **다른 값**：없음

    * **`bool $secure`**
      * **기능**：`안전한 HTTPS 연결을 통해 Cookie를 전송할지 여부 지정`
      * **기본값**：''
      * **다른 값**：없음

    * **`bool $httponly`**
      * **기능**：`HttpOnly 속성이있는 Cookie에 대한 브라우저의 JavaScript 액세스를 허용하는지 여부`, `true`는 허용하지 않음, `false`는 허용함
      * **기본값**：false
      * **다른 값**：없음

    * **`string $samesite`**
      * **기능**：`보안 위험을 줄이기 위해 제3자 Cookie를 제한`, `Strict`, `Lax`, `None` 중 하나의 값 선택 가능
      * **기본값**：''
      * **다른 값**：없음

    * **`string $priority`**
      * **기능**：`Cookie 우선 순위, 규정을 초과하면 낮은 우선 순위의 것이 먼저 삭제됨`, `Low`, `Medium`, `High` 중 하나의 값 선택 가능
      * **기본값**：''
      * **다른 값**：없음
  
  * **반환값** 

    * 설정 실패 시 `false` 반환
    * 설정 성공 시 `true` 반환

* **주의사항**

  !> - `cookie` 설정은[end](/http_server?id=end) 메소드 이전에 이루어져야 합니다  
  - `v4.4.6` 버전부터 `$samesite` 매개변수가 지원됩니다, `v4.5.8` 버전부터 `$priority` 매개변수가 지원됩니다  
  - `Swoole`은 자동으로 `$value`를 `urlencode` 인코딩하며, `rawCookie()` 메소드를 사용하여 `$value`의 인코딩 처리를 해제할 수 있습니다  
  - `Swoole`은 동일한 `$key`의 여러 `COOKIE`를 설정할 수 있습니다
### rawCookie()

?> **`HTTP` 응답에 쿠키 정보 설정**

!> `rawCookie()`의 매개변수는 이전 `cookie()`와 동일하지만 인코딩 처리를 수행하지 않습니다.
### status()

?> **`Http` 상태 코드를 보냅니다. 별칭`setStatusCode()`**

```php
Swoole\Http\Response->status(int $http_status_code, string $reason = ''): bool
```

* **매개변수** 

  * **`int $http_status_code`**
    * **기능**：`HttpCode`를 설정합니다.
    * **기본값**：없음
    * **다른 값**：없음

  * **`string $reason`**
    * **기능**：상태 코드의 원인
    * **기본값**：''
    * **다른 값**：없음

  * **반환 값** 

    * 설정 실패 시 `false`를 반환합니다.
    * 설정 성공 시 `true`를 반환합니다.

* **팁**

  * 만약 첫 번째 매개변수만 전달하는 경우, `$http_status_code`는 반드시 유효한 `HttpCode`여야 합니다. 예를 들어 `200`, `502`, `301`, `404` 등이어야 하며, 그렇지 않으면 `200` 상태 코드로 설정됩니다.
  * 두 번째 매개변수 `$reason`을 설정한 경우, `$http_status_code`는 `499`와 같이 정의되지 않은 `HttpCode`를 포함하여 임의의 값이 될 수 있습니다.
  * `status` 메서드를 실행하기 전에 반드시 [$response->end()](/http_server?id=end)를 실행해야 합니다.
### gzip()

!> 이 메서드는 `4.1.0` 버전 이상에서 폐기되었으며, [http_compression](/http_server?id=http_compression)으로 이동하세요. 새로운 버전에서는 `gzip` 메서드 대신 `http_compression` 구성 옵션을 사용합니다.  
주된 이유는 `gzip()` 메서드가 브라우저 클라이언트에서 전송되는 `Accept-Encoding` 헤더를 확인하지 않아서, 클라이언트가 `gzip` 압축을 지원하지 않는 경우 강제로 사용하면 클라이언트가 압축을 풀 수 없습니다.  
새로운 `http_compression` 구성 옵션은 클라이언트의 `Accept-Encoding` 헤더에 따라 자동으로 압축 여부를 선택하고 최적의 압축 알고리즘을 자동으로 선택합니다.

?> **`Http GZIP` 압축을 활성화합니다. 압축을 사용하면 `HTML` 콘텐츠 크기를 줄일 수 있어 네트워크 대역폭을 효과적으로 절약하고 응답 시간을 향상시킬 수 있습니다. `write/end`로 내용을 보내기 전에 반드시 `gzip`를 실행해야 하며, 그렇지 않으면 오류가 발생할 수 있습니다.**
```php
Swoole\Http\Response->gzip(int $level = 1);
```

* **매개변수** 
   
     * **`int $level`**
       * **기능** : 압축 레벨, 레벨이 높을수록 압축된 크기가 작아지지만 `CPU` 소비가 더 많이 발생합니다.
       * **기본값** : 1
       * **다른 값** : `1-9`

!> `gzip` 메서드 호출 후에 내부적으로 `Http` 인코딩 헤더가 자동으로 추가됩니다. PHP 코드에서 관련 `Http` 헤더를 다시 설정하면 안 됩니다. `jpg/png/gif` 형식의 이미지는 이미 압축되었기 때문에 다시 압축할 필요가 없습니다.

!> `gzip` 기능은 `zlib` 라이브러리에 의존하며, swoole을 컴파일할 때 시스템에 `zlib`가 있는지 확인합니다. `zlib`가 없는 경우 `gzip` 메서드를 사용할 수 없습니다. `yum` 또는 `apt-get`을 사용하여 `zlib` 라이브러리를 설치할 수 있습니다:

```shell
sudo apt-get install libz-dev
```
### redirect()

?> **`Http` 리다이렉션을 보냅니다. 이 메서드를 호출하면 자동적으로 `end`가 발생하고 응답이 종료됩니다.**

```php
Swoole\Http\Response->redirect(string $url, int $http_code = 302): bool
```

  * **Parameters (파라미터)** 
* **Parameters (파라미터)** 
  * **Parameters (파라미터)** 
* **Parameters (파라미터)** 
  * **Parameters (파라미터)** 

    * **`string $url`**
      * **기능**: 새 주소로의 리다이렉션이 이루어질 때 `Location` 헤더로 보냄
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $http_code`**
      * **기능**: 상태 코드 [기본값은 `302`로 임시 리다이렉션, `301`을 전달하면 영구 리다이렉션을 표시합니다]
      * **기본값**: `302`
      * **기타 값**: 없음

  * **Return Value (반환 값)** 

    * 호출 성공 시 `true` 반환, 호출 실패 또는 연결된 컨텍스트가 없을 경우 `false` 반환

* **Example (예시)**

```php
$http = new Swoole\Http\Server("0.0.0.0", 9501, SWOOLE_BASE);

$http->on('request', function ($req, Swoole\Http\Response $resp) {
    $resp->redirect("http://www.baidu.com/", 301);
});

$http->start();
```
### write()

?> **`Http Chunk`을 활성화하여 브라우저에 응답 콘텐츠를 전송합니다.**

?> `Http Chunk`에 대한 자세한 내용은 `Http` 프로토콜 표준 문서를 참조해주세요.

```php
Swoole\Http\Response->write(string $data): bool
```

  * **매개변수** 

    * **`string $data`**
      * **기능**：전송할 데이터 내용【최대 길이는 `2M`를 초과할 수 없으며 [buffer_output_size](/server/setting?id=buffer_output_size) 구성 항목에 의해 제어됨】
      * **기본값**：없음
      * **다른 값**：없음

  * **반환 값** 
  
    * 호출이 성공하면 `true`가 반환되고, 호출이 실패하거나 연결 컨텍스트가 없는 경우에는 `false`가 반환됩니다.

* **팁**

  * 데이터를 `write`를 통해 분할하여 전송한 후, [end](/http_server?id=end) 메서드는 어떤 매개변수도 받지 않습니다. `end`를 호출하면 데이터 전송이 완료된 것을 나타내는 길이가 `0`인 `Chunk`가 전송됩니다.
  * 만약 `Swoole\Http\Response::header()` 메서드로 `Content-Length`를 설정한 후 이 메서드를 호출하면, `Swoole`은 `Content-Length` 설정을 무시하고 경고를 발생시킵니다.
  * `Http2`에서는 이 함수를 사용할 수 없으며, 그렇다면 경고가 발생합니다.
  * 클라이언트가 응답 압축을 지원하는 경우, `Swoole\Http\Response::write()`는 응답 압축을 강제로 비활성화합니다.
### sendfile()

?> **브라우저로 파일을 전송합니다.**

```php
Swoole\Http\Response->sendfile(string $filename, int $offset = 0, int $length = 0): bool
```

  * **매개변수** 

    * **`string $filename`**
      * **기능**：전송할 파일의 이름【파일이 없거나 액세스 권한이 없으면 `sendfile`이 실패합니다.】
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $offset`**
      * **기능**：파일의 오프셋【파일의 중간 부분부터 데이터를 전송할 수 있습니다. 이 기능은 이어받기를 지원하는 데 사용될 수 있습니다.】
      * **기본값**：`0`
      * **기타 값**：없음

    * **`int $length`**
      * **기능**：전송할 데이터의 크기
      * **기본값**：파일의 크기
      * **기타 값**：없음

  * **반환 값** 

      * 호출 성공 시 `true`를 반환하고, 호출 실패 또는 연결 컨텍스트가 없는 경우 `false`를 반환합니다.

* **팁**

  * 하위 수준에서 전송할 파일의 MIME 형식을 추론할 수 없기 때문에 응용 프로그램 코드에서 `Content-Type`을 지정해야 합니다.
  * `sendfile`을 호출하기 전에 `write` 메서드를 사용하여 `Http-Chunk`를 보내서는 안 됩니다.
  * `sendfile`을 호출한 후에는 하위 수준에서 자동으로 `end`가 실행됩니다.
  * `sendfile`은 `gzip` 압축을 지원하지 않습니다.

* **예제**

```php
$response->header('Content-Type', 'image/jpeg');
$response->sendfile(__DIR__.$request->server['request_uri']);
```
### end()

?> **`Http` 응답 바디를 전송하고 요청 처리를 종료합니다.**

```php
Swoole\Http\Response->end(string $html): bool
```

  * **매개변수** 
  
    * **`string $html`**
      * **기능** : 전송할 내용
      * **기본값** : 없음
      * **다른 값** : 없음

  * **반환값** 

    * 호출이 성공하면 `true`를 반환하고, 호출이 실패하거나 연결 상황이 없으면 `false`를 반환합니다

* **팁**

  * `end`는 한 번만 호출 가능하며, 클라이언트에 데이터를 여러 번 보내야 하는 경우 [write](/http_server?id=write) 메서드를 사용해야 합니다.
  * 클라이언트가 [KeepAlive](/coroutine_client/http_client?id=keep_alive)를 활성화한 경우, 연결은 유지되며 서버는 다음 요청을 기다립니다.
  * 클라이언트가 `KeepAlive`를 비활성화한 경우, 서버는 연결을 끊습니다.
  * `end`에 전송할 내용은 [output_buffer_size](/server/setting?id=buffer_output_size) 제한을 받기 때문에 기본값은 `2M`입니다. 이 한계를 초과하면 응답이 실패하고 다음과 같은 오류가 발생합니다:

!> 문제를 해결하기 위해 [sendfile](/http_server?id=sendfile), [write](/http_server?id=write)를 사용하거나 [output_buffer_size](/server/setting?id=buffer_output_size)를 조정하세요.

```bash
WARNING finish (ERRNO 1203): The length of data [262144] exceeds the output buffer size[131072], please use the sendfile, chunked transfer mode or adjust the output_buffer_size
```
### detach()

?> **응답 객체를 분리합니다.** 이 메서드를 사용하면 `$response` 객체가 제거될 때 자동으로 종료되지 않고, [Http\Response::create](/http_server?id=create) 및 [Server->send](/server/methods?id=send)와 함께 사용합니다.

```php
Swoole\Http\Response->detach(): bool
```

  * **반환값** 

    * 성공적으로 호출되면 `true`를 반환하고, 실패하거나 연결 컨텍스트가 없으면 `false`를 반환합니다.

* **예시** 

  * **프로세스 간 응답**

  ?> 일부 상황에서 [작업자 프로세스](/learn?id=taskworker-프로세스)에서 클라이언트에게 응답을 보내야 하는 경우가 있습니다. 이때 `detach`를 사용하여 `$response` 객체를 독립적으로 만들 수 있습니다. [작업자 프로세스](/learn?id=taskworker-프로세스)에서는 `$response`를 재구성하여 `Http` 요청 응답을 발생시킬 수 있습니다.

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->set(['task_worker_num' => 1, 'worker_num' => 1]);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->task(strval($resp->fd));
  });

  $http->on('finish', function () {
      echo "task finish";
  });

  $http->on('task', function ($serv, $task_id, $worker_id, $data) {
      var_dump($data);
      $resp = Swoole\Http\Response::create($data);
      $resp->end("in task");
      echo "async task\n";
  });

  $http->start();
  ```

  * **임의의 내용 전송**

  ?> 일부 특수한 시나리오에서 클라이언트에게 특수한 응답 콘텐츠를 전송해야 하는 경우가 있습니다. `Http\Response` 객체에 내장된 `end` 방법만으로는 요구 사항을 충족시키지 못하는 경우가 있습니다. `detach`를 사용하여 응답 객체를 분리한 후 HTTP 프로토콜 응답 데이터를 직접 조립하고 `Server->send`를 사용하여 데이터를 전송할 수 있습니다.

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->send($resp->fd, "HTTP/1.1 200 OK\r\nServer: server\r\n\r\nHello World\n");
  });

  $http->start();
  ```
### create()

?> **새로운 `Swoole\Http\Response` 객체를 생성합니다.**

!> 이 메서드를 사용하기 전에 이전의 `$response` 객체를 분리하기 위해 `detach` 메서드를 호출해야 합니다. 그렇지 않으면 동일한 요청에 두 번의 응답 내용을 보낼 수 있습니다.

```php
Swoole\Http\Response::create(object|array|int $server = -1, int $fd = -1): Swoole\Http\Response
```

* **매개변수**

  * **`int $server`**
    * **기능**: `Swoole\Server` 또는 `Swoole\Coroutine\Socket` 객체, 배열 (배열은 `Swoole\Server` 객체와 `Swoole\Http\Request` 객체 두 개의 매개변수만 포함해야 합니다), 또는 파일 기술자
    * **기본값**: -1
    * **다른 값**: 없음

  * **`int $fd`**
    * **기능**: 파일 기술자. 매개변수 `$server`가 `Swoole\Server` 객체인 경우, `$fd`가 필수입니다.
    * **기본값**: -1
    * **다른 값**: 없음

* **반환값**

  * 성공 시 새로운 `Swoole\Http\Response` 객체를 반환하고, 실패 시 `false`를 반환합니다.

* **예시**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
    $resp->detach();
    // 예시1
    $resp2 = Swoole\Http\Response::create($req->fd);
    // 예시2
    $resp2 = Swoole\Http\Response::create($http, $req->fd);
    // 예시3
    $resp2 = Swoole\Http\Response::create([$http, $req]);
    // 예시4
    $socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
    $socket->connect('127.0.0.1', 9501)
    $resp2 = Swoole\Http\Response::create($socket);
    $resp2->end("hello world");
});

$http->start();
```
### isWritable()

?> **`Swoole\Http\Response` 객체가 종료되었거나 분리되었는지 확인합니다.**

```php
Swoole\Http\Response->isWritable(): bool
```

  * **반환값** 

    * `Swoole\Http\Response` 객체가 종료되지 않았거나 분리되지 않았으면 `true`를 반환하고, 그렇지 않으면 `false`를 반환합니다.


!> Swoole 버전 >= `v4.6.0`에서 사용 가능

* **예시**

```php
use Swoole\Http\Server;
use Swoole\Http\Request;
use Swoole\Http\Response;

$http = new Server('0.0.0.0', 9501);

$http->on('request', function (Request $req, Response $resp) {
    var_dump($resp->isWritable()); // true
    $resp->end('hello');
    var_dump($resp->isWritable()); // false
    $resp->setStatusCode(403); // http response is unavailable (maybe it has been ended or detached)
});

$http->start();
```
## 설정 옵션
### http_parse_cookie

?> **`Swoole\Http\Request` 객체를 대상으로 하는 설정입니다. `Cookie` 파싱을 비활성화하여 `header`에 처리되지 않은 원시 `Cookies` 정보가 유지됩니다. 기본적으로 활성화됩니다.**

```php
$server->set([
    'http_parse_cookie' => false,
]);
```
?> **`Swoole\Http\Request` 객체를 위한 설정으로, POST 메시지 파싱을 활성화 또는 비활성화할 수 있습니다. 기본값은 활성화 상태입니다.**

* `Content-Type`이 `x-www-form-urlencoded`인 요청의 본문을 자동으로 `POST` 배열로 파싱합니다.
* `false`로 설정하면 `POST` 파싱이 비활성화됩니다.

```php
$server->set([
    'http_parse_post' => false,
]);
```
### http_parse_files

?> **`Swoole\Http\Request` 객체를 대상으로 하는 설정으로, 파일 업로드 구문 분석 스위치를 설정합니다. 기본적으로 활성화되어 있습니다.**

```php
$server->set([
    'http_parse_files' => false,
]);
```
### http_compression

?> **`Swoole\Http\Response` 객체에 대한 구성으로 압축을 활성화합니다. 기본적으로 활성화되어 있습니다.**

!> - `http-chunk`는 분할된 데이터의 개별 압축을 지원하지 않으며, [write](/http_server?id=write) 메서드를 사용할 경우 압축이 강제로 비활성화됩니다.  
- `http_compression`은 `v4.1.0` 이상에서 사용 가능합니다.

```php
$server->set([
    'http_compression' => false,
]);
```

현재 지원되는 압축 형식은 `gzip`, `br`, `deflate` 세 가지이며, 하위에서는 브라우저 클라이언트가 전송하는 `Accept-Encoding` 헤더를 기반으로 자동으로 압축 방식을 선택합니다 (압축 알고리즘 우선 순위: `br` > `gzip` > `deflate`).

**의존성:**

`gzip`와 `deflate`는 `zlib` 라이브러리에 의존하며, `Swoole`을 컴파일할 때 하위에서 시스템에 `zlib`가 있는지 확인합니다.

`zlib` 라이브러리를 설치하려면 `yum` 또는 `apt-get`을 사용할 수 있습니다:

```shell
sudo apt-get install libz-dev
```

`br` 압축 형식은 `google`의 `brotli` 라이브러리에 의존하며, 설치 방법에 대해서는 `install brotli on linux`를 검색하여 확인할 수 있습니다. `Swoole`을 컴파일할 때 하위에서 시스템에 `brotli`가 있는지 확인합니다.
### http_compression_level / compression_level / http_gzip_level

?> ** `Swoole\Http\Response` 객체에 대한 압축 수준 구성**
  
!> `$level` 압축 수준은 `1-9`의 범위이며, 높은 수준일수록 압축 후 크기가 더 작지만 `CPU` 소비가 많아집니다. 기본값은 `1`이며, 최대값은 `9`입니다.
### http_compression_min_length / compression_min_length

?> **`Swoole\Http\Response` 객체에 대한 설정으로, 압축을 시작하기 전에 지정된 최소 바이트 수를 설정합니다. 기본값은 20바이트입니다.**

!> Swoole 버전 `v4.6.3` 이상에서 사용 가능

```php
$server->set([
    'compression_min_length' => 128,
]);
```
### upload_tmp_dir

?> **업로드 파일의 임시 디렉터리를 설정합니다. 디렉터리 이름의 최대 길이는 `220`바이트를 초과할 수 없습니다.**

```php
$server->set([
    'upload_tmp_dir' => '/data/uploadfiles/',
]);
```
### upload_max_filesize

?> **업로드 파일의 최대 크기 설정**

```php
$server->set([
    'upload_max_filesize' => 5 * 1024,
]);
```  
### enable_static_handler

정적 파일 요청 처리 기능을 활성화하려면 `document_root`와 함께 사용해야 합니다. 기본값은 `false`입니다.
### http_autoindex

`http autoindex` 기능을 활성화합니다. 기본적으로 비활성화되어 있습니다.
```php
$server->set([
    'document_root' => '/data/webroot/example.com',
    'enable_static_handler' => true,
    'http_autoindex' => true,
    'http_index_files' => ['indesx.html', 'index.txt'],
]);
```
```plaintext
`http_autoindex`와 함께 사용하여 색인할 파일 목록을 지정합니다.
```
### http_compression_types / compression_types

?> **`Swoole\Http\Response` 객체에 대한 응답 유형 압축을 설정합니다.**

```php
$server->set([
        'http_compression_types' => [
            'text/html',
            'application/json'
        ],
    ]);
```

!> `Swoole` 버전 >= `v4.8.12` 에서 사용 가능
### static_handler_locations

?> **정적 핸들러의 경로를 설정합니다. 배열 형식이며 기본값은 사용되지 않습니다.**

!> Swoole 버전 >= `v4.4.0`에서 사용 가능

```php
$server->set([
    'static_handler_locations' => ['/static', '/app/images'],
]);
```

* `Nginx`의 `location` 지시문과 유사하며, 하나 이상의 경로를 정적 경로로 지정할 수 있습니다. 지정된 경로 아래에 있는 `URL`만 정적 파일 핸들러를 활성화하고, 그렇지 않으면 동적 요청으로 처리됩니다.
* `location` 항목은 반드시 슬래시(`/`)로 시작해야 합니다.
* `/app/images`와 같은 다중 경로를 지원합니다.
* `static_handler_locations`를 활성화한 후 요청에 해당하는 파일이 없으면 직접 404 오류를 반환합니다.
### open_http2_protocol

?> **`HTTP2` 프로토콜 파싱 활성화**【기본값：`false`】

!> [--enable-http2](/environment?id=编译选项) 옵션을 사용하여 컴파일해야 하며, `Swoole5`부터는 http2가 기본으로 컴파일됩니다.
### document_root

?> **정적 파일 루트 디렉토리를 설정하고 `enable_static_handler`와 함께 사용합니다.**

!> 이 기능은 매우 간단합니다. 공용 네트워크 환경에서 직접 사용하지 마십시오.

```php
$server->set([
    'document_root' => '/data/webroot/example.com', // v4.4.0 이하 버전에서, 여기에는 절대 경로를 지정해야 합니다.
    'enable_static_handler' => true,
]);
```

* `document_root`를 설정하고 `enable_static_handler`를 `true`로 설정한 후, 하위 레벨에서 `Http` 요청을 받으면 먼저 document_root 경로에 파일이 있는지 확인하고, 있다면 파일 내용을 직접 클라이언트로 전송하여 [onRequest](/http_server?id=on) 콜백을 호출하지 않습니다.
* 정적 파일 처리 기능을 사용할 때는 동적 PHP 코드와 정적 파일을 분리하여야 합니다. 정적 파일을 특정 디렉토리에 저장해야 합니다.
### max_concurrency

?> **`HTTP1/2` 서비스의 최대 동시 요청 수를 제한할 수 있으며, 초과 시 `503` 에러를 반환합니다. 기본값은 4294967295이며, 즉 부호 없는 정수의 최댓값입니다.**

```php
$server->set([
    'max_concurrency' => 1000,
]);
```
### worker_max_concurrency

?> **일괄 코루틴 활성화 후 `worker` 프로세스는 계속해서 요청을 받습니다. 과도한 압력을 피하기 위해 `worker`프로세스의 요청 실행 수를 제한하는 `worker_max_concurrency`를 설정할 수 있습니다. 요청 수가 해당 값을 초과하면`worker`프로세스는 초과된 요청을 대기열에 보관합니다. 기본값은 4294967295로, 부호 없는 int의 최대값입니다. 만약 `worker_max_concurrency`를 설정하지 않았지만, `max_concurrency`를 설정한 경우, 하위 레벨에서`worker_max_concurrency`가 자동으로 `max_concurrency`와 동일하게 설정됩니다.**

```php
$server->set([
    'worker_max_concurrency' => 1000,
]);
```

!> Swoole 버전 >= `v5.0.0`에서 사용 가능합니다.
### http2_header_table_size

?> HTTP/2 네트워크 연결의 최대 'header table' 크기를 정의합니다.

```php
$server->set([
  'http2_header_table_size' => 0x1
])
```
### http2_enable_push

?> 이 구성은 HTTP2 푸시를 활성화 또는 비활성화하는 데 사용됩니다.

```php
$server->set([
  'http2_enable_push' => 0x2
])
```
### http2_max_concurrent_streams

?> 각 HTTP/2 네트워크 연결에서 허용되는 다중화 스트림의 최대 수를 설정합니다.

```php
$server->set([
  'http2_max_concurrent_streams' => 0x3
])
```  
### http2_init_window_size

?> HTTP/2 트래픽 제어 창의 초기 크기를 설정합니다.

```php
$server->set([
  'http2_init_window_size' => 0x4
])
```
### http2_max_frame_size

?> 단일 HTTP/2 프로토콜 프레임의 바디에 대한 최대 크기를 설정합니다.

```php
$server->set([
  'http2_max_frame_size' => 0x5
])
```
### http2_max_header_list_size

?> HTTP/2 스트림에서 전송할 수 있는 헤더의 최대 크기를 설정합니다. 

```php
$server->set([
  'http2_max_header_list_size' => 0x6
])
```
