# Coroutine\Socket

`Swoole\Coroutine\Socket`モジュールは、[Coroutine Style Server](/server/co_init)および[Coroutine Client](/coroutine_client/init)関連モジュールよりもより細かい`IO`操作を実現できる`Socket`より高度な機能を提供します。

!> `Co\Socket`を使用してクラス名を簡略化できます。このモジュールはかなり低レベルなものであるため、利用者はソケットプログラミングの経験が望ましいです。
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    $socket = new Coroutine\Socket(AF_INET, SOCK_STREAM, 0);

    $retval = $socket->connect('127.0.0.1', 9601);
    while ($retval)
    {
        $n = $socket->send('hello');
        var_dump($n);

        $data = $socket->recv();
        var_dump($data);

        //発生したエラーまたは相手が接続を切断した場合、この側も閉じる必要があります
        if ($data === '' || $data === false) {
            echo "errCode: {$socket->errCode}\n";
            $socket->close();
            break;
        }

        Coroutine::sleep(1.0);
    }

    var_dump($retval, $socket->errCode, $socket->errMsg);
});
```    
## コルーチンスケジューリング

`Coroutine\Socket`モジュールが提供する`IO`操作のインターフェースはすべて同期プログラミングスタイルです。これらは、[協力的マルチタスクスケジューラ](/coroutine?id=協力的マルチタスクスケジューラ)を使用して、[非同期IO](/learn?id=同期io非同期io)を実装します。
## エラーコード

`socket`関連のシステムコールを実行すると、-1のエラーが返されることがあります。この場合、下位レベルで`Coroutine\Socket->errCode`プロパティがシステムエラーコード`errno`に設定されますので、対応する`man`ドキュメントを参照してください。たとえば、`$socket->accept()`がエラーを返す場合、`errCode`の意味は`man accept`に記載されているエラーコードドキュメントを参照してください。
Sure! Could you please provide the text you would like me to translate into Japanese?
### fd

`socket`のファイル記述子`ID`
### errCode

エラーコード
このセクションは特にコードブロックが含まれていないため、翻訳の必要はありません。
### __construct()

`Coroutine\Socket`オブジェクトを構築します。

```php
Swoole\Coroutine\Socket::__construct(int $domain, int $type, int $protocol);
```

!> 詳細については、`man socket`ドキュメントを参照してください。

  * **パラメータ** 

    * **`int $domain`**
      * **機能**：プロトコルドメイン【`AF_INET`、`AF_INET6`、`AF_UNIX`を使用できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $type`**
      * **機能**：タイプ【`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`を使用できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $protocol`**
      * **機能**：プロトコル【`IPPROTO_TCP`、`IPPROTO_UDP`、`IPPROTO_STCP`、`IPPROTO_TIPC`、`0`を使用できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> 構造関数は`socket`システムコールを呼び出し、`socket`ハンドルを作成します。呼び出しに失敗した場合、`Swoole\Coroutine\Socket\Exception`例外がスローされます。また、`$socket->errCode`プロパティが設定されます。このプロパティの値に基づいて、システムコールの失敗原因を取得できます。
### getOption()

設定を取得します。

!> このメソッドは `getsockopt` システムコールに対応しており、詳細については `man getsockopt` ドキュメントを参照してください。  
このメソッドは `sockets` 拡張の `socket_get_option` 関数と同等であり、[PHPドキュメント](https://www.php.net/manual/zh/function.socket-get-option.php) を参照してください。

!> Swooleバージョン >= v4.3.2

```php
Swoole\Coroutine\Socket->getOption(int $level, int $optname): mixed
```

  * **Parameters** 

    * **`int $level`**
      * **Description**: Specifies the protocol level where the option resides
      * **Default**: None
      * **Other values**: None

      !> For example, to retrieve an option at the socket level, the `level` parameter will use `SOL_SOCKET`.  
      Other levels can be used by specifying the protocol number of that level, such as `TCP`. Protocol numbers can be found using the [getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php) function.

    * **`int $optname`**
      * **Description**: Available socket options are the same as those in the [socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php) function
      * **Default**: None
      * **Other values**: None
### setOption()

設定選項。

!> このメソッドは、`setsockopt`システムコールに対応しており、詳細については`man setsockopt`ドキュメントを参照してください。このメソッドは`sockets`拡張機能の`socket_set_option`機能と同等であり、[PHPドキュメント](https://www.php.net/manual/zh/function.socket-set-option.php)を参照してください。

!> Swoole バージョン >= v4.3.2

```php
Swoole\Coroutine\Socket->setOption(int $level, int $optname, mixed $optval): bool
```

  * **パラメータ** 

    * **`int $level`**
      * **機能**：オプションが存在するプロトコルレベルを指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> 例えば、ソケットレベルでオプションを取得する場合、`SOL_SOCKET`の `level` パラメータが使用されます。  
      他のレベルを使用する場合は、そのレベルのプロトコル番号を指定することができます。たとえば、`TCP`。プロトコル番号は[getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php)関数を使用して見つけることができます。

    * **`int $optname`**
      * **機能**：利用可能なソケットオプションは、[socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php)関数のソケットオプションと同じです
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $optval`**
      * **機能**：オプションの値 【`int`、`bool`、`string`、`array` などが含まれます。`level` と `optname` に応じて異なります。】
      * **デフォルト値**：なし
      * **その他の値**：なし
### setProtocol()

`socket`にプロトコル処理機能を付与し、`SSL`暗号化転送の有効化や[TCPデータパケットの境界問題](/learn?id=tcpデータパケットの境界問題)の解決などを設定できます。

!> Swoole version >= v4.3.2

```php
Swoole\Coroutine\Socket->setProtocol(array $settings): bool
```

  * **$settingsで指定可能なパラメータ**

