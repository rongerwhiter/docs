# Swoole\Server\TaskResult

`Swoole\Server\TaskResult`の詳細について説明しています。

## プロパティ

### $task_id
`Reactor`スレッドIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\TaskResult->task_id
```

### $task_worker_id
実行結果がどの`task`プロセスから来たかを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\TaskResult->task_worker_id
```

### $dispatch_time
接続が持つ`data`を返します。このプロパティは`?string`型の文字列です。

```php
Swoole\Server\TaskResult->dispatch_time
```

### $data
接続が持つ`data`を返します。このプロパティは`string`型の文字列です。

```php
Swoole\Server\TaskResult->data
```
