# Swoole\Server\StatusInfo

こちらは`Swoole\Server\StatusInfo`の詳細です。

## プロパティ

### $worker_id
現在の`worker`プロセスIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->worker_id
```

### $worker_pid
現在の`worker`プロセスの親プロセスIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->worker_pid
```

### $status
プロセスの状態`status`を返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->status
```

### $exit_code
プロセスの終了ステータスコード`exit_code`を返します。このプロパティは`int`型の整数で、範囲は`0-255`です。

```php
Swoole\Server\StatusInfo->exit_code
```

### $signal
プロセスの終了シグナル`signal`を返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->signal
```
