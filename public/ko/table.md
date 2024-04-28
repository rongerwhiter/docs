# 고성능 공유 메모리 테이블

`PHP` 언어가 다중 스레드를 지원하지 않기 때문에 `Swoole`은 다중 프로세스 모드를 사용하며, 다중 프로세스 모드에서는 프로세스 간 메모리 격리가 존재하여 작업 프로세스 내에서 `global` 전역 변수 및 슈퍼 글로벌 변수를 수정해도 다른 프로세스에는 적용되지 않습니다.

> `worker_num=1`로 설정하면 프로세스 격리가 없으며 전역 변수를 사용하여 데이터를 저장할 수 있습니다.

```php
$fds = array();
$server->on('connect', function ($server, $fd){
    echo "connection open: {$fd}\n";
    global $fds;
    $fds[] = $fd;
    var_dump($fds);
});
```

`$fds`는 전역 변수이지만 현재 프로세스 내에서만 유효합니다. `Swoole` 서버의 하위에는 여러 `Worker` 프로세스가 생성되며 `var_dump($fds)`로 출력되는 값은 일부 연결의 `fd`만 포함합니다.

해당 문제에 대한 해결 방법은 외부 저장 서비스를 사용하는 것입니다:

* 데이터베이스, 예: `MySQL`, `MongoDB`
* 캐시 서버, 예: `Redis`, `Memcache`
* 디스크 파일, 다중 프로세스 동시 읽기/쓰기 시 잠금이 필요합니다.

일반적인 데이터베이스 및 디스크 파일 작업은 많은 `IO` 대기 시간이 발생합니다. 따라서 다음을 권장합니다:

* `Redis` 메모리 데이터베이스, 읽기 및 쓰기 속도가 매우 빠르지만 TCP 연결 등의 문제가 있고 성능도 가장 높지 않습니다.
* `/dev/shm` 메모리 파일 시스템, 모든 읽기/쓰기 동작이 메모리에만 완료되어 `IO` 소모가 없으며 성능이 매우 높지만 데이터가 형식화되지 않고 데이터 동기화 문제가 있습니다.

?> 위에서 언급한 저장 방법 이외에도 데이터를 저장하는 데 공유 메모리를 사용하는 것을 권장하며, `Swoole\Table`은 공유 메모리와 락을 사용하여 구현된 초고성능 동시 데이터 구조입니다. 다중 프로세스/다중 스레드 데이터 공유 및 동기화 락 문제를 해결하는 데 사용됩니다. `Table`의 메모리 용량은 `PHP`의 `memory_limit` 제어를 받지 않습니다.

!> 배열 방식으로 `Table`을 읽거나 쓰지 마십시오. 꼭 문서에서 제공하는 API를 사용하여 작업해야 합니다.   
배열 방식으로 가져온 `Table\Row` 객체는 일회성 객체이므로 많은 작업에 의존하지 마십시오.
`v4.7.0` 버전부터 배열 방식으로 `Table`을 읽거나 쓰는 것은 더 이상 지원되지 않으며 `Table\Row` 객체가 제거되었습니다.

* **장점**

  * 강력한 성능, 단일 스레드당 초당 `200`만 번의 읽기/쓰기 가능;
  * 응용 프로그램 코드에서 잠금을 걸 필요가 없으며, `Table`은 행 락 동기화 락을 내장하고 있으므로 모든 작업은 멀티스레드/다중프로세스 안전합니다. 사용자층에서 데이터 동기화 문제를 고려할 필요가 없습니다;
  * 다중 프로세스 지원, `Table`은 다중 프로세스 간 데이터 공유에 사용할 수 있습니다;
  * 행 락 사용, 전역 락 사용이 아닌 경우, 2개 프로세스가 동일 `CPU` 시간에 동시에 같은 데이터를 읽을 때에만 경쟁적 락이 발생합니다.

* **반복**

!> 반복 중에 삭제 작업을 수행하지 마십시오(모든 `key`를 가져온 후 삭제).

`Table` 클래스는 반복자 및 `Countable` 인터페이스를 구현했으며 `foreach`를 사용하여 반복하거나 `count`를 사용하여 현재 행 수를 계산할 수 있습니다.

```php
foreach($table as $row)
{
  var_dump($row);
}
echo count($table);
```
## 속성
### size

표의 최대 행 수를 가져옵니다.

```php
Swoole\Table->size;
```
### memorySize

`memorySize` 속성은 메모리 소비 크기를 바이트 단위로 가져옵니다.

