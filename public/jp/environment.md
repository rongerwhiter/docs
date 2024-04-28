# Swoole 설치

`Swoole` 확장은 `PHP` 표준 확장으로 구축됩니다. 컴파일 및 검사 스크립트를 생성하기 위해 `phpize`를 사용하고, 컴파일 구성 검사를 위해 `./configure`를 사용하며, 컴파일을 위해 `make`를 사용하고, 설치를 위해 `make install`를 사용합니다.

* 특별한 요구사항이 없는 경우, 반드시 `Swoole`의 최신 [Swoole](https://github.com/swoole/swoole-src/releases/) 버전을 컴파일하여 설치하십시오.
* 현재 사용자가 `root`가 아닌 경우, `PHP` 설치 디렉토리에 쓰기 권한이 없을 수 있으므로 설치할 때 `sudo` 또는 `su`가 필요할 수 있습니다.
* `git` 브랜치에서 직접 `git pull`로 코드를 업데이트 하는 경우, 다시 컴파일하기 전에 반드시 `make clean`을 실행해야 합니다.
* `Linux` (2.3.32 이상의 커널), `FreeBSD`, `MacOS` 세 가지 운영 체제만 지원됩니다.
* 낮은 버전의 Linux 시스템 (예: `CentOS 6`)은 `RedHat`에서 제공하는 `devtools`를 사용하여 컴파일할 수 있습니다. [참고 문서](https://blog.csdn.net/ppdouble/article/details/52894271).
* `Windows` 플랫폼에서는 `WSL(Windows Subsystem for Linux)` 또는 `CygWin`을 사용할 수 있습니다.
* 일부 확장은 `Swoole` 확장과 호환되지 않을 수 있습니다. [확장 충돌](/getting_started/extension)을 참조하십시오.
## 설치 준비

다음 소프트웨어가 시스템에 설치되어 있는지 확인해야 합니다.

- `4.8` 버전은 `php-7.2` 이상이 필요합니다.
- `5.0` 버전은 `php-8.0` 이상이 필요합니다.
- `gcc-4.8` 이상 버전
- `make`
- `autoconf`
## 빠른 설치

> 1. 소프트웨어 소스 다운로드

* [https://github.com/swoole/swoole-src/releases](https://github.com/swoole/swoole-src/releases)
* [https://pecl.php.net/package/swoole](https://pecl.php.net/package/swoole)
* [https://gitee.com/swoole/swoole/tags](https://gitee.com/swoole/swoole/tags)

> 2. 소스 코드 컴파일 및 설치

소스 코드를 다운로드 한 후 터미널에서 소스 코드 디렉토리로 이동하여 아래 명령어를 실행하여 컴파일 및 설치합니다.

각 명령어는 줄 단위로 실행되어야 합니다. ubuntu에서 `phpize`가 없는 경우, `sudo apt-get install php-dev` 명령어를 실행하여 `phpize`를 설치합니다.

```shell
cd swoole-src && \
phpize && \
./configure && \
sudo make && sudo make install
```

> 3. 확장 기능 활성화

시스템에 성공적으로 설치한 후, `php.ini` 파일에 `extension=swoole.so` 라인을 추가하여 Swoole 확장 기능을 활성화합니다.
## 고급 완전 편집 예제

!> Swoole에 대해 처음 접하는 개발자는 먼저 위의 간단한 컴파일을 시도한 후, 필요에 따라 컴파일 매개변수를 구체적인 요구 사항과 버전에 맞게 조정할 수 있습니다. [컴파일 매개변수 참조](/environment?id=编译选项)

다음 스크립트는 `master` 브랜치의 소스 코드를 다운로드하고 컴파일합니다. 모든 종속성이 설치되어 있는지 확인해야 합니다. 그렇지 않으면 다양한 종속성 오류가 발생할 수 있습니다.

```shell
mkdir -p ~/build && \
cd ~/build && \
rm -rf ./swoole-src && \
curl -o ./tmp/swoole.tar.gz https://github.com/swoole/swoole-src/archive/master.tar.gz -L && \
tar zxvf ./tmp/swoole.tar.gz && \
mv swoole-src* swoole-src && \
cd swoole-src && \
phpize && \
./configure \
--enable-openssl --enable-sockets --enable-mysqlnd --enable-swoole-curl --enable-cares --enable-swoole-pgsql && \
sudo make && sudo make install
```
## PECL

> Note: PECL의 공개 일정이 GitHub의 공개 일정보다 늦을 수 있습니다.

Swoole 프로젝트는 PHP 공식 확장 라이브러리에 추가되었습니다. 수동으로 다운로드하고 컴파일하는 대신 PHP 공식 `pecl` 명령을 사용하여 간편하게 다운로드하고 설치할 수 있습니다.

```shell
pecl install swoole
```

PECL을 통해 Swoole을 설치할 때, 설치 과정 중에 특정 기능을 활성화할지 묻는데, 이를 설치하기 전에 제공할 수도 있습니다. 예를 들어 다음과 같습니다:

```shell
pecl install -D 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole

#또는
pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole
```  
## php.ini에 Swoole 추가

마지막으로, 컴파일 및 설치를 완료한 후, `php.ini` 파일을 수정하여 다음을 추가합니다.

```ini
extension=swoole.so
```

`swoole.so`가 성공적으로 로드되었는지 확인하려면 `php -m`을 사용합니다. 로드되지 않은 경우 `php.ini`의 경로가 잘못되었을 수 있습니다.  
`php --ini`를 사용하여 `php.ini`의 절대 경로를 확인할 수 있습니다. `Loaded Configuration File` 항목에 나오는 파일이 로드된 `php.ini` 파일입니다. 값이 `none`인 경우 PHP에서 `php.ini` 파일이 로드되지 않았으며 직접 파일을 만들어야 합니다.

!> `PHP` 버전 지원 및 `PHP` 공식 유지 관리 버전과 일치해야 합니다. [PHP 버전 지원 기간 표](http://php.net/supported-versions.php)를 참조하세요.
# Swooleのインストール

`swoole`拡張機能は、`PHP`の標準拡張機能に従って構築されています。`phpize`を使用してコンパイル検査スクリプトを生成し、`./configure`でコンパイル構成を確認し、`make`でコンパイルし、`make install`でインストールします。

* 特別な要件がない場合、必ず`Swoole`の最新バージョンをコンパイルしてインストールしてください。[Swoole](https://github.com/swoole/swoole-src/releases/)
* 現在のユーザーが`root`でない場合、`PHP`のインストールディレクトリに書き込み権限がない可能性があります。インストール時に`sudo`または`su`を使用する必要があります。
* `git`ブランチで直接`git pull`してコードを更新する場合は、再コンパイルする前に`make clean`を実行してください。
* `Linux`(2.3.32以上のカーネル)、`FreeBSD`、`MacOS`の3つのオペレーティングシステムのみをサポートしています。
* 低いバージョンのLinuxシステム（`CentOS 6`など）では、`RedHat`が提供する`devtools`を使用してコンパイルできます。[参考文献](https://blog.csdn.net/ppdouble/article/details/52894271)
* `Windows`プラットフォームでは、`WSL(Windows Subsystem for Linux)`または`CygWin`を使用できます。
* 一部の拡張機能と`swoole`拡張機能が互換性がない場合があります。[拡張機能の衝突](/getting_started/extension)を参照してください。
## インストールの準備

インストールする前に、システムに以下のソフトウェアがインストールされていることを確認してください。

- `4.8` バージョンは `php-7.2` またはそれ以上のバージョンが必要です
- `5.0` バージョンは `php-8.0` またはそれ以上のバージョンが必要です
- `gcc-4.8` またはそれ以上のバージョン
- `make`
- `autoconf`
## 速やかなインストール

> 1. Swooleのソースコードをダウンロード

* [https://github.com/swoole/swoole-src/releases](https://github.com/swoole/swoole-src/releases)
* [https://pecl.php.net/package/swoole](https://pecl.php.net/package/swoole)
* [https://gitee.com/swoole/swoole/tags](https://gitee.com/swoole/swoole/tags)

> 2. ソースコードからコンパイルしてインストール

ソースコードをダウンロードした後、ターミナルでソースコードディレクトリに移動し、以下のコマンドを実行してコンパイルおよびインストールを行います。

!> Ubuntuに`phpize`がインストールされていない場合は、`sudo apt-get install php-dev`コマンドを実行して`phpize`をインストールしてください。

```shell
cd swoole-src && \
phpize && \
./configure && \
sudo make && sudo make install
```

> 3. 拡張機能の有効化

システムにコンパイルされた後、Swoole拡張機能を有効にするには、`php.ini`ファイルに`extension=swoole.so`という行を追加する必要があります。
## 上級者向けの完全コンパイル例

初めてSwooleに触れる開発者は、まずは上記の簡単なコンパイルを試してみてください。さらなるニーズがある場合は、具体的な要件やバージョンに基づいて、以下のコンパイルパラメータを調整することができます。[コンパイルパラメータの参考](/environment?id=コンパイルオプション)

以下のスクリプトは、`master`ブランチのソースコードをダウンロードしてコンパイルします。すべての依存関係がインストールされていることを確認してください。そうでない場合、さまざまな依存関係のエラーに遭遇する可能性があります。

```shell
mkdir -p ~/build && \
cd ~/build && \
rm -rf ./swoole-src && \
curl -o ./tmp/swoole.tar.gz https://github.com/swoole/swoole-src/archive/master.tar.gz -L && \
tar zxvf ./tmp/swoole.tar.gz && \
mv swoole-src* swoole-src && \
cd swoole-src && \
phpize && \
./configure \
--enable-openssl --enable-sockets --enable-mysqlnd --enable-swoole-curl --enable-cares --enable-swoole-pgsql && \
sudo make && sudo make install
```
## PECL

> Note: PECL release times are later than GitHub release times

Swooleプロジェクトは、PHP公式拡張ライブラリに収録されました。手動でダウンロードしてコンパイルするだけでなく、PHP公式提供の`pecl`コマンドを使用して簡単にダウンロードしてインストールすることもできます。

```shell
pecl install swoole
```

PECLを使用してSwooleをインストールする際には、インストールプロセス中に特定の機能を有効にするかどうかを尋ねてきます。これは、例えばインストールを実行する前に次のように提供することもできます：

```shell
pecl install -D 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole

#または
pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole
```
## php.ini に Swoole を追加する

最後に、コンパイルしてインストールが成功したら、`php.ini` ファイルを編集して次の行を追加します。

```ini
extension=swoole.so
```

`swoole.so` が正しくロードされたかどうかを確認するには、`php -m` を使用します。もしロードされていない場合は、`php.ini` のパスが間違っている可能性があります。  
`php --ini` を使用して、`php.ini` ファイルの絶対パスを特定できます。`Loaded Configuration File` の項目で、読み込まれた php.ini ファイルが表示されます。値が `none` であれば、まったく php.ini ファイルが読み込まれていないことを示し、自分で作成する必要があります。

!> `PHP` のバージョンサポートと`PHP` 公式メンテナンスバージョンは同じに保つことをお勧めします。[PHPサポートバージョンのスケジュール](http://php.net/supported-versions.php)を参照してください。
## 他のプラットフォームのコンパイル

ARMプラットフォーム（Raspberry PI）

- `GCC`をクロスコンパイルに使用
- `Swoole`をコンパイルする際、`Makefile`を手動で変更して、`-O2`コンパイルオプションを削除する必要があります。

MIPSプラットフォーム（OpenWrtルーター）

- GCCをクロスコンパイルに使用

Windows WSL

`Windows 10`システムは `Linux` サブシステムサポートを追加し、`BashOnWindows`環境で`Swoole`を使用することもできます。インストールコマンド

```shell
apt-get install php7.0 php7.0-curl php7.0-gd php7.0-gmp php7.0-json php7.0-mysql php7.0-opcache php7.0-readline php7.0-sqlite3 php7.0-tidy php7.0-xml  php7.0-bcmath php7.0-bz2 php7.0-intl php7.0-mbstring  php7.0-mcrypt php7.0-soap php7.0-xsl  php7.0-zip
pecl install swoole
echo 'extension=swoole.so' >> /etc/php/7.0/mods-available/swoole.ini
cd /etc/php/7.0/cli/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
cd /etc/php/7.0/fpm/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
```

!> `WSL`環境では、`daemonize`オプションを無効にする必要があります  
バージョンが`17101`未満の`WSL`では、ソースのインストール後に`configure`を変更して`config.h`を閉じ、`HAVE_SIGNALFD`を無効にする必要があります。
## Docker公式イメージ

- GitHub: [https://github.com/swoole/docker-swoole](https://github.com/swoole/docker-swoole)
- dockerhub: [https://hub.docker.com/r/phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)
## コンパイルオプション

ここは`./configure`コンパイル設定の追加パラメータです。特定の機能を有効にするために使用されます。
### General Parameters
#### --enable-openssl

`SSL`サポートを有効にする

> OSで提供されている`libssl.so`ダイナミックリンクライブラリを使用します。
#### --with-openssl-dir

`SSL`サポートを有効にし、`openssl`ライブラリのパスを指定します。 パス引数を指定する必要があります。 例: `--with-openssl-dir=/opt/openssl/`
#### --enable-http2

**HTTP2**のサポートを有効にする

> **nghttp2**ライブラリに依存します。バージョン**V4.3.0**以降では依存関係のインストールは不要ですが、内蔵されています。ただし、**http2**のサポートを有効にするにはこのコンパイルオプションを追加する必要があります。**Swoole5**ではデフォルトでこのオプションが有効になっています。
#### --enable-swoole-json

[swoole_substr_json_decode](/functions?id=swoole_substr_json_decode)のサポートを有効にします。`Swoole5`からこのパラメータがデフォルトで有効になります

> 依存関係:`json`拡張、`v4.5.7`バージョンで使用可能
#### --enable-swoole-curl

[SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl)のサポートを有効にする

> `v4.6.0`から利用可能です。コンパイル時に`curl/curl.h: No such file or directory`のエラーが発生した場合は、[installation issues](/question/install?id=libcurl)を参照してください
#### --enable-cares

`c-ares` のサポートを有効にする

> `c-ares` ライブラリに依存しており、`v4.7.0` バージョンが利用可能です。コンパイル中に`ares.h: No such file or directory`というエラーが表示された場合は、[インストールの問題](/question/install?id=libcares)を参照してください。
#### --with-jemalloc-dir

`jemalloc` サポートを有効にします
#### --enable-brotli

`libbrotli` 圧縮サポートを有効にします。
#### --with-brotli-dir

`libbrotli`の圧縮サポートを有効にし、`libbrotli`ライブラリのパスを指定します。 パス引数を指定する必要があります。例：`--with-brotli-dir=/opt/brotli/`
#### --enable-swoole-pgsql

`PostgreSQL`データベースのコルーチン化を有効にします。

> `Swoole5.0`以前では`PostgreSQL`をコルーチン化するためにコルーチンクライアントを使用していましたが、`Swoole5.1`以降では、コルーチンクライアントを使用して`PostgreSQL`をコルーチン化するだけでなく、ネイティブの`pdo_pgsql`を使用して`PostgreSQL`をコルーチン化することもできます。
#### --with-swoole-odbc

`pdo_odbc`のコルーチン化を有効にして、すべての`odbc`インターフェースをサポートするデータベースがコルーチン化されるようにします。

>`v5.1.0`以降利用可能で、unixodbc-devが依存関係に必要です。

設定の例

```
with-swoole-odbc="unixODBC,/usr"
```
#### --with-swoole-oracle

`pdo_oci`のコルーチンを有効にし、このパラメータを有効にすると`oracle`データベースの挿入、削除、更新、検索すべてがコルーチン操作をトリガーします。

>`v5.1.0`バージョン以降で利用可能
#### --enable-swoole-sqlite

`pdo_sqlite`のコルーチン化を有効にします。このオプションを有効にすると、`sqlite`データベースの追加、削除、更新、検索はすべてコルーチン操作を発生させます。

> `v5.1.0`以降で利用可能
### 特別なパラメータ

!> **履歴がない場合はお勧めしません**
#### --enable-mysqlnd

`mysqlnd`サポートを有効にし、`Coroutine\MySQL::escapse`メソッドを有効にします。このオプションを有効にすると、`PHP`に`mysqlnd`モジュールが必要です。 それ以外の場合、`Swoole`が実行できなくなります。

> `mysqlnd`拡張に依存します。
#### --enable-sockets

PHPの`sockets`リソースをサポートするために有効にします。このパラメータを有効にすると、[Swoole\Event::add](/event?id=add) を使用して`sockets`拡張で作成された接続を`Swoole`の[イベントループ](/learn?id=什么是eventloop)に追加できます。
`Server`および`Client`の[getSocket()](/server/methods?id=getsocket)メソッドも、このコンパイルパラメータに依存しています。

> `sockets`拡張に依存しており、バージョン`v4.3.2`以降では、このパラメータの役割が弱まっています。なぜなら、Swooleに組み込まれている[Coroutine\Socket](/coroutine_client/socket)がほとんどの機能を代替できるからです。
### Debug Parameters

!> **Do not enable in production environment**

### デバッグパラメータ

!> **本番環境で有効にしないでください**
#### --enable-debug

デバッグモードを有効にします。`Swoole`をコンパイルする際に`gdb`を使用してトレースする必要があります。
#### --enable-debug-log

カーネルのDEBUGログを有効にします。**(Swooleバージョン >= 4.2.0)**
#### --enable-trace-log

追跡ログを有効にします。このオプションを有効にすると、Swooleはさまざまなデバッグログの詳細を出力します。カーネル開発時にのみ使用してください。
#### --enable-swoole-coro-time

协程の実行時間を計測するために有効にする。このオプションを有効にすると、Swoole\Coroutine::getExecuteTime()を使用して単位時間の実行時間を計算することができます。I/Oの待機時間は含まれません。
### PHPコンパイルパラメータ

I noticed the article is for PHP, so I've translated the title as such. Let me know if you need any more help!
#### --enable-swoole

Swoole拡張機能をPHPに静的にコンパイルするには、次の手順に従って`--enable-swoole`オプションを使用できます。

```shell
cp -r /home/swoole-src /home/php-src/ext
cd /home/php-src
./buildconf --force
./configure --help | grep swoole
```

!> このオプションは、SwooleではなくPHPをコンパイルする際に使用されます。
## よくある質問

* [Swooleのインストールに関する質問](/question/install)
