# Downward Incompatible Changes
## v5.0.0
* Change the default running mode of `Server` to `SWOOLE_BASE`
* Increase the minimum `PHP` version requirement to `8.0`
* All class methods and functions have added type constraints, changing to strict typing mode
* Removed the underscore `PSR-0` class alias and only kept namespace-style class names, such as changing `swoole_server` to `Swoole\Server`
* `Swoole\Coroutine\Redis` and `Swoole\Coroutine\MySQL` are marked as deprecated, please use `Runtime Hook` + native `Redis`/`MySQL` client
## v4.8.0

- In `BASE` mode, the `onStart` callback will always be invoked when the first worker process (with `workerId` as `0`) starts, before `onWorkerStart` is executed. Coroutines API can always be used in the `onStart` function. When `Worker-0` encounters a fatal error and restarts, `onStart` will be called again.
In previous versions, `onStart` would be called in `Worker-0` when there was only one worker process. When there were multiple worker processes, it would be executed in the `Manager` process.
## v4.7.0

- Removed `Table\Row`. `Table` no longer supports read and write operations in array format.
## v4.6.0

- `session id`の最大制限が削除され、もはや繰り返されません
- コルーチンを使用する場合、`pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`などのセキュリティリスクのある機能が無効になります
- デフォルトで coroutine フックが有効化されました
- PHP7.1のサポートは終了しました
- `Event::rshutdown()`を非推奨とし、代わりに Coroutine\run を使用してください
## v4.5.4

- `SWOOLE_HOOK_ALL` と `SWOOLE_HOOK_CURL` が含まれています
- `ssl_method` が削除され、`ssl_protocols` がサポートされます
## v4.4.12

- このバージョンではWebSocketフレームの圧縮がサポートされました。pushメソッドの3番目の引数がflagsに変更されました。strict_typesが設定されていない場合、コードの互換性は影響を受けません。しかし、strict_typesが設定されていると、boolをintに暗黙的に変換できない型のエラーが発生します。この問題はv4.4.13で修正されます。
## v4.4.1

- Registered signals are no longer considered as a condition to maintain the event loop. **If a program only registers signals without performing other tasks, it will be considered idle and will exit immediately** (in this case, you can prevent the process from exiting by registering a timer).
## v4.4.0

- 和`PHP`官方保持一致, 不再支持`PHP7.0` (@matyhtf)
- 移除`Serialize`模块, 在单独的 [ext-serialize](https://github.com/swoole/ext-serialize) 扩展中维护
- 移除`PostgreSQL`模块，在单独的 [ext-postgresql](https://github.com/swoole/ext-postgresql) 扩展中维护
- `Runtime::enableCoroutine`不再会自动兼容协程内外环境, 一旦开启, 则一切阻塞操作必须在协程内调用 (@matyhtf)
- 由于引入了全新的协程`MySQL`客户端驱动, 底层设计更加规范, 但有一些小的向下不兼容的变化 (详见 [4.4.0更新日志](https://wiki.swoole.com/wiki/page/p-4.4.0.html))
## v4.3.0

- 移除了所有异步模块, 详见 [独立异步扩展](https://wiki.swoole.com/wiki/page/p-async_ext.html) 或 [4.3.0更新日志](https://wiki.swoole.com/wiki/page/p-4.3.0.html)
## v4.2.13

> 由于历史API设计存在问题导致的不可避免的不兼容变更

* 协程Redis客户端订阅模式操作变更, 详见[订阅模式](https://wiki.swoole.com/#/coroutine_client/redis?id=%e8%ae%a2%e9%98%85%e6%a8%a1%e5%bc%8f)
## v4.2.12

> Experimental feature + Inevitable incompatible changes due to historical API design issues

- Removed the `task_async` configuration option, replaced with [task_enable_coroutine](https://wiki.swoole.com/#/server/setting?id=task_enable_coroutine)
## v4.2.5

- Removed support for UDP clients in `onReceive` and `Server::getClientInfo`
## v4.2.0

- Removed `swoole_http2_client` asynchronous client, please use coroutine HTTP2 client
## v4.0.4

From this version onwards, the asynchronous `Http2\Client` will trigger `E_DEPRECATED` notices and will be removed in the next version. Please use `Coroutine\Http2\Client` instead.

The `body` attribute of `Http2\Response` has been renamed to `data`. This modification is made to ensure the consistency between `request` and `response` and to align more closely with the frame type names of the HTTP2 protocol.

Starting from this version, `Coroutine\Http2\Client` has gained comprehensive support for the HTTP2 protocol, meeting the needs of enterprise-level production applications such as `grpc`, `etcd`, etc. Therefore, the series of changes regarding HTTP2 are essential.
## v4.0.3

`swoole_http2_response` と `swoole_http2_request` が一貫しているように `headers` と `cookies` のすべての属性名を複数形に変更しました。
## v4.0.2

> Due to the complexity of the underlying implementation, which is difficult to maintain, and the frequent misconceptions by users about its usage, the following API has been temporarily removed:

- `Coroutine\Channel::select`

However, a second parameter `timeout` has been added to the `Coroutine\Channel->pop` method to meet development needs.
## v4.0

> Due to the upgrade of coroutine kernel, coroutines can be invoked at any place in any function without special treatment, so the following APIs have been removed

- `Coroutine::call_user_func`
- `Coroutine::call_user_func_array`