```php
Swoole\Table->memorySize;
```
## 방법
### __construct()

메모리 테이블을 생성합니다.

```php
Swoole\Table::__construct(int $size, float $conflict_proportion = 0.2);
```

  * **매개변수** 

    * **`int $size`**
      * **기능**：테이블의 최대 행 수를 지정합니다.
      * **기본값**：없음
      * **다른 값**：없음

      !> `Table`은 공유 메모리 위에 구축되므로 동적으로 확장할 수 없습니다. 따라서 `$size`는 직접 계산하여 설정해야하며, `Table`이 저장할 수 있는 최대 행 수는 `$size`와 관련이 있지만 완전히 일치하지는 않습니다. 예를 들어, `$size`가 `1024`인 경우 저장 가능한 행 수는 **1024보다 적습니다**. 만약 `$size`가 너무 크면 기기 메모리가 부족하여 `Table` 생성에 실패할 수 있습니다.

    * **`float $conflict_proportion`**
      * **기능**：해시 충돌의 최대 비율
      * **기본값**：`0.2` (즉, `20%`)
      * **다른 값**：최소 `0.2`, 최대 `1`

  * **용량 계산**

      * 만약 `$size`가 `2`의 `N`승이 아닌 경우, `1024`, `8192`, `65536`와 같이 자동으로 근접한 숫자로 조정됩니다. 만약 `1024`보다 작으면 기본값으로 `1024`로 설정되며, 최소 값으로 `1024`가 됩니다. `v4.4.6` 버전부터 최소 값은 `64`입니다.
      * `Table`이 차지하는 총 메모리 크기는 (`HashTable 구조체 길이` + `키 길이 64바이트` + `$size 값`) * (`1 + $conflict_proportion 값으로 해시 충돌`) * (`열 크기`)입니다.
      * 데이터 `Key` 및 해시 충돌율이 `20%`를 초과하면 충돌 메모리 블록 용량이 부족하여 새로운 데이터를 `set`하면 `Unable to allocate memory` 오류가 발생하고 `false`를 반환하여 저장에 실패하며, 이 경우 `$size` 값을 늘리고 서비스를 다시 시작해야 합니다.
      * 충분한 메모리가 있는 경우에는 이 값을 크게 설정하십시오.
### column()

메모리 테이블에 열을 추가합니다.

```php
Swoole\Table->column(string \$name, int \$type, int \$size = 0);
```

  * **매개변수**

    * **`string \$name`**
      * **기능**: 필드 이름을 지정합니다.
      * **기본 값**: 없음
      * **기타 값**: 없음

    * **`int \$type`**
      * **기능**: 필드 유형을 지정합니다.
      * **기본 값**: 없음
      * **기타 값**: `Table::TYPE_INT`, `Table::TYPE_FLOAT`, `Table::TYPE_STRING`

    * **`int \$size`**
      * **기능**: 문자열 필드의 최대 길이를 지정합니다.【문자열 유형의 필드는 반드시`$size`를 지정해야 합니다.】
      * **값의 단위**: 바이트
      * **기본 값**: 없음
      * **기타 값**: 없음

  * **`$type` 유형 설명**
  
유형 | 설명
---|---
Table::TYPE_INT | 기본값은 8바이트입니다.
Table::TYPE_STRING | 설정 후, 설정된 문자열은 `$size`로 지정된 최대 길이를 초과할 수 없습니다.
Table::TYPE_FLOAT | 8바이트의 메모리를 차지합니다.
### create()

메모리 테이블을 생성합니다. 테이블 구조를 정의한 후에 `create`를 실행하여 운영 체제에 메모리를 요청하고 테이블을 생성합니다.

```php
Swoole\Table->create(): bool
```

`create` 메소드를 사용하여 테이블을 생성한 후에는 [memorySize](/memory/table?id=memorysize) 속성을 사용하여 실제 메모리 사용량을 얻을 수 있습니다.

  * **팁** 

    * `create`를 호출하기 전에 `set` 또는 `get`과 같은 데이터 읽기/쓰기 작업 메소드를 사용할 수 없습니다.
    * `create`를 호출한 후에는 `column` 메소드를 사용하여 새 필드를 추가할 수 없습니다.
    * 시스템 메모리가 부족하면 요청 실패하여 `create`가 `false`를 반환합니다.
    * 메모리 요청에 성공하면 `create`가 `true`를 반환합니다.

    !> `Table`은 데이터를 보관하기 위해 공유 메모리를 사용하며, 자식 프로세스를 생성하기 전에 `Table->create()`를 실행해야 합니다.  
    `Server`에서 `Table`을 사용할 경우, `Table->create()`는 `Server->start()` 이전에 실행되어야 합니다.

  * **사용 예시**

