# Coroutine\Channel

> It is recommended to first check the [overview](/coroutine) to understand some basic concepts of coroutines before reading this section.

チャネルは、コルーチン間の通信に使用され、複数の生産者コルーチンおよび複数の消費者コルーチンをサポートします。 チャネルは、コルーチンの切り替えとスケジューリングを自動的に実行します。
## 実装原理

  * チャネルは`PHP`の`Array`に似ており、メモリだけを使用し、その他の余分なリソースを要求しません。すべての操作はメモリ操作であり、`IO`コストはありません。
  * チャネルは`PHP`の参照カウントを利用して実装されており、メモリコピーは行われません。したがって、巨大な文字列や配列を渡しても追加のパフォーマンスコストは発生しません。
  * `channel`は参照カウントに基づいており、ゼロコピーです。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

run(function(){
    $channel = new Channel(1);
    Coroutine::create(function () use ($channel) {
        for($i = 0; $i < 10; $i++) {
            Coroutine::sleep(1.0);
            $channel->push(['rand' => rand(1000, 9999), 'index' => $i]);
            echo "{$i}\n";
        }
    });
    Coroutine::create(function () use ($channel) {
        while(1) {
            $data = $channel->pop(2.0);
            if ($data) {
                var_dump($data);
            } else {
                assert($channel->errCode === SWOOLE_CHANNEL_TIMEOUT);
                break;
            }
        }
    });
});
```
このセクションでは、特定の方法や手順について説明します。
### __construct()

チャネルのコンストラクター。

```php
Swoole\Coroutine\Channel::__construct(int $capacity = 1)
```

  * **引数** 

    * **`int $capacity`**
      * **機能**：容量を設定します 【1以上の整数である必要があります】
      * **デフォルト値**：`1`
      * **その他の許容値**：なし

!> 変数を保存するためにPHPの参照カウントが使用され、バッファ領域は ` $capacity * sizeof(zval) ` バイトのメモリを必要とします。`PHP7`の場合、`zval` は `16` バイトです。つまり、`$capacity = 1024`の場合、`Channel` は最大で `16K` バイトのメモリを使用します。

!> `Server`で使用する場合、必ず[onWorkerStart](/server/events?id=onworkerstart)の後に作成してください。
### push()

データをチャネルに書き込みます。

```php
Swoole\Coroutine\Channel->push(mixed $data, float $timeout = -1): bool
```

  * **パラメータ** 

    * **`mixed $data`**
      * **機能**：データをプッシュします 【無名関数やリソースを含む任意の種類のPHP変数が可能です】
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> 混乱を避けるため、`0`、`false`、`空の文字列`、`null`などの空のデータをチャネルに書き込まないでください。

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点数をサポートします。例：`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：`-1`
      * **その他の値**：なし
      * **影響バージョン**：Swoole v4.2.12以上

      !> チャネルがいっぱいの場合、`push`は現在のコルーチンを一時中断し、指定された時間内に消費者がデータを消費しない場合はタイムアウトが発生し、バックエンドが現在のコルーチンを復元し、`push`呼び出しはすぐに`false`を返し、書き込みに失敗します。

  * **返り値**

    * 成功すると`true`が返されます
    * チャネルが閉じられている場合、失敗すると`false`が返され、`$channel->errCode`を使用してエラーコードを取得できます

  * **拡張**

    * **チャネルがいっぱいの場合**

      * 現在のコルーチンを自動的に`yield`し、他の消費者コルーチンがデータを`pop`して空きが出ると、現在のコルーチンが再開されます。
      * 複数の生産者コルーチンが同時に`push`する場合、バックエンドは自動的にキューイングを行い、順番にこれらの生産者コルーチンを1つずつ再開します。

    * **チャネルが空の場合**

      * 消費者コルーチンの1つを自動的に起動します。
      * 複数の消費者コルーチンが同時に`pop`する場合、バックエンドは自動的にキューイングを行い、順番にこれらの消費者コルーチンを1つずつ再開します。

