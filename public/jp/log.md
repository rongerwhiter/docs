# バージョン更新履歴

`v1.5`から厳格なバージョン更新履歴が確立されました。現在の平均イテレーション時間は半年ごとの大規模なバージョンと、`2-4`週ごとの小規模なバージョンです。
## 推奨されるPHPバージョン

- 7.4
- 8.0
- 8.1
- 8.2
## 推奨されるSwooleバージョン
`Swoole5.x`および`Swoole4.8.x`

両者の違いは、`v5.x`がアクティブなイテレーションブランチであるのに対し、`v4.8.x`は**非**アクティブなイテレーションブランチであり、単に`BUG`を修正しています

!> バージョン`v4.x`以降では、[enable_coroutine](/server/setting?id=enable_coroutine)を設定して、非コルーチンバージョンに切り替えることができます
## バージョンタイプ

* `alpha` フィーチャーのプレビューバージョン。開発計画のタスクが完了し、プレビューが公開されていることを示します。多くの`BUG`が存在する可能性があります。
* `beta` テストバージョン。開発環境でのテストに使用できることを示します。`BUG`が存在する可能性があります。
* `rc[1-n]` リリース候補バージョン。リリースサイクルに入り、大規模なテストが行われていることを示します。この期間中に`BUG`が見つかる可能性があります。
* 接尾辞がない場合は、安定版を表します。このバージョンは開発が完了し、正式に使用できることを示します。
## 現在のバージョン情報を確認

```shell
php --ri swoole
```
## v6.0.0
- 增加了多线程模式的支持
- 移除了 `Coroutine\Redis`、`Coroutine\MySQL`、`Coroutine\PostgreSQL` 客户端，已被 `ext-redis`、`mysqli`、`pdo_mysql`、`pdo_pgsql` 取代
## v5.1.0
### 新機能
- `pdo_pgsql`のコルーチンサポートを追加
- `pdo_odbc`のコルーチンサポートを追加
- `pdo_oci`のコルーチンサポートを追加
- `pdo_sqlite`のコルーチンサポートを追加
- `pdo_pgsql`、`pdo_odbc`、`pdo_oci`、`pdo_sqlite`のコネクションプール構成を追加
### 増強
- `Http\Server`のパフォーマンスが向上し、限界では`60%`の改善が見込まれています。
### 修正
- 修正`WebSocket`协程客户端每次请求造成的内存泄漏
- 修正`http协程服务端`优雅退出导致客户端不退出的问题
- 修正编译的时候加入了`--enable-thread-context`选项会导致`Process::signal()`不起作用的问题
- 修正在`SWOOLE_BASE`模式下，当进程非正常退出时，连接数统计错误的问题
- 修正`stream_select()`函数签名错误
- 修正文件MIME信息大小写敏感错误
- 修正`Http2\Request::$usePipelineRead`拼写错误导致在PHP8.2的环境下会抛出警告
- 修正在`SWOOLE_BASE`模式下的内存泄漏问题
- 修正当`Http\Response::cookie()`设置cookie的过期时间导致的内存泄漏问题
- 修正在`SWOOLE_BASE`模式下的连接泄漏问题
### 内核
- Fix an issue with the function signature of `php_url_encode` in Swoole under PHP 8.3
- Fix unit test options issue
- Optimize and refactor code
- Compatible with PHP 8.3
- Compilation on 32-bit operating systems is not supported

### 内核
- PHP 8.3でのSwooleの`php_url_encode`関数のシグネチャの問題を修正
- ユニットテストのオプションの問題を修正
- コードを最適化してリファクタリング
- PHP 8.3と互換性あり
- 32ビットのオペレーティングシステムでのコンパイルはサポートされていません
## v5.0.3
### Enhancement
- `--with-nghttp2_dir`オプションが追加され、システムの`nghttp2`ライブラリを使用するためのものです
- バイト長やサイズに関連するオプションがサポートされています
- `Process\Pool::sendMessage()`関数が追加されました
- `Http\Response:cookie()`が`max-age`をサポートしています
### 修复
- 修复`Server task/pipemessage/finish`事件导致内存泄漏
### 内核
- `http`レスポンスヘッダの競合はエラーをスローしません
- `Server`接続のクローズはエラーをスローしません
## v5.0.2
### 強化
- `http2` のデフォルト設定の構成をサポート
- 8.1 以上の `xdebug` をサポート
- 複数のソケットを持つ curl ハンドルをサポートするために、ネイティブの curl を再構築しました。たとえば、curl ftp プロトコルをサポート
- `Process::setPriority/getPriority` に `who` パラメーターを追加
- `Coroutine\Socket::getBoundCid()` メソッドを追加
- `Coroutine\Socket::recvLine/recvWithBuffer` メソッドの `length` パラメーターのデフォルト値を `65536` に調整しました
- クロスコルーチン終了機能を再構築し、メモリ解放をより安全にし、致命的なエラーが発生した際のクラッシュ問題を解決しました
- `Coroutine\Client`、`Coroutine\Http\Client`、`Coroutine\Http2\Client` に `socket` プロパティを追加し、`socket` リソースを直接操作可能にしました
- `Http\Server` で `http2` クライアントに空のファイルを送信する機能を追加
- `Coroutine\Http\Server` のグレースフルな再起動をサポート。サーバーが閉じられるとき、クライアント接続は強制的に閉じず、新規リクエストの受付のみが停止します
- `pcntl_rfork` と `pcntl_sigwaitinfo` を安全でない関数リストに追加し、コルーチンコンテナが開始されるときに閉じられます
- `SWOOLE_BASE` モードのプロセスマネージャを再構築し、終了とリロードの動作を `SWOOLE_PROCESS` と一貫させました
##  v5.0.1
### 強化
- `PHP-8.2`をサポートし、コルーチンの例外処理が改善され、`ext-soap`と互換性が向上しました
- `pgsql`コルーチンクライアントの`LOB`サポートが追加されました
- `websocket`クライアントが改善され、ヘッダーに`=`を使用するのではなく`websocket`をアップグレードしました
- サーバーが`connection close`を送信するときに`http クライアント`が`keep-alive`を無効化するように最適化されました
- 圧縮ライブラリがない場合には`Accept-Encoding`ヘッダーを追加しないように、`http クライアント`が最適化されました
- デバッグ情報が向上し、`PHP-8.2`でパスワードが機密パラメータとして設定されます
- `Server::taskWaitMulti()`が強化され、コルーチン環境でブロッキングされなくなりました
- ログ関数が最適化され、ログファイルへの書き込みに失敗したときにはもはや画面には出力されません
### 修正
- 修正了`Coroutine::printBackTrace()`和`debug_print_backtrace()`之间的参数兼容性问题
- 修正了`Event::add()`函数对套接字资源的支持
- 修正了在没有安装`zlib`时的编译错误
- 修正了当解包服务器任务时遇到意外字符串造成崩溃的问题
- 修正了添加小于`1ms`的定时器被强制设置为`0`的问题
- 修正了在调用`Table::getMemorySize()`之前添加列导致崩溃的问题
- 将`Http\Response::setCookie()`方法中的过期参数名称修改为`expires`
## V5.0.0
### 新機能
- `Server`の`max_concurrency`オプションが追加されました
- `Coroutine\Http\Client`の`max_retries`オプションが追加されました
- `name_resolver`グローバルオプションが追加されました。`Server`に`upload_max_filesize`オプションが追加されました
- `Coroutine::getExecuteTime()`メソッドが追加されました
- `Server`に`SWOOLE_DISPATCH_CONCURRENT_LB`の`dispatch_mode`が追加されました
- すべての関数のパラメータと戻り値に型が追加され、型システムが強化されました
- エラー処理が最適化され、すべてのコンストラクターは失敗した場合に例外をスローします
- `Server`のデフォルトモードが調整され、デフォルトで`SWOOLE_BASE`モードになりました
- `pgsql`コルーチンクライアントがコアライブラリに移行されました。`4.8.x`ブランチのすべてのバグ修正が含まれています
### 削除
- `PSR-0`スタイルのクラス名を削除しました
- 終了関数での`Event::wait()`の自動追加機能を削除しました
- `Server::tick/after/clearTimer/defer`のエイリアスを削除しました
- `--enable-http2/--enable-swoole-json`を削除し、デフォルトで有効にしました
### 廃止
- Coroutine\RedisおよびCoroutine\MySQLのコルーチンクライアントはデフォルトで廃止されました。
## v4.8.13  
### 強化
- 原生のcurlをリファクタリングし、複数のソケットを持つcurlハンドル（例：curl FTPプロトコル）をサポートします。
- `http2`設定を手動で設定できるようにサポートします。
- `WebSocketクライアント`を改善し、ヘッダーに`websocket`を含めるようにアップグレードします（`equal`ではなく）。
- HTTPクライアントを最適化し、サーバーからの接続クローズ時に`keep-alive`を無効にします。
- デバッグ情報を改善し、PHP-8.2でパスワードを機密パラメーターに設定します。
- `HTTP Range Requests`をサポートします。
### 修正
- 修复了 `Coroutine::printBackTrace()` 和 `debug_print_backtrace()` 的参数兼容性问题
- 修复了在同时启用 `HTTP2` 和 `WebSocket` 协议时解析长度错误的问题
- 修复了在 `Server::send()`、`Http\Response::end()`、`Http\Response::write()` 和 `WebSocket/Server::push()` 中发生 `send_yield` 时的内存泄漏问题
- 修复了在添加列之前使用 `Table::getMemorySize()` 时导致崩溃的问题。
## v4.8.12
### 改善
- PHP8.2のサポート
- `Event::add()`関数が`sockets resources`をサポート
- `Http\Client::sendfile()`で4Gを超えるファイルをサポート
- `Server::taskWaitMulti()`がコルーチン環境をサポート
### 修復
- 修復接收到錯誤的`multipart body`會拋出錯誤訊息
- 修復定時器的超時時間小於`1ms`導致的錯誤
- 修復硬碟已滿導致的死鎖問題
## v4.8.11
### Enhancements
- サポート `Intel CET` セキュリティ防御メカニズム
- 追加 `Server::$ssl`プロパティ
- `pecl`を使用して`swoole`をコンパイルする際、`enable-cares`プロパティを追加
- `multipart_parser`解析器を再構築
### 修复
- 修复`pdo`持久化连接抛出异常导致的段错误
- 修复析构函数使用协程导致段错误
- 修复`Server::close()`的不正确的错误信息
## v4.8.10
### 修復

