# Coroutine\MySQL

코루틴 MySQL 클라이언트.

!> 이 클라이언트는 더 이상 권장되지 않으며, Swoole\Runtime::enableCoroutine + PDO 또는 Mysqli 방식을 권장합니다. 즉, [원생 PHP의 MySQL 클라이언트를 코루틴화하는 방법](/runtime)을 추천합니다.

!> 구식인 `Swoole 1.x` 시대의 비동기 콜백 방식과 이 코루틴 MySQL 클라이언트를 동시에 사용하지 마십시오.
## 사용 예시

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('select sleep(1)');
    var_dump($res);
});
```
## defer 특성

[concurrency Client](/coroutine/multi_call)를 참조하십시오.
## 저장 프로시저

`4.0.0` 버전 이후부터, `MySQL` 저장 프로시저와 다중 결과 집합을 지원합니다.
`Swoole-4.0.1` 또는 그 이상의 버전에서는 `MySQL8`의 모든 보안 인증 기능을 지원하며, 비밀번호 설정을 롤백할 필요 없이 클라이언트를 정상적으로 사용할 수 있습니다.
### 4.0.1 이하 버전

`MySQL-8.0`은 기본적으로 더 강력한 보안을 제공하는 `caching_sha2_password` 플러그인을 사용합니다. 만약 `5.x`에서 업그레이드 했다면 모든 `MySQL` 기능을 바로 사용할 수 있지만, 새로운 `MySQL`을 만든 경우에는 다음과 같은 작업을 수행해야 합니다.

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
flush privileges;
```

위 문장에서 `'root'@'localhost'`를 사용중인 사용자로, `password`를 해당 사용자의 비밀번호로 대체해야 합니다.

만약 계속해서 사용에 문제가 있는 경우, `my.cnf` 파일에서 `default_authentication_plugin = mysql_native_password` 설정을 해야 합니다.
## 속성
### serverInfo

연결 정보입니다. 이는 연결 함수에 전달된 배열을 저장합니다.
### 양말

연결에 사용되는 파일 서술자.
### 연결됨

`MySQL` 서버에 연결되었는지 확인합니다.

!> [연결되었음에도 'connected' 속성과 연결 상태가 일치하지 않는 경우](/question/use?id=connected属性和连接状态不一致)를 참고하세요.
### connect_error

서버에 연결할 때 `connect`를 실행하는 중 발생한 오류 메시지입니다.
### connect_errno

`connect` 함수를 사용하여 서버에 연결할 때 발생하는 오류 코드이며, 정수형입니다.
### 에러

'MySQL' 지시를 실행할 때 서버에서 반환된 오류 메시지입니다.
### errno

`MySQL` 명령을 실행할 때 서버가 반환하는 오류 코드로, 정수형입니다.
### affected_rows

영향을 받은 행의 수입니다.
### insert_id

가장 최근에 삽입된 레코드의 `id`.
## Methods
### connect()

MySQL 연결을 설정합니다.

```php
Swoole\Coroutine\MySQL->connect(array $serverInfo): bool
```

!> `$serverInfo`：매개변수는 배열 형태로 전달됩니다.

```php
[
    'host'        => 'MySQL IP 주소', // 로컬 UNIX 소켓인 경우, `unix://tmp/your_file.sock` 형식으로 작성해야 함
    'user'        => '데이터 사용자',
    'password'    => '데이터베이스 비밀번호',
    'database'    => '데이터베이스 이름',
    'port'        => 'MySQL 포트 기본값은 3306 선택 사항',
    'timeout'     => '연결 설정 시간 초과', // connect 시간 초과만 영향을 줌, query와 execute 메서드에는 영향을 주지 않음, '클라이언트 시간 초과 규칙'을 참조
    'charset'     => '문자 집합',
    'strict_type' => false, // 엄격 모드 활성화, query 메서드로 반환된 데이터도 강제 형변환됨
    'fetch_mode'  => true,  // fetch 모드 활성화, pdo와 같이 fetch/fetchAll을 통해 행 단위로 차례로 또는 즉시 전체 결과 집합을 가져올 수 있음 (4.0 버전 이상)
]
```
### query()

SQL 문을 실행합니다.

```php
Swoole\Coroutine\MySQL->query(string $sql, float $timeout = 0): array|false
```

  * **매개변수** 

    * **`string $sql`**
      * **기능**：SQL 문
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간 【지정된 시간 내에 `MySQL` 서버가 데이터를 반환하지 않으면 하부에서 `false`를 반환하고 오류 코드를 `110`으로 설정하고 연결을 끊음】
      * **단위**：초, 최소 정밀도는 밀리초(`0.001` 초)
      * **기본값**：`0`
      * **다른 값**：없음
      * **참고[클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)**


  * **반환 값**

    * 타임아웃 또는 에러 시 `false`를 반환하고, 그렇지 않으면 쿼리 결과를 `array` 형식으로 반환합니다.

  * **지연 수신**

  !> `defer`를 설정한 후 `query`를 호출하면 직접적으로 `true`를 반환합니다. `recv`를 호출해야만 쿼리 결과를 받아올 수 있음.

  * **예시**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('show tables');
    if ($res === false) {
        return;
    }
    var_dump($res);
});
```  
### prepare()

