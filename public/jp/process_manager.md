# プロセス\マネージャ

プロセスマネージャは、[プロセス\プール](/process/process_pool)に基づいて実装されたものです。複数のプロセスを管理できます。`プロセス\プール`と比較して、異なるタスクを実行する複数のプロセスを非常に簡単に作成し、各プロセスがコルーチン環境にいるかどうかを制御できます。

## 対応バージョン

| バージョン | クラス名                        | 更新内容                                 |
| ------ | ----------------------------- | ---------------------------------------- |
| v4.5.3 | Swoole\Process\ProcessManager | -                                        |
| v4.5.5 | Swoole\Process\Manager        | 名前が変更され、ProcessManager は Manager の別名となりました |

!> `v4.5.3`以上のバージョンで利用可能です。

## 使用例

```php
use Swoole\Process\Manager;
use Swoole\Process\Pool;

$pm = new Manager();

for ($i = 0; $i < 2; $i++) {
    $pm->add(function (Pool $pool, int $workerId) {
    });
}

$pm->start();
```

## メソッド

### __construct()

コンストラクタ。

```php
Swoole\Process\Manager::__construct(int $ipcType = SWOOLE_IPC_NONE, int $msgQueueKey = 0);
```

* **パラメータ**

  * **`int $ipcType`**
    * **機能**：プロセス間通信のモード、`Process\Pool`の`$ipc_type`と同じ【デフォルトは`0`で、特定のプロセス間通信機能を使用しないことを示します】
    * **デフォルト値**：`0`
    * **その他の値**：無し

  * **`int $msgQueueKey`**
    * **機能**：メッセージキューの `key`、`Process\Pool`の`$msgqueue_key`と同じ
    * **デフォルト値**：無し
    * **その他の値**：無し

### setIPCType()

ワーカープロセス間の通信方法を設定します。

```php
Swoole\Process\Manager->setIPCType(int $ipcType): self;
```

* **パラメータ**

  * **`int $ipcType`**
    * **機能**：プロセス間通信のモード
    * **デフォルト値**：無し
    * **その他の値**：無し

### getIPCType()

ワーカープロセス間の通信方法を取得します。

```php
Swoole\Process\Manager->getIPCType(): int;
```

### setMsgQueueKey()

メッセージキューの `key` を設定します。

```php
Swoole\Process\Manager->setMsgQueueKey(int $msgQueueKey): self;
```

* **パラメータ**

  * **`int $msgQueueKey`**
    * **機能**：メッセージキューの `key`
    * **デフォルト値**：無し
    * **その他の値**：無し

### getMsgQueueKey()

メッセージキューの `key` を取得します。

```php
Swoole\Process\Manager->getMsgQueueKey(): int;
```

### add()

ワーカープロセスを追加します。

```php
Swoole\Process\Manager->add(callable $func, bool $enableCoroutine = false): self;
```

* **パラメータ**

  * **`callable $func`**
    * **機能**：現在のプロセスで実行されるコールバック関数
    * **デフォルト値**：無し
    * **その他の値**：無し

  * **`bool $enableCoroutine`**
    * **機能**：このプロセスでコールバック関数を実行するためにコルーチンを作成するかどうか
    * **デフォルト値**：false
    * **その他の値**：無し

### addBatch()

ワーカープロセスを一括追加します。

```php
Swoole\Process\Manager->addBatch(int $workerNum, callable $func, bool $enableCoroutine = false): self
```

* **パラメータ**

  * **`int $workerNum`**
    * **機能**：プロセスを一括で追加する数
    * **デフォルト値**：無し
    * **その他の値**：無し

  * **`callable $func`**
    * **機能**：これらのプロセスで実行されるコールバック関数
    * **デフォルト値**：無し
    * **その他の値**：無し

  * **`bool $enableCoroutine`**
    * **機能**：これらのプロセスでコールバック関数を実行するためにコルーチンを作成するかどうか
    * **デフォルト値**：無し
    * **その他の値**：無し

### start()

ワーカープロセスを起動します。

```php
Swoole\Process\Manager->start(): void
```
