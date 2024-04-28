# プロセス間のロック

`PHP`コードでは、データ同期を実現するために簡単にロックを作成することができます。 `Lock`クラスは`5`種類のロックタイプをサポートしています。

| ロックの種類 | 説明 |
| --- | --- |
| SWOOLE_MUTEX | ミューテックスロック |
| SWOOLE_RWLOCK | リードライトロック |
| SWOOLE_SPINLOCK | スピンロック |
| SWOOLE_FILELOCK | ファイルロック (非推奨) |
| SWOOLE_SEM | セマフォ (非推奨) |

!> [onReceive](/server/events?id=onreceive)などのコールバック関数内でロックを作成しないでください。そうしないとメモリが継続的に増加し、メモリリークが発生します。

## 使用例

```php
$lock = new Swoole\Lock(SWOOLE_MUTEX);
echo "[Master]create lock\n";
$lock->lock();
if (pcntl_fork() > 0)
{
  sleep(1);
  $lock->unlock();
} 
else
{
  echo "[Child] Wait Lock\n";
  $lock->lock();
  echo "[Child] Get Lock\n";
  $lock->unlock();
  exit("[Child] exit\n");
}
echo "[Master]release lock\n";
unset($lock);
sleep(1);
echo "[Master]exit\n";
```

## 警告

!> コルーチン内ではロックを使用できません。慎重に使用し、`lock`および`unlock`操作の間にコルーチン切り替えを引き起こす可能性のある`API`を使用しないでください。

### 間違った例

!> このコードはコルーチンモードで`100%`デッドロックします。[この記事](https://course.swoole-cloud.com/article/2)を参照してください。

```php
$lock = new Swoole\Lock();
$c = 2;

while ($c--) {
  go(function () use ($lock) {
      $lock->lock();
      Co::sleep(1);
      $lock->unlock();
  });
}
```

## メソッド

### __construct()

コンストラクタ。

```php
Swoole\Lock::__construct(int $type = SWOOLE_MUTEX, string $lockfile = '');
```

!> ロックオブジェクトを繰り返し作成/破棄しないでください。そうしないとメモリリークが発生します。

  * **パラメータ** 

    * **`int $type`**
      * **機能**：ロックのタイプ
      * **デフォルト値**：`SWOOLE_MUTEX`【ミューテックスロック】
      * **その他の値**：なし

    * **`string $lockfile`**
      * **機能**：ファイルロックのパスを指定します【タイプが`SWOOLE_FILELOCK`の場合は必ず渡す必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> 各タイプのロックはそれぞれ異なるメソッドをサポートします。たとえば、リードライトロック、ファイルロックは`$lock->lock_read()`をサポートすることができます。また、ファイルロック以外のロックタイプは、親プロセス内で作成する必要があります。これにより、`fork`された子プロセス同士がロックを競合させることができます。

### lock()

ロックを取得します。他のプロセスがロックを保持している場合、ここではブロックされ、保持しているプロセスが`unlock()`でロックを解放するまで待機します。

```php
Swoole\Lock->lock(): bool
```

### trylock()

ロックを取得します。`lock`メソッドとは異なり、`trylock()`はブロックせず、即座にリターンします。

```php
Swoole\Lock->trylock(): bool
```

  * **戻り値**

    * ロックに成功すると`true`が返され、この時点で共有変数を変更できます
    * ロックに失敗すると`false`が返され、他のプロセスがロックを保持していることを示します

!> `SWOOlE_SEM`セマフォには`trylock`メソッドがありません

### unlock()

ロックを解放します。

```php
Swoole\Lock->unlock(): bool
```

### lock_read()

リードロックを取得します。

```php
Swoole\Lock->lock_read(): bool
```

* リードロックを保持している間、他のプロセスは引き続きリードロックを取得でき、リード操作を続行できます。
* ただし、`$lock->lock()`または`$lock->trylock()`はできません。これらのメソッドは排他ロックを取得し、排他ロックを取得している間は他のプロセスがどのようなロック操作も行えません。
* 排他ロック(つまり、`$lock->lock()`/`$lock->trylock()`を呼び出す)を保持している別のプロセスがある場合、`$lock->lock_read()`はブロックされ、排他ロックを保持しているプロセスがロックを解放するまで待機します。

!> `SWOOLE_RWLOCK`および`SWOOLE_FILELOCK`タイプのロックのみリードロックをサポートします

### trylock_read()

ロックします。このメソッドは`lock_read()`と同じですが、ブロックしません。

```php
Swoole\Lock->trylock_read(): bool
```

!> コールは即座に戻ります。ロックを取得できたかどうかを確認するために戻り値をチェックする必要があります。

### lockwait()

ロックを取得します。`lock()`メソッドと同じ効果がありますが、`lockwait()`ではタイムアウトを設定できます。

```php
Swoole\Lock->lockwait(float $timeout = 1.0): bool
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウトを指定します
      * **値の単位**：秒【浮動小数点数もサポートします。たとえば、`1.5`は`1s` + `500ms`を表します】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **戻り値**

    * 指定された時間内にロックを取得できない場合は`false`が返されます
    * ロックに成功すると`true`が返されます

!> `Mutex`タイプのロックのみ`lockwait`がサポートされます
