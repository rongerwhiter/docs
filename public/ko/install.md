# 문제 해결

이 코드 블록에는 특정한 문제에 대한 해결 방법이 포함되어 있습니다.
## Swoole 버전 업그레이드

pecl을 사용하여 설치 및 업그레이드할 수 있습니다.

```shell
pecl upgrade swoole
```

또는 새 버전을 직접 github/gitee/pecl에서 다운로드하고 다시 설치하여 컴파일할 수 있습니다.

* Swoole 버전을 업데이트하려면 이전 버전의 Swoole을 제거하거나 삭제할 필요가 없습니다. 새로운 버전이 이전 버전을 덮어씁니다.
* Swoole을 컴파일하고 설치한 후에는 추가적인 파일이 없으며, swoole.so 파일 하나만 있습니다. 다른 기계에서 컴파일된 바이너리 버전인 경우, 그냥 서로 swoole.so를 덮어쓰면 버전 이전이 가능합니다.
* git clone으로 코드를 가져온 경우, git pull을 실행해서 코드를 업데이트한 후에 `phpize`, `./configure`, `make clean`, `make install`을 다시 실행해야 합니다.
* 해당하는 docker를 사용하여 Swoole 버전을 업그레이드할 수도 있습니다.
```php
<?php
phpinfo();
```

phpinfo에는 있지만 php -m에는 없는 확장 프로그램이 있습니다.

먼저 CLI 모드에서 확인해보세요. 명령줄에 `php --ri swoole`을 입력하세요.

만약 Swoole 확장 프로그램 정보가 표시된다면 설치가 성공한 것입니다!

**99.999%의 사람들은 여기까지 성공하면 swoole을 바로 사용할 수 있습니다!**

`swoole`이 php -m 또는 phpinfo 웹 출력에 있는지 여부는 신경 쓸 필요가 없습니다.

왜냐하면 Swoole은 CLI 모드에서 실행되며, 전통적인 fpm 모드에서는 기능이 매우 제한적입니다.

fpm 모드에서는 모든 비동기/코루틴 기능 등 **사용할 수 없습니다.** 99.999%의 사람들이 fpm 모드에서 원하는 것을 얻지 못하는데, fpm 모드에서 확장 프로그램 정보가 표시되지 않는 것에 대해 고민합니다.

**Swoole의 실행 모드를 정말로 이해했는지 먼저 확인한 후 설치 정보 문제를 계속 따져보세요!** 
```
### 이유

Swoole을 컴파일하여 설치한 후에는 `php-fpm/apache`의 `phpinfo` 페이지에서 볼 수 있지만 명령줄의 `php -m`에서는 볼 수 없는데, 이는 `cli/php-fpm/apache`가 서로 다른 php.ini 설정을 사용하기 때문일 수 있습니다.
### 해결 방법

1. php.ini 파일의 위치 확인

`cli` 명령줄에서 `php -i | grep php.ini` 또는 `php --ini`를 실행하여 php.ini 파일의 절대 경로를 찾습니다.

`php-fpm/apache`의 경우 `phpinfo` 페이지를 확인하여 php.ini 파일의 절대 경로를 찾습니다.

2. 해당 php.ini 파일에 `extension=swoole.so`이 있는지 확인합니다.

```shell
cat /path/to/php.ini | grep swoole.so
```
## pcre.h: No such file or directory

Swoole extension을 컴파일하는 동안 다음과 같은 오류가 발생하였습니다.

```bash
fatal error: pcre.h: No such file or directory
```

원인은 pcre이 누락되었습니다. libpcre을 설치해야합니다.
### 우분투/데비안

```shell
sudo apt-get install libpcre3 libpcre3-dev
```
### centos/redhat

```shell
sudo yum install pcre-devel
```
### 다른 Linux

[PCRE 공식 웹 사이트](http://www.pcre.org/)에서 소스 코드를 다운로드하여 `pcre` 라이브러리를 컴파일 및 설치하십시오.

`PCRE` 라이브러리를 설치한 후에는 `swoole`을 다시 컴파일하여 설치해야하며, 그런 다음 `php --ri swoole`을 사용하여 `swoole` 확장 프로그램 관련 정보에서 `pcre => enabled`가 있는지 확인하십시오.

## '__builtin_saddl_overflow' was not declared in this scope

 ```