MySQL 서버에 SQL 준비 요청을 보냅니다.

!> `prepare`는 `execute`와 함께 사용해야 합니다. 준비 요청이 성공한 후에는 `execute` 메서드를 호출하여 데이터 매개변수를 MySQL 서버로 보냅니다.

```php
Swoole\Coroutine\MySQL->prepare(string $sql, float $timeout): Swoole\Coroutine\MySQL\Statement|false;
```

  * **매개변수** 

    * **`string $sql`**
      * **기능**：준비된 문장【매개변수 자리 표시자로 `?`를 사용합니다】
      * **기본값**：없음
      * **다른 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간
      * **값의 단위**：초, 최소 정밀도는 밀리초(`0.001`초)
      * **기본값**：`0`
      * **다른 값**：없음
      * **[클라이언트 타임아웃 규칙](/coroutine_client/init?id=超时规则)을 참조하세요**

  * **반환 값**

    * 실패 시 `false`를 반환하며, `$db->error` 및 `$db->errno`를 확인하여 오류 원인을 판단할 수 있습니다.
    * 성공 시 `Coroutine\MySQL\Statement` 객체를 반환하며, 객체의 [execute](/coroutine_client/mysql?id=statement-gtexecute) 메서드를 호출하여 매개변수를 보낼 수 있습니다.

  * **예시**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10));
        var_dump($ret2);
    }
});
```
### escape()

SQL 문의 특수 문자를 이스케이프하여 SQL 인젝션 공격을 방지합니다. `mysqlnd`에 기반한 기능으로, `PHP`의 `mysqlnd` 확장에 의존합니다.

!> 컴파일 시 [--enable-mysqlnd](/environment?id=编译选项)를 추가하여 활성화해야 합니다.

```php
Swoole\Coroutine\MySQL->escape(string $str): string
```

  * **매개변수** 

    * **`string $str`**
      * **목적**：문자열 이스케이프
      * **기본값**：없음
      * **다른 값**：없음

  * **사용 예시**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $data = $db->escape("abc'efg\r\n");
});
```
### begin()

트랜잭션을 시작합니다. `commit` 및 `rollback`을 함께 사용하여 `MySQL` 트랜잭션 처리를 구현합니다.

```php
Swoole\Coroutine\MySQL->begin(): bool
```

!> `MySQL` 트랜잭션을 시작하고, 성공 시 `true`를 반환하고, 실패 시 `false`를 반환합니다. 오류 코드를 얻으려면 `$db->errno`을 확인하세요.
  
!> 동일한 `MySQL` 연결 객체는 한 번에 하나의 트랜잭션만 시작할 수 있습니다;  
이전 트랜잭션이 `commit` 또는 `rollback`될 때까지 새 트랜잭션을 시작해야 합니다;  
그렇지 않으면 하위 수준에서 `Swoole\MySQL\Exception` 예외가 발생합니다. 예외 `code`는 `21`입니다.

  * **예시**

    ```php
    $db->begin();
    $db->query("update userinfo set level = 22 where id = 1");
    $db->commit();
    ```
### commit()

트랜잭션을 커밋합니다.

!> `begin`과 함께 사용되어야 합니다.

```php
Swoole\Coroutine\MySQL->commit(): bool
```

!> 성공 시 `true`를 반환하고, 실패 시 `false`를 반환하며, 오류 코드를 얻으려면 `$db->errno`을 확인하십시오.
### rollback()

트랜잭션을 롤백합니다.

!> `begin`과 함께 사용해야 합니다.

```php
Swoole\Coroutine\MySQL->rollback(): bool
```

