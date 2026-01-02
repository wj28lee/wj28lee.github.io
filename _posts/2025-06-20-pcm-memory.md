---
layout: post
title: "Linux에서 실시간 B/W를 조회하는 방법 (DIMM/CXL DRAM)"
categories: tech
tags: [dimm, cxl, intel, pcm, performance, counter, monitor]
---
Intel 서버 환경에서 메모리 장치의 실시간 B/W를 모니터링해야할 때가 있다. 이 때는 Intel에서 제공하는 오픈소스인 **Intel® Performance Counter Monitor (Intel® PCM)**[^1]를 활용하면 된다.

특히, Intel SPR(Sapphire Rapids) 이후 시스템에 장착된 CXL 메모리에 대한 bandwidth 측정 기능도 제공하기 때문에, CXL 메모리 장치에 대한 벤치마킹 시 사용하면 유익하다.

아래는 github repo.에서 소스를 다운로드 받고, 빌드하고, 설치하는 과정이다.
```bash
$ git clone https://github.com/intel/pcm.git
$ cd pcm
$ mkdir bulid
$ cd build
$ cmake ..
$ cmake --build .
```

모니터링을 위해 주기적으로 bandwidth 정보를 화면에 띄우고 싶다면, 아래와 같이 사용하면 된다. 실행에는 sudo 권한이 필요하다.
```bash
$ sudo ./pcm-memory -u # -u: update measurements instead of printing new ones
```

아래 명령어를 사용하면 1초마다 test.log 파일에 csv 형태로 각 dimm/cxl dram의 read, write bandwidth가 누적되어 기록된다.
```bash
$ sudo ./pcm-memory 1 -csv=test.log -- <your program>
```

[^1]: [https://github.com/intel/pcm](https://github.com/intel/pcm)