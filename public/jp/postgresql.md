# Coroutine\PostgreSQL

`PostgreSQL`のクライアントライブラリです。

Swoole 5.0では完全に新しい形で再構築されており、古いバージョンとは完全に異なる使い方をしています。古いバージョンをご利用中の方は、[古いバージョンのドキュメント](/coroutine_client/postgresql-old.md)を参照してください。
## コンパイルとインストール

- システムに`libpq`ライブラリがインストールされていることを確認してください
- `mac`では、`postgresql`のインストール後に`libpq`ライブラリが含まれていますが、環境によって異なるため、`ubuntu`では`apt-get install libpq-dev`が必要になるかもしれません。`centos`では`yum install postgresql10-devel`を実行するかもしれません
- Swooleをコンパイルする際には、コンパイルオプションとして次を追加してください：`./configure --enable-swoole-pgsql`
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
### 事务处理

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
Sure! Feel free to provide me with the text you would like translated into Japanese.
### エラー

エラーメッセージを取得します。
```python
def greet():
    print("Hello, World!")
```

### connect()

`postgresql`の非同期接続を確立します。

```php
Swoole\Coroutine\PostgreSQL->connect(string $conninfo, float $timeout = 2): bool
```

!> `$conninfo` は接続情報であり、接続に成功すると`true`が返り、失敗すると`false`が返ります。エラー情報は[error](/coroutine_client/postgresql?id=error)属性を使用して取得できます。
  * **例**

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

SQLステートメントを実行します。非同期非ブロッキングコルーチンコマンドを送信します。

```php
Swoole\Coroutine\PostgreSQL->query(string $sql): \Swoole\Coroutine\PostgreSQLStatement|false;
```

  * **パラメータ** 

    * **`string $sql`**
      * **機能**：SQLステートメント
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **例**

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

    * **insert idを返す**

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

    * **トランザクション**

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

表のメタデータを表示します。非同期でブロッキングしないコルーチン版。

```php
Swoole\Coroutine\PostgreSQL->metaData(string $tableName): array
```
    
  * **使用例**

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

前処理。

```php
$stmt = Swoole\Coroutine\PostgreSQL->prepare(string $sql);
$stmt->execute(array $params);
```

  - **使用例**

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

クラス名：`Swoole\Coroutine\PostgreSQLStatement`

すべてのクエリは `PostgreSQLStatement` オブジェクトを返します。
### fetchAll()

```php
Swoole\Coroutine\PostgreSQLStatement->fetchAll(int $result_type = SW_PGSQL_ASSOC): false|array;
```

  * **参数**
    * **`$result_type`**
      * **機能**：定数。オプションのパラメータで、戻り値の初期化方法を制御します。
      * 形式**：`SW_PGSQL_ASSOC`
      * 他の値**：なし

      値 | 戻り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキー値とする連想配列を返します
      SW_PGSQL_NUM | フィールドをキー値とする配列を返します
      SW_PGSQL_BOTH | 両方をキー値として返します

  * **戻り値**

    * 瑕結果をすべて行を配列として返します。性
### affectedRows()

返されたレコードの数を示します。

```php
Swoole\Coroutine\PostgreSQLStatement->affectedRows(): int
```  
### numRows()

戻り値は行数です。

```php
Swoole\Coroutine\PostgreSQLStatement->numRows(): int
```
### fetchObject()

1行をオブジェクトとして取得します。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchObject(int $row, ?string $class_name = null, array $ctor_params = []): object;
```

  * **Example**

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
### fetchAssoc()

一行を関連配列として取得します。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchAssoc(int $row, int $result_type = SW_PGSQL_ASSOC): array
```
```php
Swoole\Coroutine\PostgreSQLStatement->fetchArray(int $row, int $result_type = SW_PGSQL_BOTH): array|false
```

  * **パラメータ**
    * **`int $row`**
      * **機能**: `row` は取得したい行（レコード）の番号です。最初の行は `0` です。
      * **デフォルト値**: なし
      * **他の値**: なし
    * **`$result_type`**
      * **機能**: 定数。オプションのパラメータで、返り値の初期化方法を制御します。
      * **デフォルト値**: `SW_PGSQL_BOTH`
      * **他の値**: なし

      値 | 返り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとする関連配列を返す
      SW_PGSQL_NUM | フィールド番号をキーとする配列を返す
      SW_PGSQL_BOTH | 両方をキーとする配列を返す

  * **返り値**

    * 取得した行（タプル/レコード）に対応する配列を返します。取得できる行がもうない場合は `false` を返します。

  * **使用例**

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

指定された`result`リソースから1行のデータ（レコード）を配列として取得して返します。取得した各列は、`0`オフセットから配列に格納されます。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchRow(int $row, int $result_type = SW_PGSQL_NUM): array|false
```

  * **Parameters**
    * **`int $row`**
      * **Purpose**：取得したい行（レコード）の番号です。最初の行は`0`です。
      * **Default**：None
      * **Other values**：None
    * **`$result_type`**
      * **Purpose**：定数です。オプションのパラメータで、返り値の初期化方法を制御します。
      * **Default**：`SW_PGSQL_NUM`
      * **Other values**：None

      Value | Return Value
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとする連想配列を返す
      SW_PGSQL_NUM | フィールド番号をキーとする配列を返す
      SW_PGSQL_BOTH | 両方をキーとして返す

  * **Return Value**

    * 返される配列は取得した行と一致します。取得可能な行（`row`）がない場合は`false`を返します。

  * **Example**

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