```php
$table = new Swoole\Table(1024);
$table->column('id', Swoole\Table::TYPE_INT);
$table->column('name', Swoole\Table::TYPE_STRING, 64);
$table->column('num', Swoole\Table::TYPE_FLOAT);
$table->create();

$worker = new Swoole\Process(function () {}, false, false);
$worker->start();

//$serv = new Swoole\Server('127.0.0.1', 9501);
//$serv->start();
``` 
### set()

데이터 행을 설정합니다. `Table`은 데이터에 접근하기 위해 `key-value` 방식을 사용합니다.

```php
Swoole\Table->set(string $key, array $value): bool
```

  * **매개변수** 

    * **`string $key`**
      * **기능**：데이터의 `key`
      * **기본값**：없음
      * **기타**：없음

      !> 동일한 `$key`는 동일한 행 데이터에 해당합니다. 만약 동일한 `key`를 `set`한다면, 이전 데이터가 덮어씌워집니다. `key`의 최대 길이는 63바이트여야 합니다.

    * **`array $value`**
      * **기능**：데이터의 `value`
      * **기본값**：없음
      * **기타**：없음

      !> 배열이어야 하며, 필드 정의한 `$name`과 완전히 일치해야 합니다.

  * **반환 값**

    * 설정 성공 시 `true` 반환
    * 실패 시 `false` 반환, 이는 해시 충돌이 너무 많아 동적 공간에 메모리를 할당할 수 없는 경우일 수 있으며, 생성자의 두 번째 매개변수를 더 크게 설정할 수 있습니다.

!> -`Table->set()`은 모든 필드의 값을 설정할 수도 있고 일부 필드만 수정할 수도 있습니다.  
   -`Table->set()`을 설정하지 않으면, 해당 행 데이터의 모든 필드가 비어 있습니다.  
   -`set`/`get`/`del`은 행 락을 자동으로 포함하므로 `lock`을 잠글 필요가 없습니다.  
   -**키는 이진 보안이 아니며, 문자열 유형이어야 하며 이진 데이터를 전달해서는 안 됩니다.**

  * **사용 예시**

```php
$table->set('1', ['id' => 1, 'name' => 'test1', 'age' => 20]);
$table->set('2', ['id' => 2, 'name' => 'test2', 'age' => 21]);
$table->set('3', ['id' => 3, 'name' => 'test3', 'age' => 19]);
```

  * **최대 길이를 초과하는 문자열 설정**
    
    문자열 길이가 열 정의 시 설정한 최대 크기를 초과하는 경우, 하위 수준에서 자동으로 잘립니다.
    
    ```php
    $table->column('str_value', Swoole\Table::TYPE_STRING, 5);
    $table->set('hello', array('str_value' => 'world 123456789'));
    var_dump($table->get('hello'));
    ```

    * `str_value`열의 최대 크기는 5바이트이지만, `5`바이트보다 큰 문자열을 `set`하였습니다.
    * 하위 수준에서 5바이트 데이터를 자동으로 잘라내어, 최종 `str_value` 값은 `world`가 됩니다.

!> `v4.3` 버전부터, 하위 수준에서는 메모리 길이를 정렬 처리합니다. 문자열 길이는 8의 배수여야 하며, 길이가 5인 경우 자동으로 8바이트로 정렬되므로 `str_value` 값은 `world 12`입니다.
### incr()

원자 증가 작업.

```php
Swoole\Table->incr(string $key, string $column, mixed $incrby = 1): int
```

  * **매개변수**

    * **`string $key`**
      * **기능**: 데이터의 `key`【`$key`에 해당하는 행이 없으면 기본 열 값은 `0`입니다.】
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`string $column`**
      * **기능**: 열 이름 지정【실수 및 정수 필드만 지원됩니다.】
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`string $incrby`**
      * **기능**: 증가분 【열이 `int`인 경우, `$incrby`는 `int` 형이어야 하며, 열이 `float` 형인 경우, `$incrby`는 `float` 유형이어야 합니다.】
      * **기본값**: `1`
      * **기타 값**: 없음

  * **반환값**

    결과 숫자를 반환합니다.
### decr()

원자 감소 작업.