パラメータ | 型
---|---
open_ssl | bool
ssl_cert_file | string
ssl_key_file | string
open_eof_check | bool
open_eof_split | bool
open_mqtt_protocol | bool
open_fastcgi_protocol | bool
open_length_check | bool
package_eof | string
package_length_type | string
package_length_offset | int
package_body_offset | int
package_length_func | callable
package_max_length | int

!> 上記の全てのパラメータの意味は[Server->set()](/server/setting?id=open_eof_check)と完全に一致しますが、ここでは繰り返し説明しません。

  * **例**

```php
$socket->setProtocol([
    'open_length_check'     => true,
    'package_max_length'    => 1024 * 1024,
    'package_length_type'   => 'N',
    'package_length_offset' => 0,
    'package_body_offset'   => 4,
]);
```
### bind()

アドレスとポートをバインドします。

!> このメソッドは`IO`操作を行いません。したがって、コルーチンの切り替えは発生しません。

```php
Swoole\Coroutine\Socket->bind(string $address, int $port = 0): bool
```

  * **パラメーター**

    * **`string $address`**
      * **機能**： バインドするアドレス【例：`0.0.0.0`、`127.0.0.1`】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **機能**： バインドするポート【デフォルトは`0`で、システムは使用可能なポートをランダムにバインドします。[getsockname](/coroutine_client/socket?id=getsockname)メソッドでシステムが割り当てた`port`を取得できます】
      * **デフォルト値**：`0`
      * **その他の値**：なし

  * **戻り値**

    * バインドに成功すると`true`が返されます
    * バインドに失敗すると`false`が返され、失敗の理由は`errCode`プロパティを確認してください
# 코루틴\ 소켓

`Swoole\Coroutine\Socket` 모듈은 [코루틴 스타일 서버](/server/co_init) 및 [코루틴 클라이언트](/coroutine_client/init)와 관련된 모듈과 비교하여 보다 세부적인 `IO` 작업을 수행할 수 있습니다.

!> `Co\Socket`를 사용하여 클래스 이름을 간단하게 할 수 있습니다. 이 모듈은 상당히 낮은 수준이므로 사용자는 소켓 프로그래밍 경험이 있어야 합니다.
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    $socket = new Coroutine\Socket(AF_INET, SOCK_STREAM, 0);

    $retval = $socket->connect('127.0.0.1', 9601);
    while ($retval)
    {
        $n = $socket->send('hello');
        var_dump($n);

        $data = $socket->recv();
        var_dump($data);

        //에러가 발생하거나 상대방이 연결을 끊었을 때, 클라이언트도 닫아야 함
        if ($data === '' || $data === false) {
            echo "errCode: {$socket->errCode}\n";
            $socket->close();
            break;
        }

        Coroutine::sleep(1.0);
    }

    var_dump($retval, $socket->errCode, $socket->errMsg);
});
```  
## 코루틴 스케줄링

`Coroutine\Socket` 모듈은 제공하는 `IO` 작업 인터페이스가 동기식 프로그래밍 스타일이며, 내부적으로 [코루틴 스케줄러](/coroutine?id=코루틴 스케줄링)를 활용하여 [비동기 IO](/learn?id=동기io비동기io)를 구현합니다.
## 오류 코드

`socket` 관련 시스템 호출을 실행하는 동안에 -1 오류가 발생할 수 있으며, 이 경우 하위 수준에서는 `errno` 시스템 오류 번호로 `Coroutine\Socket->errCode` 속성이 설정됩니다. 이 때 발생한 오류에 대한 의미를 확인하려면 해당하는 `man` 페이지를 참고하시기 바랍니다. 예를 들어, `$socket->accept()`가 오류를 반환했을 때, `errCode`의 의미는 `man accept`에 나열된 오류 코드 문서를 참고하시기 바랍니다.
## 속성
### fd

`socket`에 대응하는 파일 기술자 `ID`
### errCode

오류 코드
해당 문서는 메소드에 대해 설명하고 있습니다.
### __construct()

`Coroutine\Socket` 객체를 만드는 생성자입니다.

```php
Swoole\Coroutine\Socket::__construct(int $domain, int $type, int $protocol);
```

!> 자세한 내용은 `man socket` 문서를 참조하십시오.

  * **매개변수** 

    * **`int $domain`**
      * **기능**：프로토콜 도메인【`AF_INET`、`AF_INET6`、`AF_UNIX` 사용 가능】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $type`**
      * **기능**：유형【`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW` 사용 가능】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $protocol`**
      * **기능**：프로토콜【`IPPROTO_TCP`、`IPPROTO_UDP`、`IPPROTO_STCP`、`IPPROTO_TIPC` 사용 가능, `0`】
      * **기본값**：없음
      * **다른 값**：없음

!> 생성자는 `socket` 시스템 콜을 사용하여 `socket` 핸들을 만듭니다. 호출에 실패하면 `Swoole\Coroutine\Socket\Exception` 예외가 발생하며 `$socket->errCode` 속성이 설정됩니다. 해당 속성의 값에 따라 시스템 콜이 실패한 이유를 확인할 수 있습니다.
### getOption()

설정을 가져옵니다.

!> 이 메소드는 `getsockopt` 시스템 호출에 해당하며, 자세한 내용은 `man getsockopt` 문서를 참조하십시오.  
이 메소드는 `sockets` 확장의 `socket_get_option` 기능과 동등하며, [PHP 문서](https://www.php.net/manual/zh/function.socket-get-option.php)를 참조할 수 있습니다.

!> Swoole 버전 >= v4.3.2

```php
Swoole\Coroutine\Socket->getOption(int $level, int $optname): mixed
```

  * **파라미터** 

    * **`int $level`**
      * **기능**：옵션이 위치한 프로토콜 레벨 지정
      * **기본값**：없음
      * **다른 값**：없음

      !> 예를 들어, 소켓 레벨에서 옵션을 가져오려면 `SOL_SOCKET`의 `level` 매개변수를 사용합니다.  
      다른 레벨을 사용하려면 해당 레벨의 프로토콜 번호를 지정할 수 있습니다. 예를 들어 `TCP`와 같은 다른 레벨을 사용할 수 있으며, 프로토콜 번호는 [getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php) 함수를 사용하여 찾을 수 있습니다.

    * **`int $optname`**
      * **기능**：사용 가능한 소켓 옵션으로 [socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php) 함수의 소켓 옵션과 동일합니다
      * **기본값**：없음
      * **다른 값**：없음
### setOption()

설정을 설정합니다.

!> 이 메서드는 `setsockopt` 시스템 호출에 해당하며, 자세한 내용은 `man setsockopt` 문서를 참조하십시오. 이 메서드는 `sockets` 확장 기능의 `socket_set_option`과 동등하며, [PHP 문서](https://www.php.net/manual/zh/function.socket-set-option.php)를 참조할 수 있습니다.

!> Swoole 버전 >= v4.3.2

```php
Swoole\Coroutine\Socket->setOption(int $level, int $optname, mixed $optval): bool
```

  * **파라미터** 

    * **`int $level`**
      * **기능**：옵션의 프로토콜 레벨을 지정합니다.
      * **기본값**：없음
      * **다른 값**：없음

      !> 예를 들어, 소켓 레벨에서 옵션을 검색하려면 `SOL_SOCKET`의 `level` 매개변수를 사용합니다.  
      다른 레벨을 사용하려면 해당 레벨의 프로토콜 번호를 지정할 수 있습니다. 예를 들어 `TCP`와 같은 것이 있습니다. 프로토콜 번호를 찾으려면 [getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php) 함수를 사용합니다.

    * **`int $optname`**
      * **기능**：사용 가능한 소켓 옵션이 [socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php) 함수의 소켓 옵션과 동일합니다.
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $optval`**
      * **기능**：옵션의 값 【`int`、`bool`、`string`、`array`가 될 수 있습니다. `level`과 `optname`에 따라 달라집니다.】
      * **기본값**：없음
      * **다른 값**：없음
