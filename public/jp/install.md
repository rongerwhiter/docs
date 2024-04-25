Sure! Please tell me about the installation problem you are experiencing so that I can assist you.
## Swooleバージョンのアップグレード

peclを使用してインストールおよびアップグレードできます

```shell
pecl upgrade swoole
```

また、新しいバージョンをGithub / Gitee / peclから直接ダウンロードして、再インストールおよびコンパイルすることもできます。

- Swooleのバージョンを更新する際、古いバージョンをアンインストールまたは削除する必要はありません。新しいバージョンで上書きされます。
- Swooleをコンパイルしてインストールすると、追加のファイルはなく、swoole.soのみがあります。他のマシンでコンパイルされたバイナリバージョンになります。swoole.soを直接上書きして、バージョンを切り替えることができます。
- git cloneしたコードを更新するために`git pull`を実行した場合は、必ず再度`phpize`、`./configure`、`make clean`、`make install`を実行してください。
- 対応するDockerを使用して、Swooleバージョンをアップグレードすることもできます。
```php
## Make sure to check in phpinfo for any extensions not shown in php -m

Check in CLI mode by running `php --ri swoole`

If you see the Swoole extension information, then your installation was successful!

**99.999% of people who succeed at this step can directly use Swoole!**

You don't need to worry about whether Swoole shows up in `php -m` or `phpinfo` web page.

This is because Swoole runs in CLI mode, and its functionality is very limited in traditional FPM mode.

In FPM mode, you **cannot use** any of the main asynchronous/coroutine features. 99.999% of people cannot achieve what they want in FPM mode, yet they get stuck wondering why the extension information is not showing.

**First, make sure you truly understand how Swoole operates in its running mode before continuing to investigate installation issues!**
``` 

## PHPのリストに表示されていないphpinfoに存在するもの

`php --ri swoole`の実行によってCLIモードで確認してください

Swooleの拡張機能情報が表示されれば、インストールが正常に完了したことになります！

**この段階で成功した99.999％の人は、Swooleを直接使用できます！**

`php -m`や`phpinfo`のページにSwooleが表示されるかどうかを気にする必要はありません。

これは、SwooleがCLIモードで実行され、従来のFPMモードでは機能が非常に限られているためです。

FPMモードでは、主要な非同期/コルーチン機能を**使用できません**。99.999％の人はFPMモードで望むことを達成できず、拡張機能情報が表示されていないことに困惑してしまいます。

**インストールの問題を調査する前に、Swooleの動作モードを正しく理解しているか確認してから続行してください。**
### 原因

Swooleをコンパイルしてインストールした後、`php-fpm/apache`の`phpinfo`ページには表示されるが、コマンドラインの`php -m`には表示されない場合、原因はおそらく`cli/php-fpm/apache`が異なるphp.ini設定を使用しているためです。
### 解決策

1. php.ini の場所を確認します

`cli` コマンドラインで `php -i | grep php.ini` または `php --ini` を実行して、php.ini の絶対パスを見つけます

`php-fpm/apache` では `phpinfo` ページで php.ini の絶対パスを見つけます

2. 対応する php.ini に `extension=swoole.so` があるかを確認します

```shell
cat /path/to/php.ini | grep swoole.so
```
## pcre.h: ファイルまたはディレクトリがありません

Swoole拡張をコンパイルすると、

```bash
fatal error: pcre.h: No such file or directory
```

というエラーが表示されます。

これはpcreが不足しているためで、libpcreをインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libpcre3 libpcre3-dev
```
### centos/redhat

```shell
sudo yum install pcre-devel
```
### 他のLinux

[PCRE公式ウェブサイト](http://www.pcre.org/)からソースコードをダウンロードし、`pcre`ライブラリをコンパイルしてインストールします。

`PCRE`ライブラリをインストールした後、`swoole`を再コンパイルしてインストールする必要があります。その後、`php --ri swoole`を使用して`swoole`拡張機能に関連する情報を確認し、`pcre => enabled`があるかどうかを確認します。
## '__builtin_saddl_overflow' がこのスコープで宣言されていません

 ```
