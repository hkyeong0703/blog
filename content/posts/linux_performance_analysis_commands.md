---
title: "리눅스 성능을 분석할 수 있는 명령어"
date: 2021-05-12T19:52:09+09:00
description: ""
tags: [
	"linux",
	"uptime",
	"dmesg",
	"vmstat",
	"mpstat",
	"pidstat",
	"iostat",
	"free",
	"sar",
	"top",
]
categories: []
disableComments: false
---

Netflix Tech Blog [60초 내에 Linux 성능 분석](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55) 에서 리눅스 명령어를 통해 성능을 분석할 수 있는 방법이 나온다.

번역과 함께 읽으며 내가 찾아본 내용들을 같이 추후 참고용으로 기록해두려고한다. 



아래 명령어들을 사용해 60초 안에 리눅스 환경에서 CLI를 이용해 어떤 것들을 확인할지 순서대로 알아보자.
일부 명령어는 sysstat 패키지를 설치해야한다. 

```shell
$ uptime
$ dmesg | tail
$ vmstat 1
$ mpstat -P ALL 1
$ pidstat 1
$ iostat -xz 1
$ free -m
$ sar -n DEV 1
$ sar -n TCP,ETCP 1
$ top
```



## 자세히 알아보기

### uptime

```shell
$ uptime
23:51:26 up 21:31, 1 user, load average - 30.02, 26.43, 19.02
```

uptime 명령어는 **현재시간, 시스템 실행시간, 로그인된 사용자 수, 그리고 부하율**을 확인 할 수 있다.
현재 대기중인 프로세스가 얼마나 있는지 나타내는 load average 값을 확인하는 가장 쉬운 방법이다.
대기 중인 프로세스뿐만 아니라 disk I/O와 같은 I/O작업으로 block된 프로세스까지 포함되어 있다. 
얼마나 많은 리소스가 사용되고 있는지 확인 할 수 있지만, 정확하게는 알 수 없다.

load average 옆에 나오는 3개의 숫자는 각각 **1분, 5분, 15분간의 부하율**을 나타낸다.
이를 통해서 시간의 변화를 알 수 있다. 예를들어, 장애가 발생했다는 소식을 듣고 해당 instance에 로그인 했을 때 1분 동안의 값이 15분 값에 비해 작다면 이는 장애가 발생하고선 내가 너무 뒤늦게 로그인했음을 알 수 있다. 
위의 예제에서 1분 값이 약 30이고 15분 값이 19정도 되는 것으로 볼 때 최근에 상승한 것을 알 수 있다. 

load average 값을 비교하며 CPU 수요에 문제가 있을거라 추측은 할 수 있지만, 정확한 원인 파악을 위해서는 뒤에 나오는 vmstat나 mpstat 같은 명령어를 이용해서 확인 할 수 있다.

### dmesg | tail

```shell
$ dmesg | tail
[1880957.563150] perl invoked oom-killer - gfp_mask=0x280da, order=0, oom_score_adj=0
[...]
[1880957.563400] Out of memory - Kill process 18694 (perl) score 246 or sacrifice child
[1880957.563408] Killed process 18694 (perl) total-vm:1972392kB, anon-rss:1953348kB, file-rss:0kB
[2320864.954447] TCP - Possible SYN flooding on port 7001. Dropping request.  Check SNMP counters.
```

시스템 부팅 메세지를 확인하는 명령어다.
 **커널에서 출력되는 메세지를 일정 수준 기록하는 버퍼 역할**을하며 커널 부팅 중 에러가 났다면 어느 단계에서 에러가 났는지 범위를 좁히고 찾아내는데 도움이 된다.
 `/var/log/dmesg` 에 로그 파일이 위치한다.

부팅시부터 모든 커널메세지가 출력되기 때문에 tail 명령어를 이용해서 마지막 10줄만 출력해보았다. 이 메세지를 통해서 성능에 문제를 줄 수 있는 에러를 찾을 수 있는데 위의 예제에서는 `OOM` , TCP request가 drop된 것을 확인 할 수 있다.

