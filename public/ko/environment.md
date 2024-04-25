# Swoole 설치

`Swoole` 확장은 `PHP`의 표준 확장에 따라 구축됩니다. `phpize`를 사용하여 컴파일 검사 스크립트를 생성하고, `./configure`를 사용하여 컴파일 구성 검사를 하며, `make`를 사용하여 컴파일을 수행하고, `make install`로 설치합니다.

* 특별한 요구사항이 없는 경우, 반드시 최신 버전의 [Swoole](https://github.com/swoole/swoole-src/releases/)을 컴파일하여 설치하십시오.
* 현재 사용자가 `root`가 아닌 경우, `PHP` 설치 디렉토리에 쓰기 권한이 없을 수 있으므로 설치 시 `sudo` 또는 `su`가 필요할 수 있습니다.
* `git` 브랜치에서 직접 `git pull`로 코드를 업데이트하는 경우, 다시 컴파일하기 전에 반드시 `make clean`을 실행해야 합니다.
* `Linux`(2.3.32 이상의 커널) 및 `FreeBSD`, `MacOS` 세 가지 운영 체제만 지원됩니다.
* 낮은 버전의 Linux 시스템(예: `CentOS 6`)은 `레드햇`이 제공하는 `devtools`를 사용하여 컴파일할 수 있습니다. [참고 문서](https://blog.csdn.net/ppdouble/article/details/52894271) 참조.
* `Windows` 플랫폼에서는 `WSL(Windows Subsystem for Linux)` 또는 `CygWin`을 사용할 수 있습니다.
* 일부 확장은 `Swoole` 확장과 호환되지 않을 수 있으니 [확장 충돌](/getting_started/extension)을 참조하십시오.
## 설치 준비

설치하기 전에 시스템에 다음 소프트웨어가 설치되어 있는지 확인해야 합니다.

- `4.8` 버전은 `php-7.2` 이상의 버전이 필요합니다
- `5.0` 버전은 `php-8.0` 이상의 버전이 필요합니다
- `gcc-4.8` 이상의 버전
- `make`
- `autoconf`
## 빠른 설치

> 1. Swoole 소스 코드 다운로드

* [https://github.com/swoole/swoole-src/releases](https://github.com/swoole/swoole-src/releases)
* [https://pecl.php.net/package/swoole](https://pecl.php.net/package/swoole)
* [https://gitee.com/swoole/swoole/tags](https://gitee.com/swoole/swoole/tags)

> 2. 소스 코드로부터 컴파일 및 설치

소스 코드 패키지를 다운로드 한 후 터미널에서 소스 코드 디렉토리로 이동하여 아래 명령어를 실행하여 컴파일 및 설치합니다.

!> 우분투에 `phpize`가 설치되어 있지 않은 경우, `sudo apt-get install php-dev` 명령으로 `phpize`를 설치하세요.

```shell
cd swoole-src && \
phpize && \
./configure && \
sudo make && sudo make install
```

> 3. 확장 기능 활성화

시스템에 성공적으로 컴파일 및 설치 한 후, `php.ini`에 `extension=swoole.so` 한 줄을 추가하여 Swoole 확장을 활성화하세요.
## 고급 완전한 컴파일 예제

!> Swoole에 처음 접하는 개발자는 먼저 위의 간단한 컴파일을 시도한 다음 추가 요구에 따라 아래 예제의 컴파일 매개변수를 조정할 수 있습니다. [컴파일 매개변수 참고](/environment?id=编译选项)

다음 스크립트는 `master` 브랜치의 소스 코드를 다운로드하고 컴파일합니다. 모든 종속성이 설치되어 있어야 하며, 그렇지 않으면 다양한 종속성 오류가 발생할 수 있습니다.

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

> Note: PECL release is later than GitHub release

Swoole project has been included in the PHP official extension library. Besides manually downloading and compiling, you can also use the `pecl` command provided by PHP official to download and install in one click.

```shell
pecl install swoole
```

When installing Swoole through PECL, it will ask whether to enable certain features during the installation process. This can also be provided before running the installation, for example:

```shell
pecl install -D 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole

#or
pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole
```
## php.ini에 Swoole 추가

마지막으로 컴파일 및 설치가 성공했다면, `php.ini` 파일을 수정하여 다음을 추가해주세요.

```ini
extension=swoole.so
```

`swoole.so`가 올바르게 로드되었는지 확인하기 위해 `php -m`을 사용해보세요. 만약 로드되지 않았다면 `php.ini` 파일의 경로가 잘못된 것일 수 있습니다.  
`php --ini` 명령을 사용하여 `php.ini` 파일의 절대 경로를 찾을 수 있습니다. `Loaded Configuration File` 항목에서 로드된 php.ini 파일을 알 수 있으며, 값이 `none`이면 어떤 `php.ini` 파일도 로드되지 않았다는 것을 의미합니다. 이 경우 직접 파일을 생성해야 합니다.

!> `PHP` 버전 지원과 `PHP` 공식 유지 관리된 버전을 일치시키세요. [PHP 버전 지원 일정](http://php.net/supported-versions.php)을 참고하세요.
## 다른 플랫폼에서 컴파일

ARM 플랫폼 (라즈베리 파이 Raspberry PI)

- `GCC` 교차 컴파일 사용
- `Swoole`을 컴파일 할 때, `-O2` 컴파일 옵션을 제거하기 위해 `Makefile`을 수동으로 수정해야 합니다.

MIPS 플랫폼 (OpenWrt 라우터)

- GCC 교차 컴파일 사용

Windows WSL

`Windows 10` 시스템에는 `Linux` 하위 시스템 지원이 추가되어 `BashOnWindows` 환경에서도 `Swoole`을 사용할 수 있습니다. 설치 명령어

```shell
apt-get install php7.0 php7.0-curl php7.0-gd php7.0-gmp php7.0-json php7.0-mysql php7.0-opcache php7.0-readline php7.0-sqlite3 php7.0-tidy php7.0-xml  php7.0-bcmath php7.0-bz2 php7.0-intl php7.0-mbstring  php7.0-mcrypt php7.0-soap php7.0-xsl  php7.0-zip
pecl install swoole
echo 'extension=swoole.so' >> /etc/php/7.0/mods-available/swoole.ini
cd /etc/php/7.0/cli/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
cd /etc/php/7.0/fpm/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
```

!> `WSL` 환경에서는 `daemonize` 옵션을 비활성화해야 합니다.  
`17101` 미만의 `WSL`의 경우, `configure`를 통해 소스를 설치한 후 `config.h`를 수정하여 `HAVE_SIGNALFD`를 비활성화해야 합니다.
## Docker 공식 이미지

- GitHub: [https://github.com/swoole/docker-swoole](https://github.com/swoole/docker-swoole)
- dockerhub: [https://hub.docker.com/r/phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)
## 컴파일 옵션

여기에는 `./configure` 컴파일 구성을 위한 추가 매개변수가 있습니다. 특정 기능을 활성화하는 데 사용됩니다.
### 일반 매개변수
#### --enable-openssl

SSL 지원 활성화

> 운영 체제에서 제공하는 `libssl.so` 동적 연결 라이브러리 사용
#### --with-openssl-dir

`SSL` 지원을 활성화하고 `openssl` 라이브러리의 경로를 지정합니다. 경로 매개변수와 함께 사용하십시오. 예: `--with-openssl-dir=/opt/openssl/`
#### --enable-http2

`HTTP2` 지원 활성화

> `nghttp2` 라이브러리에 의존합니다. 버전 `V4.3.0` 이후에는 의존성을 설치할 필요가 없어졌으며 내장되었지만, 여전히 이 컴파일 매개변수를 추가하여 `http2` 지원을 활성화해야 합니다. `Swoole5`는 기본적으로 이 매개변수를 활성화합니다.
#### --enable-swoole-json

[swoole_substr_json_decode](/functions?id=swoole_substr_json_decode)에 대한 지원을 활성화하려면 `Swoole5` 이후에는 기본적으로이 매개 변수가 활성화되어 있습니다.

> `json` 확장이 필요하며, `v4.5.7` 버전에서 사용할 수 있습니다.
#### --enable-swoole-curl

[SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl) 지원을 활성화합니다.

> `v4.6.0` 이후 버전에서 사용 가능합니다. 만약 `curl/curl.h: No such file or directory` 라는 컴파일 오류가 발생하면 [설치 문제](/question/install?id=libcurl)를 확인해주세요.
#### --enable-cares

`c-ares` 지원 활성화

> `c-ares` 라이브러리에 의존하며 `v4.7.0` 버전에서 사용 가능합니다. 컴파일 중에 `ares.h: No such file or directory` 오류가 발생하면 [설치 문제](/question/install?id=libcares)를 확인하십시오.
#### --with-jemalloc-dir

`jemalloc` 지원 사용
#### --enable-brotli

`libbrotli` 압축 지원을 활성화합니다.
#### --with-brotli-dir

`libbrotli` 압축 지원을 활성화하고 `libbrotli` 라이브러리 경로를 지정하십시오. 경로 매개변수를 첨부해야 합니다. 예: `--with-brotli-dir=/opt/brotli/`
#### --enable-swoole-pgsql

`PostgreSQL` 데이터베이스를 코루틴으로 활성화합니다.

> `Swoole5.0` 이전에는 `PostgreSQL`을 코루틴으로 처리하기 위해 코루틴 클라이언트를 사용했습니다. `Swoole5.1` 이후에는 코루틴 클라이언트를 사용하는 것 외에도 네이티브 `pdo_pgsql`를 사용하여 `PostgreSQL`을 코루틴으로 처리할 수 있습니다.
#### --with-swoole-odbc

`pdo_odbc`을 코루틴으로 실행하기 위해 활성화하면 `odbc` 인터페이스를 지원하는 모든 데이터베이스를 코루틴으로 사용할 수 있습니다.

>`v5.1.0` 버전부터 사용 가능하며, unixodbc-dev에 대한 의존성이 필요합니다.

구성 예시

```
with-swoole-odbc="unixODBC,/usr"
```  
#### --with-swoole-oracle

`pdo_oci`의 코루틴을 활성화합니다. 이 매개변수를 활성화하면 `oracle` 데이터베이스의 삽입, 삭제, 업데이트 및 조회가 모두 코루틴 작업을 트리거합니다.

>`v5.1.0` 버전 이후 사용 가능
#### --enable-swoole-sqlite

`pdo_sqlite`의 코루틴 활성화합니다. 이 매개변수를 활성화하면 `sqlite` 데이터베이스의 삽입, 삭제, 업데이트, 조회 작업은 모두 코루틴 작업을 트리거합니다.

>`v5.1.0` 버전부터 사용 가능합니다.
### 특수 매개변수

!> **역사적 이유가 없는 한 활성화하지 않는 것이 좋습니다**
#### --enable-mysqlnd

`mysqlnd` 지원을 활성화하고 `Coroutine\MySQL::escape` 메서드를 활성화합니다. 이 매개 변수를 활성화하면 `PHP`에서 `mysqlnd` 모듈이 필요하며 그렇지 않으면 `Swoole`이 작동하지 않습니다.

> `mysqlnd` 확장이 필요합니다.
#### --enable-sockets

PHP의 `sockets` 리소스를 지원하도록 추가합니다. 이 매개변수를 활성화하면 [Swoole\Event::add](/event?id=add)를 사용하여 `sockets` 확장으로 생성된 연결을 `Swoole`의 [이벤트 루프](/learn?id=什게/eventloop)에 추가할 수 있습니다.
`Server`와 `Client`의 [getSocket()](/server/methods?id=getsocket) 메소드도 이 컴파일 매개변수에 의존합니다.

> `sockets` 확장에 의존하며, `v4.3.2` 버전 이후에는 이 매개변수의 역할이 약화되었습니다. 왜냐하면 Swoole에 내장된 [Coroutine\Socket](/coroutine_client/socket)이 대부분의 작업을 수행할 수 있기 때문입니다.
### 디버그 매개변수

!> **운영 환경에서는 활성화되어선 안 됩니다**
#### --enable-debug

디버그 모드를 활성화합니다. `Swoole`를 컴파일할 때 이 옵션을 추가하면 `gdb`를 사용하여 추적할 수 있습니다.
#### --enable-debug-log

커널 DEBUG 로그를 활성화합니다. **(Swoole 버전 >= 4.2.0)**
#### --enable-trace-log

추적 로그를 활성화하십시오. 이 옵션을 활성화하면 Swoole이 다양한 디버그 세부 정보를 출력하며, 내부 커널 개발 시에만 사용합니다.
#### --enable-swoole-coro-time

협동 실행 시간 측정을 활성화합니다. 이 옵션을 활성화하면 Swoole\Coroutine::getExecuteTime()을 사용하여 협동 실행 시간을 계산할 수 있습니다. I/O 대기 시간은 포함되지 않습니다.
### PHP 컴파일 매개변수
#### --enable-swoole

PHP에 Swoole 확장을 정적으로 컴파일하려면 다음 명령어를 실행하여 `--enable-swoole` 옵션을 사용할 수 있습니다.

```shell
cp -r /home/swoole-src /home/php-src/ext
cd /home/php-src
./buildconf --force
./configure --help | grep swoole
```

!> 이 옵션은 Swoole이 아니라 PHP를 컴파일할 때 사용됩니다.
## 자주 묻는 질문

* [Swoole 설치에 대한 자주 묻는 질문](/question/install)