!> `Coroutine\Channel`はローカルメモリを使用します。異なるプロセス間のメモリは分離されます。`push`および`pop`操作は同じプロセス内の異なるコルーチンでのみ行うことができます。
### pop()

データをチャネルから読み取ります。

```php
Swoole\Coroutine\Channel->pop(float $timeout = -1): mixed
```

  * **パラメーター**

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **値の単位**：秒【浮動小数点数もサポートされており、例えば`1.5`は`1秒`と`500ミリ秒`を表します】
      * **デフォルト値**：`-1`【タイムアウトしないことを表します】
      * **他の値**：なし
      * **影響するバージョン**：Swooleバージョン >= v4.0.3

  * **戻り値**

    * 戻り値は、PHP変数の任意のタイプであり、クロージャやリソースも含まれます
    * チャネルが閉じられた場合、失敗時に`false`が返されます

  * **拡張**

    * **チャネルが満杯の場合**

      * `pop`でデータを消費すると、自動的に1つの生産者コルーチンが起こされ、新しいデータを書き込むように促します
      * 複数の生産者コルーチンが同時に`push`する場合、内部でキューイングされ、順番にこれらの生産者コルーチンすべてが`resume`されます

    * **チャネルが空の場合**

      * 現在のコルーチンが自動的に`yield`され、他の生産者コルーチンがデータを`push`すると、チャネルが読み込み可能になり、現在のコルーチンが再度`resume`されます
      * 複数の消費者コルーチンが同時に`pop`する場合、内部でキューイングされ、順番にこれらの消費者コルーチンすべてが`resume`されます
### stats()

通道の状態を取得します。

```php
Swoole\Coroutine\Channel->stats(): array
```

 - **戻り値**

  配 す。バッファ付きチャネルの場合は、`4` 項目の情報が含まれますが、バッファなしチャネルの場合は `2` 項目の情報が含まれます。

  - `consumer_num` 消費者の数を表し、現在のチャネルが空であることを示します。`N` 個のコルーチンが他のコルーチンに`push` メソッドを呼び出してデータを生成するのを待っています。
  - `producer_num` 生産者の数を表し、現在のチャネルがいっぱいであることを示します。`N` 個のコルーチンが他のコルーチンに`pop` メソッドを呼び出してデータを消費するのを待っています。
  - `queue_num` チャネル内の要素数

```php
array(
  "consumer_num" => 0,
  "producer_num" => 1,
  "queue_num" => 10
);
```
### close()

チャネルを閉じます。待機中のすべての読み書きコルーチンを起こします。

```php
Swoole\Coroutine\Channel->close(): bool
```

!> すべての生産者コルーチンを起こし、`push`メソッドは`false`を返します。すべての消費者コルーチンを起こし、`pop`メソッドは`false`を返します。
### length()

通道内の要素数を取得します。

```php
Swoole\Coroutine\Channel->length(): int
```
### isEmpty()

現在のチャネルが空かどうかを判断します。

```php
Swoole\Coroutine\Channel->isEmpty(): bool
```
### isFull()

判断現在のチャネルがいっぱいになっているかどうか。

```php
Swoole\Coroutine\Channel->isFull(): bool
```
このセクションでは、オブジェクトの属性に関する情報を提供します。
### capacity

The capacity of the channel buffer.

The capacity set in the [constructor](/coroutine/channel?id=__construct) will be stored here, but if the set capacity is less than 1, this variable will be equal to 1.

```php
Swoole\Coroutine\Channel->capacity: int
```
### errCode

エラーコードを取得します。

```php
Swoole\Coroutine\Channel->errCode: int
```

  * **Return Value**

値 | Constant | Description
---|---|---
0 | SWOOLE_CHANNEL_OK | デフォルト成功
-1 | SWOOLE_CHANNEL_TIMEOUT | タイムアウトして pop に失敗した場合(タイムアウト)
-2 | SWOOLE_CHANNEL_CLOSED | チャネルが閉じられており、引き続きチャネルを操作しようとしています
