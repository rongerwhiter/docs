# Coroutine\PostgreSQL 오래된 버전

코루틴 `PostgreSQL` 클라이언트입니다. 이 기능을 활성화하려면 [ext-postgresql](https://github.com/swoole/ext-postgresql) 확장을 컴파일해야 합니다.

> 이 문서는 Swoole < 5.0에만 해당됩니다.
## 컴파일 및 설치

소스 코드 다운로드: [https://github.com/swoole/ext-postgresql](https://github.com/swoole/ext-postgresql), 해당 releases 버전이 Swoole 버전과 일치해야 합니다.

* 시스템에 `libpq` 라이브러리가 설치되어 있는지 확인해야 합니다.
* `mac`에는 `postgresql`이 설치될 때 `libpq` 라이브러리가 함께 설치되지만, 환경에 따라 다양한 차이가 있습니다. `ubuntu`의 경우 `apt-get install libpq-dev`를 실행해야 할 수 있고, `centos`의 경우 `yum install postgresql10-devel`을 실행해야 할 수도 있습니다.
* `libpq` 라이브러리 디렉토리를 직접 지정할 수도 있습니다. 예시: `./configure --with-libpq-dir=/etc/postgresql`
## 사용 예시

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
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchAll($result);
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
    $result = $pg->query('SELECT * FROM test');
    $arr = $pg->fetchAll($result);
    $pg->query('COMMIT');
    var_dump($arr);
});
```
## 속성
### 에러

에러 메시지를 가져옵니다.
해당 주제에 대해 추가 정보가 필요하신가요?
### connect()

`postgresql` 비차단 코루틴 연결을 설정합니다.

```php
Swoole\Coroutine\PostgreSQL->connect(string $connection_string): bool
```

!> `$connection_string`은 연결 정보이며, 연결 성공시 true를 반환하고, 연결 실패시 false를 반환합니다. 에러 정보는 [error](/coroutine_client/postgresql?id=error) 속성을 사용하여 얻을 수 있습니다.
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
```php
Swoole\Coroutine\PostgreSQL->query(string $sql): resource;
```

  * **매개변수** 

    * **`string $sql`**
      * **기능**：SQL 쿼리
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
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
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
        $result = $pg->query("insert into test (id,text) VALUES (24,'text') RETURNING id ;");
        $arr = $pg->fetchRow($result);
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
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
        $pg->query('COMMIT;');
        var_dump($arr);
    });
    ```
### fetchAll()

```php
Swoole\Coroutine\PostgreSQL->fetchAll(resource $queryResult, $resultType = SW_PGSQL_ASSOC):? array;
```

  * **매개변수**
    * **`$resultType`**
      * **기능**: 상수. 선택적 매개변수로, 반환 값의 초기화를 조절합니다.
      * **기본값**: `SW_PGSQL_ASSOC`
      * **다른 값**: 없음

      값 | 반환 값
      ---|---
      SW_PGSQL_ASSOC | 필드 이름을 키로 사용하여 연관 배열을 반환합니다.
      SW_PGSQL_NUM | 필드 번호를 키로 사용하여 반환합니다.
      SW_PGSQL_BOTH | 둘 다를 키로 사용하여 반환합니다.

  * **반환 값**

    * 모든 행을 추출하여 배열로 반환합니다.
### affectedRows()

```php
Swoole\Coroutine\PostgreSQL->affectedRows(resource $queryResult): int
```
```php
Swoole\Coroutine\PostgreSQL->numRows(resource $queryResult): int
```

### numRows()

행의 수를 반환합니다.
### fetchObject()

한 행을 객체로 가져옵니다.

```php
Swoole\Coroutine\PostgreSQL->fetchObject(resource $queryResult, int $row): object;
```

  * **예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    for ($row = 0; $row < $pg->numRows($result); $row++) {
        $data = $pg->fetchObject($result, $row);
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
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    while ($data = $pg->fetchObject($result, $row)) {
        echo $data->id . " \n ";
        $row++;
    }
});
```
### fetchAssoc()

한 줄을 연관 배열로 추출합니다.

```php
Swoole\Coroutine\PostgreSQL->fetchAssoc(resource $queryResult, int $row): array
```
### fetchArray()

한 줄을 배열로 추출합니다.

```php
Swoole\Coroutine\PostgreSQL->fetchArray(resource $queryResult, int $row, $resultType = SW_PGSQL_BOTH): array|false
```

  * **매개변수**
    * **`int $row`**
      * **기능**: `row`는 가져올 행(레코드)의 번호입니다. 첫 번째 행은 `0`입니다.
      * **기본값**: 없음
      * **다른 값**: 없음
    * **`$resultType`**
      * **기능**: 상수. 선택적 매개변수로, 반환 값을 어떻게 초기화할지를 제어합니다.
      * **기본값**: `SW_PGSQL_BOTH`
      * **다른 값**: 없음

      값 | 반환값
      ---|---
      SW_PGSQL_ASSOC | 필드 이름을 키 값 인덱스로 하는 연관 배열을 반환
      SW_PGSQL_NUM | 필드 번호를 키 값으로 사용하여 반환
      SW_PGSQL_BOTH | 두 가지를 모두 키 값으로 사용하여 반환

  * **반환 값**

    * 추출한 행(튜플/레코드)과 일치하는 배열을 반환합니다. 더 이상 추출할 행이 없으면 `false`를 반환합니다.

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchArray($result, 1, SW_PGSQL_ASSOC);
    var_dump($arr);
});
```
### fetchRow()

지정된 `result` 리소스에 따라 한 행의 데이터를 배열로 반환합니다. 각 열은 0부터 시작하는 오프셋에 순서대로 저장됩니다.

```php
Swoole\Coroutine\PostgreSQL->fetchRow(resource $queryResult, int $row, $resultType = SW_PGSQL_NUM): array|false
```

  * **매개변수**
    * **`int $row`**
      * **기능**：`row`는 가져올 행(레코드)의 번호입니다. 첫 번째 행은 `0`입니다.
      * **기본값**：없음
      * **기타 값**：없음
    * **`$resultType`**
      * **기능**：상수입니다. 선택적 매개변수로 반환 값을 초기화하는 방법을 제어합니다.
      * **기본값**：`SW_PGSQL_NUM`
      * **기타 값**：없음

      값 | 반환 값
      ---|---
      SW_PGSQL_ASSOC | 필드 이름을 키 값 인덱스로 사용하는 연관 배열 반환
      SW_PGSQL_NUM | 필드 번호를 키 값으로 사용하여 반환
      SW_PGSQL_BOTH | 둘 다를 키 값으로 사용하여 반환

  * **반환값**

    * 반환된 배열은 추출된 행과 일치합니다. 더 이상 가져올 행 `row`이 없는 경우 `false`를 반환합니다.

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    while ($row = $pg->fetchRow($result)) {
        echo "name: $row[0]  mobile: $row[1]" . PHP_EOL;
    }
});
```
### metaData()

테이블의 메타데이터를 확인합니다. 비동기 및 블로킹되지 않는 코루틴 버전입니다.

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

전처리.

```php
Swoole\Coroutine\PostgreSQL->prepare(string $name, string $sql);
Swoole\Coroutine\PostgreSQL->execute(string $name, array $bind);
```

  * **사용 예시**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=112");
    $pg->prepare("my_query", "select * from  test where id > $1 and id < $2");
    $res = $pg->execute("my_query", array(1, 3));
    $arr = $pg->fetchAll($res);
    var_dump($arr);
});
```
