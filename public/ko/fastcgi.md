# 코루틴 FastCGI 클라이언트

PHP-FPM은 효율적인 이진 프로토콜인 'FastCGI 프로토콜'을 사용하여 통신하며, FastCGI 클라이언트를 사용하면 PHP-FPM 서비스와 직접 상호 작용할 수 있으므로 어떠한 HTTP 리버스 프록시도 필요하지 않습니다.

[PHP 소스 코드 디렉터리](https://github.com/swoole/library/blob/master/src/core/Coroutine/FastCGI)
## 간단한 사용 예시

[더 많은 예시 코드](https://github.com/swoole/library/tree/master/examples/fastcgi)

!> 아래 예시 코드는 코루틴 내에서 호출해야 합니다.
```php
#greeter.php
echo 'Hello ' . ($_POST['who'] ?? 'World');
```

```php
echo \Swoole\Coroutine\FastCGI\Client::call(
    '127.0.0.1:9000', // FPM 리스닝 주소, 또는 unix:/tmp/php-cgi.sock와 같은 유닉스 소켓 주소
    '/tmp/greeter.php', // 실행할 진입 파일
    ['who' => 'Swoole'] // 포함된 POST 정보
);
```  
```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1:9000', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withScriptFilename(__DIR__ . '/greeter.php')
        ->withMethod('POST')
        ->withBody(['who' => 'Swoole']);
    $response = $client->execute($request);
    echo "Result: {$response->getBody()}\n";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "Error: {$exception->getMessage()}\n";
}
```
```php
#var.php
var_dump($_SERVER);
var_dump($_GET);
var_dump($_POST);
```

```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withDocumentRoot(__DIR__)
        ->withScriptFilename(__DIR__ . '/var.php')
        ->withScriptName('var.php')
        ->withMethod('POST')
        ->withUri('/var?foo=bar&bar=char')
        ->withHeader('X-Foo', 'bar')
        ->withHeader('X-Bar', 'char')
        ->withBody(['foo' => 'bar', 'bar' => 'char']);
    $response = $client->execute($request);
    echo "결과: \n{$response->getBody()}";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "오류: {$exception->getMessage()}\n";
}
```
### WordPress를 위한 원 클릭 프록시

!> 이 사용법은 생산적이지 않으며, 프로덕션 환경에서 프록시는 일부 오래된 API 인터페이스의 HTTP 요청을 이전 FPM 서비스로 프록싱하는 데 사용될 수 있습니다 (전체 사이트를 프록싱하는 것이 아닌).

```php
use Swoole\Constant;
use Swoole\Coroutine\FastCGI\Proxy;
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\Http\Server;

$documentRoot = '/var/www/html'; # WordPress 프로젝트 루트 디렉토리
$server = new Server('0.0.0.0', 80, SWOOLE_BASE); # 이곳의 포트는 WordPress 구성과 일치해야 하며, 일반적으로 특정 포트를 지정하지 않고 80포트일 것입니다
$server->set([
    Constant::OPTION_WORKER_NUM => swoole_cpu_num() * 2,
    Constant::OPTION_HTTP_PARSE_COOKIE => false,
    Constant::OPTION_HTTP_PARSE_POST => false,
    Constant::OPTION_DOCUMENT_ROOT => $documentRoot,
    Constant::OPTION_ENABLE_STATIC_HANDLER => true,
    Constant::OPTION_STATIC_HANDLER_LOCATIONS => ['/wp-admin', '/wp-content', '/wp-includes'], # 정적 자원 경로
]);
$proxy = new Proxy('127.0.0.1:9000', $documentRoot); # 프록시 객체 생성
$server->on('request', function (Request $request, Response $response) use ($proxy) {
    $proxy->pass($request, $response); # 원 클릭으로 요청 프록싱
});
$server->start();
```
## 방법
### call

**정적 메서드, 새로운 클라이언트 연결을 직접 생성하여 FPM 서버에 요청을 보내고 응답 본문을 수신합니다.**

!> FPM은 짧은 연결만 지원하므로 일반적으로 지속적인 개체 생성에 큰 의미가 없습니다.

```php
Swoole\Coroutine\FastCGI\Client::call(string $url, string $path, $data = '', float $timeout = -1): string
```

  * **매개변수** 

    * **`string $url`**
      * **기능**：FPM이 수신 대기하는 주소【예: `127.0.0.1:9000`, `unix:/tmp/php-cgi.sock` 등】
      * **기본값**：없음
      * **다른 값**：없음

    * **`string $path`**
      * **기능**：실행하려는 진입 파일
      * **기본값**：없음
      * **다른 값**：없음

    * **`$data`**
      * **기능**：요청 데이터를 첨부합니다.
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간을 설정합니다【기본값은 -1이며 무한대기를 의미함】
      * **단위**：초【부동 소수점을 지원하며, 예: 1.5는 1초 500밀리초를 의미함】
      * **기본값**：`-1`
      * **다른 값**：없음

  * **반환값** 

    * 서버의 응답 바디 내용을 반환합니다.
    * 오류 발생 시 `Swoole\Coroutine\FastCGI\Client\Exception` 예외를 던집니다.
### __construct

클라이언트 객체의 생성자 메서드, 대상 FPM 서버 지정

```php
Swoole\Coroutine\FastCGI\Client::__construct(string $host, int $port = 0)
```

  * **매개변수** 

    * **`string $host`**
      * **기능**：대상 서버의 주소【예: `127.0.0.1` 또는 `unix://tmp/php-fpm.sock`와 같은】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $port`**
      * **기능**：대상 서버 포트【대상 주소가 UNIXSocket인 경우 전달할 필요 없음】
      * **기본값**：없음
      * **다른 값**：없음
### 실행

요청을 실행하고 응답을 반환합니다.

```php
Swoole\Coroutine\FastCGI\Client->execute(Request $request, float $timeout = -1): Response
```

  * **매개변수** 

    * **`Swoole\FastCGI\Request|Swoole\FastCGI\HttpRequest $request`**
      * **기능**：요청 정보를 포함하는 객체로, 일반적으로 HTTP 요청을 모방하기 위해 `Swoole\FastCGI\HttpRequest`를 사용하며 특별한 요구 사항이 있을 때만 FPM 프로토콜의 원시 요청 클래스인 `Swoole\FastCGI\Request`를 사용합니다.
      * **기본값**：없음
      * **기타 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간을 설정합니다.【기본값으로 `-1`은 절대 타임아웃하지 않음을 의미합니다】
      * **시간 단위**：초【부동 소수점을 지원하며, `1.5`는 `1초` + `500밀리초`를 나타냅니다】
      * **기본값**：`-1`
      * **기타 값**：없음

  * **반환 값** 

    * 요청 객체의 유형에 맞는 Response 객체가 반환되며, `Swoole\FastCGI\HttpRequest`의 경우 `Swoole\FastCGI\HttpResponse` 객체가 반환되어 FPM 서버의 응답 정보가 포함됩니다.
    * 오류 발생시에는 `Swoole\Coroutine\FastCGI\Client\Exception` 예외가 발생합니다.
## 관련 요청/응답 클래스

라이브러리는 PSR의 방대한 종속성 구현을 가져올 수 없고 확장을 항상 PHP 코드 실행 전에 로드하기 때문에 관련된 요청 응답 객체는 PSR 인터페이스를 상속하고 있지는 않습니다. 그러나 개발자가 빠르게 사용할 수 있도록 PSR 스타일을 최대한 따르도록 노력했습니다.

FastCGI를 흉내내는 HTTP 요청 응답 클래스의 소스 코드는 다음과 같습니다. 매우 간단하며 코드가 문서입니다:

[Swoole\FastCGI\HttpRequest](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpRequest.php)
[Swoole\FastCGI\HttpResponse](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpResponse.php)
