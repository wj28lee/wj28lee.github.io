---
layout: post
title: "CXL 메모리 장치를 위한 lspci 명령어 사용법"
categories: tech
tags: [cxl, lspci]
---
CXL 메모리 장치에 대한 PCIe 정보를 조회할 때 lspci 명령을 사용한다. 우선 cxl 키워드로 장치를 검색하여 원하는 장치의 BDF (bus/device/func) 정보를 알아낸 뒤, -vvv 옵션을 붙여 verbose 모드로 실행한 뒤 상세 정보를 조회하는 식이다.
```bash
$ lspci | grep -i cxl
$ sudo lspci -s <bus>:<device>.<func> -vvv
```

이 때 CXL RegLoc와 같은 CXL용 vendor-spefic extented capabilities를 조회하고자 한다면 최신 lspci를 사용해야 한다. 그렇지 않으면 human-readable하게 파싱해주지 않고 '데이터가 있는 건 알겠는데, 어떻게 파싱할지는 모르겠습니다.'라는 무책임(?)한 출력문만 보게 된다.

lspci는 pciutils repository[^1]에서 다운로드 받아 설치할 수 있다.

```bash
$ git clone https://github.com/pciutils/pciutils.git
$ cd pciutils
$ make -j && sudo make install
$ lspci | grep -i CXL
$ sudo lspci -s <bus>:<device>.<func> -vvv
```

[^1]: [https://github.com/pciutils/pciutils.git](https://github.com/pciutils/pciutils.git)