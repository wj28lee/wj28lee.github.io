---
layout: post
title: "Liux kernel 코드 빌드 및 설치하는 방법"
categories: tech
tags: [linux, kernel]
---
종종 리눅스 커널의 최신 기능 실험(?)을 위해, github repo.[^1]에서 코드를 다운로드 받아 빌드/설치하는 경우가 있다. 사실 방법이야 단순하지만, 커널 개발자가 아니라면 굳이 모든 history를 다운로드 받을 필요는 없다. 시간도 오래 걸리고.

아래는 특정 버전의 커널 코드만 다운로드 받아 빌드 및 설치하는 방법이다. (예로 v6.15 버전을 다운로드 받았다.)
```bash
$ git clone --branch v6.15 --depth 1 https://github.com/torvalds/linux.git
$ cd linux
$ make olddefconfig # 기존 .config 정보 승계
$ make menuconfig # 변경하고자 하는 config가 있다면 바꿔주자
$ make -j`nproc`
$ sudo make INSTALL_MOD_STRIP=1 modules_install -j`nproc`
$ sudo make install
```

만일 kernel command line[^2]이나 부팅해야 하는 커널 이미지 순서에 대한 변경이 필요하다면, 아래 단계를 참고하자.
```bash
$ sudo vi /etc/default/grub
# GRUB_DEFAULT: 부팅할 커널 이미지 선택. "1>이미지이름" 등으로 입력하면 된다.
# GRUB_CMDLINE_LINUX_DEFAULT: 사용할 Kernel command line 입력. (e.g. memhp_default_state=offline)
$ sudo update-grub2
```

작업이 완료되면, 재부팅해주면 된다.
```bash
$ sudo reboot
```

[^1]: [https://github.com/torvalds/linux.git](https://github.com/torvalds/linux.git)
[^2]: [The kernel’s command-line parameters](https://docs.kernel.org/admin-guide/kernel-parameters.html)