```php
Swoole\Table->decr(string $key, string $column, mixed $decrby = 1): int
```

  * **매개변수**

    * **`string $key`**
      * **기능**: 데이터의`key`【`$key`에 대한 행이 없는 경우, 기본 열 값은`0`입니다】
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`string $column`**
      * **기능**: 지정된 열 이름【실수형과 정수형 필드만 지원됨】
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`string $decrby`**
      * **기능**: 감소분【열이`int`인 경우`$decrby`는`int`형이어야 하며, 열이`float`형인 경우,`$decrby`는`float`형이어야 합니다】
      * **기본값**: `1`
      * **다른 값**: 없음

  * **반환값**

    최종 결과 값 반환

    !> 결과 값이`0`일 때 감소하면 음수가 됩니다.
### get()

한 줄의 데이터를 가져옵니다.

```php
Swoole\Table->get(string $key, string $field = null): array|false
```

  * **매개변수** 

    * **`string $key`**
      * **기능**：데이터의 `key`【문자열 형식이어야 함】
      * **기본값**：없음
      * **기타 값**：없음

    * **`string $field`**
      * **기능**：`$field`을 지정하면 해당 필드의 값만 반환하고 전체 레코드는 반환하지 않음
      * **기본값**：없음
      * **기타 값**：없음
      
  * **반환 값**

    * `$key`가 존재하지 않으면 `false`를 반환
    * 성공 시 결과 배열 반환
    * `field`가 지정된 경우 해당 필드의 값만 반환, 전체 레코드는 반환하지 않음
### exist()

특정 키가 테이블에 존재하는지 확인합니다.

```php
Swoole\Table->exist(string $key): bool
```

  * **파라미터** 

    * **`string $key`**
      * **기능**：데이터의 `key`【반드시 문자열 유형이어야 함】
      * **기본값**：없음
      * **기타 값**：없음
```php
Swoole\Table->count(): int
```
### del()

데이터를 삭제합니다.

!> `Key`는 이진 보안이 아니며, 문자열 유형이어야하며 바이너리 데이터를 전달해서는 안됩니다; **반복문 중에 삭제하지 마십시오**.

```php
Swoole\Table->del(string $key): bool
```

  * **Return**

    * 해당 `$key`의 데이터가 존재하지 않으면 `false`가 반환됩니다.
    * 성공적으로 삭제되면 `true`가 반환됩니다.
### stats()

`Swoole\Table`의 상태를 가져옵니다.

```php
Swoole\Table->stats(): array
```

!> Swoole 버전 >= `v4.8.0`에서 사용 가능
## 보조 함수 :id=swoole_table

사용자가 `Swoole\Table`을 빠르게 생성할 수 있도록 도와주는 함수입니다.

```php
function swoole_table(int $size, string $fields): Swoole\Table
```

!> Swoole 버전 >= `v4.6.0`에서 사용 가능합니다. `$fields`의 형식은 `foo:i/foo:s:num/foo:f`입니다.

| 축약 | 전체 이름 | 타입                  |
| ---- | ------ | ------------------ |
| i    | int    | Table::TYPE_INT    |
| s    | string | Table::TYPE_STRING |
| f    | float  | Table::TYPE_FLOAT  |

예시:

```php
$table = swoole_table(1024, 'fd:int, reactor_id:i, data:s:64');
var_dump($table);

$table = new Swoole\Table(1024, 0.25);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();
var_dump($table);
```
```php
<?php
$table = new Swoole\Table(1024);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();

$serv = new Swoole\Server('127.0.0.1', 9501);
$serv->set(['dispatch_mode' => 1]);
$serv->table = $table;

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {

	$cmd = explode(" ", trim($data));

	//get
	if ($cmd[0] == 'get')
	{
		//get self
		if (count($cmd) < 2)
		{
			$cmd[1] = $fd;
		}
		$get_fd = intval($cmd[1]);
		$info = $serv->table->get($get_fd);
		$serv->send($fd, var_export($info, true)."\n");
	}
	//set
	elseif ($cmd[0] == 'set')
	{
		$ret = $serv->table->set($fd, array('reactor_id' => $data, 'fd' => $fd, 'data' => $cmd[1]));
		if ($ret === false)
		{
			$serv->send($fd, "ERROR\n");
		}
		else
		{
			$serv->send($fd, "OK\n");
		}
	}
	else
	{
		$serv->send($fd, "command error.\n");
	}
});

$serv->start();
```  
