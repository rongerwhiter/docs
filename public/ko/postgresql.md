# 코루틴\PostgreSQL

Swoole 5.0 버전에서 완전히 새롭게 구성된 코루틴 `PostgreSQL` 클라이언트입니다.

이전 버전을 사용 중이라면 [이전 버전 문서](/coroutine_client/postgresql-old.md)를 확인해 주세요.
## 컴파일 및 설치

- 시스템에 `libpq` 라이브러리가 설치되어 있는지 확인해야 합니다.
- `mac`에는 `postgresql`이 설치되면 `libpq` 라이브러리가 기본으로 제공되지만, 환경에 따라 차이가 있습니다. `ubuntu`의 경우 `apt-get install libpq-dev`가 필요할 수 있고, `centos`의 경우 `yum install postgresql10-devel`이 필요할 수 있습니다.
- Swoole을 컴파일할 때 다음과 같은 옵션을 추가하세요: `./configure --enable-swoole-pgsql`
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    if (!$conn) {
        var_dump($pg->error);
        return;
    }
    $stmt = $pg->query('SELECT * FROM test;');
    $arr = $stmt->fetchAll();
    var_dump($arr);
});
```
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    $pg->query('BEGIN');
    $stmt = $pg->query('SELECT * FROM test');
    $arr = $stmt->fetchAll();
    $pg->query('COMMIT');
    var_dump($arr);
});
```
## 속성
### 오류

오류 정보를 가져옵니다.
## 방법
### connect()

`postgresql`의 비차단 코루틴 연결을 설정합니다.

```php
Swoole\Coroutine\PostgreSQL->connect(string $conninfo, float $timeout = 2): bool
```

!> `$conninfo`는 연결 정보이며, 연결 성공 시 true를 반환하고 연결 실패 시 false를 반환합니다. 에러 정보는 [error](/coroutine_client/postgresql?id=error) 속성을 사용하여 얻을 수 있습니다.
  * **예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
    var_dump($pg->error, $conn);
});
```
### query()

SQL 문을 실행합니다. 비동기적인 논블로킹 코루틴 명령을 보냅니다.

```php
Swoole\Coroutine\PostgreSQL->query(string $sql): \Swoole\Coroutine\PostgreSQLStatement|false;
```

  * **매개변수** 

    * **`string $sql`**
      * **기능**：SQL 문
      * **기본값**：없음
      * **다른 값**：없음

  * **예시**

    * **select**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $stmt = $pg->query('SELECT * FROM test;');
        $arr = $stmt->fetchAll();
        var_dump($arr);
    });
    ```

    * **insert id 반환**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
        $stmt = $pg->query("insert into test (id,text) VALUES (24,'text') RETURNING id ;");
        $arr = $stmt->fetchRow();
        var_dump($arr);
    });
    ```

    * **트랜잭션**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $pg->query('BEGIN;');
        $stmt = $pg->query('SELECT * FROM test;');
        $arr = $stmt->fetchAll();
        $pg->query('COMMIT;');
        var_dump($arr);
    });
    ```
### metaData()

테이블의 메타데이터를 확인합니다. 비동기 논블로킹 코루틴 버전입니다.

```php
Swoole\Coroutine\PostgreSQL->metaData(string $tableName): array
```

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->metaData('test');
    var_dump($result);
});
```
### prepare()

준비 작업.

```php
$stmt = Swoole\Coroutine\PostgreSQL->prepare(string $sql);
$stmt->execute(array $params);
```

  * **사용 예**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=112");
    $stmt = $pg->prepare("select * from test where id > $1 and id < $2");
    $res = $stmt->execute(array(1, 3));
    $arr = $stmt->fetchAll();
    var_dump($arr);
});
```
## PostgreSQLStatement

Class: `Swoole\Coroutine\PostgreSQLStatement`

모든 쿼리는 `PostgreSQLStatement` 객체를 반환합니다.
### fetchAll()

```php
Swoole\Coroutine\PostgreSQLStatement->fetchAll(int $result_type = SW_PGSQL_ASSOC): false|array;
```

  * **매개변수**
    * **`$result_type`**
      * **기능**：상수. 선택적 매개변수로, 반환값의 초기화 및 구성을 제어합니다.
      * **기본값**：`SW_PGSQL_ASSOC`
      * **다른 값**：없음

      값 | 반환값
      ---|---
      SW_PGSQL_ASSOC | 키를 필드 이름으로 하는 연관 배열을 반환
      SW_PGSQL_NUM | 키를 필드 번호로 하는 배열을 반환
      SW_PGSQL_BOTH | 둘 다를 키로 하는 배열을 반환

  * **반환값**

    * 결과에서 모든 행을 배열로 반환합니다.