### vmstat 1

```shell
$ vmstat 1
procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
34  0    0 200889792  73708 591828    0    0     0     5    6   10 96  1  3  0  0
32  0    0 200889920  73708 591860    0    0     0   592 13284 4282 98  1  1  0  0
32  0    0 200890112  73708 591860    0    0     0     0 9501 2154 99  1  0  0  0
32  0    0 200889568  73712 591856    0    0     0    48 11900 2459 99  0  0  0  0
32  0    0 200890208  73712 591860    0    0     0     0 15898 4840 98  1  1  0  0
^C
```

virtual memory stat의 약자로 대부분 환경에서 사용 가능한 명령어다. 현재 **프로세스 정보, 메모리 사용량, 스왑, IO 상태 및 CPU 활동 상황**에 대한 정보를 확인 할 수 있다. 첫번째 라인에서 부팅 된 뒤 평균적인 값을 보여준다.

뒤에 1을 인자로 줬는데 이는 1초마다 정보를 보여주라는 뜻이다. 

많은 항목 중 우리가 <u>**확인해봐야 할 항목**</u>은 

|                    |                                                              |
| ------------------ | ------------------------------------------------------------ |
| r                  | CPU에서 동작 중인 프로세스 수, CPU 자원이 포화 상태인지 확인 할 때 참고 할 수 있다. **r 값이 CPU 수보다 큰 경우에 포화 상태**라고 해석 할 수 있다. |
| free               | free memory를 kb 단위로 나타낸다. 자리수가 너무 많은 경우 `free -m` 명령어를 이용하면 조금 더 편하게 확인 가능하다. |
| si, so             | swap-in 메모리와 swap-out에 대한 값이다. **0이 아니라면 현재 시스템에 메모리가 부족한 상황**이다. |
| us, sy, id, wa, st | 모든 CPU의 평균적인 CPU time을 측정할 수 있다.               |



그 외에도 여러 필드들을 보여주고있기에 각 필드에 대해 간단히 정리하였다.

위에서 언급된 필드는 <span style="color:red">빨간색</span>으로 표시해두었다.

- procs 필드

  |                                  |                                               |
  | -------------------------------- | --------------------------------------------- |
  | <span style="color:red">r</span> | CPU 접근 대기 중인 실행 가능한 프로세스 수    |
  | b                                | I/O 자원을 할당 받지 못해 블록 된 프로세스 수 |

- memory 필드

  |                                     |                                    |
  | ----------------------------------- | ---------------------------------- |
  | swapd                               | 사용된 가상 메모리의 용량          |
  | <span style="color:red">free</span> | 사용 가능한 여유 메모리의 용량     |
  | buffer                              | 버퍼에 사용된 메모리의 총량        |
  | cache                               | 페이지 캐시에 사용된 메모리의 용량 |

- swap 필드

  |                                   |                                                              |
  | --------------------------------- | ------------------------------------------------------------ |
  | <span style="color:red">si</span> | swap-in 된 메모리의 양 (kb)                                  |
  | <span style="color:red">so</span> | swap-out 된 메모리의 양 (kb), 지속적으로 발생한다면 메모리 부족을 의심해봐야 함 |

- I/O 필드

  |      |                                  |
  | ---- | -------------------------------- |
  | bi   | 블록 디바이스로부터 입력 블록 수 |
  | bo   | 블록 디바이스에 쓰기 블록 수     |

- system 필드

  |      |                                 |
  | ---- | ------------------------------- |
  | in   | 초당 발생한 interrupts 수       |
  | cs   | 초당 발생한 context switches 수 |