- `stream_select`のタイムアウトパラメータが`1ms`未満の場合、`0`にリセットします。
- コンパイル時に`-Werror=format-security`を追加すると、コンパイルに失敗する問題を修正します。
- `curl`を使用した場合に発生する`Swoole\Coroutine\Http\Server`のセグメンテーション違反を修正します。
## v4.8.9
### 增强

- 支持 `Http2` 服务器下的 `http_auto_index` 选项
### 修复

- 优化 `Cookie` 解析器，支持传入 `HttpOnly` 选项
- 修复 #4657，Hook `socket_create` 方法返回类型问题
- 修复 `stream_select` 内存泄漏
### CLI 更新

- `CygWin` 下携带了 SSL 证书链，解决了 SSL 认证出错的问题
- 更新至 `PHP-8.1.5`
## v4.8.8
### 最適化

- SW_IPC_BUFFER_MAX_SIZE を 64k に減らす
- http2 の header_table_size 設定を最適化
### 修復

- 修復使用 enable_static_handler 下載靜態文件大量套接字錯誤
- 修復 http2 server NPN 錯誤
## v4.8.7
### 強化

- curl_share サポートを追加
### 修复

- 修复 arm32 架构下的未定义符号错误
- 修复 `clock_gettime()` 兼容性
- 修复当内核缺乏大块内存时，PROCESS 模式服务器发送失败的问题
## v4.8.6
### 修复

- 为 boost/context API 名称添加了前缀
- 优化配置选项
## v4.8.5
### 修复

- Restore the parameter type of Table
- Fix crash when receiving incorrect data using Websocket protocol
## v4.8.4
### 修复

- 修复 sockets hook 与 PHP-8.1 的兼容性
- 修复 Table 与 PHP-8.1 的兼容性
- 修复在部分情况下协程风格的 HTTP 服务器解析 `Content-Type` 为 `application/x-www-form-urlencoded` 的 `POST` 参数不符合预期
## v4.8.3
### 新しい API

- `Coroutine\Socket::isClosed()` メソッドを追加
### 修正

- 修正在 PHP 8.1 版本下的 curl 原生钩子兼容性问题
- 修正在 PHP 8 版本下的 sockets 钩子兼容性问题
- 修正 sockets 钩子函数返回值错误
- 修正 Http2Server 的 sendfile 无法设置 content-type 问题
- 优化 HttpServer date header 性能，增加了缓存
## v4.8.2
### 修正

- proc_openフック内のメモリリークの問題を修正
- curlネイティブフックとPHP-8.0、PHP-8.1の互換性の問題を修正
- Managerプロセスで接続を正常に閉じられない問題を修正
- ManagerプロセスでsendMessageが使用できない問題を修正
- `Coroutine\Http\Server`が大容量のPOSTデータを正しく解析できない問題を修正
- PHP8環境で致命的エラーが発生した際に直接終了しない問題を修正
- コルーチン`max_concurrency`構成項目を調整し、`Co::set()`でのみ使用できるようにする
- `Coroutine::join()`で存在しないコルーチンを無視するように調整
## v4.8.1
### 新しい API

