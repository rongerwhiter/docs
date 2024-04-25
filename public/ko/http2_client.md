# Coroutine\Http2\Client

코루틴 Http2 클라이언트
```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $domain = 'www.zhihu.com';
    $cli = new Client($domain, 443, true);
    $cli->set([
        'timeout' => -1,
        'ssl_host_name' => $domain
    ]);
    $cli->connect();
    $req = new Request();
    $req->method = 'POST';
    $req->path = '/api/v4/answers/300000000/voters';
    $req->headers = [
        'host' => $domain,
        'user-agent' => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip'
    ];
    $req->data = '{"type":"up"}';
    $cli->send($req);
    $response = $cli->recv();
    var_dump(assert(json_decode($response->data)->error->code === 10002));
});
```
```python
def greet(name):
    return f"Hello, {name}!"
```

위의 코드는 이름을 인자로 받아 "Hello, {name}!"을 반환하는 함수를 정의합니다.
### __construct()

构造方法。

```php
Swoole\Coroutine\Http2\Client::__construct(string $host, int $port, bool $open_ssl = false): void
```

  * **参数** 

    * **`string $host`**
      * **功能**：目标主机的IP地址【`$host`如果为域名底层需要进行一次`DNS`查询】
      * **默认值**：无
      * **其它值**：无

    * **`int $port`**
      * **功能**：目标端口【`Http`一般为`80`端口，`Https`一般为`443`端口】
      * **默认值**：无
      * **其它值**：无

    * **`bool $open_ssl`**
      * **功能**：是否开启`TLS/SSL`隧道加密 【`https`网站必须设置为`true`】
      * **默认值**：`false`
      * **其它值**：`true`

  * **주의**

    !> -외부 URL에 요청해야 하는 경우 `timeout` 값을 큰 값으로 수정해야 합니다. [클라이언트 타임아웃 규칙](/coroutine_client/init?id=超时规则) 확인  
    -`$ssl`은 `openssl`에 의존하기 때문에 `Swoole` 컴파일 시 [--enable-openssl](/environment?id=编译选项)를 활성화해야 합니다.
```php
Swoole\Coroutine\Http2\Client->set(array $options): void
```
### connect()

대상 서버에 연결합니다. 이 메서드는 매개변수가 없습니다.

!> `connect`를 호출하면 하위 레벨에서 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 자동으로 진행되며, 연결이 성공하거나 실패할 때까지 `connect`가 반환됩니다. 연결이 성공하면 `send` 메서드를 사용하여 서버에 요청을 보낼 수 있습니다.

```php
Swoole\Coroutine\Http2\Client->connect(): bool
```

  * **반환 값**

    * 연결 성공 시 `true` 반환
    * 연결 실패 시 `false` 반환하며, 오류 코드를 얻으려면 `errCode` 속성을 확인하십시오.
```php
Swoole\Coroutine\Http2\Client->stats([$key]): array|bool
```
### isStreamExist()

주어진 스트림이 존재하는지 확인합니다.

```php
Swoole\Coroutine\Http2\Client->isStreamExist(int $stream_id): bool
```
### send()

서버에 요청을 보내면 하위 수준에서 `Http2`의 `stream`이 자동으로 생성됩니다. 여러 요청을 동시에 보낼 수 있습니다.

```php
Swoole\Coroutine\Http2\Client->send(Swoole\Http2\Request $request): int|false
```

  * **매개변수**

    * **`Swoole\Http2\Request $request`**
      * **기능**: Swoole\Http2\Request 객체를 전송합니다.
      * **기본값**: 없음
      * **다른 값**: 없음

  * **반환 값**

    * 성공 시 해당 스트림의 번호가 할당됩니다. 번호는 `1`부터 시작하여 홀수로 증가합니다.
    * 실패 시 `false`가 반환됩니다.

  * **팁**

    * **Request 객체**

      !> `Swoole\Http2\Request` 객체에는 메서드가 없으며, 객체 속성을 설정하여 요청 관련 정보를 입력해야 합니다.

      * `headers` 배열: `HTTP` 헤더
      * `method` 문자열: 요청 방법 설정, 예: `GET`, `POST`
      * `path` 문자열: `URL` 경로 설정, 예: `/index.php?a=1&b=2`, 반드시 /로 시작해야 함
      * `cookies` 배열: `COOKIES` 설정
      * `data` 요청 본문 설정. 문자열인 경우 바로 `RAW form-data`로 전송됨
      * `data`이 배열인 경우, 하위 수준에서 `x-www-form-urlencoded` 형식의 `POST` 내용으로 자동으로 패킹되며, `Content-Type`이 `application/x-www-form-urlencoded`로 설정됨
      * `pipeline` 불린: `true`로 설정하면 `send` 후 `stream`을 종료하지 않고 데이터 내용을 계속 입력할 수 있음

    * **파이프라인**

      * 기본적으로 `send` 메서드는 요청을 보낸 후 현재의 `Http2 Stream`을 종료합니다. `pipeline`을 활성화하면 하위 수준에서 `stream` 흐름을 유지하고 `write` 메서드를 여러 번 호출하여 데이터 프레임을 서버로 보낼 수 있습니다. `write` 메서드를 참고하세요.
