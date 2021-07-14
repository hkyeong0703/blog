---
title: "Linux_system_infomation"
date: 2021-07-13T21:00:31+09:00
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

[DevOps와 SE를 위한 리눅스 커널이야기](http://www.yes24.com/Product/Goods/44376723 ) 를 오랜만에 다시 꺼내 읽어보며, 정리해보려고한다.



1장에서는 리눅스 시스템 구성 정보를 확인하는 방법이 나온다. 시스템의 문제점을 분석하고 확인하는데 제일 기본인 현재 시스템을 파악하는데 필요한 방법을 알아보자.



## 커널 정보 확인

가장 대표적인 방법은 `uname` 명령을 사용하는 것이다.

```shell
$ uname -a
Linux ip-172-31-27-89 5.4.0-1024-aws #24-Ubuntu SMP Sat Sep 5 06:19:55 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```



`dmesg` 명령을 통해 몇가지 정보를 더 확인 할 수 있다. `dmesg` 는 커널 디버그 메세지로 커널이 부팅할 때 나오는 메세지와 운영 중에 발생하는 메세지를 볼 수 있게 해준다.

```shell
$ dmesg | grep -i kernel | more
[    0.000000] KERNEL supported cpus:
[    0.000000] Netfront and the Xen platform PCI driver have been compiled for this kernel: unplug emulated NICs.
[    0.000000] Blkfront and the Xen platform PCI driver have been compiled for this kernel: unplug emulated disks.
               in your root= kernel command line option
[    0.043677] Booting paravirtualized kernel on Xen HVM
[    0.044494] Kernel command line: BOOT_IMAGE=/boot/vmlinuz-5.4.0-1024-aws root=PARTUUID=a2f52878-01 ro console=tty1 c
onsole=ttyS0 nvme_core.io_timeout=4294967295 panic=-1
[    0.065479] Memory: 4019924K/4193908K available (14339K kernel code, 2326K rwdata, 4632K rodata, 2656K init, 5120K b
ss, 173984K reserved, 0K cma-reserved)
[    0.065657] Kernel/User page tables isolation: enabled
```