- CPU 필드

  |                                   |                                                              |
  | --------------------------------- | ------------------------------------------------------------ |
  | <span style="color:red">us</span> | CPU가 사용자 수준 코드를 실행한 시간 (단위 %)                |
  | <span style="color:red">sy</span> | CPU가 시스템 수준 코드를 실행한 시간 (단위 %)                |
  | <span style="color:red">id</span> | Idle 시간                                                    |
  | <span style="color:red">wa</span> | I/O wait 시간                                                |
  | <span style="color:red">st</span> | stolen time, hyperisor가 가상 CPU를 서비스 하는 동안 실제 CPU를 차지한 시간 <br />(즉, 가상 머신으로부터 뺏긴 시간) |

  

### mpstat -P ALL 1

```shell
$ mpstat -P ALL 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

07:38:49 PM  CPU   %usr  %nice   %sys %iowait   %irq  %soft  %steal  %guest  %gnice  %idle
07:38:50 PM  all  98.47   0.00   0.75    0.00   0.00   0.00    0.00    0.00    0.00   0.78
07:38:50 PM    0  96.04   0.00   2.97    0.00   0.00   0.00    0.00    0.00    0.00   0.99
07:38:50 PM    1  97.00   0.00   1.00    0.00   0.00   0.00    0.00    0.00    0.00   2.00
07:38:50 PM    2  98.00   0.00   1.00    0.00   0.00   0.00    0.00    0.00    0.00   1.00
07:38:50 PM    3  96.97   0.00   0.00    0.00   0.00   0.00    0.00    0.00    0.00   3.03
[...]
```

기본적으로 리눅스에 설치되어 있지 않고 sysstat 패키지를 설치하면 함께 설치된다. 

CPU와 Core별로 사용율을 모니터링 할 수 있어 **CPU time을 CPU 별로 측정 할 수 있기에 각 CPU별로 뷸균형한 상태를 확인 가능**하다. 1개의 CPU만 일하고 있는 것은 applicatoin이 single thread로 동작한다는 이야기다.



### pidstat 1

```shell
$ pidstat 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

07:41:02 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
07:41:03 PM     0         9    0.00    0.94    0.00    0.94     1  rcuos/0
07:41:03 PM     0      4214    5.66    5.66    0.00   11.32    15  mesos-slave
07:41:03 PM     0      4354    0.94    0.94    0.00    1.89     8  java
07:41:03 PM     0      6521 1596.23    1.89    0.00 1598.11    27  java
07:41:03 PM     0      6564 1571.70    7.55    0.00 1579.25    28  java
07:41:03 PM 60004     60154    0.94    4.72    0.00    5.66     9  pidstat

07:41:03 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
07:41:04 PM     0      4214    6.00    2.00    0.00    8.00    15  mesos-slave
07:41:04 PM     0      6521 1590.00    1.00    0.00 1591.00    27  java
07:41:04 PM     0      6564 1573.00   10.00    0.00 1583.00    28  java
07:41:04 PM   108      6718    1.00    0.00    0.00    1.00     0  snmp-pass
07:41:04 PM 60004     60154    1.00    4.00    0.00    5.00     9  pidstat
^C
```

process당 `top` 명령어를 수행하는 것과 비슷하다. 차이점은 스크린 전체에 표시하는 것이 아니라 **지속적으로 변화하는 상황을 띄워주기 때문에 패턴을 관찰하고 변화를 기록하기 좋다.**

뒤에 1을 인자로 줬는데 이는 1초마다 정보를 출력하라는 뜻이다. 



###  iostat -xz 1

```shell
$ iostat -xz 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

avg-cpu -  %user   %nice %system %iowait  %steal   %idle
          73.96    0.00    3.73    0.03    0.06   22.21

Device -   rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
xvda        0.00     0.23    0.21    0.18     4.52     2.08    34.37     0.00    9.98   13.80    5.42   2.44   0.09
xvdb        0.01     0.00    1.02    8.94   127.97   598.53   145.79     0.00    0.43    1.78    0.28   0.25   0.25
xvdc        0.01     0.00    1.02    8.86   127.79   595.94   146.50     0.00    0.45    1.82    0.30   0.27   0.26
dm-0        0.00     0.00    0.69    2.32    10.47    31.69    28.01     0.01    3.23    0.71    3.98   0.13   0.04
dm-1        0.00     0.00    0.00    0.94     0.01     3.78     8.00     0.33  345.84    0.04  346.81   0.01   0.00
dm-2        0.00     0.00    0.09    0.07     1.35     0.36    22.50     0.00    2.55    0.23    5.62   1.78   0.03
[...]
^C
```