### setProtocol()

`socket`에 프로토콜 처리 능력을 부여하여 `SSL` 암호화 전송을 활성화하거나 [TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터패킷경계문제)를 해결할 수 있습니다.

!> Swoole 버전 >= v4.3.2

```php
Swoole\Coroutine\Socket->setProtocol(array $settings): bool
```

  * **$settings 지원하는 매개변수**

매개변수 | 타입
---|---
open_ssl | bool
ssl_cert_file | string
ssl_key_file | string
open_eof_check | bool
open_eof_split | bool
open_mqtt_protocol | bool
open_fastcgi_protocol | bool
open_length_check | bool
package_eof | string
package_length_type | string
package_length_offset | int
package_body_offset | int
package_length_func | callable
package_max_length | int

!> 상기 모든 매개변수의 의미는 [Server->set()](/server/setting?id=open_eof_check)과 완전히 일치하며, 여기서 다시 설명하지 않습니다.

  * **예시**

```php
$socket->setProtocol([
    'open_length_check'     => true,
    'package_max_length'    => 1024 * 1024,
    'package_length_type'   => 'N',
    'package_length_offset' => 0,
    'package_body_offset'   => 4,
]);
```  
### bind()

주소와 포트를 바인딩합니다.

!> 이 메소드에는 `IO` 작업이 없으며, 코루틴 전환을 일으키지 않습니다.

