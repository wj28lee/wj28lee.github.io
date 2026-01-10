---
layout: post
title: "CXL 장치를 위한 daxctl 사용법"
categories: tech
tags: [cxl, daxctl, devdax, system-ram]
---
리눅스 시스템에 CXL 메모리 장치가 연결되어 있는 경우, ndctl project[^1]의 daxctl 명령어[^2]를 사용하면 장치를 손쉽게 관리할 수 있다. 관리라 함은, 장치의 목록을 조회하고, 장치의 모드(devdax mode, system-ram mode)를 전환하고, 장치 메모리를 온라인/오프라인 시키는 등의 작업을 의미하며, 시스템 자원에 대한 설정을 조회, 변경하는 것이므로 보통 sudoer 권한이 필요하다.

### 설치 방법
Ubuntu 공식 저장소에는 이미 dax 관련된 패키지가 등록되어 있다. daxctl은 DAX (Direct Access) 서브시스템을 관리하는 유틸리티 패키지이고, libdaxctl1은 daxctl 관련 라이브러리 패키지이다.
```bash
$ apt search dax
```

다만, CXL 메모리 장치에 대한 Linux 커널 지원 범위가 버전에 따라 크게 달라지므로, 최신 커널을 설치하여 사용하는 경우[^3]에는 ndctl github에서 소스 코드를 다운로드 받아 빌드, 설치한 daxctl을 사용하는 것이 속편하다. 이 경우에는 기존에 설치되어 있던 daxctl, libdaxctl1을 삭제한 뒤 설치해야 충돌없이 사용할 수 있다.

우선 기존 daxctl과 libdaxctl을 삭제한다.
```bash
$ sudo apt remove --purge -y daxctl libdaxctl1
$ dpkg -l | grep -E "daxctl" # 실제로 삭제되었는지 확인
```
소스 설치 방법은 다음과 같다.
```bash
$ git clone https://github.com/pmem/ndctl.git
$ cd ndctl
$ git checkout v83 # 2026-01-10 기준 최신 Release
$ meson setup build
$ meson compile -C build
$ sudo meson install -C build
```
meson 컴파일 시, 필요한 패키지가 설치되어 있지 않은 경우, 에러 로그를 잘 살펴보며 필요한 패키지를 설치해주도록 하자.
```bash
$ sudo apt install -y meson
```
### 사용 방법
우선 daxctl로 장치 목록을 조회하는 방법이다. -u 옵션을 넣어야 GiB 단위로 장치 메모리 용량을 표기해준다.
```bash
$ daxctl list -u
```
장치의 상태를 변경하는 방법은 아래와 같다.
```bash
$ sudo daxctl reconfigure-device --mode=devdax dax0.0
$ sudo daxctl reconfigure-device --mode=system-ram dax0.0
```

장치가 System RAM mode라면, 다음 명령어로 on/offline이 가능하다.
```bash
$ sudo daxctl offline-memory dax0.0
$ sudo daxctl online-memory dax0.0
```

메모리의 온/오프라인 상태를 바꾼 경우, lsmem이나 numactl -H 명령어를 통해 의도한 대로 메모리 설정이 변경되었는지 꼭 확인하도록 하자.
```bash
$ lsmem
$ numactl -H
```

[^1]: [https://github.com/pmem/ndctl](https://github.com/pmem/ndctl)
[^2]: [DAXCTL Man Pages](https://docs.pmem.io/ndctl-user-guide/daxctl-man-pages)
[^3]: [Liux kernel 코드 빌드 및 설치하는 방법](https://wj28lee.github.io/tech/2025/06/19/how-to-build-kernel.html)
