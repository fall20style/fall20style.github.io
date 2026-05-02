---
title: "Docker와 binfmt_misc, QEMU를 활용한 이기종(ARM) 바이너리 디버깅 가이드"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - qemu
  - binfmt_misc
categories:
  - TodoList
toc: true
toc_sticky: true
toc_label: 목차
description: "Docker와 binfmt_misc, QEMU를 활용한 이기종(ARM) 바이너리 디버깅 가이드"
---


## QEMU 환경 기반 ARM 디버깅 가이드## 1. 호스트 환경 설정 (binfmt_misc 등록)

* x86_64 호스트에서 ARM 바이너리를 실행하기 위해 QEMU 인터프리터를 커널에 등록함.
* privileged 권한으로 실행하여 호스트 커널의 /proc/sys/fs/binfmt_misc 설정을 변경함.

```
docker run --privileged --rm tonistiigi/binfmt --install all
```

## 2. 디버깅 컨테이너 실행

* --platform linux/arm64 옵션으로 ARM 환경을 강제함.
* GDB가 프로세스에 접근할 수 있도록 보안 옵션을 해제함.
* 호스트의 작업 디렉토리(test_qemu)를 컨테이너 내부로 마운트함.

```
docker run -it --rm \
  --privileged \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  -v /home/USER/work/test_qemu:/work \
  -w /work \
  --platform linux/arm64 \
  ubuntu:22.04 /bin/bash
```

## 3. 디버깅 도구 설치

* 컨테이너 진입 후 필요한 패키지를 설치함.

```
apt update && apt install -y gcc gdb file
```

## 4. 컴파일 및 실행

* 디버깅 심볼(-g)을 포함하여 빌드함.
* 필요 시 코어 덤프 생성을 위해 ulimit 설정을 변경함.

```
gcc -g -o test_arm main.c
ulimit -c unlimited  # 코어 덤프 제한 해제
./test_arm           # 실행 및 크래시 유도
```

## 5. GDB 코어 덤프 분석

* 주의: 인자 순서는 반드시 gdb <실행파일> <코어파일> 순이어야 함.
* 순서가 바뀌면 not in executable format 에러가 발생함.

# 방법 1: 실행 시 인자 지정

```
gdb test_arm qemu_test_arm_...core
```

# 방법 2: GDB 내부에서 로드

```
(gdb) file test_arm
(gdb) core-file qemu_test_arm_...core
(gdb) bt  # 콜 스택 확인
```

## 핵심 요약

* binfmt_misc는 커널이 ARM 바이너리를 만났을 때 QEMU를 자동으로 호출하게 해줌.
* Docker Buildx는 내부적으로 이 기능을 사용하여 멀티 아키텍처 빌드를 수행함.
* GDB 분석 시 파일 타입이 인식되지 않으면 file 명령어로 아키텍처 일치 여부를 먼저 확인해야 함.

## 인자 목록 분석
```
docker run -it --rm		\
               --privileged	\
               --cap-add=SYS_PTRACE	\
               --security-opt seccomp=unconfined	\
               -v /home/USER/work/test_qemu:/work	\
               -w /work	\
               --platform linux/arm64	\
               ubuntu:22.04 /bin/bash	\
```