```php
Swoole\Coroutine\Socket->bind(string $address, int $port = 0): bool
```

  * **매개변수** 

    * **`string $address`**
      * **기능**：바인딩할 주소【예: `0.0.0.0`, `127.0.0.1`】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $port`**
      * **기능**：바인딩할 포트【기본값은 `0`이며, 시스템은 사용 가능한 포트 중 하나를 무작위로 바인딩하며, [getsockname](/coroutine_client/socket?id=getsockname) 메소드를 사용하여 시스템이 할당한 `port`를 얻을 수 있음】
      * **기본값**：`0`
      * **다른 값**：없음

  * **반환 값** 

    * 바인딩 성공 시 `true` 반환
    * 바인딩 실패 시 `false` 반환. 실패 원인을 얻으려면 `errCode` 속성을 확인하세요.
### listen()

`Socket`를 청취합니다.

!> 이 방법은 `IO` 작업을 수행하지 않으며 코루틴 전환을 일으키지 않습니다.

```php
Swoole\Coroutine\Socket->listen(int $backlog = 0): bool
```

  * **매개변수** 

    * **`int $backlog`**
      * **기능**：대기열 길이를 설정합니다【기본값은 `0`이며, 시스템 하부에서 `epoll`을 사용하여 비동기 `IO`를 구현했기 때문에 차단이 없으므로 `backlog`의 중요도가 높지 않음】
      * **기본값**：`0`
      * **다른 값**：없음

      !> 응용프로그램에 차단 또는 시간이 많이 걸리는 로직이 있으면 `accept`에 연결을 즉시 수락하지 못하여 새로운 생성된 연결이 `backlog` 대기열에 쌓이면서, `backlog` 길이를 초과하면 서비스가 새 연결 요청을 거부할 수 있습니다

  * **반환 값** 

    * 성공적으로 바인딩되면 `true`를 반환합니다
    * 실패 시 `false`를 반환하며, 실패 원인을 확인하려면 `errCode` 속성을 확인하세요

  * **커널 파라미터** 

    `backlog`의 최대값은 커널 파라미터 `net.core.somaxconn`의 제한이며, `Linux`에서는 `sysctl` 도구를 사용하여 모든 `kernel` 파라미터를 동적으로 조정할 수 있습니다. 동적 조정은 커널 파라미터 값이 변경된 즉시 적용됩니다. 그러나 이 적용은 `OS` 수준에서만 가능하며, 애플리케이션을 다시 시작해야 실제로 적용됩니다. 명령어 `sysctl -a`를 통해 모든 커널 파라미터 및 값을 표시할 수 있습니다.

    ```shell
    sysctl -w net.core.somaxconn=2048
    ```

    위 명령어는 커널 파라미터 `net.core.somaxconn`의 값을 `2048`로 변경합니다. 이러한 변경은 즉시 적용되지만, 장비를 다시 시작하면 기본 값으로 복원됩니다. 변경 사항을 영구적으로 유지하려면 `/etc/sysctl.conf` 파일을 수정하여 `net.core.somaxconn=2048`를 추가한 후 명령어 `sysctl -p`를 실행하여 변경을 적용해야 합니다.
### accept()

클라이언트가 요청한 연결을 수락합니다.

이 메서드를 호출하면 현재 코루틴이 즉시 일시 중지되고 [EventLoop](/learn?id=이벤트루프란 무엇인가)에 읽기 가능한 이벤트를 감시하며, `Socket`이 읽기 가능한 연결이 도착하면 해당 코루틴을 자동으로 다시 실행하고 해당 클라이언트 연결의 `Socket` 객체를 반환합니다.

!> 이 메서드는 `listen` 메서드를 사용한 후에만 사용해야 하며, `Server` 측에 사용됩니다.

```php
Swoole\Coroutine\Socket->accept(float $timeout = 0): Coroutine\Socket|false;
```

  * **인수** 

    * **`float $timeout`**
      * **기능**: 타임아웃 설정【타임아웃 매개변수를 설정하면 밑바닥에서 타이머를 설정하며, 지정된 시간 내에 클라이언트 연결이 없을 경우 `accept` 메서드는 `false`를 반환합니다】
      * **단위**: 초【부동 소수점을 지원하며, 예를 들어 `1.5`는 `1초`+`500밀리초`를 의미합니다】
      * **기본값**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃 규칙) 참조
      * **기타 값**: 없음

  * **반환 값** 

    * 타임아웃 또는 `accept` 시스템 호출 오류 시 `false`가 반환되며, `errCode` 속성을 사용하여 오류 코드를 얻을 수 있으며, 여기서 타임아웃 오류 코드는 `ETIMEDOUT`입니다.
    * 성공 시 클라이언트 연결의 `socket`을 반환하며, 타입은 여전히 `Swoole\Coroutine\Socket` 객체입니다. 이를 통해 `send`, `recv`, `close` 등의 작업을 수행할 수 있습니다.

  * **예제**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
$socket = new Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
$socket->bind('127.0.0.1', 9601);
$socket->listen(128);

    while(true) {
        echo "Accept: \n";
        $client = $socket->accept();
        if ($client === false) {
            var_dump($socket->errCode);
        } else {
            var_dump($client);
        }
    }
});
```
### connect()

대상 서버에 연결합니다.

이 메서드를 호출하면 비동기적인 `connect` 시스템 호출이 시작되고 현재 코루틴이 일시 중단됩니다. 내부적으로 쓰기 가능 여부를 감시하며, 연결이 완료되거나 실패한 후 해당 코루틴이 복원됩니다.

이 메서드는 `Client` 측에서 사용되며, `IPv4`, `IPv6`, [unixSocket](/learn?id=什么是IPC)을 지원합니다.