### write()

더 많은 데이터 프레임을 서버로 전송하려면 `write`를 여러 번 호출하여 동일한 스트림에 데이터 프레임을 작성할 수 있습니다.

```php
Swoole\Coroutine\Http2\Client->write(int $streamId, mixed $data, bool $end = false): bool
```

  * **Parameters** 

    * **`int $streamId`**
      * **Description**：스트림 번호, `send` 메소드로 반환된 값 입니다
      * **Default Value**：없음
      * **Other Values**：없음

    * **`mixed $data`**
      * **Description**：데이터 프레임의 내용, 문자열 또는 배열이 될 수 있습니다
      * **Default Value**：없음
      * **Other Values**：없음

    * **`bool $end`**
      * **Description**：스트림을 닫을지 여부
      * **Default Value**：`false`
      * **Other Values**：`true`

  * **Usage Example**

```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9518);
    $cli->set(['timeout' => 1]);
    var_dump($cli->connect());

    $req3 = new Request();
    $req3->path = "/index.php";
    $req3->headers = [
        'host' => "localhost",
        "user-agent" => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip',
    ];
    $req3->pipeline = true;
    $req3->method = "POST";
    $streamId = $cli->send($req3);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    //end stream
    $cli->write($streamId, ['int' => rand(1000, 9999), 'end' => true], true);
    var_dump($cli->recv());
    $cli->close();
});
```

!> 데이터 프레임을 세분화하여 보내려면, `send` 요청 시 `$request->pipeline`을 `true`로 설정해야 합니다.  
`end`가 `true`로 설정된 데이터 프레임을 전송한 후에는 스트림이 닫히며, 이후에는 더 이상 `write`를 사용하여이 `stream`에 데이터를 보낼 수 없습니다.
### recv()

요청을 수신합니다.

!> 이 메서드를 호출하면 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 발생합니다.

```php
Swoole\Coroutine\Http2\Client->recv(float $timeout): Swoole\Http2\Response;
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：타임아웃 시간을 설정합니다. [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)을 참조하세요.
      * **값 단위**：초【부동 소수점 지원, 예: `1.5`는 `1초`+`500밀리초`를 의미함】
      * **기본값**：없음
      * **다른 값**：없음

  * **반환 값**

성공 시 Swoole\Http2\Response 객체를 반환합니다.

```php
/**@var $resp Swoole\Http2\Response */
var_dump($resp->statusCode); // 서버가 보낸 Http 상태 코드, 예: 200, 502와 같은 값
var_dump($resp->headers); // 서버가 보낸 헤더 정보
var_dump($resp->cookies); // 서버가 설정한 쿠키 정보
var_dump($resp->set_cookie_headers); // 서버 측에서 반환한 원본 쿠키 정보, domain 및 path 항목이 포함됨
var_dump($resp->data); // 서버가 보낸 응답 바디
```

!> Swoole 버전이 < [v4.0.4](/version/bc?id=_404)인 경우, `data` 속성은 `body` 속성입니다. Swoole 버전이 < [v4.0.3](/version/bc?id=_403)인 경우, `headers` 및 `cookies`는 단수 형태입니다.
### read()

`recv()`과 기본적으로 유사하지만 `pipeline` 유형의 응답에 대해 다른 점은 `read`가 여러 번에 걸쳐 읽을 수 있지만 각각의 읽기는 메모리를 절약하거나 빠르게 푸시된 정보를 수신하기 위해 일부 내용을 부분적으로 읽을 수 있다는 것입니다. 그에 비해 `recv`는 항상 모든 프레임을 하나의 완전한 응답으로 연결한 후에만 반환합니다.

!> 이 메서드를 호출하면 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 발생합니다.

```php
Swoole\Coroutine\Http2\Client->read(float $timeout): Swoole\Http2\Response;
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：시간 초과 설정, 자세한 내용은 [클라이언트 시간 초과 규칙](/coroutine_client/init?id=시간-초과-규칙)을 참조하십시오.
      * **값의 단위**: 초【부동 소수점 지원, 예: `1.5`는 `1s`+`500ms`를 의미함】
      * **기본값**: 없음
      * **기타 값**: 없음

  * **반환값**

    성공 시 Swoole\Http2\Response 객체가 반환됩니다.
### goaway()

GOAWAY 프레임은 연결을 종료하거나 심각한 오류 상태 신호를 전송하는 데 사용됩니다.

```php
Swoole\Coroutine\Http2\Client->goaway(int $error_code = SWOOLE_HTTP2_ERROR_NO_ERROR, string $debug_data): bool
```
### ping()

PING 프레임은 송신 측의 최소 왕복 시간을 측정하고 유휴 연결이 여전히 유효한지 확인하는 데 사용되는 메커니즘입니다.

```php
Swoole\Coroutine\Http2\Client->ping(): bool
```
### close()

연결을 닫습니다.

```php
Swoole\Coroutine\Http2\Client->close(): bool
```
