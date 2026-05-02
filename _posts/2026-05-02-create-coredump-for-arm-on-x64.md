---
title: "x64 리눅스에서 ARM 코어덤프 생성 과정"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - qemu
categories:
  - TodoList
toc: true
toc_sticky: true
toc_label: 목차
description: "x64 리눅스에서 ARM 코어덤프 생성 과정"
---

## 1. 크로스 컴파일러 및 에뮬레이터 설치

* ARM64 빌드를 위한 GCC 설치함: `sudo apt install gcc-aarch64-linux-gnu`
* x64에서 ARM 실행을 위한 QEMU 설치함: `sudo apt install qemu-user-static`
* 멀티 아키텍처 디버깅을 위한 GDB 설치함: `sudo apt install gdb-multiarch`


## 2. 메모리 오류 유도 코드 작성 (test.c)


``` c
#include <stdio.h>

int main() {
    int *p = NULL;
    *p = 10; // NULL 포인터 역참조로 크래시 유도
    return 0;
}
```


## 3. ARM 바이너리 빌드

* 디버깅 정보를 포함하여 정적 빌드 수행함:

```
aarch64-linux-gnu-gcc -g -static -o test_arm test.c
```

## 4. 코어덤프 생성 환경 설정

* 현재 쉘에서 덤프 파일 크기 제한 해제함: `ulimit -c unlimited`
* 덤프 파일명을 core로 고정하고 현재 디렉토리에 생성되도록 설정함 (루트 권한):

```
echo "core" | sudo tee /proc/sys/kernel/core_pattern
```


## 5. 실행 및 덤프 생성

* QEMU를 이용하여 ARM 파일 실행함: `./test_arm` (또는 `qemu-aarch64-static ./test_arm`)
* Segmentation fault (core dumped) 문구 확인 및 core 파일 생성 여부 체크함


## 6. GDB 분석

* 생성된 코어덤프 파일을 열어 크래시 지점 확인함:

```
gdb-multiarch ./test_arm core
```

* GDB 프롬프트에서 bt (backtrace) 명령어로 소스 코드의 몇 번째 줄인지 확인함