```php
Swoole\Coroutine\Socket->connect(string $host, int $port = 0, float $timeout = 0): bool
```

  * **매개변수** 

    * **`string $host`**
      * **기능**：대상 서버의 주소【예: `127.0.0.1`, `192.168.1.100`, `/tmp/php-fpm.sock`, `www.baidu.com` 등, `IP` 주소, `Unix Socket` 경로 또는 도메인을 전달할 수 있습니다. 도메인인 경우 백그라운드에서 비동기적으로 `DNS`를 해결하며 차단을 일으키지 않습니다.】
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $port`**
      * **기능**：대상 서버 포트【`Socket`의 `domain`이 `AF_INET`, `AF_INET6`인 경우 반드시 포트를 설정해야 합니다.】
      * **기본값**：없음
      * **기타 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간 설정【내부적으로 타이머를 설정하여 지정된 시간 내에 연결을 설정할 수 없으면 `connect`는 `false`를 반환합니다.】
      * **단위**：초【부동 소수점 지원, 예: `1.5`는 `1초` + `500밀리초`를 의미합니다.】
      * **기본값**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=超时规则) 참고
      * **기타 값**：없음

  * **반환 값** 

    * 타임아웃이나 `connect` 시스템 호출에서 오류가 발생한 경우 `false`를 반환하며 `errCode` 속성을 사용하여 오류 코드를 얻을 수 있으며, 타임아웃 오류 코드는 `ETIMEDOUT`입니다.
    * 성공한 경우 `true`를 반환합니다.
### checkLiveness()

시스템 호출을 사용하여 연결이 살아 있는지 확인합니다 (비정상적으로 끊겼을 때에는 무용지물이며, 상대방이 정상적으로 종료하여 연결이 끊긴 것만 감지할 수 있음)

!> Swoole 버전 >= `v4.5.0`에서 사용 가능

```php
Swoole\Coroutine\Socket->checkLiveness(): bool
```

  * **반환값** 

    * 연결이 살아 있으면 `true`를 반환하고, 그렇지 않으면 `false`를 반환합니다.
### send()

상대방에게 데이터를 보냅니다.

send 메소드는 데이터를 즉시 보내기 위해 `send` 시스템 호출을 실행합니다. `send` 시스템 호출이 `EAGAIN` 오류를 반환하면, 하위 수준에서 자동으로 쓰기 이벤트를 감지하고, 현재 코루틴을 중단시키고 쓰기 이벤트가 트리거될 때 `send` 시스템 호출을 다시 실행하여 해당 코루틴을 다시 깨웁니다.

send가 너무 빠르고 recv가 너무 느릴 경우, 최종적으로 운영 체제 버퍼가 가득 차게 되어 현재 코루틴이 send 메소드에 매달릴 수 있습니다. 이때 버퍼 크기를 조정해야 하며, 이를 위해 [/proc/sys/net/core/wmem_max와 SO_SNDBUF](https://stackoverflow.com/questions/21856517/whats-the-practical-limit-on-the-size-of-single-packet-transmitted-over-domain)를 참조하십시오.

```php
Swoole\Coroutine\Socket->send(string $data, float $timeout = 0): int|false
```

  * **Parameters** 

    * **`string $data`**
      * **Description**: 데이터 내용을 보낼 것입니다【텍스트 또는 이진 데이터 사용 가능】
      * **Default**: None
      * **Other values**: None

    * **`float $timeout`**
      * **Description**: 시간 초과 설정
      * **Unit**: 초【부동 소수점 형식을 지원합니다. 예를 들어 `1.5`는 `1초` + `500ms`를 나타냅니다】
      * **Default**: [클라이언트의 시간 초과 규칙](/coroutine_client/init?id=시간 초과 규칙)을 참조하세요
      * **Other values**: None

  * **Return Value** 

    * 성공적으로 전송하면 쓰여진 바이트 수가 반환됩니다. **실제로 전송된 데이터는 `$data` 매개변수의 길이보다 작을 수 있습니다**. 응용 프로그램 코드는 반환 값을 `strlen($data)`와 비교하여 전송이 완료되었는지를 판단해야 합니다.
    * 전송 실패 시 `false`가 반환되며 `errCode` 속성이 설정됩니다.
### sendAll()

상대방에 데이터를 보냅니다. `send` 메서드와 다른 점은 `sendAll`은 가능한 한 데이터를 완전히 보내기 위해 실행되고, 모든 데이터를 성공적으로 보내거나 오류가 발생할 때까지 계속됩니다.

!> `sendAll` 메서드는 데이터를 보낼 때 즉시 여러 번의 send 시스템 호출을 실행합니다. `send` 시스템 호출이 `EAGAIN` 오류를 반환하면, 하부에서는 자동으로 쓰기 가능 이벤트를 감시하고 현재 코루틴을 일시 중단하여 쓰기 가능 이벤트가 발생하면 데이터를 보내기 위해 다시 `send` 시스템 호출을 실행하고, 데이터가 전송되거나 오류가 발생할 때까지 해당 코루틴을 깨웁니다.

!> Swoole 버전 >= v4.3.0

```php
Swoole\Coroutine\Socket->sendAll(string $data, float $timeout = 0) : int | false;
```

  * **Parameters**

    * **`string $data`**
      * **Purpose**: 전송할 데이터 내용【텍스트 또는 이진 데이터가 될 수 있음】
      * **Default**: 없음
      * **Others**: 없음

    * **`float $timeout`**
      * **Purpose**: 타임아웃 설정
      * **Unit**: 초【부동 소수점 지원, 예: `1.5`는 `1초 + 500ms`를 의미함】
      * **Default**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)을 참조
      * **Others**: 없음

  * **Return Value**
  
    * `sendAll`은 데이터가 모두 성공적으로 전송되는 것을 보장하지만, `sendAll` 중에 상대방이 연결을 끊을 수 있으며, 이 경우 일부 데이터가 성공적으로 전송되었을 수 있습니다. 반환 값은 이 성공적인 데이터의 길이를 반환하며, 응용 프로그램 코드에서는 반환 값과 `strlen($data)`이 일치하는지 확인하여 전송이 완료되었는지 판단하고 비즈니스 요구에 따라 재전송이 필요할 수 있습니다.
    * 전송에 실패하면 `false`를 반환하고 `errCode` 속성을 설정합니다.
### peek()

데이터를 읽어 캐시 버퍼를 엿보는 것으로, 시스템 호출의 `recv(length, MSG_PEEK)`와 비슷합니다.

!> `peek`는 즉시 완료되며, 코루틴을 일시 중지시키지 않지만 한 번의 시스템 호출 오버헤드가 있습니다.

```php
Swoole\Coroutine\Socket->peek(int $length = 65535): string|false
```

  * **매개변수** 

    * **`int $length`**
      * **기능**：엿보는 데이터를 복사하기 위한 메모리 크기를 지정합니다. (참고: 여기서 메모리를 할당하며, 너무 큰 길이는 메모리 고갈로 이어질 수 있습니다.)
      * **단위**：바이트
      * **기본값**：없음
      * **다른 값**：없음

  * **반환값** 

    * 엿보기 성공시 데이터 반환
    * 엿보기 실패시 `false`가 반환되며, `errCode` 속성이 설정됩니다.
### recv()

데이터를 수신합니다.

!> `recv` 메소드는 즉시 현재 코루틴을 일시 중단시키고 읽을 수 있는 이벤트를 대기하며, 상대방이 데이터를 보낼 때까지 기다린 후 읽을 수 있는 이벤트가 트리거되면 `recv` 시스템 호출을 실행하여 `socket` 버퍼 영역의 데이터를 가져오고 해당 코루틴을 깨웁니다.

```php
Swoole\Coroutine\Socket->recv(int $length = 65535, float $timeout = 0): string|false
```

  * **매개변수** 

    * **`int $length`**
      * **기능**：데이터를 수신하는 데 사용할 메모리 크기 지정 (주의: 여기서는 메모리를 할당하며, 큰 길이는 메모리 고갈을 유발할 수 있음)
      * **값 단위**：바이트
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간 설정
      * **값 단위**：초【부동 소수점 지원, 예: `1.5`는 `1초` + `500ms`를 의미함】
      * **기본값**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃 규칙) 참조
      * **다른 값**：없음

  * **반환 값** 

    * 수신 성공 시 실제 데이터 반환
    * 수신 실패 시 `false` 반환하고 `errCode` 속성 설정
    * 수신 타임아웃 시 오류 코드는 `ETIMEDOUT`

!> 반환 값은 항상 기대한 길이와 일치하지 않을 수 있으며, 이 호출로 수신된 데이터의 길이를 직접 확인해야 합니다. 특정 길이의 데이터를 한 번에 얻으려면 `recvAll` 메소드를 사용하거나 직접 루프를 돌려야 합니다  
TCP 데이터 패킷 경계 문제에 대해서는 `setProtocol()` 메소드를 참고하거나 `sendto()`를 사용하세요;
### recvAll()

데이터를 수신합니다. `recv`와 다르게, `recvAll`은 응답 길이의 데이터를 가능한 완전히 수신하고, 완료되거나 오류가 발생할 때까지 계속 수신합니다.

!> `recvAll` 메서드는 현재 코루틴을 즉시 중단시키고 읽을 수 있는 이벤트를 대기합니다. 상대방이 데이터를 보내면 읽을 수 있는 이벤트가 트리거되고, `recv` 시스템 호출을 실행하여 `socket` 버퍼 영역에서 데이터를 가져옵니다. 지정된 길이의 데이터를 수신하거나 오류가 발생할 때까지 이 작업을 반복하고, 해당 코루틴을 다시 깨웁니다.

!> Swoole 버전 >= v4.3.0

```php
Swoole\Coroutine\Socket->recvAll(int $length = 65535, float $timeout = 0): string|false
```

  * **Parameters** 

    * **`int $length`**
      * **기능**：받기를 기대하는 데이터 크기 (참고: 여기서 메모리가 할당되며, 큰 길이는 메모리 고갈을 초래할 수 있습니다)
      * **값 단위**：바이트
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간 설정
      * **값 단위**：초【부동 소수점을 지원하며, `1.5`는 `1초` + `500밀리초`를 의미합니다】
      * **기본값**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)을 참조
      * **다른 값**：없음

  * **Return Value** 

    * 성공적으로 수신한 경우에는 실제 데이터를 반환하고, 반환된 문자열 길이는 매개변수 길이와 일치합니다.
    * 수신에 실패한 경우에는 `false`를 반환하고, `errCode` 속성을 설정합니다.
    * 수신 시간 초과의 경우 오류 코드는 `ETIMEDOUT`입니다.
### readVector()

데이터를 분할하여 수신합니다.

!> `readVector` 메서드는 즉시 `readv` 시스템 호출을 사용하여 데이터를 읽습니다. `readv` 시스템 호출이 `EAGAIN` 오류를 반환하면, 기초부에서는 자동으로 읽기 이벤트를 감시하고 현재 코루틴을 일시 중지시킨 후, 읽기 이벤트가 발생하면 다시 `readv` 시스템 호출을 실행하여 데이터를 읽고 해당 코루틴을 깨웁니다.

!> Swoole 버전 >= v4.5.7

```php
Swoole\Coroutine\Socket->readVector(array $io_vector, float $timeout = 0): array|false
```

  * **파라미터**

    * **`array $io_vector`**
      * **기능**: 받고자 하는 데이터의 분할 크기
      * **값 단위**: 바이트
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`float $timeout`**
      * **기능**: 타임아웃 시간 설정
      * **값 단위**: 초【부동 소수점 지원, 예: `1.5`는 `1초`+`500밀리초`를 의미함】
      * **기본값**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=超时规则)을 참조
      * **다른 값**: 없음

  * **반환 값**

    * 성공적으로 수신한 분할 데이터
    * 수신 실패 시 빈 배열을 반환하고 `errCode` 속성을 설정
    * 수신 시간 초과시 오류 코드는 `ETIMEDOUT`

  * **예시**

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// 상대가 helloworld를 보냈다면
$ret = $socket->readVector([5, 5]);
// 그렇다면, $ret은 ['hello', 'world']가 됩니다.
```
### readVectorAll()

