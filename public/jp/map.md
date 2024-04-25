# 並行マップ

並行性のある `Map` 構造を作成し、スレッドに渡すことができるようにします。他のスレッドで読み書きするときに可視性があります。

## 特徴
- `Map`、`ArrayList`、`Queue` は自動的にメモリを割り当て、`Table` のように固定する必要はありません
- ロックを自動的に適用する底層のため、スレッドセーフです
- `null/bool/int/float/string` タイプのみをサポートし、他のタイプは書き込み時に自動的にシリアライズされ、読み込み時に逆シリアライズされます
- イテレータはサポートされず、イテレータ内で要素を削除するとメモリエラーが発生します
- スレッドを作成する前に `Map`、`ArrayList`、`Queue` オブジェクトをスレッドパラメータとして子スレッドに渡す必要があります

## 使用方法
`Thread\Map` は `ArrayAccess` インターフェースを実装しており、配列操作として直接使用できます。

## 例

```php
use Swoole\Thread;
use Swoole\Thread\Map;

$args = Thread::getArguments();
if (empty($args)) {
    $map = new Map;
    $thread = Thread::exec(__FILE__, $i, $map);
    sleep(1);
    $map['test'] = unique();
    $thread->join();
} else {
    $map = $args[1];
    sleep(2);
    var_dump($map['test']);
}
```

## メソッド

### Map::count() 
要素数を取得します

### Map::keys()
すべての `key` を返します

### Map::clean()
すべての要素をクリアします