!> 성공 시 `true`를 반환하며, 실패 시 `false`를 반환합니다. 오류 코드를 얻으려면 `$db->errno`을 확인해주세요.
### Statement->execute()

MySQL 서버에 SQL 준비 데이터 매개변수를 보냅니다.

!> `execute`는 `prepare`와 함께 사용해야 하며, `execute`를 호출하기 전에 먼저 `prepare`를 호출하여 준비 요청을 보내야 합니다.

!> `execute` 메서드는 여러 번 호출할 수 있습니다.

```php
Swoole\Coroutine\MySQL\Statement->execute(array $params, float $timeout = -1): array|bool
```

  * **Parameters** 

    * **`array $params`**
      * **Purpose**：준비 데이터 매개변수 【`prepare` 문의 매개변수 수와 동일해야 합니다. `$params`는 숫자 색인 배열이어야 하며, 매개변수 순서는 `prepare` 문과 동일해야 합니다】
      * **Default**：없음
      * **Other values**：없음

    * **`float $timeout`**
      * **Purpose**：타임아웃 시간 【지정된 시간 내에`MySQL` 서버가 데이터를 반환하지 못한 경우, 하위 수준에서는 `false`를 반환하고, 오류 코드를 `110`으로 설정하고 연결을 끊습니다】
      * **Value Unit**：초, 최소 정밀도는 밀리초(`0.001`초)
      * **Default**：`-1`
      * **Other values**：없음
      * **Reference[Client Timeout Rules](/coroutine_client/init?id=timeout-rules)**

  * **Return Value** 

    * 성공 시 `true`를 반환하며, `connect`의 `fetch_mode` 매개변수를 `true`로 설정한 경우
    * 성공 시 데이터 세트 배열인 `array`를 반환하며, 위의 경우가 아닐 때,
    * 실패하면 `false`를 반환하며, 오류 원인을 판단할 수 있도록 `$db->error` 및 `$db->errno`를 확인할 수 있습니다

  * **Usage Example** 

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=? and name=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10, 'rango'));
        var_dump($ret2);

        $ret3 = $stmt->execute(array(13, 'alvin'));
        var_dump($ret3);
    }
});
```
### Statement->fetch()

결과 집합에서 다음 행을 가져옵니다.

```php
Swoole\Coroutine\MySQL\Statement->fetch(): ?array
```

!> Swoole 버전 >= `4.0-rc1`에서는 `connect` 할 때 `fetch_mode => true` 옵션을 추가해야 합니다.

  * **예시**

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
while ($ret = $stmt->fetch()) {
    var_dump($ret);
}
```

!> 새로운 `MySQL` 드라이버인 `v4.4.0`부터, `fetch`는 이전 예제의 방식으로 `NULL`을 읽을 때까지 사용해야하며, 그렇지 않으면 새 요청을 시작할 수 없습니다 (필요에 따라 메모리를 절약하기 위해).
### Statement->fetchAll()

결과 집합에 있는 모든 행을 포함하는 배열을 반환합니다.

```php
Swoole\Coroutine\MySQL\Statement->fetchAll():? array
```

!> Swoole 버전 >= `4.0-rc1`에서는 `connect` 시에 `fetch_mode => true` 옵션을 추가해야 합니다.

  * **예시** 

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
$stmt->fetchAll();
```
### Statement->nextResult()

한 개 이상의 응답 결과를 다루기 위한 문장 핸들을 다음 응답 결과로 전진시킵니다 (예: 저장 프로시저의 다중 결과 반환).

```php
Swoole\Coroutine\MySQL\Statement->nextResult():? bool
```

  * **반환 값**

    * 성공할 시 `TRUE` 반환
    * 실패할 시 `FALSE` 반환
    * 다음 결과가 없는 경우 `NULL` 반환

  * **예시** 

    * **fetch 모드가 아닌 경우**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $res  = $stmt->execute(['hello mysql!']);
    do {
      var_dump($res);
    } while ($res = $stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

    * **fetch 모드인 경우**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $stmt->execute(['hello mysql!']);
    do {
      $res = $stmt->fetchAll();
      var_dump($res);
    } while ($stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

!> 새 `MySQL` 드라이버인 `v4.4.0`부터, `fetch`는 `NULL`까지 반복해서 읽어야하며, 그렇지 않을 경우 새로운 요청이 시작되지 않습니다. (요구에 따른 동적 읽기 메커니즘으로 인해 메모리 절약됨)
