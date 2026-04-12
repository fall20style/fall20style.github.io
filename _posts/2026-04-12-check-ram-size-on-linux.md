---
title: Linux에서 RAM 사이즈 확인하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - H/W
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


리눅스에서 free -h를 입력했을 때 나오는 결과 예시

## 출력 예시

```
              total        used        free      shared  buff/cache   available
Mem:           16Gi       4.5Gi       2.1Gi       1.2Gi       9.4Gi        10Gi
Swap:         2.0Gi        0.5Gi       1.5Gi
```

## 주요 항목

* total: 내장된 전체 물리 램 용량임 (예: 16GB).
* used: 현재 프로세스들이 실제로 점유 중인 용량임.
* free: 완전히 비어 있는 순수 여유 공간임.
* buff/cache: 커널이 속도를 높이려고 임시로 빌려 쓰는 공간임. (앱이 필요하면 즉시 반환됨)
* available: 가장 중요한 수치임. 새로운 프로그램을 실행할 때 실제로 사용 가능한 여유량임. (free + buff/cache 일부)

## Swap 항목

* Swap: 램이 모자랄 때 쓰는 가상 메모리(디스크)임.
* 여기 used가 높으면 시스템이 매우 느려질 수 있음.

- VS Code 같은 무거운 프로젝트를 빌드할 때는 available 수치가 최소 8Gi 이상인지 확인하는 것이 좋음.



