# 협동 코루틴 HTTP/WebSocket 클라이언트

협동 코루틴 버전의 `HTTP` 클라이언트는 순수 `C`로 작성되어 있으며 어떠한 서드 파티 확장 라이브러리에도 의존하지 않으며 매우 뛰어난 성능을 자랑합니다.

* `Http-Chunk`, `Keep-Alive` 기능을 지원하며, `form-data` 형식을 지원합니다.
* `HTTP` 프로토콜 버전은 `HTTP/1.1` 입니다.
* `WebSocket` 클라이언트로 업그레이드 가능합니다.
* `gzip` 압축 형식은 `zlib` 라이브러리에 의존합니다.
* 클라이언트는 핵심 기능만 구현되어 있으며, 실제 프로젝트에서는 [Saber](https://github.com/swlib/saber)를 사용하는 것이 좋습니다.
## 속성
### errCode

오류 상태 코드. `connect/send/recv/close` 작업이 실패하거나 시간 초과되면 `Swoole\Coroutine\Http\Client->errCode`의 값이 자동으로 설정됩니다.

```php
Swoole\Coroutine\Http\Client->errCode: int
```

`errCode` 값은 `Linux errno`와 같습니다. `socket_strerror` 함수를 사용하여 오류 코드를 오류 메시지로 변환할 수 있습니다.

```php
// 만약 connect refuse라면, 오류 코드는 111입니다
// 만약 시간 초과라면, 오류 코드는 110입니다
echo socket_strerror($client->errCode);
```

!> 참조: [Linux 에러 코드 목록](/other/errno?id=linux)
### body

마지막 요청의 응답 본문을 저장합니다.

```php
Swoole\Coroutine\Http\Client->body: string
```

  * **예제**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->get('/get');
    echo $cli->body;
    $cli->close();
});
```
### statusCode

HTTP 상태 코드, 예를 들어 200, 404 등입니다. 음수인 경우 연결에 문제가 있음을 나타냅니다. [더 보기](/coroutine_client/http_client?id=getstatuscode)

```php
Swoole\Coroutine\Http\Client->statusCode: int
```
아래는 웹 애플리케이션의 메인 페이지를 렌더링하는 Express 라우팅 코드입니다.

```javascript
app.get('/', (req, res) => {
  res.render('index');
});
```
### __construct()

생성자 메서드.

```php
Swoole\Coroutine\Http\Client::__construct(string $host, int $port, bool $ssl = false);
```

  * **매개변수**

    * **`string $host`**
      * **기능**: 대상 서버 호스트 주소【IP 또는 도메인이 될 수 있으며, 하부에서 자동으로 도메인을 해석합니다. 로컬 UNIX 소켓인 경우에는 `unix://tmp/your_file.sock` 형식으로 입력해야 합니다. 도메인인 경우에는 `http://` 또는 `https://` 프로토콜 헤더를 적지 않아도 됩니다.】
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`int $port`**
      * **기능**: 대상 서버 호스트 포트
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`bool $ssl`**
      * **기능**: `SSL/TLS` 터널 암호화를 사용할지 여부. 대상 서버가 https인 경우에는 `true`로 설정해야 합니다.
      * **기본값**: `false`
      * **다른 값**: 없음

  * **예시**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->set(['timeout' => 1]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```
### set()

클라이언트 매개변수를 설정합니다.

```php
Swoole\Coroutine\Http\Client->set(array $options);
```

이 메서드는 `Swoole\Client->set`이 받는 매개변수와 완전히 동일하며, [Swoole\Client->set](/client?id=set) 메서드의 설명을 참조할 수 있습니다.

`Swoole\Coroutine\Http\Client`는 `HTTP` 및 `WebSocket` 클라이언트를 제어하기 위해 몇 가지 추가 옵션을 추가했습니다.
#### 추가 옵션
##### 시간 초과 제어

`timeout` 옵션을 설정하여 HTTP 요청 시간 초과 감지를 활성화합니다. 시간은 초 단위로 지정하며, 최소 값은 밀리초까지 지원됩니다.

```php
$http->set(['timeout' => 3.0]);
```

- 연결 시간 초과 또는 서버에서 연결이 종료될 경우 `statusCode`가 `-1`로 설정됩니다.
- 약속된 시간 내에 서버가 응답을 반환하지 않으면 요청 시간이 초과되었음을 나타내기 위해 `statusCode`가 `-2`로 설정됩니다.
- 요청 시간 초과 후에는 하위 수준에서 자동으로 연결이 끊깁니다.
- [클라이언트 시간 초과 규칙](/coroutine_client/init?id=시간 초과 규칙)을 참조하세요.
##### keep_alive

`keep_alive` 옵션을 설정하여 HTTP 장기 연결을 활성화하거나 비활성화합니다.

```php
$http->set(['keep_alive' => false]);
```
##### websocket_mask

> RFC 규정에 따라, v4.4.0 이후에는이 설정이 기본적으로 활성화되지만 성능 저하를 초래할 수 있습니다. 서버 측에서 강제로 필요하지 않은 경우 false로 설정하여 비활성화할 수 있습니다.

`WebSocket` 클라이언트에서 마스크를 활성화하거나 비활성화합니다. 기본값은 활성화입니다. 활성화되면 `WebSocket` 클라이언트가 보낸 데이터에 마스크를 사용하여 데이터 변환을 수행합니다.

```php
$http->set(['websocket_mask' => false]);
```
##### websocket_compression

> `v4.4.12` 이상의 버전이 필요합니다.

`true`로 설정하면 프레임을 zlib으로 압축하는 것을 **허용**합니다. 실제로 압축을 수행할 수 있는지는 서버가 압축을 처리할 수 있는지에 따라 달라지며, 이는 핸드셰이크 정보에 따라 결정됩니다 (`RFC-7692` 참조).

특정 프레임을 실제로 압축하려면 `SWOOLE_WEBSOCKET_FLAG_COMPRESS` 플래그 매개변수와 함께 사용해야하며, 자세한 사용 방법은 [이 부분을 참조하세요](/websocket_server?id=websocket-frames-compression-(rfc-7692))

```php
$http->set(['websocket_compression' => true]);
```
### setMethod()

요청 방법을 설정합니다. 현재 요청에서만 유효하며 요청을 보낸 후 바로 방법 설정이 지워집니다.

```php
Swoole\Coroutine\Http\Client->setMethod(string $method): void
```

  * **매개변수** 

    * **`string $method`**
      * **기능**：방법 설정 
      * **기본값**：없음
      * **다른 값**：없음

      !> `$method`는 `HTTP` 표준에 부합하는 방법 이름이어야 하며, 잘못된 설정일 경우 `HTTP` 서버에서 요청이 거부될 수 있습니다.

  * **예시**

```php
$http->setMethod("PUT");
```
### setHeaders()

HTTP 요청 헤더를 설정합니다.

```php
Swoole\Coroutine\Http\Client->setHeaders(array $headers): void
```

  * **매개변수** 

    * **`array $headers`**
      * **기능**：요청 헤더를 설정합니다. 【키-값 쌍 배열이어야 하며, 하위 수준에서`$key`: `$value` 형식의`HTTP` 표준 헤더 형식으로 자동 변환됩니다】
      * **기본값**：없음
      * **다른 값**：없음

!> `setHeaders`로 설정된 `HTTP` 헤더는 `Coroutine\Http\Client` 객체의 수명 동안 모든 요청에 영구적으로 적용됩니다. `setHeaders`를 다시 호출하면 이전 설정이 덮어쓰기됩니다.
### setCookies()

`Cookie`를 설정하고 값은 `urlencode`로 처리되며, 원본 정보를 유지하려면 `Cookie`라는 `header` 이름을 직접 설정해야 합니다.

```php
Swoole\Coroutine\Http\Client->setCookies(array $cookies): void
```

  * **Parameters** 

    * **`array $cookies`**
      * **Description**: `COOKIE`를 설정합니다. 【배열 형태여야 함】
      * **Default**: 없음
      * **Other values**: 없음

!> - `COOKIE`를 설정하면 클라이언트 객체의 수명 동안 지속됩니다.  
- 서버에서 설정한 `COOKIE`는 `cookies` 배열에 병합되며, `$client->cookies` 속성을 통해 현재 `HTTP` 클라이언트의 `COOKIE` 정보를 얻을 수 있습니다.  
- `setCookies` 메소드를 반복 호출하면 현재 `Cookies` 상태가 덮어씌워집니다. 이는 이전에 서버에서 발급한 `COOKIE` 및 이전에 설정한 `COOKIE`를 제거하는 결과를 가져옵니다.
### setData()

HTTP 요청의 본문을 설정합니다.

```php
Swoole\Coroutine\Http\Client->setData(string|array $data): void
```

  * **매개변수** 

    * **`string|array $data`**
      * **기능** : 요청의 본문을 설정합니다.
      * **기본값** : 없음
      * **다른 값** : 없음

  * **팁**

    * `$data`를 설정한 후에 `$method`를 설정하지 않은 경우, 하위 레벨에서 자동으로 POST로 설정됩니다.
    * `$data`가 배열이고 `Content-Type`이 `urlencoded` 형식인 경우 하위 레벨에서 자동으로 `http_build_query`를 수행합니다.
    * `addFile` 또는 `addData`를 사용하여 `form-data` 형식을 활성화한 경우, `$data` 값이 문자열인 경우(형식이 다른 경우) 무시되지만 배열인 경우 하위 레벨에서 배열의 필드를 `form-data` 형식으로 추가합니다.
### addFile()

파일을 POST로 추가합니다.

!> `addFile`를 사용하면 `POST`의 `Content-Type`이 자동으로 `form-data`로 변경됩니다. `addFile`은 `sendfile`을 기반으로 하며 대용량 파일의 비동기 전송을 지원합니다.

```php
Swoole\Coroutine\Http\Client->addFile(string $path, string $name, string $mimeType = null, string $filename = null, int $offset = 0, int $length = 0): void
```

  * **Parameters** 

    * **`string $path`**
      * **기능**： 파일 경로【필수 매개변수로, 빈 파일이거나 존재하지 않는 파일은 허용되지 않음】
      * **기본값**： 없음
      * **다른 값**： 없음

    * **`string $name`**
      * **기능**： 폼 이름【필수 매개변수로, `FILES` 매개변수에서의 키】
      * **기본값**： 없음
      * **다른 값**： 없음

    * **`string $mimeType`**
      * **기능**： 파일의 `MIME` 형식【선택적 매개변수로, 파일의 확장자를 기반으로 자동 추론됨】
      * **기본값**： 없음
      * **다른 값**： 없음

    * **`string $filename`**
      * **기능**： 파일 이름【선택적 매개변수】
      * **기본값**： `$path`의 기본 이름
      * **다른 값**： 없음

    * **`int $offset`**
      * **기능**： 파일 전송의 오프셋【선택적 매개변수로, 파일의 중간 부분부터 데이터를 전송할 수 있음. 이 기능은 이어받기를 지원하는 데 사용됨.】
      * **기본값**： 없음
      * **다른 값**： 없음

    * **`int $length`**
      * **기능**： 데이터 크기 전송【선택적 매개변수】
      * **기본값**： 파일 전체 크기로 설정됨
      * **다른 값**： 없음

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $cli->set(['timeout' => -1]);
    $cli->addFile(__FILE__, 'file1', 'text/plain');
    $cli->post('/post', ['foo' => 'bar']);
    echo $cli->body;
    $cli->close();
});
```
### addData()

파일을 업로드할 내용을 문자열로 작성합니다.

!> `addData`는 `v4.1.0` 이상 버전에서 사용할 수 있습니다.

```php
Swoole\Coroutine\Http\Client->addData(string $data, string $name, string $mimeType = null, string $filename = null): void
```

  * **Parameters** 

    * **`string $data`**
      * **Description**: 데이터 내용【필수 매개변수이며, 최대 길이는 [buffer_output_size](/server/setting?id=buffer_output_size)를 초과할 수 없음】
      * **Default**: 없음
      * **Other values**: 없음

    * **`string $name`**
      * **Description**: 폼의 이름【필수 매개변수, `$_FILES` 매개변수의 `key`】
      * **Default**: 없음
      * **Other values**: 없음

    * **`string $mimeType`**
      * **Description**: 파일의 `MIME` 형식【선택적 매개변수, 기본값은 `application/octet-stream`】
      * **Default**: 없음
      * **Other values**: 없음

    * **`string $filename`**
      * **Description**: 파일 이름【선택적 매개변수, 기본값은 `$name`】
      * **Default**: 없음
      * **Other values**: 없음

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('httpbin.org', 80);
    $client->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $client->set(['timeout' => -1]);
    $client->addData(Co::readFile(__FILE__), 'file1', 'text/plain');
    $client->post('/post', ['foo' => 'bar']);
    echo $client->body;
    $client->close();
});
```
### get()

GET 요청을 시작합니다.

```php
Swoole\Coroutine\Http\Client->get(string $path): void
```

  * **매개변수** 

    * **`string $path`**
      * **기능**：`URL` 경로를 설정합니다【예: `/index.html`，여기에는 `http://domain`을 전달할 수 없음】
      * **기본값**：없음
      * **기타값**：없음

  * **예시**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```

!> `get`을 사용하면 `setMethod`로 설정한 요청 방법이 무시되고 `GET`이 강제로 사용됩니다.
### post()

POST 요청을 보냅니다.

```php
Swoole\Coroutine\Http\Client->post(string $path, mixed $data): void
```

* **매개변수**

  * **`string $path`**
    * **기능**: `URL` 경로를 설정합니다. [`/index.html`과 같이 `http://domain`은 전달할 수 없음]
    * **기본값**: 없음
    * **기타값**: 없음

  * **`mixed $data`**
    * **기능**: 요청 패킷 데이터
    * **기본값**: 없음
    * **기타값**: 없음

    !> `$data`가 배열인 경우 하위 수준에서 자동으로 `x-www-form-urlencoded` 형식의 `POST` 내용으로 패키지화되며 `Content-Type`은 `application/x-www-form-urlencoded`로 설정됩니다.

