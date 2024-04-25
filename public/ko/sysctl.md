# 내 커널 매개변수 조정

## ulimit 설정

`ulimit -n`을 100000 또는 그 이상으로 조정해야 합니다. 명령 줄에서 `ulimit -n 100000`를 실행하여 수정할 수 있습니다. 수정이 안 된다면 `/etc/security/limits.conf`을 설정해야 합니다. 다음 내용을 추가하세요.

```
* soft nofile 262140
* hard nofile 262140
root soft nofile 262140
root hard nofile 262140
* soft core unlimited
* hard core unlimited
root soft core unlimited
root hard core unlimited
```

`limits.conf` 파일을 수정한 후 시스템을 다시 시작하여 변경 사항을 적용해야 합니다.

## 커널 설정

`Linux` 운영 체제에서는 내 커널 매개변수를 다음 세 가지 방법으로 조정할 수 있습니다.

- `/etc/sysctl.conf` 파일을 수정하여 설정 옵션을 추가하고 `sysctl -p`를 호출하여 새 설정을 로드합니다.
- `sysctl` 명령을 사용하여 일시적으로 수정합니다. 예: `sysctl -w net.ipv4.tcp_mem="379008 505344 758016"`
- `/proc/sys/` 디렉토리의 파일을 직접 수정합니다. 예: `echo "379008 505344 758016" > /proc/sys/net/ipv4/tcp_mem`

> 첫 번째 방법은 운영 체제 재부팅 후 자동으로 적용되지만, 두 번째와 세 번째 방법은 재부팅 후에 적용되지 않습니다.

### net.unix.max_dgram_qlen = 100

swoole은 프로세스간 통신을 위해 유닉스 소켓 dgram을 사용합니다. 요청 양이 많은 경우 이 매개변수를 조정해야 합니다. 시스템 기본값은 10이며, 100이나 더 큰 값으로 설정하거나 worker 프로세스 수를 늘려야 할 수도 있습니다.

### net.core.wmem_max

소켓 버퍼 크기를 증가시키기 위해 이 매개변수를 수정합니다.

```
net.ipv4.tcp_mem  =   379008       505344  758016
net.ipv4.tcp_wmem = 4096        16384   4194304
net.ipv4.tcp_rmem = 4096          87380   4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

### net.ipv4.tcp_tw_reuse

소켓 재사용 여부를 나타냅니다. 서버 재시작시 청취 포트를 빠르게 재사용할 수 있게 해줍니다.

### net.ipv4.tcp_tw_recycle

빠른 소켓 재활용을 위한 매개변수입니다. 단절형 서버에서는 이 매개변수를 활성화해야 합니다. 이 매개변수는 TCP 연결 중 TIME-WAIT 소켓의 빠른 재수집을 나타냅니다. Linux 시스템에서 기본값은 0으로 별도의 설정이 필요합니다. 이 매개변수를 활성화하면 NAT 사용자 연결이 불안정해질 수 있으므로 신중한 테스트 후에 활성화해야 합니다.

## 메시지 큐 설정

프로세스간 통신 방법으로 메시지 큐를 사용할 때 이 내 커널 매개변수를 조정해야 합니다.

- kernel.msgmnb = 4203520 (메시지 큐의 최대 바이트 수)
- kernel.msgmni = 64 (생성할 수 있는 최대 메시지 큐 수)
- kernel.msgmax = 8192 (메시지 큐 단일 데이터의 최대 길이)

## FreeBSD/MacOS

- sysctl -w net.local.dgram.maxdgram=8192
- sysctl -w net.local.dgram.recvspace=200000
  Unix 소켓의 버퍼 영역 크기를 조정합니다.

## CoreDump 활성화

내 커널 매개변수를 설정합니다.

```
kernel.core_pattern = /data/core_files/core-%e-%p-%t
```

현재 `coredump` 파일 제한을 확인하려면 `ulimit -c` 명령을 사용하세요.

```shell
ulimit -c
```

0이라면 `/etc/security/limits.conf`을 수정하여 제한을 설정해야 합니다.

> Core-dump를 활성화하면 프로그램이 예외를 발생시킬 때 프로세스가 파일로 내보내집니다. 프로그램 문제를 조사하는 데 큰 도움이 됩니다.

## 기타 중요 설정

- net.ipv4.tcp_syncookies=1
- net.ipv4.tcp_max_syn_backlog=81920
- net.ipv4.tcp_synack_retries=3
- net.ipv4.tcp_syn_retries=3
- net.ipv4.tcp_fin_timeout = 30
- net.ipv4.tcp_keepalive_time = 300
- net.ipv4.tcp_tw_reuse = 1
- net.ipv4.tcp_tw_recycle = 1
- net.ipv4.ip_local_port_range = 20000 65000
- net.ipv4.tcp_max_tw_buckets = 200000
- net.ipv4.route.max_size = 5242880

## 설정이 올바르게 적용되었는지 확인

예를 들어 `net.unix.max_dgram_qlen = 100`을 수정한 후 다음과 같이 확인할 수 있습니다.

```shell
cat /proc/sys/net/unix/max_dgram_qlen
```

수정이 제대로 되었다면 여기에 새로운 값이 나와야 합니다.