### affectedRows()

데이터베이스에 영향을 준 레코드 수를 반환합니다.

```php
Swoole\Coroutine\PostgreSQLStatement->affectedRows(): int
```
### numRows()

행의 수를 반환합니다.

```php
Swoole\Coroutine\PostgreSQLStatement->numRows(): int
```
### fetchObject()

한 줄을 객체로 가져옵니다.

```php
Swoole\Coroutine\PostgreSQLStatement->fetchObject(int $row, ?string $class_name = null, array $ctor_params = []): object;
```

  * **예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    for ($row = 0; $row < $stmt->numRows(); $row++) {
        $data = $stmt->fetchObject($row);
        echo $data->id . " \n ";
    }
});
```
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    while ($data = $stmt->fetchObject($row)) {
        echo $data->id . " \n ";
        $row++;
    }
});
```  
```php
Swoole\Coroutine\PostgreSQLStatement->fetchAssoc(int $row, int $result_type = SW_PGSQL_ASSOC): array
```

`fetchAssoc()` 함수는 한 행을 연관 배열로 추출합니다.
```  
### fetchArray()

한 행을 배열로 추출합니다.

```php
Swoole\Coroutine\PostgreSQLStatement->fetchArray(int $row, int $result_type = SW_PGSQL_BOTH): array|false
```

  * **매개변수**
    * **`int $row`**
      * **기능**: `row`는 추출하려는 행(레코드)의 번호입니다. 첫 번째 행은 `0`입니다.
      * **기본값**: 없음
      * **다른 값**: 없음
    * **`$result_type`**
      * **기능**: 상수입니다. 선택적 매개변수로, 반환 값을 어떻게 초기화할지 제어합니다.
      * **기본값**: `SW_PGSQL_BOTH`
      * **다른 값**: 없음

      값 | 반환 값
      ---|---
      SW_PGSQL_ASSOC | 필드 이름을 키 값 인덱스로 사용하는 연관 배열을 반환합니다
      SW_PGSQL_NUM | 필드 번호를 키 값으로 사용하여 반환합니다
      SW_PGSQL_BOTH | 두 가지를 동시에 키 값으로 사용하여 반환합니다

  * **반환 값**

    * 추출한 행(튜플/레코드)과 일치하는 배열을 반환합니다. 더 이상 추출할 행이 없으면 `false`를 반환합니다.

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    $arr = $stmt->fetchArray(1, SW_PGSQL_ASSOC);
    var_dump($arr);
});
```
### fetchRow()

지정된 `result` 리소스를 기반으로 한 행 데이터를 배열로 추출하여 반환합니다. 각 열은 `0`에서 시작하는 오프셋으로 배열에 저장됩니다.

```php
Swoole\Coroutine\PostgreSQLStatement->fetchRow(int $row, int $result_type = SW_PGSQL_NUM): array|false
```

  * **매개변수**
    * **`int $row`**
      * **기능**：`row`는 가져올 행(레코드)의 번호입니다. 첫 번째 행은 `0`입니다.
      * **기본값**：없음
      * **다른 값**：없음
    * **`$result_type`**
      * **기능**：상수. 선택적 매개변수로, 반환 값을 초기화하는 방식을 제어합니다.
      * **기본값**：`SW_PGSQL_NUM`
      * **다른 값**：없음

      값 | 반환 값
      ---|---
      SW_PGSQL_ASSOC | 필드 이름을 키로 하는 연관 배열 반환
      SW_PGSQL_NUM | 필드 번호를 키로 하는 배열 반환
      SW_PGSQL_BOTH | 두 가지를 모두 키로 하는 배열 반환

  * **반환 값**

    * 반환된 배열은 추출된 행과 일치합니다. 더 이상 추출할 행 `row`이 없으면 `false`를 반환합니다.

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    while ($row = $stmt->fetchRow()) {
        echo "name: $row[0]  mobile: $row[1]" . PHP_EOL;
    }
});
```
