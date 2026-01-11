---
layout: post
title: "mlc --loaded_latency 사옹법"
categories: tech
tags: [cxl, lspci]
---
**Intel Memory Latency Checker(MLC)**는 프로세서 캐시와 메모리 시스템의 latency, bandwidth를 측정용 툴이다. 흔히 **메모리 장치에 대한 성능 벤치마킹을 수행할 때** 많이 사용한다. 'intel mlc'로 구글링하면 intel 공식 사이트[^1]에서 다운로드 받을 수 있다.

현재 최신 버전은 v3.12이고, 다운로드 받은 파일명은 **mlc_v3.12.tgz**이다. 압축 해제하면 Linux, Windows용 mlc 파일이 있고, readme (사용자 가이드) PDF 문서도 확인할 수 있다. 실행 전에 꼭! 이 가이드를 숙지해서, 대체 내가 뭘 평가하고 있는 건지 사전에 공부해두도록 하자. 안 그러면 나중에 결과 해석하는데 크게 고생할 수 있다.

```bash
$ tree mlc_v3.12
mlc_v3.12
├── Intel Memory Latency Tools Outbound License Agreement.pdf
├── Linux
│   ├── mlc
│   └── redist.txt
├── Windows
│   ├── mlc.exe
│   ├── mlcdrv.sys
│   └── redist.txt
└── readme_mlc_v3.12.pdf

3 directories, 7 files
```

### mlc --loaded_latency
`mlc --loaded_latency` 명령어는 메모리 장치에 부하가 걸린 상태에서 latency를 측정하는 모드다. 실제 장치에 부하가 발생할 때의 체감 성능을 확인하는 용도라고 생각하면 된다.

이 명령을 실행하면 MLC는 CPU를 두 그룹으로 나눈다. **대부분의 코어는 메모리 트래픽을 만들기 위해 쓰이고**, Hyperthreading이 꺼져 있는 경우, CPU당 thread가 하나씩, 켜져 있는 경우 thread가 두 개씩 생성된다. 그리고 **단 하나의 코어만이 latency를 재는 데 쓰이는데**, 기본적으로 CPU 0이 그 역할을 맡는다.

### How latency is measured

Latency를 재는 thread는 **pointer chasing** 방식을 사용하여 평균 latency를 측정한다. 메모리 안의 각 cache line이 다음에 접근해야 할 주소를 들고 있고, 이전 load가 끝나야만 다음 load를 할 수 있는 구조다.

**Google의 multichase**[^2]도 같은 방식을 사용하며, mlc와 달리 오픈소스이니 궁금하다면 코드를 확인해보도록 하자. 물론, ChatGPT에게 "pointer chasing 방식으로 average latendy를 측정하는 microbenchmark를 구현해달라"고 요청하면 심플 코드를 확인할 수도 있다.

### Prefetcher control

Latency를 재는 코어에서는 **하드웨어 prefetcher**가 꺼진다. 프리페처가 켜져 있으면 다음 주소를 미리 가져와서 실제 메모리 접근이 발생하지 않을 수 있기 때문이다. 그러면 DRAM이나 CXL의 latency를 재는 게 아니라 L2나 L3 hit 시간을 재게 된다.

반대로 부하를 만드는 코어들에서는 prefetcher를 켜 둔다. 이쪽은 최대한 많은 메모리 트래픽을 만들어야 하므로 가능한 한 aggressive하게 동작해야 한다.

### Injection delay and bandwidth sweep

Load generation용 쓰레드는 N-1개 CPUs로 고정된다. 즉, thread 개수로 부하의 정도를 조절하지 않는다. MLC는 **injeciton delay**라는 개념을 사용하는데, 메모리 접근을 몇 번 수행한 다음 일정한 사이클 동안 일부러 쉰다 (== injection delay). 이 쉬는 시간의 길이를 바꿔 가면서 메모리 트래픽을 조절하는 것이다.

delay가 0이면 코어는 쉬지 않고 메모리를 때려서 시스템을 포화시킨다. delay가 커질수록 요청 사이 간격이 늘어나고, 시스템은 점점 idle에 가까워진다. MLC는 이 값을 0부터 수천 사이클까지 단계적으로 바꿔 가며 각 단계에서 latency를 측정한다.

### What the output means

출력은 **delay, latency (ns), bandwidth (MB/sec)** 순서로 출력된다. Inject Delay가 커질수록 load generation thread에 의한 bandwidth는 감소하고, latency thread에 의해 측정되는 latency는 감소하는 경향성을 확인할 수 있다.

```bash
Inject Latency Bandwidth
Delay (ns) MB/sec
==========================
00000 196.54 76701.3
00002 196.13 76784.2
00008 196.14 77053.2
.......
09000 71.74 2206.4
20000 71.32 1489.7
```

Intel MLC의 `--loaded_latency`는 기본적으로 **19개의 고정된 injection delay 포인트**에서만 측정을 수행한다. 하지만 실제 실험을 하다 보면, bandwidth가 saturation되는 구간이나, latency가 꺾이는 변곡점을 확인하기 위해, 훨씬 더 촘촘하게 보고 싶은 경우가 생긴다. 이럴 때를 위해, mlc에서 제공하는 **Injection Delay를 사용자 정의 값으로 설정하는 기능**을 활용하면 된다.

`--loaded_latency`를 아무 옵션 없이 실행하면 MLC는 다음과 같이 **하드코딩된 delay 테이블**을 사용한다:

```text
{ 
  0, 2, 8, 15, 50, 100, 200, 300, 400, 500,
  700, 1000, 1300, 1700, 2500, 3500,
  5000, 9000, 20000 
}
```

-g 옵션으로 내가 사용하고자 하는 delay를 정의한 인풋 파일을 전달해주면 된다.
이 파일에는 **한 줄에 하나씩 십진수 형태의 injection delay(ns 단위)** 값을 나열한다.

예를 들어 `delay.txt`:
```text
100
150
200
250
300
350
400
450
500
```
로 작성하고 `mlc --loaded_latency -g delay.txt`로 실행하면, 100ns에서 500ns까지, 50ns 간격으로 inject delay를 스윕하며 테스트하게 된다. 이렇게 출력된 값은 엑셀에서 bandwidth를 x축으로, latency를 y축으로 두고 그래프를 그리면 시각화시킬 수 있다.


[^1]: [인텔® Memory Latency Checker (인텔® MLC)](https://www.intel.co.kr/content/www/kr/ko/download/736633/intel-memory-latency-checker-intel-mlc.html)
[^2]: [https://github.com/google/multichase](https://github.com/google/multichase)