**block device(HDD, SSD, ...) 가 어떻게 동작하는지 이해하기 좋은 툴**이다. 기본적으로 리눅스에 설치되어 있지 않고 sysstat 패키지를 설치하면 함께 설치된다. 

우리가 <u>**확인해봐야 할 항목**</u>은 

|                        |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| r/s, w/s, rkB/s, wkB/s | read 요청과 write 요청, read kB/s, write kB/s를 나타낸다. **어떤 요청이 가장 많이 들어오는지 확인**해볼 수 있는 중요한 지표다. 성능 문제는 생각보다 과도한 요청때문에 발생하는 경우도 있기 때문이다. |
| await                  | **I/O처리 평균 시간(ms)을 표현한 값**이다. application한테는 I/O요청을 queue하고 서비스를 받는데 걸리는 시간이기 때문에 application이 이 시간동안 대기하게 된다. 일반적인 장치의 요청 처리 시간보다 긴 경우에는 블럭장치 자체의 문제가 있거나 장치가 포화된 상태임을 알 수 있다. |
| avgqu-sz               | 해당 디바이스에 발생된 request들의 queue의 평균 length다. **1보다 큰 값은 포화 상태**를 나타낸다. |
| %util                  | **디바이스 사용률**이다. 60보다 큰 값은 장치에 따라 다르지만 일반적으로 성능 저하로 이어진다. 100에 근접 할 경우 device saturation이 발생한다. |



###  free -m

```shell
$ free -m
             total       used       free     shared    buffers     cached
Mem -        245998      24545     221453         83         59        541
-/+ buffers/cache -      23944     222053
Swap -            0          0          0
```

리눅스에서 **전체적인 현황**을 빠르게 살펴 볼 수 있는 명령어이다. 전체 메모리의 크기와 사용 중인 메모리 크기, 그 밖에 공유 메모리와 buffer, cache 메모리 및 swap의 크기를 알 수 있다.

우리가 <u>**확인해봐야 할 항목**</u>은 

|         |                                          |
| ------- | ---------------------------------------- |
| buffers | Block 장치 I/O의 buffer 캐시, 사용량     |
| cached  | 파일 시스템에서 사용되는 page cache의 양 |

2개의 값은 **0에 가까워지면 안된다. 이는 곧 높은 Disk I/O가 발생하고 있음을 의미**한다.



그 외 필드들이 의미하는 바를 간단히 정리해본다.

|                   |                                                              |
| ----------------- | ------------------------------------------------------------ |
| total             | 현재 시스템에 설치되어있는 전체 메모리의 크기                |
| used              | 현재 사용 중인 메모리의 크기 (toal - free - buffer/cache)    |
| free              | 시스템에서 사용 가능한 잔여 메모리의 크기                    |
| shared            | 프로세스 사이에서 공유되는 메모리의 크기, 주로 프로세스 또는 스레드 간 통신에 사용 |
| -/+ buffers/cache | 사용중인 메모리와 여유 메모리의 양                           |



### sar -n DEV 1