- `swoole_error_log_ex()` および `swoole_ignore_error()` 関数が追加されました。 (#4440) (@matyhtf)
- admin apiをext-swooleからext-swoole_plusに移動する(#4441) (@matyhtf)
- admin serverにget_composer_packagesコマンドを追加(swoole/library@07763f46) (swoole/library@8805dc05) (swoole/library@175f1797) (@sy-records) (@yunbaoi)
- 書き込み操作のPOSTメソッドリクエスト制限を追加(swoole/library@ac16927c) (@yunbaoi)
- admin serverがクラスメソッド情報を取得できるようにサポート(swoole/library@690a1952) (@djw1028769140) (@sy-records)
- admin serverコードを最適化(swoole/library#128) (swoole/library#131) (@sy-records)
- admin serverが複数のターゲットと複数のAPIに対する並列リクエストをサポート(swoole/library#124) (@sy-records)
- admin serverがインターフェース情報を取得できるようにサポート(swoole/library#130) (@sy-records)
- SWOOLE_HOOK_CURLがCURLOPT_HTTPPROXYTUNNELをサポート(swoole/library#126) (@sy-records)
### 修復

- join メソッドで同じコルーチンを並行して呼び出すことが禁止された (#4442) (@matyhtf)
- Table の原子ロックが誤って解放される問題を修正 (#4446) (@Txhua) (@matyhtf)
- 欠落していたヘルパーオプションを修正 (swoole/library#123) (@sy-records)
- get_static_property_value メソッドの引数エラーを修正 (swoole/library#129) (@sy-records)
## v4.8.0
### Non-breaking change

- In base mode, the onStart callback will always be called when the first worker process (worker id 0) starts, before onWorkerStart (#4389) (@matyhtf)
### 新增 API

- 新增 `Co::getStackUsage()` 方法 (#4398) (@matyhtf) (@twose)
- 新增 `Coroutine\Redis` 的一些 API (#4390) (@chrysanthemum)
- 新增 `Table::stats()` 方法 (#4405) (@matyhtf)
- 新增 `Coroutine::join()` 方法 (#4406) (@matyhtf)
### 新機能

- 支持 server command (#4389) (@matyhtf)
- 支持 `Server::onBeforeShutdown` 事件コールバック (#4415) (@matyhtf)
- Websocket pack fail時にエラーコードを設定する(swoole/swoole-src@d27c5a5) (@matyhtf)
- `Timer::exec_count`フィールドを追加しました(#4402) (@matyhtf)
- hook mkdirはopen_basedir ini設定を使用することができます(#4407) (@NathanFreeman)
- ライブラリにvendor_init.phpスクリプトを追加しました(swoole/library@6c40b02) (@matyhtf)
- SWOOLE_HOOK_CURLはCURLOPT_UNIX_SOCKET_PATHをサポートします(swoole/library#121) (@sy-records)
- Clientはssl_ciphers設定をサポートします(#4432) (@amuluowin)
- `Server::stats()`に新しい情報を追加しました(#4410) (#4412) (@matyhtf)
### 修复

- 修复文件上传时，对文件名字进行不必要的 URL decode (swoole/swoole-src@a73780e) (@matyhtf)
- 修复 HTTP2 max_frame_size 问题 (#4394) (@twose)
- 修复 curl_multi_select bug #4393 (#4418) (@matyhtf)
- 修复丢失的 coroutine options (#4425) (@sy-records)
- 修复当发送缓冲区满的时候，连接无法被 close 的问题 (swoole/swoole-src@2198378) (@matyhtf)
## v4.7.1
### 強化

- `System::dnsLookup` は `/etc/hosts` のクエリをサポートする (#4341) (#4349) (@zmyWL) (@NathanFreeman)
- mips64 の boost コンテキストサポートを追加 (#4358) (@dixyes)
- `SWOOLE_HOOK_CURL` が `CURLOPT_RESOLVE` オプションをサポート (swoole/library#107) (@sy-records)
- `SWOOLE_HOOK_CURL` が `CURLOPT_NOPROGRESS` オプションをサポート (swoole/library#117) (@sy-records)
- riscv64 の boost コンテキストサポートを追加 (#4375) (@dixyes)
### 修复

- 修复 PHP-8.1 在 on shutdown 时产生的内存错误 (#4325) (@twose)
- 修复 8.1.0beta1 的不可序列化类 (#4335) (@remicollet)
- 修复多个协程递归创建目录失败的问题 (#4337) (@NathanFreeman)
- 修复 native curl 在外网发送大文件偶发超时的问题，以及在 CURL WRITEFUNCTION 中使用协程文件 API 出现 crash 的问题 (#4360) (@matyhtf)
- 修复 `PDOStatement::bindParam()` 期望参数1为字符串的问题 (swoole/library#116) (@sy-records)
## v4.7.0
### 新しいAPI

- 新しく`Process\Pool::detach()`メソッドを追加しました (#4221) (@matyhtf)
- `Server`が`onDisconnect`コールバック関数をサポートしています (#4230) (@matyhtf)
- `Coroutine::cancel()`と`Coroutine::isCanceled()`メソッドを追加しました (#4247) (#4249) (@matyhtf)
- `Http\Client`が`http_compression`と`body_decompression`オプションをサポートしています (#4299) (@matyhtf)
### 強化

- 協力者 MySQL クライアントの`prepare`時に厳密な型フィールドをサポートします（＃4238）（@ Yurunsoft）
- DNS サポート`c-ares`ライブラリ（＃4275）（@matyhtf）
- 複数のポートでリッスンするときに、`Server`は異なるポートに心拍検出時間を設定できます（＃4290）（@matyhtf）
- `Server`の`dispatch_mode`は、`SWOOLE_DISPATCH_CO_CONN_LB`および`SWOOLE_DISPATCH_CO_REQ_LB`モードをサポートします（＃4318）（@matyhtf）
- `ConnectionPool::get()`は`timeout`パラメータをサポートします（swoole/library＃108）（@leocavalcante）
- フック Curlは`CURLOPT_PRIVATE`オプションをサポートします（swoole/library＃112）（@sy-records）
- `PDOStatementProxy::setFetchMode()`メソッドの関数宣言を最適化しました（swoole/library＃109）（@yespire）
### 修正

- 複数のコルーチンを作成する際にスレッドコンテキストを使用するとスレッドを作成できない例外がスローされる問題を修正 (8ce5041) (@matyhtf)
- Swooleをインストールする際にphp_swoole.hヘッダーファイルが欠落している問題を修正 (#4239) (@sy-records)
- EVENT_HANDSHAKEの下位互換性の問題を修正 (#4248) (@sy-records)
- SW_LOCK_CHECK_RETURNマクロが関数を2回呼び出す可能性がある問題を修正 (#4302) (@zmyWL)
- M1チップ上での`Atomic\Long`の問題を修正 (e6fae2e) (@matyhtf)
- `Coroutine\go()`の返り値が欠落する問題を修正 (swoole/library@1ed49db) (@matyhtf)
- `StringObject`の返り値の型問題を修正 (swoole/library#111) (swoole/library#113) (@leocavalcante) (@sy-records)
### Kernel

- Prevent hooking of functions that PHP has disabled (#4283) (@twose)
### テスト

- `Cygwin` 環境のビルドを追加 (#4222) (@sy-records)
- `alpine 3.13` と `3.14` のコンパイルテストを追加 (#4309) (@limingxinleo)
## v4.6.7
### 強化

- マネージャープロセスとタスク同期プロセスが`Process::signal()`関数を呼び出すサポートを追加 (#4190) (@matyhtf)
### 修复

- 修复信号不能被重复注册的问题 (#4170) (@matyhtf)
- 修复在 OpenBSD/NetBSD 上编译失败的问题 (#4188) (#4194) (@devnexen)
- 修复监听可写事件时特殊情况 onClose 事件丢失 (#4204) (@matyhtf)
- 修复 Symfony HttpClient 使用 native curl 的问题 (#4204) (@matyhtf)
- 修复`Http\Response::end()`方法总是返回 true 的问题 (swoole/swoole-src@66fcc35) (@matyhtf)
- 修复 PDOStatementProxy 产生的 PDOException (swoole/library#104) (@twose)
### 内核

- 重构 worker buffer，给 event data 加上 msg id 标志 (#4163) (@matyhtf)
- 修改 Request Entity Too Large 日志等级为 warning 级别 (#4175) (@sy-records)
- 替换 inet_ntoa and inet_aton 函数 (#4199) (@remicollet)
- 修改 output_buffer_size 默认值为 UINT_MAX (swoole/swoole-src@46ab345) (@matyhtf)  
## v4.6.6
### 増強

- FreeBSDでMasterプロセスが終了した後にManagerプロセスにSIGTERMシグナルを送信する機能をサポート (#4150) (@devnexen)
- SwooleをPHPに静的にコンパイルする機能をサポート (#4153) (@matyhtf)
- SNIがHTTPプロキシを使用する機能をサポート (#4158) (@matyhtf)
### 修复

- 修复同步客户端异步连接的错误 (#4152) (@matyhtf)
- 修复 Hook 原生 curl multi 导致的内存泄漏 (swoole/swoole-src@91bf243) (@matyhtf)
## v4.6.5
### 新しいAPIの追加

- WaitGroupにcountメソッドを追加します（swoole/library#100）（@sy-records）（@deminy）
### 更新内容

- オリジナルの curl multi をサポート (#4093) (#4099) (#4101) (#4105) (#4113) (#4121) (#4147) (swoole/swoole-src@cd7f51c) (@matyhtf) (@sy-records) (@huanghantao)
- HTTP/2 を使用するレスポンスで配列を使用してヘッダーを設定することができます
### 修复

- 修复 NetBSD 构建 (#4080) (@devnexen)
- 修复 OpenBSD 构建 (#4108) (@devnexen)
- 修复 illumos/solaris 构建，只有成员别名 (#4109) (@devnexen)
- 修复握手未完成时，SSL 连接的心跳检测不生效 (#4114) (@matyhtf)
- 修复 Http\Client 使用代理时`host`中存在`host:port`产生的错误 (#4124) (@Yurunsoft)
- 修复 Swoole\Coroutine\Http::request 中 header 和 cookie 的设置 (swoole/library#103) (@leocavalcante) (@deminy)
### 内核

- 支持 BSD 上的 asm context (#4082) (@devnexen)
- 在 FreeBSD 下使用 arc4random_buf 来实现 getrandom (#4096) (@devnexen)
- 优化 darwin arm64 context：删除 workaround 使用 label (#4127) (@devnexen)
### テスト

- アルパインのビルドスクリプトを追加 (#4104) (@limingxinleo)
## v4.6.4
### 新しいAPI

- 新しいCoroutine\Http::request、Coroutine\Http::post、Coroutine\Http::get関数を追加しました（swoole/library#97）（@matyhtf）
### 強化

- ARM 64 ビルドのサポート (#4057) (@devnexen)
- Swoole TCP サーバーで open_http_protocol を設定するサポート (#4063) (@matyhtf)
- ssl クライアントが証明書を設定するだけのサポート (91704ac) (@matyhtf)
- FreeBSD の tcp_defer_accept オプションのサポート (#4049) (@devnexen)
### 修正

- 修正使用 Coroutine\Http\Client 时缺少代理授权的问题 (edc0552) (@matyhtf)
- 修正 Swoole\Table 的内存分配问题 (3e7770f) (@matyhtf)
- 修正 Coroutine\Http2\Client 并发连接时的 crash (630536d) (@matyhtf)
- 修正 DTLS 的 enable_ssl_encrypt 问题 (842733b) (@matyhtf)
- 修正 Coroutine\Barrier 内存泄漏(swoole/library#94) (@Appla) (@FMiS)
- 修正由 CURLOPT_PORT 和 CURLOPT_URL 顺序引起的偏移错误 (swoole/library#96) (@sy-records)
- 修正`Table::get($key, $field)`当字段类型为 float 时的错误 (08ea20c) (@matyhtf)
- 修正 Swoole\Table 内存泄漏 (d78ca8c) (@matyhtf)
## v4.4.24
### 修复

- 修复 http2 客户端并发连接时的 crash (#4079)
## v4.6.3
### 新しいAPI

- 新しい Swoole\Coroutine\go 関数を追加しました (swoole/library@82f63be) (@matyhtf)
- 新しい Swoole\Coroutine\defer 関数を追加しました (swoole/library@92fd0de) (@matyhtf)
### 増強

- HTTP サーバーに compression_min_length オプションを追加します（#4033）（@matyhtf）
- アプリケーションレベルで Content-Length HTTP ヘッダーを設定できるようにします（#4041）（@doubaokun）
### 修正

- 修复程序达到文件打开限制时的 core dump (swoole/swoole-src@709813f) (@matyhtf)
- 修复 JIT 被禁用问题 (#4029) (@twose)
- 修复 `Response::create()` 参数错误问题 (swoole/swoole-src@a630b5b) (@matyhtf)
- 修复 ARM 平台下投递 task 时 task_worker_id 误报 (#4040) (@doubaokun)
- 修复 PHP8 开启 native curl hook 时的 core dump (#4042)(#4045) (@Yurunsoft) (@matyhtf)
- 修复 fatal error 时 shutdown 阶段的内存越界错误 (#4050) (@matyhtf)
### 内核

- 优化 ssl_connect/ssl_shutdown (#4030) (@matyhtf)
- 发生 fatal error 时直接退出进程 (#4053) (@matyhtf)
## v4.6.2
### 新たなAPI

- `Http\Request\getMethod()` メソッドを追加しました (#3987) (@luolaifa000)
- `Coroutine\Socket->recvLine()` メソッドを追加しました (#4014) (@matyhtf)
- `Coroutine\Socket->readWithBuffer()` メソッドを追加しました (#4017) (@matyhtf)
- `Response\create()` メソッドを強化し、Server の使用に独立して使用できるようにします (#3998) (@matyhtf)
- `Coroutine\Redis->hExists` が compatibility_mode を設定した後に bool 型を返すようにサポートされます (swoole/swoole-src@b8cce7c) (@matyhtf)
- `socket_read` に PHP_NORMAL_READ オプションを設定するサポートが追加されました (swoole/swoole-src@b1a0dcc) (@matyhtf)
### 修复

- 修复 `Coroutine::defer` 在 PHP8 下 coredump 的问题 (#3997) (@huanghantao)
- 修复当使用 thread context 的时候，错误设置 `Coroutine\Socket::errCode` 的问题 (swoole/swoole-src@004d08a) (@matyhtf)
- 修复在最新的 macos 下 Swoole 编译失败的问题 (#4007) (@matyhtf)
- 修复当 md5_file 参数传入 url 导致 php stream context 为空指针的问题 (#4016) (@ZhiyangLeeCN)  
### 内核

- 使用 AIO 线程池 hook stdio（解决之前把 stdio 视为 socket 导致的多协程读写问题） (#4002) (@matyhtf)
- 重构 HttpContext (#3998) (@matyhtf)
- 重构 `Process::wait()` (#4019) (@matyhtf)
## v4.6.1
### 増強

- `--enable-thread-context` コンパイルオプションを追加 (#3970) (@matyhtf)
- セッションIDを処理する際に接続の存在をチェックする (#3993) (@matyhtf)
- CURLOPT_PROXY の強化 (swoole/library#87) (@sy-records)
### 修復

- 修復 pecl 安裝中的最小 PHP 版本 (#3979) (@remicollet)
- 修復 pecl 安裝時沒有 `--enable-swoole-json` 和 `--enable-swoole-curl` 選項 (#3980) (@sy-records)
- 修復 openssl 線程安全問題 (b516d69f) (@matyhtf)
- 修復 enableSSL coredump (#3990) (@huanghantao)
### カーネル

- ipc writev の最適化、イベントデータが空の場合にコアダンプが発生しないように回避します（9647678）（@matyhtf）
## v4.5.11
### 増強

- Swoole\Tableの最適化 (#3959) (@matyhtf)
- CURLOPT_PROXYの増強 (swoole/library#87) (@sy-records)
### 修复

- 修复 Table 递增和递减时不能清除所有列问题 (#3956) (@matyhtf) (@sy-records)
- 修复编译时产生的`clock_id_t`错误 (49fea171) (@matyhtf)
- 修复 fread bugs (#3972) (@matyhtf)
- 修复 ssl 多线程 crash (7ee2c1a0) (@matyhtf)
- 兼容 uri 格式错误导致报错 Invalid argument supplied for foreach (swoole/library#80) (@sy-records)
- 修复 trigger_error 参数错误 (swoole/library#86) (@sy-records)
## v4.6.0
### 向下非互換の変更

- `session id`の最大制限が削除され、重複しなくなりました (#3879) (@matyhtf)
- コルーチンを使用する際に、`pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`などの安全でない機能が無効になりました (#3880) (@matyhtf)
- デフォルトでコルーチンフックが有効になりました (#3903) (@matyhtf)
### 削除

- PHP7.1 のサポートが終了しました (4a963df) (9de8d9e) (@matyhtf)
### 廃止

- `Event::rshutdown()` を非推奨としてマークし、代わりに Coroutine\run を使用してください (#3881) (@matyhtf)
### 新しいAPI

- setPriority/getPriorityのサポート (#3876) (@matyhtf)
- native-curlフックのサポート (#3863) (@matyhtf) (@huanghantao)
- サーバーイベントコールバック関数へのオブジェクトスタイルパラメータの渡しをサポートし、デフォルトではオブジェクトスタイルのパラメータを渡さないようにします (#3888) (@matyhtf)
- socketsの拡張をフックするサポート (#3898) (@matyhtf)
- 重複ヘッダーのサポート (#3905) (@matyhtf)
- SSL sniのサポート (#3908) (@matyhtf)
- stdioをフックするサポート (#3924) (@matyhtf)
- stream_socketのcapture_peer_certオプションのサポート (#3930) (@matyhtf)
- Http\Request::create/parse/isCompletedを追加しました (#3938) (@matyhtf)
- Http\Response::isWritableを追加しました (db56827) (@matyhtf)
### 強化

- サーバーのすべての時間精度をintからdoubleに変更 (#3882) (@matyhtf)
- swoole_client_select関数内でpoll関数のEINTR状況を確認する (#3909) (@shiguangqi)
- コルーチンデッドロック検出を追加する (#3911) (@matyhtf)
- 別のプロセスでSWOOLE_BASEモードを使用して接続を閉じる機能をサポートする (#3916) (@matyhtf)
- サーバーマスタープロセスとワーカープロセス間の通信のパフォーマンスを最適化し、メモリコピーを削減する (#3910) (@huanghantao) (@matyhtf)
### 修正

- Coroutine\Channel が閉じられたときに、すべてのデータをポップする (960431d) (@matyhtf)
- JIT を使用したときのメモリエラーを修正 (#3907) (@twose)
- `port->set()` dtls のコンパイルエラーを修正 (#3947) (@Yurunsoft)
- connection_list のエラーを修正 (#3948) (@sy-records)
- SSL 検証を修正 (#3954) (@matyhtf)
- Table のインクリメントおよびデクリメント時にすべての列がクリアされない問題を修正 (#3956) (@matyhtf) (@sy-records)
- LibreSSL 2.7.5 を使用した際のコンパイルエラーを修正 (#3962) (@matyhtf)
- 未定義定数 CURLOPT_HEADEROPT と CURLOPT_PROXYHEADER を修正 (swoole/library#77) (@sy-records)    
### 内核

- 默认情况下忽略 SIGPIPE 信号 (9647678) (@matyhtf)
- 支持同时运行 PHP 协程和 C 协程 (c94bfd8) (@matyhtf)
- 添加 get_elapsed 测试 (#3961) (@luolaifa000)
- 添加 get_init_msec 测试 (#3964) (@luffluo)
## v4.5.10
### 修复

- 修复使用 Event::cycle 时产生的 coredump (93901dc) (@matyhtf)
- 兼容 PHP8 (f0dc6d3) (@matyhtf)
- 修复 connection_list 错误 (#3948) (@sy-records)
## v4.4.23  
### 修复

- 修复 Swoole\Table 自减时数据错误 (bcd4f60d)(0d5e72e7) (@matyhtf)
- 修复同步客户端错误信息 (#3784)
- 修复解析表单数据边界时出现的内存溢出问题 (#3858)
- 修复 channel 的bug，关闭后无法 pop 已有数据
## v4.5.9
### 増強

- Coroutine\Http\Client に SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED 定数を追加 (#3873) (@sy-records)
### 修复

- 兼容 PHP8 (#3868) (#3869) (#3872) (@twose) (@huanghantao) (@doubaokun)
- 修复未定义的常量 CURLOPT_HEADEROPT 和 CURLOPT_PROXYHEADER (swoole/library#77) (@sy-records)
- 修复 CURLOPT_USERPWD (swoole/library@7952a7b) (@twose)
## v4.5.8
### 新しいAPI

- swoole_error_log関数を追加し、log_rotationを最適化しました（swoole/swoole-src@67d2bff）（@matyhtf）
- readVectorとwriteVectorはSSLをサポートします（#3857）（@huanghantao）
### 強化

- 子プロセスが終了したら、System::waitがブロックを解除します（#3832）（@matyhtf）
- DTLSは16Kのパケットをサポートします（#3849）（@matyhtf）
- Response::cookieメソッドはpriorityパラメータをサポートします（#3854）（@matyhtf）
- より多くのCURLオプションをサポートします（swoole/library#71）（@sy-records）
- CURL HTTPヘッダーの名前の大文字と小文字を区別しない場合に上書きされる問題を解決します（swoole/library#76）（@filakhtov）（@twose）（@sy-records）
### 修正

- 修正 readv_all 和 writev_all 处理 EAGAIN 错误的问题 (#3830) (@huanghantao)
- 修正 PHP8 编译警告的问题 (swoole/swoole-src@03f3fb0) (@matyhtf)
- 修正 Swoole\Table 二进制安全性问题 (#3842) (@twose)
- 修正 MacOS 下 System::writeFile 追加文件覆盖的问题 (swoole/swoole-src@a71956d) (@matyhtf)
- 修正 CURL 的 CURLOPT_WRITEFUNCTION (swoole/library#74) (swoole/library#75) (@sy-records)
- 修正解析 HTTP form-data 时内存泄漏的问题 (#3858) (@twose)
- 修正在 PHP8 中 `is_callable()` 无法访问类私有方法的问题 (#3859) (@twose)
### 内核

- 重构内存分配函数，使用 SwooleG.std_allocator (#3853) (@matyhtf)
- 重构管道 (#3841) (@matyhtf)
## v4.5.7
### 新增 API

- Coroutine\Socket クライアントに、writeVector、writeVectorAll、readVector、readVectorAll メソッドが追加されました (#3764) (@huanghantao)
- server->statsにtask_worker_numとdispatch_countを追加しました（#3771）（#3806）（@sy-records）（@matyhtf）
- 拡張依存関係（json、mysqlnd、socketsなど）を追加しました（#3789）（@remicollet）
- server->bindのuidの最小値をINT32_MINに制限しました（#3785）（@sy-records）
- swoole_substr_json_decodeにコンパイルオプションを追加して、負のオフセットをサポートしました（#3809）（@matyhtf）
- CURLのCURLOPT_TCP_NODELAYオプションをサポートしました（swoole/library#65）（@sy-records）（@deminy）
### 修复

- 修复同步客户端连接信息错误 (#3784) (@twose)
- 修复 hook scandir 函数的问题 (#3793) (@twose)
- 修复协程屏障 barrier 中的错误 (swoole/library#68) (@sy-records)
### カーネル

- boost.stacktraceを使用してprint-backtraceを最適化しました（＃3788）（@matyhtf）
## v4.5.6
### 新しい API

- [swoole_substr_unserialize](/functions?id=swoole_substr_unserialize) と [swoole_substr_json_decode](/functions?id=swoole_substr_json_decode) を追加しました (#3762) (@matyhtf)
### 増強

- `Coroutine\Http\Server` の `onAccept` メソッドをプライベートに変更しました (dfcc83b) (@matyhtf)
### 修复

- 修复 coverity 的问题 (#3737) (#3740) (@matyhtf)
- 修复 Alpine 环境下的一些问题 (#3738) (@matyhtf)
- 修复 swMutex_lockwait (0fc5665) (@matyhtf)
- 修复 PHP-8.1 安装失败 (#3757) (@twose)
### カーネル

- `Socket::read/write/shutdown` にアクティビティ検出を追加しました (#3735) (@matyhtf)
- session_id と task_id の型を int64 に変更しました (#3756) (@matyhtf)
## v4.5.5

!> このバージョンでは、[設定項目](/server/setting)の検出機能が追加されました。Swooleが提供していないオプションが設定されている場合、警告が発生します。

```shell
PHP Warning:  unsupported option [foo] in @swoole-src/library/core/Server/Helper.php 
```

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->set(['foo' => 'bar']);

$http->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```
### 新規 API

- Process\ProcessManagerをProcess\Managerへの別名として追加 (swoole/library#eac1ac5) (@matyhtf)
- HTTP2サーバーGOAWAYのサポート (#3710) (@doubaokun)
- `Co\map()` 関数を追加 (swoole/library#57) (@leocavalcante)
### 強化

- HTTP2のUNIXソケットクライアントをサポートするようになりました。 (#3668) (@sy-records)
- ワーカープロセスが終了した後、ワーカープロセスの状態をSW_WORKER_EXITに設定するようになりました。 (#3724) (@matyhtf)
- `Server::getClientInfo()`の戻り値にsend_queued_bytesとrecv_queued_bytesを追加しました。 (#3721) (#3731) (@matyhtf) (@Yurunsoft)
- Serverがstats_file構成オプションをサポートするようになりました。 (#3725) (@matyhtf) (@Yurunsoft)
### 修正

- 修正 PHP8 下的编译問題 (zend_compile_string 変更) (#3670) (@twose)
- 修正 PHP8 下的编译問題 (ext/sockets 互換性) (#3684) (@twose)
- 修正 PHP8 下的编译問題 (php_url_encode_hash_ex 変更) (#3713) (@remicollet)
- 'const char*'を 'char*'に変換する間違った型変換を修正 (#3686) (@remicollet)
- HTTP2 クライアントが HTTP プロキシ下で動作しない問題を修正 (#3677) (@matyhtf) (@twose)
- PDO 接続が再接続時にデータが混在する問題を修正 (swoole/library#54) (@sy-records)
- IPv6を使用するときにUDPサーバーのポート解析が間違っている問題を修正
- Lock::lockwait のタイムアウトが無効の問題を修正
## v4.5.4
### Non-backward compatible changes

- SWOOLE_HOOK_ALL includes SWOOlE_HOOK_CURL (#3606) (@matyhtf)
- Remove ssl_method, add ssl_protocols (#3639) (@Yurunsoft)
### 新規APIの追加

- 配列に firstKey と lastKey メソッドを追加します (swoole/library#51) (@sy-records)
### 強化

- `open_websocket_ping_frame`、`open_websocket_pong_frame`設定項目をWebSocketサーバに追加します (#3600) (@Yurunsoft)
### 修復

- 修復文件大於 2G 時，fseek ftell 不正確的問題 (#3619) (@Yurunsoft)
- 修復 Socket barrier 的問題 (#3627) (@matyhtf)
- 修復 http proxy 握手的問題 (#3630) (@matyhtf)
- 修復對端傳送 chunk 數據時，解析 HTTP Header 出錯的問題 (#3633) (@matyhtf)
- 修復 zend_hash_clean 斷言失敗的問題 (#3634) (@twose)
- 修復無法從事件循環移除 broken fd 的問題 (#3650) (@matyhtf)
- 修復收到無效的 packet 時導致核心轉儲的問題 (#3653) (@matyhtf)
- 修復 array_key_last 的 bug (swoole/library#46) (@sy-records)
### カーネル

- コードの最適化 (#3615) (#3617) (#3622) (#3635) (#3640) (#3641) (#3642) (#3645) (#3658) (@matyhtf)
- Swoole Table にデータを書き込む際の不要なメモリ操作を減らす (#3620) (@matyhtf)
- AIO のリファクタリング (#3624) (@Yurunsoft)
- readlink/opendir/readdir/closedir フックのサポート (#3628) (@matyhtf)
- swMutex_create の最適化、SW_MUTEX_ROBUST をサポート (#3646) (@matyhtf)
## v4.5.3
### 新しいAPIの追加

- `Swoole\Process\ProcessManager` を追加しました (swoole/library#88f147b) (@huanghantao)
- ArrayObject::append, StringObject::equals を追加しました (swoole/library#f28556f) (@matyhtf)
- [Coroutine::parallel](/coroutine/coroutine?id=parallel) を追加しました (swoole/library#6aa89a9) (@matyhtf)
- [Coroutine\Barrier](/coroutine/barrier) を追加しました (swoole/library#2988b2a) (@matyhtf)
- usePipelineReadを追加して、http2クライアントストリーミングをサポートする（#3354）（@twose）
- ファイルをダウンロードする際、データを受信する前にファイルを作成しないようにhttpクライアントを更新する（#3381）（@twose）
- httpクライアントが`bind_address`および`bind_port`設定をサポートする（#3390）（@huanghantao）
- httpクライアントが`lowercase_header`設定をサポートする（#3399）（@matyhtf）
- `Swoole\Server`が`tcp_user_timeout`設定をサポートする（#3404）（@huanghantao）
- `Coroutine\Socket`にイベントバリアを追加して、コルーチンの切り替えを減らす（#3409）（@matyhtf）
- 特定のswStringに`memory allocator`を追加する（#3418）（@matyhtf）
- cURLが`__toString`をサポートする（swoole/library#38）（@twose）
- `WaitGroup`のコンストラクタで`wait count`を直接設定することをサポートする（swoole/library#2fb228b8）（@matyhtf）
- `CURLOPT_REDIR_PROTOCOLS`を追加する（swoole/library#46）（@sy-records）
- http1.1サーバーがトレイラーをサポートする（#3485）（@huanghantao）
- コルーチンのsleep時間が1ms未満の場合、現在のコルーチンがyieldされる（#3487）（@Yurunsoft）
- http静的ハンドラがシンボリックリンクされたファイルをサポートする（#3569）（@LeiZhang-Hunter）
- Serverがcloseメソッドを呼び出した後、WebSocket接続を即座に閉じる（#3570）（@matyhtf）
- `stream_set_blocking`をフックすることをサポートする（#3585）（@Yurunsoft）
- 非同期HTTP2サーバーがストリーム制御をサポートする（#3486）（@huanghantao）（@matyhtf）
- `onPackage`コールバック関数の実行が完了した後にソケットバッファを解放する（#3551）（@huanghantao）（@matyhtf）
### 修復

- 修復 WebSocket coredump, 處理協議錯誤的狀態 (#3359) (@twose)
- 修復 swSignalfd_setup 函數以及 wait_signal 函數裡的空指針錯誤 (#3360) (@twose)
- 修復在設置了 dispatch_func 時，調用`Swoole\Server::close`會報錯的問題 (#3365) (@twose)
- 修復`Swoole\Redis\Server::format`函數中 format_buffer 初始化問題 (#3369) (@matyhtf) (@twose)
- 修復 MacOS 上無法獲取 mac 地址的問題 (#3372) (@twose)
- 修復 MySQL 測試用例 (#3374) (@qiqizjl)
- 修復多處 PHP8 兼容性問題 (#3384) (#3458) (#3578) (#3598) (@twose)
- 修復 hook 的 socket write 中丟失了 php_error_docref, timeout_event 和返回值問題 (#3383) (@twose)
- 修復異步 Server 無法在`WorkerStart`回調函數中關閉 Server 的問題 (#3382) (@huanghantao)
- 修復心跳線程在操作 conn->socket 的時候，可能會發生 coredump 的問題 (#3396) (@huanghantao)
- 修復 send_yield 的邏輯問題 (#3397) (@twose) (@matyhtf)
- 修復 Cygwin64 上的編譯問題 (#3400) (@twose)
- 修復 WebSocket finish 屬性無效的問題 (#3410) (@matyhtf)
- 修復遺漏的 MySQL transaction 錯誤狀態 (#3429) (@twose)
- 修復 hook 後的`stream_select`與 hook 之前返回值行為不一致的問題 (#3440) (@Yurunsoft)
- 修復使用`Coroutine\System`來創建子進程時丟失`SIGCHLD`信號的問題 (#3446) (@huanghantao)
- 修復`sendwait`不支持 SSL 的問題 (#3459) (@huanghantao)
- 修復`ArrayObject`和`StringObject`的若干問題 (swoole/library#44) (@matyhtf)
- 修復 mysqli 異常信息錯誤 (swoole/library#45) (@sy-records)
- 修復當設置`open_eof_check`後，`Swoole\Client`無法獲取正確的`errCode`的問題 (#3478) (@huanghantao)
- 修復 MacOS 上 `atomic->wait()`/`wakeup()`的若干問題 (#3476) (@Yurunsoft)
- 修復`Client::connect`連接拒絕的時候，返回成功狀態的問題 (#3484) (@matyhtf)
- 修復 alpine 環境下 nullptr_t 沒有被聲明的問題 (#3488) (@limingxinleo)
- 修復 HTTP Client 下載文件的時候，double-free 的問題 (#3489) (@Yurunsoft)
- 修復`Server`被銷毀時候，`Server\Port`沒釋放導致的內存洩漏問題 (#3507) (@twose)
- 修復 MQTT 協議解析問題 (318e33a) (84d8214) (80327b3) (efe6c63) (@GXhua) (@sy-records)
- 修復`Coroutine\Http\Client->getHeaderOut`方法導致的 coredump 問題 (#3534) (@matyhtf)
- 修復 SSL 驗證失敗後，丟失了錯誤信息的問題 (#3535) (@twose)
- 修復 README 中，`Swoole benchmark`鏈接錯誤的問題 (#3536) (@sy-records) (@santalex)
- 修復在`HTTP header/cookie`中使用`CRLF`後導致的`header`注入問題 (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 修復 issue #3463 中提到的變量錯誤的問題 (#3547) (chromium1337) (@huanghantao)
- 修復 pr #3463 中提到的錯別字問題 (#3547) (@deminy)
- 修復協程 WebSocket 服務器 frame->fd 為空的問題 (#3549) (@huanghantao)
- 修復心跳線程錯誤判斷連接狀態導致的連接洩漏問題 (#3534) (@matyhtf)
- 修復`Process\Pool`中阻塞了信號的問題 (#3582) (@huanghantao) (@matyhtf)
- 修復`SAPI`中使用 send headers 的問題 (#3571) (@twose) (@sshymko)
- 修復`CURL`執行失敗的時候，未設置`errCode`和`errMsg`的問題 (swoole/library#1b6c65e) (@sy-records)
- 修復當調用了`setProtocol`方法後，`swoole_socket_coro`accept coredump 的問題 (#3591)
### カーネル

- C ++スタイルを使用します（＃3349）（＃3351）（＃3454）（＃3479）（＃3490）（@huanghantao）（@matyhtf）
- パフォーマンスの改善として `PHP`オブジェクトのプロパティ読み取りに`Swoole known strings`を追加します（＃3363）（@huanghantao）
- 多くのコードを最適化します（＃3350）（＃3356）（＃3357）（＃3423）（＃3426）（＃3461）（＃3463）（＃3472）（＃3557）（＃3583）（@huanghantao）（@twose）（@matyhtf）
- 多くのテストコードを最適化します（＃3416）（＃3481）（＃3558）（@matyhtf）
- `Swoole\Table`の`int`型を簡素化します（＃3407）（@matyhtf）
- `sw_memset_zero`を追加し、`bzero`関数を置換します（＃3419）（@CismonX）
- ロギングモジュールを最適化します（＃3432）（@matyhtf）
- 多くの libswoole を再構築します（＃3448）（＃3473）（＃3475）（＃3492）（＃3494）（＃3497）（＃3498）（＃3526）（@matyhtf）
- 多くのヘッダーファイルインクルードを再構築します（＃3457）（@matyhtf）（@huanghantao）
- `Channel::count()`と`Channel::get_bytes()`を追加します（f001581）（@matyhtf）
- `scope guard`を追加します（＃3504）（@huanghantao）
- libswooleのカバレッジテストを追加します（＃3431）（@huanghantao）
- lib-swoole/ext-swooleのMacOS環境テストを追加します（＃3521）（@huanghantao）
- lib-swoole/ext-swooleのAlpine環境テストを追加します（＃3537）（@limingxinleo）
## v4.5.2

[v4.5.2](https://github.com/swoole/swoole-src/releases/tag/v4.5.2)、これはバグ修正バージョンで、下位互換性のない変更はありません。
### 強化

- Support `Server->set(['log_rotation' => SWOOLE_LOG_ROTATION_DAILY])` to generate logs daily (#3311) (@matyhtf)
- Support `swoole_async_set(['wait_signal' => true])`, where the reactor will not exit if there is a signal listener (#3314) (@matyhtf)
- Support `Server->sendfile` to send empty files (#3318) (@twose)
- Optimize worker idle/busy warning messages (#3328) (@huanghantao)
- Improve configuration for Host header in HTTPS proxy (use `ssl_host_name` for configuration) (#3343) (@twose)
- SSL now defaults to using ecdh-auto mode (#3316) (@matyhtf)
- SSL clients will exit silently when the connection is disconnected (#3342) (@huanghantao)
### 修复

- 修复 `Server->taskWait` 在 OSX 平台上的问题 (#3330) (@matyhtf)
- 修复 MQTT 协议解析错误的 bug (8dbf506b) (@guoxinhua) (2ae8eb32) (@twose)
- 修复 Content-Length int 类型溢出的问题 (#3346) (@twose)
- 修复 PRI 包长度检查缺失的问题 (#3348) (@twose)
- 修复 CURLOPT_POSTFIELDS 无法置空的问题 (swoole/library@ed192f64) (@twose)
- 修复 最新的连接对象在接收到下一个连接之前无法被释放的问题 (swoole/library@1ef79339) (@twose)
### コア

- ソケット書き込みゼロコピー機能 (#3327) (@twose)
- グローバル変数の読み書きを置き換えるためにswoole_get_last_error/swoole_set_last_errorを使用します (e25f262a) (@matyhtf) (#3315) (@huanghantao)
## v4.5.1

[v4.5.1](https://github.com/swoole/swoole-src/releases/tag/v4.5.1) は、バグ修正バージョンであり、`v4.5.0` で導入されるはずだったシステムファイル関数の廃止警告を追加しました。
- socket_context の bindto 構成をサポートするようになりました (#3275) (#3278) (@codinghuang)
- client::sendto での自動 DNS アドレス解決をサポート (#3292) (@codinghuang)
- Process->exit(0) はプロセスを直接終了させます。shutdown_functions を実行してから終了するには PHP の提供する exit を使用してください (a732fe56) (@matyhtf)
- `log_date_format` を構成することでログの日付形式を変更できるようになりました。ログにマイクロ秒のタイムスタンプを表示するには `log_date_with_microseconds` を使用します (baf895bc) (@matyhtf)
- CURLOPT_CAINFO および CURLOPT_CAPATH をサポートします (swoole/library#32) (@sy-records)
- CURLOPT_FORBID_REUSE をサポートします (swoole/library#33) (@sy-records)
### 修正

- 修正 32 ビット環境でのビルドエラー (#3276) (#3277) (@remicollet) (@twose)
- 協力クライアントが再接続した際に EISCONN エラーメッセージが表示されない問題を修正 (#3280) (@codinghuang)
- 潜在的な Table モジュールのバグを修正 (d7b87b65) (@matyhtf)
- 未定義の動作によるサーバー内のヌルポインタを修正(防御プログラミング) (#3304) (#3305) (@twose)
- ハートビート構成を有効にした後にヌルポインタエラーが発生する問題を修正 (#3307) (@twose)
- mysqli 設定が機能しない問題を修正 (swoole/library#35)
- レスポンス内の非標準的なヘッダー(スペースが不足している)を解析する問題を修正 (swoole/library#27) (@Yurunsoft)
### 廃止

- Coroutine\Systemの(fread/fgets/fwrite)などのメソッドを廃止としてマークします（代わりにhook機能を使用してください、PHPで提供されているファイル関数を直接使用してください） (c7c9bb40) (@twose)
### Kernel

- Use zend_object_alloc to allocate memory for custom objects (cf1afb25) (@twose)
- Some optimizations, add more configuration options for the log module (#3296) (@matyhtf)
- A lot of code optimization work and addition of unit tests (swoole/library) (@deminy)
## v4.5.0

[v4.5.0](https://github.com/swoole/swoole-src/releases/tag/v4.5.0) は、重要なバージョンアップです。v4.4.x で非推奨となっていた一部モジュールが削除されています。
### APIの追加

- DTLSサポートが追加されました。これにより、WebRTCアプリケーションを構築することができます（#3188）(@matyhtf)
- 組み込みの`FastCGI`クライアントが追加され、1行のコードでFPMにリクエストをプロキシしたり、FPMアプリケーションを呼び出すことができます（swoole/library#17）(@twose)
- `Co::wait`, `Co::waitPid`（子プロセスの回収に使用）、`Co::waitSignal`（シグナルの待機に使用）が追加されました（#3158）(@twose)
- `Co::waitEvent`（指定したイベントがソケットで発生するのを待機するために使用）が追加されました（#3197）(@twose)
- `Co::set(['exit_condition' => $callable])`（プログラムの終了条件をカスタマイズするために使用）が追加されました（#2918）（#3012）(@twose)
- `Co::getElapsed`（コルーチンが実行された時間を取得して、分析やゾンビコルーチンの特定に使用）が追加されました（#3162）(@doubaokun)
- `Socket::checkLiveness`（システムコールを使用して接続のアクティブ状態を判定）、`Socket::peek`（リードバッファをのぞき見）が追加されました（#3057）(@twose)
- `Socket->setProtocol(['open_fastcgi_protocol' => $bool])`（組み込みのFastCGIパースサポート）が追加されました（#3103）(@twose)
- `Server::get(Master|Manager|Worker)Pid`、`Server::getWorkerId`（非同期Server単一インスタンスとその情報を取得するためのメソッド）が追加されました（#2793）（#3019）(@matyhtf)
- `Server::getWorkerStatus`（workerプロセスの状態を取得し、SWOOLE_WORKER_BUSY、SWOOLE_WORKER_IDLEの定数で忙しさを表します）が追加されました（#3225）(@matyhtf)
- `Server->on('beforeReload', $callable)`および`Server->on('afterReload', $callable)`（マネージャープロセスで発生するサーバー再読み込みイベント）が追加されました（#3130）(@hantaohuang)
- `Http\Server`の静的ファイルハンドラーは、`http_index_files`と`http_autoindex`構成をサポートするようになりました（#3171）(@hantaohuang)
- `Http2\Client->read(float $timeout = -1)`メソッドがストリーミングレスポンスの読み取りをサポートするようになりました（#3011）（#3117）(@twose)
- `Http\Request->getContent`（rawContentメソッドのエイリアス）が追加されました（#3128）(@hantaohuang)
- `swoole_mime_type_(add|set|delete|get|exists)()`（mime関連API、組み込みのmimeタイプを追加、削除、取得、存在を確認できます）が追加されました（#3134）(@twose)
- masterとworkerプロセス間のメモリコピーを最適化（極限の場合、性能が4倍向上）（#3075）（#3087）（@hantaohuang）
- WebSocketのディスパッチロジックを最適化（#3076）（@matyhtf）
- WebSocketでのフレーム構築時のメモリコピーを最適化（#3097）（@matyhtf）
- SSL検証モジュールを最適化（#3226）（@matyhtf）
- SSL acceptとSSL handshakeを分離し、遅延するSSLクライアントによるコルーチンサーバーのフリーズ問題を解決しました（#3214）（@twose）
- MIPSアーキテクチャをサポート（#3196）（@ekongyun）
- UDPクライアントが受信したドメイン名を自動的に解析できるようになりました（#3236）（#3239）（@huanghantao）
- Coroutine\Http\Serverが一部の一般的なオプションをサポートするようになりました（#3257）（@twose）
- WebSocketハンドシェイク時にクッキーを設定できるようになりました（#3270）（#3272）（@twose）
- CURLOPT_FAILONERRORをサポート（swoole/library#20）（@sy-records）
- CURLOPT_SSLCERTTYPE、CURLOPT_SSLCERT、CURLOPT_SSLKEYTYPE、CURLOPT_SSLKEYをサポート（swoole/library#22）（@sy-records）
- CURLOPT_HTTPGETをサポート（swoole/library@d730bd08）（@shiguangqi）
### 削除

- `Runtime::enableStrictMode`メソッドを削除しました (b45838e3) (@twose)
- `Buffer`クラスを削除しました (559a49a8) (@twose)
- 新しいC++のAPI：coroutine::async関数は、ラムダを渡すことで非同期スレッドタスクを開始できます（#3127）（@matyhtf）
- ローレベルイベントAPI中の整数型fdをswSocketオブジェクトにリファクタリングしました（#3030）（@matyhtf）
- すべてのコアCファイルがC++ファイルに変換されました（#3030）（71f987f3）（@matyhtf）
- 一連のコード最適化が行われました（#3063）（#3067）（#3115）（#3135）（#3138）（#3139）（#3151）（#3168）（@hantaohuang）
- ヘッダーファイルの規格を最適化しました（#3051）（@matyhtf）
- 'enable_reuse_port'設定のリファクタリングを行い、より規格化されました（#3192）（@matyhtf）
- Socket関連APIのリファクタリングを行い、より規格化されました（#3193）（@matyhtf）
- バッファ予測による余分なシステムコールの削減を実装しました（3b5aa85d）（@matyhtf）
- ローレベルのリフレッシュタイマーswServerGS::nowを削除し、直接時間関数を使用して時間を取得します（#3152）（@hantaohuang）
- プロトコルコンフィギュレータの最適化が行われました（#3108）（@twose）
- より良い互換性のあるC構造の初期化方法がサポートされました（#3069）（@twose）
- bitフィールドが一貫してuchar型に統一されました（#3071）（@twose）
- 並列テストをサポートし、より高速になりました（#3215）（@twose）
### 修复

- 修复 enable_delay_receive 开启后 onConnect 无法触发的问题 (#3221) (#3224) (@matyhtf)
- 所有其它的 bug 修复都已合并到 v4.4.x 分支并在更新日志中体现, 在此不再赘述
## v4.4.22
### 修正

- 修正 HTTP2 client 在 HTTP 代理下无法正常工作的问题 (#3677) (@matyhtf) (@twose)
- 修正 PDO 断线重连时数据混乱的问题 (swoole/library#54) (@sy-records)
- 修正 swMutex_lockwait (0fc5665) (@matyhtf)
- 修正 UDP Server 使用 IPv6 时端口解析错误的问题
- 修正 systemd 文件描述符的问题
## v4.4.20

[v4.4.20](https://github.com/swoole/swoole-src/releases/tag/v4.4.20) は、バグ修正バージョンです。下位互換性に影響する変更はありません。
### 修复

- 修复在设置了 dispatch_func 时候，调用`Swoole\Server::close`会报错的问题 (#3365) (@twose)
- 修复`Swoole\Redis\Server::format`函数中 format_buffer 初始化问题 (#3369) (@matyhtf) (@twose)
- 修复 MacOS 上无法获取 mac 地址的问题 (#3372) (@twose)
- 修复 MySQL 测试用例 (#3374) (@qiqizjl)
- 修复异步 Server 无法在`WorkerStart`回调函数中关闭 Server 的问题 (#3382) (@huanghantao)
- 修复遗漏的 MySQL transaction 错误状态 (#3429) (@twose)
- 修复 HTTP Client 下载文件的时候，double-free 的问题 (#3489) (@Yurunsoft)
- 修复`Coroutine\Http\Client->getHeaderOut`方法导致的 coredump 问题 (#3534) (@matyhtf)
- 修复在`HTTP header/cookie`中使用`CRLF`后导致的`header`注入问题 (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 修复协程 WebSocket 服务器 frame->fd 为空的问题 (#3549) (@huanghantao)
- 修复 hook phpredis 产生的`read error on connection`问题 (#3579) (@twose)
- 修复 MQTT 协议解析问题 (#3573) (#3517) (9ad2b455) (@GXhua) (@sy-records)
## v4.4.19

[v4.4.19](https://github.com/swoole/swoole-src/releases/tag/v4.4.19)は、バグ修正バージョンです。互換性のない変更はありません。

!> 注意: v4.4.x はメンテナンスが終了しているため、必要な場合のみバグ修正が行われます。
### 修复

- 从 v4.5.2 合并了所有 bug 修复补丁
## v4.4.18

[v4.4.18](https://github.com/swoole/swoole-src/releases/tag/v4.4.18) は、バグ修正バージョンです。下位互換性のない変更はありません。
- UDPクライアントは、入力されたドメイン名を自動的に解析できるようになりました（＃3236）（＃3239）（@huanghantao）
- CLIモードでは、標準出力と標準エラー出力を閉じなくなりました（シャットダウン後に生成されるエラーログが表示されます）（＃3249）（@twose）
- Coroutine\Http\Serverに一般的なオプションのサポートが追加されました（＃3257）（@twose）
- WebSocketのハンドシェイク時にクッキーを設定する機能が追加されました（＃3270）（＃3272）（@twose）
- CURLOPT_FAILONERRORのサポートが追加されました（swoole/library＃20）（@sy-records）
- CURLOPT_SSLCERTTYPE、CURLOPT_SSLCERT、CURLOPT_SSLKEYTYPE、CURLOPT_SSLKEYのサポートが追加されました（swoole/library＃22）（@sy-records）
- CURLOPT_HTTPGETのサポートが追加されました（swoole/library@d730bd08）（@shiguangqi）
- すべてのPHP-Redis拡張機能のバージョンとの互換性を最大限に向上させました（異なるバージョンのコンストラクタ引数が異なります）（swoole/library＃24）（@twose）
- 接続オブジェクトのクローンを禁止しました（swoole/library＃23）（@deminy）
### 修復

- 修復 SSL 握手失敗的問題 (dc5ac29a) (@twose)
- 修復生成錯誤信息時產生的內存錯誤 (#3229) (@twose)
- 修復空白的 proxy 驗證信息 (#3243) (@twose)
- 修復 Channel 的內存泄漏問題 (並非真正的內存泄漏) (#3260) (@twose)
- 修復 Co\Http\Server 在循環引用時產生的一次性內存洩漏 (#3271) (@twose)
- 修復`ConnectionPool->fill`中的書寫錯誤 (swoole/library#18) (@NHZEX)
- 修復 curl 客戶端遭遇重定向時沒有更新連接的問題 (swoole/library#21) (@doubaokun)
- 修復產生 ioException 時空指針的問題 (swoole/library@4d15a4c3) (@twose)
- 修復 ConnectionPool@put 傳入 null 時沒有歸還新連接導致的死鎖問題 (swoole/library#25) (@Sinute)
- 修復 mysqli 代理實現導致的 write_property 錯誤 (swoole/library#26) (@twose)
## v4.4.17

[v4.4.17](https://github.com/swoole/swoole-src/releases/tag/v4.4.17) はバグ修正バージョンであり、下位互換性のない変更はありません。
### 増強

- SSLサーバーのパフォーマンスを向上させました (#3077) (85a9a595) (@matyhtf)
- HTTPヘッダーサイズの制限を解除しました (#3187) limitation (@twose)
- MIPSをサポートしました (#3196) (@ekongyun)
- CURLOPT_HTTPAUTHをサポートしました (swoole/library@570318be) (@twose)
### 修正

- 修正 package_length_func 的行为和可能导致一次性内存泄漏的问题 (#3111) (@twose)
- 修正 HTTP 状态码 304 下的错误行为 (#3118) (#3120) (@twose)
- 修正 Trace 日志错误的宏展开导致的内存错误 (#3142) (@twose)
- 修正 OpenSSL 函数签名 (#3154) (#3155) (@twose)
- 修正 SSL 错误信息 (#3172) (@matyhtf) (@twose)
- 修正 PHP-7.4 下的兼容性 (@twose) (@matyhtf)
- 修正 HTTP-chunk 的长度解析错误问题 (19a1c712) (@twose)
- 修正 chunked 模式下 multipart 请求的解析器行为 (3692d9de) (@twose)
- 修正 PHP-Debug 模式下 ZEND_ASSUME 断言失败 (fc0982be) (@twose)
- 修正 Socket 错误的地址 (d72c5e3a) (@twose)
- 修正 Socket getname (#3177) (#3179) (@matyhtf)
- 修正静态文件处理器对于空文件的错误处理 (#3182) (@twose)
- 修正 Coroutine\Http\Server 上传文件问题 (#3189) (#3191) (@twose)
- 修正 shutdown 期间可能的内存错误 (44aef60a) (@matyhtf)
- 修正 Server->heartbeat (#3203) (@matyhtf)
- 修正 CPU 调度器可能无法调度死循环的情况 (#3207) (@twose)
- 修正在不可变数组上的无效写入操作 (#3212) (@twose)
- 修正 WaitGroup 多次 wait 问题 (swoole/library@537a82e1) (@twose)
- 修正空 header 的处理 (和 cURL 保持一致) (swoole/library@7c92ed5a) (@twose)
- 修正非 IO 方法返回 false 时抛出异常的问题 (swoole/library@f6997394) (@twose)
- 修正 cURL-hook 下使用 proxy 端口号被多次添加到标头的问题 (swoole/library@5e94e5da) (@twose)
## v4.4.16

[v4.4.16](https://github.com/swoole/swoole-src/releases/tag/v4.4.16)，これはバグ修正バージョンであり、下位互換性のない変更はありません。
### 増強

- 現在、[Swoole バージョンサポート情報](https://github.com/swoole/swoole-src/blob/master/SUPPORTED.md) を入手できます。
- よりフレンドリーなエラーメッセージ (0412f442) (09a48835) (@twose)
- 特定の特殊システムでのシステムコールのデッドロックを防ぐ (069a0092) (@matyhtf)
- PDOConfig にドライバーオプションを追加(swoole/library#8) (@jcheron)
### 修复

- 修复 http2_session.default_ctx 内存错误 (bddbb9b1) (@twose)
- 修复未初始化的 http_context (ce77c641) (@twose)
- 修复 Table 模块中的书写错误 (可能会造成内存错误) (db4eec17) (@twose)
- 修复 Server 中 task-reload 的潜在问题 (e4378278) (@GXhua)
- 修复不完整协程 HTTP 服务器请求原文 (#3079) (#3085) (@hantaohuang)
- 修复 static handler (当文件为空时, 不应返回 404 响应) (#3084) (@Yurunsoft)
- 修复 http_compression_level 配置无法正常工作 (16f9274e) (@twose)
- 修复 Coroutine HTTP2 Server 由于没有注册 handle 而产生空指针错误 (ed680989) (@twose)
- 修复配置 socket_dontwait 不工作的问题 (27589376) (@matyhtf)
- 修复 zend::eval 可能会被执行多次的问题 (#3099) (@GXhua)
- 修复 HTTP2 服务器由于在连接关闭后响应而产生的空指针错误 (#3110) (@twose)
- 修复 PDOStatementProxy::setFetchMode 适配不当的问题 (swoole/library#13) (@jcheron)