error: '__builtin_saddl_overflow' がこのスコープで宣言されていません
  if (UNEXPECTED(__builtin_saddl_overflow(Z_LVAL_P(op1), 1, &lresult))) {

note: マクロ 'UNEXPECTED' の定義で
 # define UNEXPECTED(condition) __builtin_expect(!!(condition), 0)
```

これは既知の問題です。問題は、CentOS上のデフォルトのgccが必要な定義を欠いていることであり、gccをアップグレードしても、PECLが古いコンパイラを見つけてしまうことがあります。

ドライバをインストールするためには、まずgccをアップグレードするために以下のようにdevtoolsetコレクションをインストールする必要があります：

```shell
sudo yum install centos-release-scl
sudo yum install devtoolset-7
scl enable devtoolset-7 bash
```
## fatal error: 'openssl/ssl.h'ファイルが見つかりません

opensslライブラリのパスを指定するには、コンパイル時に[--with-openssl-dir](/environment?id=通用参数)パラメータを追加してください

!> [pecl](/environment?id=pecl)を使用してSwooleをインストールする際に、opensslを有効にする場合は、[--with-openssl-dir](/environment?id=通用参数)パラメータを追加することもできます。例：`opensslサポートを有効にしますか？ [no] : yes --with-openssl-dir=/opt/openssl/`
NOTICE: PHPメッセージ: PHPの警告: PHPのスタートアップ: swoole: モジュールを初期化できません  
モジュールはモジュールAPI=20090626でコンパイルされました  
PHPはモジュールAPI=20121212でコンパイルされました  
これらのオプションは一致する必要があります  
不明な行で

PHPのバージョンと`phpize`および`php-config`を使用してコンパイルする際に対応する必要があるので、絶対パスを使用してコンパイルし、PHPを実行する際にも絶対パスを使用する必要があります。

```shell
/usr/local/php-5.4.17/bin/phpize
./configure --with-php-config=/usr/local/php-5.4.17/bin/php-config

/usr/local/php-5.4.17/bin/php server.php
```
## Xdebugのインストール

```shell
git clone git@github.com:swoole/sdebug.git -b sdebug_2_9 --depth=1

cd sdebug

phpize
./configure
make clean
make
make install

#もしphpize、php-configなどの設定ファイルがデフォルトであれば、以下を直接実行できます
./rebuild.sh
```

拡張機能をロードするためにphp.iniファイルを変更し、以下の情報を追加してください。

```ini
zend_extension=xdebug.so

xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_port=8000
xdebug.idekey="xdebug"
```

正しくロードされたかどうかを確認するために、次のコマンドを実行してください。

```shell
php --ri sdebug
```
## configure: error: C preprocessor "/lib/cpp" fails sanity check

If you encounter the error message during installation:

```shell
configure: error: C preprocessor "/lib/cpp" fails sanity check
```

it means that necessary dependencies are missing. You can install them using the following commands:

```shell
yum install glibc-headers
yum install gcc-c++
```
PHP7.4.11+で新しいバージョンのSwooleをコンパイルする際に、MacOSで次のようなエラーが発生することがわかります。

```shell
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:523:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:586:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:656:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:766:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
4 errors generated.
make: *** [ext-src/php_swoole.lo] Error 1
ERROR: `make' failed
```

Solution: Modify the source code `/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h`, make sure to change it to your corresponding header file path;

Change `ZEND_USE_ASM_ARITHMETIC` to always `0`, keeping the content of the `else` in the following code

```c
#if defined(HAVE_ASM_GOTO) && !__has_feature(memory_sanitizer)
# define ZEND_USE_ASM_ARITHMETIC 1
#else
`ZEND_USE_ASM_ARITHMETIC` を `0` に定義します。
`--enable-swoole-curl`オプションを有効にした後、Swoole拡張機能をコンパイルすると、次のエラーメッセージが表示されます。

```bash
fatal error: curl/curl.h: No such file or directory
```

この原因は、curl依存関係が不足しているためで、libcurlをインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libcurl4-openssl-dev
```
### centos/redhat

```shell
sudo yum install libcurl-devel
``````fromJson
このコマンドは、CentOSやRedHat Linuxディストリビューション向けに、libcurlの開発パッケージをインストールするものです。
```
### アルパイン

```shell
apk add curl-dev
```
## fatal error: ares.h: No such file or directory :id=libcares

`--enable-cares`オプションを有効にした場合、Swoole拡張をコンパイルする際に次のエラーが発生します。

```bash
fatal error: ares.h: No such file or directory
```

これはc-aresの依存関係が不足しているため、libcaresをインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libc-ares-dev
``````Japanese
### ubuntu/debian

```shell
sudo apt-get install libc-ares-dev
```
### centos/redhat

```shell
sudo yum install c-ares-devel
`````

### centos/redhat

```shell
sudo yum install c-ares-devel
```
### アルパイン

```shell
apk add c-ares-dev
```
### MacOs

```shell
brew install c-ares
```
