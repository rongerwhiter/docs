# 툴 사용
## yasd

[yasd](https://github.com/swoole/yasd)

단계별 디버깅 도구, `Swoole` 코루틴 환경에서 사용 가능하며, `IDE` 및 콘솔 디버깅 모드를 지원합니다.
## tcpdump

네트워크 통신 프로그램을 디버깅할 때 tcpdump는 필수 도구입니다. tcpdump는 매우 강력하며 네트워크 통신의 모든 세부 정보를 볼 수 있습니다. TCP의 경우, 3-way handshake, PUSH/ACK 데이터 전송, 4-way close handshake 등의 모든 세부 정보를 확인할 수 있습니다. 각 네트워크 패킷의 바이트 수, 시간 등도 포함됩니다.
### 사용법

가장 간단한 사용 예시:

```shell
sudo tcpdump -i any tcp port 9501
```
* -i 매개변수는 네트워크 카드를 지정합니다. 여기서 any는 모든 네트워크 카드를 의미합니다.
* tcp는 TCP 프로토콜만 감청하도록 지정합니다.
* port는 감청할 포트를 지정합니다.

!> tcpdump는 root 권한이 필요하며, 통신 데이터 내용을 보려면 `-Xnlps0` 옵션을 추가해야 합니다. 더 많은 옵션은 인터넷의 문서를 참고해 주세요.
### 실행 결과

```
13:29:07.788802 IP localhost.42333 > localhost.9501: Flags [S], seq 828582357, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 0,nop,wscale 7], length 0
13:29:07.788815 IP localhost.9501 > localhost.42333: Flags [S.], seq 1242884615, ack 828582358, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 2207513,nop,wscale 7], length 0
13:29:07.788830 IP localhost.42333 > localhost.9501: Flags [.], ack 1, win 342, options [nop,nop,TS val 2207513 ecr 2207513], length 0
13:29:10.298686 IP localhost.42333 > localhost.9501: Flags [P.], seq 1:5, ack 1, win 342, options [nop,nop,TS val 2208141 ecr 2207513], length 4
13:29:10.298708 IP localhost.9501 > localhost.42333: Flags [.], ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:10.298795 IP localhost.9501 > localhost.42333: Flags [P.], seq 1:13, ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 12
13:29:10.298803 IP localhost.42333 > localhost.9501: Flags [.], ack 13, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:11.563361 IP localhost.42333 > localhost.9501: Flags [F.], seq 5, ack 13, win 342, options [nop,nop,TS val 2208457 ecr 2208141], length 0
13:29:11.563450 IP localhost.9501 > localhost.42333: Flags [F.], seq 13, ack 6, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
13:29:11.563473 IP localhost.42333 > localhost.9501: Flags [.], ack 14, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
```
* `13:29:11.563473` 시간은 마이크로초까지 정확합니다.
*  localhost.42333 > localhost.9501은 통신 방향을 나타내는데, 42333은 클라이언트이고 9501은 서버입니다.
* [S]는 SYN 요청을 나타냅니다.
* [.]는 ACK 확인 패킷을 나타내며, (client)SYN->(server)SYN->(client)ACK는 3-way 핸드셰이크 과정입니다.
* [P]는 데이터 푸시를 의미하는데, 서버에서 클라이언트로의 푸시일 수도 있고, 클라이언트에서 서버로의 푸시일 수도 있습니다.
* [F]는 FIN 패킷을 의미하며, 연결을 닫는 작업이며, 클라이언트/서버 모두가 시작할 수 있습니다.
* [R]은 RST 패킷을 의미하며, F 패킷과 동일한 역할을 하지만 RST는 연결이 닫힐 때 아직 처리되지 않은 데이터가 있음을 나타냅니다. 연결을 강제로 끊는 것으로 이해할 수 있습니다.
* win 342는 윈도우 크기를 나타냅니다.
* length 12는 데이터 패킷의 크기를 나타냅니다.
## strace

strace는 시스템 콜의 실행 상황을 추적할 수 있으며, 프로그램에 문제가 발생한 경우 strace를 사용하여 문제를 분석하고 추적할 수 있습니다.

!> FreeBSD/MacOS에서는 truss를 사용할 수 있습니다.
### 사용법

```shell
strace -o /tmp/strace.log -f -p $PID
```

* -f은 여러 스레드 및 프로세스를 추적하며, 이 옵션을 사용하지 않으면 자식 프로세스 및 스레드의 실행 상황을 포착할 수 없습니다.
* -o는 결과를 파일에 출력한다는 것을 의미합니다.
* -p $PID는 추적할 프로세스 ID를 지정합니다. `ps aux`를 통해 확인할 수 있습니다.
* -tt는 시스템 호출이 발생한 시간을 마이크로초 단위로 출력합니다.
* -s는 출력할 문자열의 길이를 제한합니다. 예를 들어 recvfrom 시스템 호출로 수신된 데이터는 기본적으로 32바이트만 인쇄됩니다.
* -c는 각 시스템 호출의 소요 시간을 실시간으로 통계합니다.
* -T는 각 시스템 호출의 소요 시간을 출력합니다.
## gdb

GDB는 GNU 오픈 소스 조직이 배포한 강력한 UNIX 프로그램 디버깅 도구로, C/C++ 프로그램을 디버깅하는 데 사용할 수 있습니다. PHP 및 Swoole은 C 언어로 개발되었으므로 PHP+Swoole 프로그램을 디버깅하는 데 GDB를 사용할 수 있습니다.

gdb 디버깅은 명령줄 대화형 방식이며 일반적인 명령어를 숙지해야 합니다.
### 사용 방법

```shell
gdb -p 프로세스ID
gdb php
gdb php core
```

gdb를 사용하는 세 가지 방법이 있습니다.

* 실행 중인 PHP 프로세스를 추적하는 경우, `gdb -p 프로세스ID`를 사용합니다.
* PHP 프로그램을 실행하고 디버그하는 경우, `gdb php`를 사용하고 `run server.php`를 사용하여 디버깅합니다.
* PHP 프로그램이 코어 덤프를 발생시킨 후 코어 메모리 이미지를 로드하여 디버깅하는 경우에는 `gdb php core`를 사용합니다.

!> 만약 PATH 환경 변수에 php가 없다면, gdb를 사용할 때 절대 경로를 지정해야 합니다. 예를 들어, `/usr/local/bin/php` 경로를 사용합니다.
### 자주 사용되는 명령어

* `p`：print，C 변수의 값을 출력합니다.
* `c`：continue，중단된 프로그램을 계속 실행합니다.
* `b`：breakpoint，중단점을 설정합니다. 함수 이름으로 설정할 수 있으며, `b zif_php_function`과 같이 설정할 수 있습니다. 또는 소스 코드의 특정 라인을 지정할 수도 있습니다. 예를 들어 `b src/networker/Server.c:1000`과 같이 지정할 수 있습니다.
* `t`：thread，스레드를 전환합니다. 프로세스에 여러 개의 스레드가 있는 경우, t 명령어를 사용하여 다른 스레드로 전환할 수 있습니다.
* `ctrl + c`：현재 실행 중인 프로그램을 중지시킵니다. c 명령어와 함께 사용됩니다.
* `n`：next，다음 라인을 실행합니다. 한 단계씩 디버깅합니다.
* `info threads`：현재 실행 중인 모든 스레드를 확인합니다.
* `l`：list，소스 코드를 보여줍니다. `l 함수명` 또는 `l 라인번호`를 사용할 수 있습니다.
* `bt`：backtrace，실행 중인 함수 호출 스택을 확인합니다.
* `finish`：현재 함수를 끝냅니다.
* `f`：frame，bt와 함께 사용되며, 함수 호출 스택의 특정 단계로 전환할 수 있습니다.
* `r`：run，프로그램을 실행합니다.
### zbacktrace

zbacktrace는 PHP 소스 코드 패키지에서 제공하는 사용자 지정 gdb 명령어로, bt 명령어와 유사한 기능을 제공하지만 bt와 다른 점은 zbacktrace는 PHP 함수 호출 스택을 볼 수 있고 C 함수를 볼 수 없다는 것입니다.

php-src를 다운로드하고 압축 해제 한 후 루트 디렉토리에서 `.gdbinit` 파일을 찾아서 gdb 쉘에서 다음과 같이 입력하세요.

```shell
source .gdbinit
zbacktrace
```
`.gdbinit` 파일에는 다른 추가 지시문들도 제공되어 있어 소스 코드를 살펴보고 자세한 정보를 알 수 있습니다.
#### gdb와 zbacktrace를 사용하여 무한 루프 문제 추적

```shell
gdb -p 프로세스ID
```

* 무한 루프가 발생한 Worker 프로세스ID를 찾기 위해 `ps aux` 도구를 사용합니다.
* `gdb -p`를 사용하여 지정된 프로세스를 추적합니다.
* `ctrl + c`를 반복해서 호출한 후, `zbacktrace`, `c`를 사용하여 프로그램이 루프에 돌고 있는 PHP 코드 부분을 확인합니다.
* 해당하는 PHP 코드를 찾아 문제를 해결합니다.
## lsof

Linux 플랫폼은 `lsof` 도구를 제공하여 특정 프로세스가 열어 놓은 파일 핸들을 볼 수 있습니다. 이 도구는 swoole 작업 프로세스가 열어 놓은 모든 소켓, 파일, 리소스를 추적하는 데 사용할 수 있습니다.
### 사용 방법

```shell
lsof -p [프로세스 ID]
```
```shell
lsof -p 26821
lsof: 경고: tracefs 파일 시스템 /sys/kernel/debug/tracing을 stat()할 수 없습니다
      출력 정보가 불완전할 수 있습니다.
COMMAND   PID USER   FD      TYPE             DEVICE SIZE/OFF    NODE NAME
php     26821  htf  cwd       DIR                8,4     4096 5375979 /home/htf/workspace/swoole/examples
php     26821  htf  rtd       DIR                8,4     4096       2 /
php     26821  htf  txt       REG                8,4 24192400 6160666 /opt/php/php-5.6/bin/php
php     26821  htf  DEL       REG                0,5          7204965 /dev/zero
php     26821  htf  DEL       REG                0,5          7204960 /dev/zero
php     26821  htf  DEL       REG                0,5          7204958 /dev/zero
php     26821  htf  DEL       REG                0,5          7204957 /dev/zero
php     26821  htf  DEL       REG                0,5          7204945 /dev/zero
php     26821  htf  mem       REG                8,4   761912 6160770 /opt/php/php-5.6/lib/php/extensions/debug-zts-20131226/gd.so
php     26821  htf  mem       REG                8,4  2769230 2757968 /usr/local/lib/libcrypto.so.1.1
php     26821  htf  mem       REG                8,4   162632 6322346 /lib/x86_64-linux-gnu/ld-2.23.so
php     26821  htf  DEL       REG                0,5          7204959 /dev/zero
php     26821  htf    0u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    1u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    2u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    3r      CHR                1,9      0t0      11 /dev/urandom
php     26821  htf    4u     IPv4            7204948      0t0     TCP *:9501 (LISTEN)
php     26821  htf    5u     IPv4            7204949      0t0     UDP *:9502 
php     26821  htf    6u     IPv6            7204950      0t0     TCP *:9503 (LISTEN)
php     26821  htf    7u     IPv6            7204951      0t0     UDP *:9504 
php     26821  htf    8u     IPv4            7204952      0t0     TCP localhost:8000 (LISTEN)
php     26821  htf    9u     unix 0x0000000000000000      0t0 7204953 type=DGRAM
php     26821  htf   10u     unix 0x0000000000000000      0t0 7204954 type=DGRAM
php     26821  htf   11u     unix 0x0000000000000000      0t0 7204955 type=DGRAM
php     26821  htf   12u     unix 0x0000000000000000      0t0 7204956 type=DGRAM
php     26821  htf   13u  a_inode               0,11        0    9043 [eventfd]
php     26821  htf   14u     unix 0x0000000000000000      0t0 7204961 type=DGRAM
php     26821  htf   15u     unix 0x0000000000000000      0t0 7204962 type=DGRAM
php     26821  htf   16u     unix 0x0000000000000000      0t0 7204963 type=DGRAM
php     26821  htf   17u     unix 0x0000000000000000      0t0 7204964 type=DGRAM
php     26821  htf   18u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   19u  a_inode               0,11        0    9043 [signalfd]
php     26821  htf   20u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   22u     IPv4            7452776      0t0     TCP localhost:9501->localhost:59056 (ESTABLISHED)
```

* so 파일은 불러온 프로세스의 동적 연결 라이브러리입니다.
* IPv4/IPv6 TCP (LISTEN)는 서버가 듣고 있는 포트입니다.
* UDP는 서버가 듣고 있는 UDP 포트입니다.
* UNIX type=DGRAM은 프로세스에서 생성한 [UNIX 소켓](/learn?id=什么是IPC)입니다.
* IPv4 (ESTABLISHED)는 서버에 연결된 TCP 클라이언트를 나타냅니다. 클라이언트의 IP, PORT 및 상태(ESTABLISHED)를 포함합니다.
* 9u / 10u는 해당 파일 핸들의 fd 값(파일 기술자)을 나타냅니다.
* 자세한 정보는 lsof 매뉴얼을 참조하세요.
## perf

`perf` 도구는 Linux 커널에서 제공하는 매우 강력한 동적 추적 도구이며, `perf top` 명령어는 실행 중인 프로그램의 성능 문제를 실시간으로 분석하는 데 사용됩니다. `callgrind`, `xdebug`, `xhprof` 등과는 달리, `perf`는 코드를 수정하여 프로파일 결과 파일을 내보낼 필요가 없습니다.
### 사용법

```shell
perf top -p [프로세스ID]
```
### 출력 결과

![perf top 출력 결과](../_images/other/perf.png)

perf 결과는 현재 프로세스 실행 중 각 C 함수의 실행 시간을 명확히 보여줍니다. 이를 통해 어떤 C 함수가 CPU 자원을 많이 사용하는지 이해할 수 있습니다.

만약 Zend VM에 익숙하다면, 특정 Zend 함수가 너무 많이 호출된 경우에는 프로그램에서 특정 함수를 대량으로 사용하여 CPU 자원을 과도하게 사용하고 있음을 나타낼 수 있습니다. 이에 대해 집중적으로 최적화를 수행할 수 있습니다.
