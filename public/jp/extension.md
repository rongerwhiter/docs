# 拡張機能の競合

一部のトレースデバッグ用の`PHP`拡張機能はグローバル変数を大量に使用しているため、`Swoole`のコルーチンがクラッシュする可能性があります。次の関連拡張機能を無効にしてください：

* phptrace
* aop
* molten
* xhprof
* phalcon（`Swoole`コルーチンは `phalcon` フレームワーク内で実行できません）

## Xdebug サポート
`5.1`バージョン以降、`xdebug`拡張機能を使用して `Swoole` プログラムをデバッグできます。コマンドライン引数を使用するか、`php.ini` を変更して有効にしてください。

```ini
swoole.enable_fiber_mock=On
```

または

```shell
php -d swoole.enable_fiber_mock=On your_file.php
```
