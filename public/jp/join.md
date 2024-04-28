# スレッド管理

## Thread::join()

子スレッドの終了を待ちます。サブスレッドがまだ実行されている場合、`join()` はブロックされます。

```php
$thread = Thread::exec(__FILE__, $i);
$thread->join();
```

## Thread::joinable()

サブスレッドが既に終了しているかどうかを確認します。

### 戻り値
- `true`：サブスレッドが終了しており、`join()` を呼び出してもブロックされません
- `false`：まだ終了していない

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->joinable());
```

## Thread::detach()

サブスレッドを親スレッドの制御から切り離し、スレッドの終了を待つ`join()`などは不要です。

```php
$thread = Thread::exec(__FILE__, $i);
$thread->detach();
unset($thread);
```

## Thread::getId()

静的メソッド、現在のスレッドの`ID`を取得するためにサブスレッドで呼び出します。

```php
var_dump(Thread::getId());
```

## Thread::getArguments()

静的メソッド、現在のスレッドの引数を取得するためにサブスレッドで呼び出します。親スレッドで`Thread::exec()`するときに渡されます。

```php
var_dump(Thread::getArguments());
```

## Thread::$id

このオブジェクトのプロパティを使用して、サブスレッドの`ID`を取得できます。

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->id);
```
