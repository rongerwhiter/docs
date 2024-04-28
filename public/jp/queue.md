# 並行キュー

並行的な `Queue` 構造を作成し、スレッドに渡すことができます。他のスレッドから読み取る際に見えるようになります。
詳細は [並行Map](thread/map.md) を参照してください。


## 使用方法
`Thread\Queue` は先入れ先出のデータ構造であり、次の2つの主要な操作があります。
- `Queue::push($value)`：データをキューに書き込みます
- `Queue::pop()`：キューの先頭からデータを取り出します

## 注意事項
- `ArrayList` には要素を追加するだけで、ランダムに削除したり代入したりすることはできません

## サンプルコード

```php
use Swoole\Thread;
use Swoole\Thread\Queue;

$args = Thread::getArguments();
$c = 4;
$n = 128;

if (empty($args)) {
    $threads = [];
    $queue = new Queue;
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i, $queue);
    }
    while ($n--) {
        $queue->push(base64_encode(random_bytes(16)), Queue::NOTIFY_ONE);
        usleep(random_int(10000, 100000));
    }
    $n = 4;
    while ($n--) {
        $queue->push('', Queue::NOTIFY_ONE);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
    var_dump($queue->count());
} else {
    $queue = $args[1];
    while (1) {
        $job = $queue->pop(-1);
        if (!$job) {
            break;
        }
        var_dump($job);
    }
}
```

## メソッドリスト

### Queue::push()

データをキューに書き込みます

```php
function Queue::push(mixed $value, int $notify = 0);
```

- `$value`：書き込むデータの内容
- `$notify`：データを読み取る待機中のスレッドに通知するかどうか、`Queue::NOTIFY_ONE` は1つのスレッドを起こし、`Queue::NOTIFY_ALL` はすべてのスレッドを起こします


### Queue::pop()

キューの先頭からデータを取得します

```php
function Queue::pop(double $timeout = 0);
```

- `$timeout=0`：デフォルト値で、待たずに、キューが空の場合はすぐに `NULL` を返します
- `$timeout!=0`：キューが空の場合は生産者の `push()` データを待ちます。`$timeout` が負の場合はタイムアウトしないことを示します

### Queue::count()
キューの要素数を取得します

### Queue::clean()
すべての要素を削除します