```shell
$ sar -n DEV 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015     _x86_64_    (32 CPU)

12:16:48 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:49 AM      eth0  18763.00   5032.00  20686.42    478.30      0.00      0.00      0.00      0.00
12:16:49 AM        lo     14.00     14.00      1.36      1.36      0.00      0.00      0.00      0.00
12:16:49 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

12:16:49 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:50 AM      eth0  19763.00   5101.00  21999.10    482.56      0.00      0.00      0.00      0.00
12:16:50 AM        lo     20.00     20.00      3.25      3.25      0.00      0.00      0.00      0.00
12:16:50 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

System Activity Report의 약어이다. CPU, memory, I/O 사용량을 수집, 레포트하고 저장하는 명령어다.

**시스템 자원 사용율 이력을 파일에 저장 한 후, 레포팅 할 때 유용**하다. 기본적으로 리눅스에 설치되어 있지 않고 sysstat 패키지를 설치하면 함께 설치된다. 

`-n` 옵션을 사용해서 **네트워크 인터페이스 처리량**(rxkB/s 및 txkB/s)을 측정 할 수 있다. 

필드들이 의하는 바를 간단히 정리해본다.

|          |                                     |
| -------- | ----------------------------------- |
| rx       | received 수신 (inbound traffic)     |
| tx       | transmitted 송신 (outbound traffic) |
| IFACE    | network interface 명                |
| rxpck/s  | 초당 rx 수                          |
| txpck/s  | 초당 tx 수                          |
| rxkB/s   | 초당 rx된 크기 (kb)                 |
| txkB/s   | 초당 tx된 크기 (kb)                 |
| rxcmp/s  | 초당 압축된 패킷의 rx 수            |
| txcmp/s  | 초당 압축된 패킷의 tx 수            |
| rxmcst/s | 초당 rx된 다중 패킷 수              |

### sar -n TCP,ETCP 1

```shell
$ sar -n TCP,ETCP 1
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

12:17:19 AM  active/s passive/s    iseg/s    oseg/s
12:17:20 AM      1.00      0.00  10233.00  18846.00

12:17:19 AM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
12:17:20 AM      0.00      0.00      0.00      0.00      0.00

12:17:20 AM  active/s passive/s    iseg/s    oseg/s
12:17:21 AM      1.00      0.00   8359.00   6039.00

12:17:20 AM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
12:17:21 AM      0.00      0.00      0.00      0.00      0.00
```

TCP 통신량을 요약해서 보여준다. 

|           |                                                              |
| --------- | ------------------------------------------------------------ |
| active/s  | 로컬에서부터 요청한 초당 TCP 커넥션 수 (예를들어, connect()를 통한 연결)<br />CLOSED -> SYN-SENT 상태로 전환한 수 |
| passive/s | 원격으로부터 요청된 초당 TCP 커넥션 수 (예를들어, accept()를 통한 연결)<br />LISTEN -> SYN-RCVD 상태로 전환한 수 |
| retrans/s | 초당 TCP 재연결 수                                           |
| iseg/s    | rx 된 총 세그먼트 수                                         |
| oseg/s    | tx 된 총 세그먼트 수                                         |

ative, passive 를 통해 서버의 부하를 대략적으로 측정하는데 편리하다. 

retrans는 네트워크나 서버의 이슈가 있음을 뜻한다. 신뢰성이 떨어지는 네트워크 환경이나(공용 인터넷), 서버가 처리 할 수 있는 용량 이상의 커넥션이 붙어서 패킷이 드랍되는 것을 이야기한다.



### top

```shell
$ top
top - 00:15:40 up 21:56,  1 user,  load average - 31.09, 29.87, 29.92
Tasks - 871 total,   1 running, 868 sleeping,   0 stopped,   2 zombie
%Cpu(s) - 96.8 us,  0.4 sy,  0.0 ni,  2.7 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem -  25190241+total, 24921688 used, 22698073+free,    60448 buffers
KiB Swap -        0 total,        0 used,        0 free.   554208 cached Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 20248 root      20   0  0.227t 0.012t  18748 S  3090  5.2  29812:58 java
  4213 root      20   0 2722544  64640  44232 S  23.5  0.0 233:35.37 mesos-slave
 66128 titancl+  20   0   24344   2332   1172 R   1.0  0.0   0:00.07 top
  5235 root      20   0 38.227g 547004  49996 S   0.7  0.2   2:02.74 java
  4299 root      20   0 20.015g 2.682g  16836 S   0.3  1.1  33:14.42 java
     1 root      20   0   33620   2920   1496 S   0.0  0.0   0:03.82 init
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.02 kthreadd
     3 root      20   0       0      0      0 S   0.0  0.0   0:05.35 ksoftirqd/0
     5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H
     6 root      20   0       0      0      0 S   0.0  0.0   0:06.94 kworker/u256:0
     8 root      20   0       0      0      0 S   0.0  0.0   2:38.05 rcu_sched