* **주의사항**

  !> `post`를 사용하면 `setMethod`로 설정한 요청 방법은 무시되고 강제로 `POST`가 사용됩니다.

* **예제**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->post('/post.php', array('a' => '123', 'b' => '456'));
    echo $client->body;
    $client->close();
});
```
### upgrade()

`WebSocket` connection 으로 업그레이드합니다.

```php
Swoole\Coroutine\Http\Client->upgrade(string $path): bool
```

  * **Parameters** 

    * **`string $path`**
      * **기능**：`URL` 경로를 설정합니다【예: `/`，여기에 `http://domain`을 전달할 수 없습니다】
      * **기본값**：없음
      * **다른 값**：없음

  * **팁**

    * 경우에 따라 요청은 성공했지만, `upgrade`는 `true`를 반환했지만 서버가 `HTTP` 상태 코드를 `101`이 아닌 `200` 또는 `403`으로 설정하지 않은 경우가 있습니다. 이는 서버가 핸드셰이크 요청을 거부했음을 의미합니다.
    * `WebSocket` 핸드셰이크가 성공하면 `push` 메서드를 사용하여 서버에 메시지를 보낼 수 있으며, `recv`를 호출하여 메시지를 수신할 수도 있습니다.
    * `upgrade`는 한 번의 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 발생시킵니다.

  * **예시**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```  
### push()

`WebSocket` 서버로 메시지를 푸시합니다.

!> `push` 메서드는 `upgrade` 성공 후에만 실행할 수 있습니다.
`push` 메서드는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 발생시키지 않으며, 전송 버퍼에 쓴 후 즉시 반환합니다.

```php
Swoole\Coroutine\Http\Client->push(mixed $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

  * **매개변수**

    * **`mixed $data`**
      * **기능** : 전송할 데이터 내용 [기본값은 `UTF-8` 텍스트 형식이며, 다른 형식 또는 이진 데이터인 경우 `WEBSOCKET_OPCODE_BINARY`을 사용하십시오.]
      * **기본값** : 없음
      * **다른 값** : 없음

      !> Swoole 버전 >= v4.2.0에서는 `$data`를 사용하여 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체를 사용할 수 있으며, 다양한 프레임 유형을 지원합니다.

    * **`int $opcode`**
      * **기능** : 작업 유형
      * **기본값** : `WEBSOCKET_OPCODE_TEXT`
      * **다른 값** : 없음

      !> `$opcode`은 유효한 `WebSocket OPCode` 여야하며, 그렇지 않은 경우 실패하고 오류 메시지 `opcode max 10`이 출력됩니다.

    * **`int|bool $finish`**
      * **기능** : 작업 유형
      * **기본값** : `SWOOLE_WEBSOCKET_FLAG_FIN`
      * **다른 값** : 없음

      !> `v4.4.12`부터 `finish` 매개변수(`bool` 유형)가 `flags`(`int` 유형)로 바뀌어 `WebSocket` 압축을 지원합니다. `finish`은 `SWOOLE_WEBSOCKET_FLAG_FIN` 값이 `1`인 `bool` 유형에서 `int` 유형으로 암시적으로 변환됩니다. 이 변경은 하위 호환성을 유지하며 영향을 미치지 않습니다. 또한 압축 `flag`는 `SWOOLE_WEBSOCKET_FLAG_COMPRESS`입니다.

  * **반환 값**

    * 전송 성공시 `true` 반환
    * 연결이 존재하지 않거나 이미 닫혀 있거나 완료되지 않은 `WebSocket`의 경우 전송 실패시 `false` 반환

  * **에러 코드**

에러 코드 | 설명
---|---
8502 | 잘못된 OPCode
8503 | 서버에 연결되지 않았거나 연결이 이미 닫혔음
8504 | 핸드셰이크 실패
### recv()

메시지를 수신합니다. `WebSocket`만 사용하는 메서드로, `upgrade()`와 함께 사용해야 합니다. 아래는 예시입니다.

```php
Swoole\Coroutine\Http\Client->recv(float $timeout = 0)
```

  * **Parameters** 

    * **`float $timeout`**
      * **기능**: `WebSocket` 연결로 업그레이드할 때만 이 매개변수가 유효합니다.
      * **단위**: 초【부동 소수점 사용 가능, 예: `1.5`는 `1초` + `500ms`를 의미합니다】.
      * **기본값**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙) 참조.
      * **다른 값**: 없음.

      !> 시간 초과를 설정하면 지정된 매개변수가 우선하며, 그 다음에는 `set` 메서드에 전달된 `timeout` 구성이 사용됩니다.
  
  * **Return Value**

    * 성공 시 frame 객체를 반환합니다.
    * 실패 시 `false`를 반환하며, `Swoole\Coroutine\Http\Client`의 `errCode` 속성을 확인해야 합니다. 코루틴 클라이언트에는 `onClose` 콜백이 없으며, 연결이 닫히면 `recv()`가 `false`를 반환하고 errCode=0이 됩니다.
 
  * **Example**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```
### download()

파일을 HTTP로 다운로드합니다.

!> download 메소드는 데이터를받은 후 메모리 내에서 HTTP Body를 연결하는 대신 디스크에 쓰기 때문에 작은 양의 메모리만 사용하여 대용량 파일 다운로드를 완료할 수 있습니다.

```php
Swoole\Coroutine\Http\Client->download(string $path, string $filename, int $offset = 0): bool
```

  * **Parameters** 

    * **`string $path`**
      * **기능**：`URL` 경로를 설정합니다.
      * **기본값**：없음
      * **다른 값**：없음

    * **`string $filename`**
      * **기능**：다운로드 내용을 쓸 파일 경로를 지정합니다. 【`downloadFile` 속성으로 자동 쓰여집니다.】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $offset`**
      * **기능**：파일에 쓸 오프셋을 지정합니다. 【이 옵션은 이어받기를 지원하며 `HTTP` 헤더 `Range:bytes=$offset`과 함께 사용할 수 있습니다.】
      * **기본값**：없음
      * **다른 값**：없음

      !> `$offset`이 `0`인 경우 파일이 이미 존재하는 경우 하위 수준에서이 파일을 자동으로 지웁니다.

  * **Return Value**

    * 성공하면 `true`를 반환합니다.
    * 파일 열기 실패 또는 하위 수준 `fseek()` 파일 실패시 `false`를 반환합니다.

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $host = 'cdn.jsdelivr.net';
    $client = new Client($host, 443, true);
    $client->set(['timeout' => -1]);
    $client->setHeaders([
        'Host' => $host,
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => '*',
        'Accept-Encoding' => 'gzip'
    ]);
    $client->download('/gh/swoole/swoole-src/mascot.png', __DIR__ . '/logo.png');
});
```
### getCookies()

`HTTP` 응답의 `cookie` 내용을 가져옵니다.

```php
Swoole\Coroutine\Http\Client->getCookies(): array|false
```

!> 쿠키 정보는 `urldecode`를 통해 디코딩됩니다. 원래의 쿠키 정보를 얻으려면 아래 설명대로 직접 분석해야 합니다.
```php
var_dump($client->set_cookie_headers);
``````