데이터를 분할해서 수신합니다.

!> `readVectorAll` 함수는 데이터를 읽기 위해 여러 번의 `readv` 시스템 호출을 즉시 실행하며, `readv` 시스템 호출이 `EAGAIN` 오류를 반환하면, 하위 수준에서 자동으로 읽기 가능한 이벤트를 청취하고 현재 코루틴을 일시 중지시킨 후, 읽기 가능한 이벤트가 발생하면 다시 `readv` 시스템 호출을 실행하여 데이터를 읽습니다. 데이터 읽기가 완료되거나 오류가 발생할 때까지 해당 코루틴을 깨웁니다.

!> Swoole 버전 >= v4.5.7

```php
Swoole\Coroutine\Socket->readVectorAll(array $io_vector, float $timeout = 0): array|false
```

  * **매개변수** 

    * **`array $io_vector`**
      * **기능**：받기를 기대하는 분할된 데이터 크기
      * **값 단위**：바이트
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간 설정
      * **값 단위**：초【부동 소수점 지원, 예: `1.5`는 `1초` + `500ms`를 나타냄】
      * **기본값**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=超时规则)을 참조
      * **다른 값**：없음

  * **반환 값**

    * 성공적으로 수신된 분할된 데이터
    * 수신 실패 시 빈 배열을 반환하고 `errCode` 속성을 설정함
    * 타임아웃이 발생한 경우 오류 코드는 `ETIMEDOUT`