```

위에서 나온 명령어들을 통해 알 수 있는 다양한 측정치를 쉽게 체크 할 수 있다. 시스템 전반적인 값을 확인하기 쉽다는 장점이 있지만, 화면이 지속적으로 바뀌는 점 때문에 패턴을 찾기에는 vmstat, pidstat 명령어가 더 적합하다.

맨 윗줄부터 명령어를 통해 얻은 결과값의 의미를 알아보자.

- 00:15:40 - 현재 서버의 시간

- 1 user - 접속한 유저 수

- load avarage - 부하율

- Tasks - 871 total,   1 running, 868 sleeping,   0 stopped,   2 

  |          |                         |
  | -------- | ----------------------- |
  | total    | 가동 중인 프로세스 수   |
  | running  | 실행 중인 프로세스 수   |
  | sleeping | 대기 중인 프로세스 수   |
  | stopped  | 멈춘 상태인 프로세스 수 |
  | zombie   | 좀비 상태의 프로세스 수 |

- CPU

  |      |                                                              |
  | ---- | ------------------------------------------------------------ |
  | us   | user level에서 사용하고 있는 CPU의 비중                      |
  | sy   | system level에서 사용하고 있는 CPU의 비중                    |
  | Id   | 유휴 상태의 CPU 비중                                         |
  | wa   | 시스템이 I/O 요청을 처리하지 못한 상태에서의 CPU idle 상태인 비중 |

- Memory

  - Mem

    |         |                           |
    | ------- | ------------------------- |
    | total   | 전체 물리 메모리          |
    | used    | 사용중인 메모리           |
    | free    | 사용되지 않는 여유 메모리 |
    | buffers | 버퍼된 메모리             |

  - Swap

    |        |                      |
    | ------ | -------------------- |
    | total  | 전체 스왑 메모리     |
    | used   | 사용중인 스왑 메모리 |
    | free   | 남아있는 스왑 메모리 |
    | cached | 캐싱 메모리          |

- 프로세스 상태

  |         |                                                              |
  | ------- | ------------------------------------------------------------ |
  | PID     | 프로세스 ID                                                  |
  | USER    | 프로세스를 실행시킨 사용자 ID                                |
  | PRI     | 프로세스의 우선순위 (priority)                               |
  | NI      | NICE 값. 일의 nice value값이다. 마이너스를 가지는 nice value는 우선순위가 높음. |
  | VIRT    | 가상 메모리의 사용량 (SWAP+RES)                              |
  | RES     | 현재 페이지가 상주하고 있는 크기 (Resident Size)             |
  | SHR     | 분할된 페이지, 프로세스에 의해 사용된 메모리를 나눈 메모리의 총합. |
  | S       | 프로세스의 상태 <br />[ S(sleeping), R(running), W(swapped out process), Z(zombies) ] |
  | %CPU    | 프로세스가 사용하는 CPU의 사용율                             |
  | %MEM    | 프로세스가 사용하는 메모리의 사용율                          |
  | COMMAND | 실행된 명령어                                                |



## 마무리

넷플릭스 블로그 글을 일고, 관련 내용을 찾아봤다고해서 당장 바로 업무에서 사용 할 수 있을정도는 아니다.

이 기록을 바탕으로 실제로 더 사용해보고, 값들을 비교해보면서 상황에 맞게 바로바로 사용할 수 있는 정도로 몸에 익히는 일이 남았다.

