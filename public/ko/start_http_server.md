# HTTP 서버

## 프로그램 코드

다음 코드를 httpServer.php에 작성하십시오.

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('Request', function ($request, $response) {
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});

$http->start();
```

`HTTP` 서버는 요청 및 응답에만 관심이 있으므로 [onRequest](/http_server?id=on) 이벤트 한 가지만 수신하면 됩니다. 새 `HTTP` 요청이 발생하면 이 이벤트가 트리거됩니다. 이벤트 콜백 함수에는 `$request` 객체와 같은 요청 관련 정보가 포함된 매개변수 두 개가 있습니다.

다른 하나는 `response` 객체입니다. `response` 객체 조작을 통해 요청에 대한 응답을 완료할 수 있습니다. `$response->end()` 메서드는 `HTML` 콘텐츠를 출력하고 해당 요청을 종료함을 나타냅니다.

* `0.0.0.0`은 모든 `IP` 주소를 수신 대기한다는 것을 의미하며, 한 대의 서버에는 `127.0.0.1`로컬 루프백 IP, `192.168.1.100`사설 네트워크 IP, `210.127.20.2`공인 IP 등 여러 IP가 있을 수 있습니다. 이곳에서는 특정 IP를 대상으로 수신 대기를 지정할 수도 있습니다.
* `9501`은 수신 대기하는 포트입니다. 사용 중인 경우 프로그램이 치명적인 오류를 throw하고 실행이 중지됩니다.

## 서버 시작

```shell
php httpServer.php
```
* 웹 브라우저를 열어 `http://127.0.0.1:9501`에 대한 결과를 확인할 수 있습니다.
* Apache의 `ab` 도구를 사용하여 서버에 대한 부하 테스트를 수행할 수도 있습니다.

## Chrome 두 번의 요청 문제

`Chrome` 브라우저를 사용하여 서버에 액세스하면 `/favicon.ico` 와 같이 추가 요청이 발생합니다. 코드에서 `404` 오류를 응답하도록 할 수 있습니다.

```php
$http->on('Request', function ($request, $response) {
	if ($request->server['path_info'] == '/favicon.ico' || $request->server['request_uri'] == '/favicon.ico') {
        $response->end();
        return;
	}
    var_dump($request->get, $request->post);
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});
```

## URL 라우팅

애플리케이션은 `$request->server['request_uri']`에 따라 라우팅을 구현할 수 있습니다. 예를들어, `http://127.0.0.1:9501/test/index/?a=1`이 있는 경우 코드에서 `URL` 라우팅을 다음과 같이 구현할 수 있습니다.

```php
$http->on('Request', function ($request, $response) {
    list($controller, $action) = explode('/', trim($request->server['request_uri'], '/'));
	// $controller, $action에 따라 다른 컨트롤러 클래스 및 메서드로 매핑합니다.
	(new $controller)->$action($request, $response);
});
```