해당 코드는 PHP에서 사용할 수 있으며, `$client` 객체의 `set_cookie_headers` 속성을 통해 중복된 `Cookie` 또는 원본 헤더 정보를 가져올 수 있습니다.
### getHeaders()

`HTTP` 응답의 헤더 정보를 반환합니다.

```php
Swoole\Coroutine\Http\Client->getHeaders(): array|false
```
### getStatusCode()

`HTTP` 응답의 상태 코드를 가져옵니다.

```php
Swoole\Coroutine\Http\Client->getStatusCode(): int|false
```

  * **참고**

    * **음수 상태 코드는 연결에 문제가 있음을 나타냅니다.**

상태 코드 | v4.2.10 이상 버전에 해당하는 상수 | 설명
---|---|---
-1 | SWOOLE_HTTP_CLIENT_ESTATUS_CONNECT_FAILED | 연결 시간 초과, 서버가 포트를 청취하지 않거나 네트워크 손실이 발생했을 수 있습니다. 구체적인 네트워크 오류 코드를 읽을 수 있는 $errCode를 확인할 수 있습니다.
-2 | SWOOLE_HTTP_CLIENT_ESTATUS_REQUEST_TIMEOUT | 요청 시간 초과, 서버가 지정된 제한 시간 내에 응답을 반환하지 않았습니다.
-3 | SWOOLE_HTTP_CLIENT_ESTATUS_SERVER_RESET | 클라이언트 요청을 전송한 후, 서버가 강제로 연결을 끊었습니다.
-4 | SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED | 클라이언트가 전송에 실패했습니다(이 상수는 Swoole 버전>=`v4.5.9`에서 사용 가능하며, 해당 버전 미만인 경우 상태 코드를 사용하십시오.) 
```php
Swoole\Coroutine\Http\Client->getBody(): string|false
``````getBody()` 함수는 `HTTP` 응답의 본문 내용을 가져옵니다.```
### close()

연결을 닫습니다.

```php
Swoole\Coroutine\Http\Client->close(): bool
```

!> `close` 후에 `get`, `post` 등의 메서드를 다시 요청할 때는 Swoole가 서버에 다시 연결해줍니다.
### execute()

`HTTP` 요청의 더 낮은 수준의 메서드로, 요청 방법과 데이터를 설정하기 위해 코드에서 [setMethod](/coroutine_client/http_client?id=setmethod) 및 [setData](/coroutine_client/http_client?id=setdata)와 같은 인터페이스를 사용해야 합니다.

```php
Swoole\Coroutine\Http\Client->execute(string $path): bool
```

* **예시**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $httpClient = new Client('httpbin.org', 80);
    $httpClient->setMethod('POST');
    $httpClient->setData('swoole');
    $status = $httpClient->execute('/post');
    var_dump($status);
    var_dump($httpClient->getBody());
});
```
## 함수

`Coroutine\Http\Client`를 사용하기 쉽게하기 위해 세 가지 함수가 추가되었습니다:

!> Swoole 버전 >= `v4.6.4`에서 사용 가능
```php
function request(string $url, string $method, $data = null, array $options = null, array $headers = null, array $cookies = null)
```
```php
function post(string $url, $data, array $options = null, array $headers = null, array $cookies = null)
```
### get()

`GET` 요청을 시작하는 데 사용됩니다.

```php
function get(string $url, array $options = null, array $headers = null, array $cookies = null)
```
```php
사용법
복제하고 붙여넣으십시오.
```
