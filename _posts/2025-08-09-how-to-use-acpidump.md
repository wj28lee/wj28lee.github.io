---
layout: post
title: "Liux에서 ACPI tables 정보 조회하는 방법"
categories: tech
tags: [cxl, ACPI tables, acpica, acpidump, acpixtract]
---
리눅스 시스템에서 종종 ACPI tables[^1] 정보를 확인해야할 때가 있다. 예를 들어, CXL 장치에 대한 NUMA nodes를 구성하기 위해 커널이 활용하는 raw data를 직접 조회하고자 할 때처럼 말이다[^2].

이 때는 **The ACPI Component Architecture (ACPICA) project**[^3]에서 제공하는 툴을 활용하면 된다.

방법은 간단하다.

### 1) acpi utility를 설치한다.

acpica-tools 패키지를 설치하거나, github repo.에서 직접 소스를 다운로드 받고 설치하는 방법이 있다.

```bash
$ sudo apt install acpica-tools
```

```bash
$ git clone https://github.com/acpica/acpica.git
$ cd acpica
$ make -j
$ sudo make install
```

CXL의 경우 아직 패키지들이 지속 업데이트 되는 경향이 있어, 나 같이 CXL 업무를 보는 경우는 최신 소스를 받아서 활용하나, 실제 리눅스 배포판에 상용 CXL 메모리 장치를 장착하여 활용하는 경우에는 그냥 acpica-tools를 설치해서 사용해도 무방하다.

### 2) acpidump를 활용하여 시스템에서 ACPI tables 정보를 덤프한다.
```bash
$ sudo acpidump -o acpidump.out
```

### 3) dump한 파일을 acpixtract을 사용하여 ACPI tables dat 파일을 얻어낸다.
```bash
$ acpixtract -a acpidump.out
$ ls *.dat
... cedt.dat ... srat.dat ...
```

### 4) iasl로 human-readable한 파일로 변경한 뒤 파일 내용을 조회한다.
```bash
$ iasl -d cedt.dat
$ iasl -d srat.dat
```

[^1]: [https://docs.kernel.org/driver-api/cxl/platform/acpi.html](https://docs.kernel.org/driver-api/cxl/platform/acpi.html)
[^2]: [https://docs.kernel.org/driver-api/cxl/platform/acpi/srat.html](https://docs.kernel.org/driver-api/cxl/platform/acpi/srat.html)
[^3]: [https://github.com/acpica/acpica](https://github.com/acpica/acpica)