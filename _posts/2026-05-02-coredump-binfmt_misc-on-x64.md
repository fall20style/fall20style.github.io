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


### 1. 기본 실행 관련

* `-it`: -i (interactive)와 -t (tty)의 조합임. 컨테이너 내부 쉘과 키보드로 상호작용할 수 있게 터미널 환경을 열어줌.
* `--rm`: 컨테이너 종료 시 컨테이너 자체를 자동으로 삭제함. 테스트용 일회성 컨테이너를 돌릴 때 찌꺼기를 남기지 않아 유용함.
* `--name` (생략됨): 컨테이너 이름을 지정하지 않으면 Docker가 무작위 이름을 붙여줌.

### 2. 권한 및 보안 관련 (디버깅 핵심)

* `--privileged`: 컨테이너가 호스트의 모든 장치(Device)에 접근할 수 있는 강력한 권한을 부여함. binfmt_misc를 조작하거나 하드웨어 제어가 필요할 때 사용함.
* `--cap-add=SYS_PTRACE`: 커널의 ptrace 시스템 호출 권한을 추가함. GDB가 다른 프로세스를 추적(Trace)하고 멈추기 위해 반드시 필요한 옵션임.
* `--security-opt seccomp=unconfined`: Docker의 보안 필터링(seccomp)을 해제함. 기본 설정에서는 GDB의 일부 동작이 차단될 수 있어 디버깅 시에는 무제한(unconfined) 상태로 둠.

### 3. 파일 및 경로 관련

* `-v /home/USER/work/test_qemu:/work`: Bind Mount 옵션임. 호스트의 디렉토리와 컨테이너 내부의 /work 폴더를 실시간으로 연결함.
* `-w /work`: 컨테이너 접속 시 시작 디렉토리(Working Directory)를 /work로 설정함.

### 4. 아키텍처 및 이미지

* `--platform linux/arm64`: 가장 중요한 부분임. 호스트(x86_64)와 다른 ARM 64비트 환경을 요청함. 이때 binfmt_misc에 등록된 QEMU가 작동하여 ARM 바이너리를 실행할 수 있게 됨.
* `ubuntu:22.04`: 사용할 베이스 이미지임. 해당 이미지의 ARM64 버전이 자동으로 다운로드됨.
* `/bin/bash`: 컨테이너 실행 후 바로 실행할 명령어임. 여기서는 Bash 쉘을 실행함.


### 요약

- "호스트 폴더를 공유하고, 모든 보안 제약을 푼 상태에서, QEMU를 통해 ARM64 환경의 우분투 쉘을 띄워라"라는 의미.