### writeVector()

데이터를 분할해서 보냅니다.

!> `writeVector` 메소드는 데이터를 보내기 위해 즉시 `writev` 시스템 호출을 실행하며, `writev` 시스템 호출이 `EAGAIN` 오류로 반환될 때, 하위 단계에서 자동으로 쓰기 가능 이벤트를 감시하고 현재 코루틴을 일시 중단시키며, 쓰기 가능 이벤트가 발생하면 다시 `writev` 시스템 호출을 실행하여 데이터를 보내고 해당 코루틴을 깨웁니다.

!> Swoole 버전 >= v4.5.7

```php
Swoole\Coroutine\Socket->writeVector(array $io_vector, float $timeout = 0): int|false
```

  * **매개변수** 

    * **`array $io_vector`**
      * **기능**：전송하려는 분할 데이터
      * **단위**：바이트
      * **기본값**：없음
      * **기타 값**：없음

    * **`float $timeout`**
      * **기능**：시간 제한 설정
      * **단위**：초【부동 소수점 지원, 예: `1.5`는 `1초`+`500ms`를 의미함】
      * **기본값**：[클라이언트 시간 제한 규칙](/coroutine_client/init?id=시간제한규칙) 참조
      * **기타 값**：없음

  * **반환 값**

    * 성공적으로 전송되면 보낸 바이트 수가 반환됨. 실제로 쓰인 데이터가 `$io_vector` 매개변수의 총 길이보다 작을 수 있으므로, 애플리케이션 레이어 코드는 반환 값과 `$io_vector` 매개변수의 총 길이가 일치하는지 확인하여 전송이 완료되었는지 판단해야 함
    * 전송 실패 시 `false`를 반환하고 `errCode` 속성을 설정함

  * **예시** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// 이 시점에서 배열 내의 순서대로 상대방에게 전송되며, 사실상 'helloworld'를 보냅니다
$socket->writeVector(['hello', 'world']);
```
### writeVectorAll()

데이터를 대상에게 보냅니다. `writeVector` 메소드와 다르게, `writeVectorAll`은 가능한 한 모든 데이터를 성공적으로 보낼 때까지 데이터를 보내거나 오류가 발생할 때까지 전송을 중지합니다.

!> `writeVectorAll` 메소드는 즉시 여러 번의 `writev` 시스템 호출을 실행하여 데이터를 보냅니다. `writev` 시스템 호출이 `EAGAIN` 오류를 반환하면, 밑바닥에서 자동으로 쓰기 가능 이벤트를 관찰하고, 현재 코루틴을 일시 중지하여 쓰기 가능 이벤트가 발생하면 다시 `writev` 시스템 호출을 실행하여 데이터를 보냅니다. 데이터 송신을 완료하거나 오류를 만날 때까지 해당 코루틴을 깨웁니다.

!> Swoole 버전 >= v4.5.7

```php
Swoole\Coroutine\Socket->writeVectorAll(array $io_vector, float $timeout = 0): int|false
```

  * **Parameters** 

    * **`array $io_vector`**
      * **Description**：전송하려는 세그먼트 데이터
      * **Unit**：바이트
      * **Default**：없음
      * **Other values**：없음

    * **`float $timeout`**
      * **Description**：타임아웃 시간 설정
      * **Unit**：초【부동 소수점 지원, 예: `1.5`는 `1초`+`500ms`를 의미】
      * **Default**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃 규칙)을 참조
      * **Other values**：없음

  * **Return Value**

    * `writeVectorAll`은 데이터가 모두 성공적으로 전송되도록 보장하지만, `writeVectorAll` 중간에 상대방이 연결을 끊을 수 있으므로 이 경우 일부 데이터가 성공적으로 전송되었을 수 있으며, 반환 값은 이 성공적인 데이터의 길이를 반환합니다. 응용 프로그램 레벨의 코드는 반환 값과 `$io_vector` 매개변수의 총 길이가 동일한지 비교하여 전송이 완료되었는지 판단하고 비즈니스 요구에 따라 이어보내야 할 수 있습니다.
    * 전송 실패 시 `false`를 반환하고 `errCode` 속성을 설정합니다

  * **Example** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// 이제 배열 안에 있는 순서대로 상대에게 보내게 됩니다. 사실상 helloworld를 보냅니다.
$socket->writeVectorAll(['hello', 'world']);
```
### recvPacket()

`setProtocol` 메소드를 통해 프로토콜을 설정한 소켓 객체에 대해, 이 메소드를 호출하여 완전한 프로토콜 데이터 패킷을 수신할 수 있습니다.

!> Swoole 버전 >= v4.4.0

```php
Swoole\Coroutine\Socket->recvPacket(float $timeout = 0): string|false
```

  * **매개변수** 
    * **`float $timeout`**
      * **기능**: 타임아웃 시간 설정
      * **단위**: 초【부동 소수점 지원, `1.5`는 `1s`+`500ms`를 의미함】
      * **기본값**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙) 참조
      * **다른 값**: 없음

  * **반환값** 

    * 성공적으로 수신하면 완전한 프로토콜 데이터 패킷을 반환
    * 수신에 실패하면 `false`를 반환하고 `errCode` 속성을 설정함
    * 수신이 타임아웃되면 오류 코드가 `ETIMEDOUT`임
### recvLine()

