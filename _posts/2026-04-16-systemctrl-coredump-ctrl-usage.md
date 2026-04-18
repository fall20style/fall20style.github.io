---
title: systemd-coredump 요약
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - coredumpctl
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## 주요 기능

* 자동 수집: 프로그램 크래시 시 메모리 상태 자동 저장함.
* 데이터 압축: lz4 등으로 압축해서 용량 아낌.
* 메타데이터: 실행 경로, PID, 시그널 등 상세 정보 같이 기록함.
* 로그 통합: journalctl이랑 연동돼서 확인 편함.


## 저장 및 관리

* 저장소: `/var/lib/systemd/coredump/`에 보관함.
* 설정: `/etc/systemd/coredump.conf`에서 용량 조절 가능함.
* 도구: coredumpctl 명령어로 관리함.


## 주요 명령어

* `coredumpctl list`: 전체 덤프 목록 보여줌.
* `coredumpctl info` [PID]: 상세 정보 확인용임.
* `coredumpctl gdb [PID]`: 바로 디버깅.


## 코어 파일 저장

* 최근 덤프 추출
`coredumpctl dump -o core.file`
* PID 지정 저장
`coredumpctl dump [PID] -o filename`
* 실행 파일명 기준
`coredumpctl dump [NAME] -o filename`

## 디버깅 및 설정

* GDB 바로 연결...
`coredumpctl debug [PID]`

* 기본 저장 경로...
`/var/lib/systemd/coredump/`

* 설정 파일...
`/etc/systemd/coredump.conf`

* 파일 안 보이면... `ulimit -c` 확인...


## Related Posts
- [gdb로 coredump를 생성하고 gemini CLI로 디버깅]({% link _posts/2026-04-15-create-gdb-core-on-ubuntu.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Superpowers와 Gstack]({% link _posts/2026-04-01-gstack-and-superpowers.md %})