error: '__builtin_saddl_overflow' was not declared in this scope
  if (UNEXPECTED(__builtin_saddl_overflow(Z_LVAL_P(op1), 1, &lresult))) {

note: in definition of macro 'UNEXPECTED'
 # define UNEXPECTED(condition) __builtin_expect(!!(condition), 0)
```

이것은 알려진 문제입니다. 문제는 CentOS의 기본 gcc에서 필요한 정의가 누락되어 있기 때문입니다. 심지어 gcc를 업그레이드한 후에도 PECL은 이전 컴파일러를 찾을 수 있습니다.

드라이버를 설치하려면 먼저 다음과 같이 devtoolset 모음을 설치하여 gcc를 업그레이드해야 합니다:

```shell
sudo yum install centos-release-scl
sudo yum install devtoolset-7
scl enable devtoolset-7 bash
```
## 치명적인 오류: 'openssl/ssl.h' 파일을 찾을 수 없음

`[--with-openssl-dir](/environment?id=通用参数)` 매개변수를 추가하여 openssl 라이브러리의 경로를 지정하십시오.

!> [pecl](/environment?id=pecl)을 사용하여 Swoole을 설치할 때 openssl을 활성화하려면 `--[with-openssl-dir](/environment?id=通用参数)` 매개변수를 추가할 수 있습니다. 예: `enable openssl support? [no] : yes --with-openssl-dir=/opt/openssl/`
## make 또는 make install이 실행되지 않거나 컴파일 오류가 발생하는 경우

알림: PHP 메시지: PHP 경고: PHP 시작: swoole: 모듈을 초기화할 수 없습니다
모듈이 module API=20090626로 컴파일되었습니다
PHP는 module API=20121212로 컴파일되었습니다
이러한 옵션들은 일치해야합니다
알 수 없음에서 줄 0에

PHP 버전과 `phpize` 및 `php-config`를 사용하여 컴파일할 때의 버전이 일치하지 않습니다. 컴파일을 위해 절대 경로를 사용하고 PHP를 실행하기 위해 절대 경로를 사용해야 합니다.

```shell
/usr/local/php-5.4.17/bin/phpize
./configure --with-php-config=/usr/local/php-5.4.17/bin/php-config

/usr/local/php-5.4.17/bin/php server.php
```
## xdebug 설치

```shell
git clone git@github.com:swoole/sdebug.git -b sdebug_2_9 --depth=1

cd sdebug

phpize
./configure
make clean
make
make install

# 만약 phpize, php-config 등의 구성 파일이 기본 값인 경우 직접 실행할 수 있습니다
./rebuild.sh
```

php.ini 파일을 수정하여 확장을 로드합니다. 다음 정보를 추가하세요.

```ini
zend_extension=xdebug.so

xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_port=8000
xdebug.idekey="xdebug"
```

로드가 성공적으로 이루어졌는지 확인하세요.

```shell
php --ri sdebug
```
## configure: error: C preprocessor "/lib/cpp" fails sanity check

If you encounter an error like the following during installation:

```shell
configure: error: C preprocessor "/lib/cpp" fails sanity check
```

It means that essential dependencies are missing. You can install them using the following commands:

```shell
yum install glibc-headers
yum install gcc-c++
```
```shell
brew update
brew upgrade
brew reinstall php
brew reinstall openssl
CC=/usr/bin/clang ./configure --prefix=/usr/local/Cellar/php/7.4.12_1 --with-openssl=/usr/local/Cellar/openssl@1.1/1.1.1h --enable-sockets
make && make install
```

```c
# define ZEND_USE_ASM_ARITHMETIC 1
#else
```
# define ZEND_USE_ASM_ARITHMETIC 0
#endif
`--enable-swoole-curl` 옵션을 활성화한 후 Swoole 확장을 컴파일하면 다음과 같은 오류가 발생합니다.

```bash
fatal error: curl/curl.h: No such file or directory
```

이는 curl 종속성이 부족하여 libcurl을 설치해야 합니다.
### 우분투/데비안

```shell
sudo apt-get install libcurl4-openssl-dev
```
### centos/redhat

```shell
sudo yum install libcurl-devel
```````

루트없이 yum이나 dnf로 패키지를 설치할 수 있으니 관리자 권한이 필요할 수 있습니다. 번거로우시겠지만 sudo를 사용해주세요!
### 알파인

```shell
apk add curl-dev
```
## fatal error: ares.h: No such file or directory :id=libcares

`--enable-cares` 옵션을 열면 Swoole 확장을 컴파일할 때 다음과 같은 오류가 발생할 수 있습니다.

```bash
fatal error: ares.h: No such file or directory
```

이것은 c-ares 종속성이 부족하기 때문에 발생하는 오류이며, libcares를 설치해야 합니다.
### 우분투/데비안

```shell
sudo apt-get install libc-ares-dev
```
### centos/redhat

```shell
sudo yum install c-ares-devel
```````
`c-ares-devel` 패키지를 설치하려면 위 명령어를 실행하십시오.
### 알파인

```shell
apk add c-ares-dev
```
### MacOs

```shell
brew install c-ares
```