[socket_read](https://www.php.net/manual/en/function.socket-read.php) 호환성 문제를 해결하기 위해 사용됩니다.

```php
Swoole\Coroutine\Socket->recvLine(int $length = 65535, float $timeout = 0): string|false
```
### recvWithBuffer()

`recv(1)`을 사용하여 한 바이트씩 수신할 때 발생하는 대량의 시스템 호출 문제를 해결하기 위해 사용됩니다.

```php
Swoole\Coroutine\Socket->recvWithBuffer(int $length = 65535, float $timeout = 0): string|false
```
### recvfrom()

데이터를 수신하고 송신 호스트의 주소와 포트를 설정합니다. `SOCK_DGRAM` 유형의 `socket`에 사용됩니다.

!> 이 메소드는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 유발하며, 하위 수준에서 현재 코루틴을 즉시 일시 중지하고 읽기 가능한 이벤트를 대기합니다. 읽기 가능한 이벤트가 발생하면 데이터를 수신하여 `recvfrom` 시스템 호출을 실행하여 데이터 패킷을 받습니다.

```php
Swoole\Coroutine\Socket->recvfrom(array &$peer, float $timeout = 0): string|false
```

* **매개변수**

    * **`array $peer`**
        * **기능**: 상대방 주소 및 포트, 참조 유형입니다. 【함수가 성공한 경우에 배열로 설정됩니다. `address`와 `port` 두 요소를 포함합니다.】
        * **기본값**: 없음
        * **다른 값**: 없음

    * **`float $timeout`**
        * **기능**: 시간 초과 설정【주어진 시간 내에 데이터가 반환되지 않으면 `recvfrom` 메소드가 `false`를 반환합니다.】
        * **단위**: 초【부동 소수점 지원. 예: `1.5`는 `1초` + `500ms`를 나타냅니다.】
        * **기본값**: [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)을 참조하세요.
        * **다른 값**: 없음

* **반환 값**

    * 데이터를 성공적으로 수신하면 데이터 내용을 반환하고 `$peer`를 배열로 설정합니다.
    * 실패하면 `false`를 반환하고 `errCode` 속성을 설정하며 `$peer`의 내용은 변경하지 않습니다.

* **예시**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    $socket = new Coroutine\Socket(AF_INET, SOCK_DGRAM, 0);
    $socket->bind('127.0.0.1', 9601);
    while (true) {
        $peer = null;
        $data = $socket->recvfrom($peer);
        echo "[Server] recvfrom[{$peer['address']}:{$peer['port']}] : $data\n";
        $socket->sendto($peer['address'], $peer['port'], "Swoole: $data");
    }
});
```
### sendto()

특정 주소와 포트로 데이터를 전송합니다. `SOCK_DGRAM` 유형의 `socket`에 사용됩니다.

!> 이 방법은 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 사용하지 않습니다. 바닥부에서 `sendto`를 즉시 호출하여 대상 호스트로 데이터를 전송합니다. 이 방법은 쓰기 가능 여부를 듣지 않으며, 버퍼가 가득 찬 경우 `false`를 반환할 수 있습니다. 이 경우 직접 처리하거나 `send` 방법을 사용하십시오.

```php
Swoole\Coroutine\Socket->sendto(string $address, int $port, string $data): int|false
```

  * **Parameters** 

    * **`string $address`**
      * **Description**: 대상 호스트의 `IP` 주소 또는 [unixSocket](/learn?id=ipc) 경로입니다. 【`sendto`는 도메인 이름을 지원하지 않으며, `AF_INET` 또는 `AF_INET6`를 사용할 때 유효한 `IP` 주소를 제공해야 합니다. 그렇지 않으면 전송이 실패할 수 있습니다.】
      * **Default value**: 없음
      * **Other values**: 없음

    * **`int $port`**
      * **Description**: 대상 호스트의 포트 번호입니다. 【브로드캐스트를 보낼 때 `0`이 될 수 있습니다.】
      * **Default value**: 없음
      * **Other values**: 없음

    * **`string $data`**
      * **Description**: 전송할 데이터입니다. 【텍스트 또는 이진 콘텐츠가 될 수 있으며, `SOCK_DGRAM`으로 전송하는 패킷의 최대 길이는 `64K`입니다.】
      * **Default value**: 없음
      * **Other values**: 없음

  * **Return value** 

    * 성공적으로 전송된 바이트 수를 반환합니다.
    * 전송 실패 시 `false`를 반환하고 `errCode` 속성을 설정합니다.

  * **Example** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_DGRAM, 0);
$socket->sendto('127.0.0.1', 9601, 'Hello');
```
### getsockname()

소켓의 주소와 포트 정보를 가져옵니다.

!> 이 메서드에는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링) 오버헤드가 없습니다.

```php
Swoole\Coroutine\Socket->getsockname(): array|false
```

  * **반환값** 

    * 성공 시 `address`와 `port`를 포함하는 배열 반환
    * 실패 시 `false`를 반환하고 `errCode` 속성을 설정
### getpeername()

`socket`의 상대방 주소 및 포트 정보를 가져옵니다. 연결이 있는 `SOCK_STREAM` 유형의 `socket`에서만 사용됩니다.

?> 이 방법은 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링) 비용이 없습니다.

```php
Swoole\Coroutine\Socket->getpeername(): array|false
```

  * **반환 값** 

    * 성공 시 호출하면 `address` 및 `port`를 포함하는 배열을 반환합니다.
    * 실패한 경우 `false`를 반환하고 `errCode` 속성을 설정합니다.
### close()

`Socket`를 닫습니다.

!> `Swoole\Coroutine\Socket` 객체가 파괴될 때 자동으로 `close`가 실행되며, 이 메소드는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링) 비용이 없습니다.

```php
Swoole\Coroutine\Socket->close(): bool
```

  * **반환 값** 

    * 성공적으로 닫히면 `true`를 반환합니다.
    * 실패하면 `false`를 반환합니다.
### isClosed()

`Socket`이 닫혔는지 여부를 나타냅니다.

```php
Swoole\Coroutine\Socket->isClosed(): bool
```
