# 다른 지식

## DNS 해석 시간 초과 및 재시도 설정

네트워크 프로그래밍에서 도메인 이름 해석을 구현하기 위해 `gethostbyname`과 `getaddrinfo`를 자주 사용합니다. 이 두 개의 `C` 함수는 타임아웃 매개변수를 제공하지 않습니다. 실제로 `/etc/resolv.conf`를 수정하여 타임아웃 및 재시도 논리를 설정할 수 있습니다.

!> `man resolv.conf` 문서를 참조하세요.

### 다중 NameServer <!-- {docsify-ignore} -->

```
nameserver 192.168.1.3
nameserver 192.168.1.5
option rotate
```

여러 `nameserver`를 구성할 수 있으며, 첫 번째 `nameserver`에서 쿼리가 실패하면 자동으로 두 번째 `nameserver`로 전환하여 재시도합니다.

`option rotate`로 구성된 것은 `nameserver` 부하 분산을 위해 라운드 로빈 모드를 사용하는 것입니다.

### 타임아웃 제어 <!-- {docsify-ignore} -->

```
option timeout:1 attempts:2
```

* `timeout` : `UDP` 수신 타임아웃 시간을 제어하며, 단위는 초이며 기본값은 `5`초입니다.
* `attempts` : 시도 횟수를 제어하며, `2`로 설정하면 최대 `2`번 시도하며, 기본값은 `5`번입니다.

예를 들어 `2`개의 `nameserver`가 있고, `attempts`가 `2`이며 타임아웃이 `1`인 경우, 모든 `DNS` 서버의 응답이 없는 경우 최대 대기 시간은 `4`초 (`2x2x1`)가 됩니다.

### 호출 추적 <!-- {docsify-ignore} -->

[strace](/other/tools?id=strace)를 사용하여 확인할 수 있습니다.

`nameserver`를 존재하지 않는 두 개의 `IP`로 설정하고 `PHP` 코드에서 `var_dump(gethostbyname('www.baidu.com'));`를 사용하여 도메인을 해석합니다.

```
위의 내용 확인 가능
```

여기서 총 `4`번 재시도되었음을 볼 수 있으며, `poll` 호출이 `1000ms` (`1초`)로 설정되어 있음을 알 수 있습니다.
