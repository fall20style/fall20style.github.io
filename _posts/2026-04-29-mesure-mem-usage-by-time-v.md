---
title: "time -v`를 이용한 메모리 측정하기"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - AI loop
  - time
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "time -v`를 이용한 메모리 측정하기"
mermaid: true
---

## `/usr/bin/time -v`를 이용한 메모리 측정 가이드

`/usr/bin/time` 도구의 상세 출력 모드(-v)를 사용하여 C/C++ 애플리케이션의 리소스 점유율을 정밀하게 측정하는 방법임.

## 개요 및 특징

* GNU time 도구임: 셸(bash, zsh) 내장 time 명령어보다 훨씬 상세한 정보를 제공함.
* 리소스 모니터링: CPU 사용률, 메모리 점유율, 컨텍스트 스위칭 등 커널 수준의 지표를 기록함.
* -v 옵션: 상세 보고서(Verbose) 모드를 활성화하여 모든 자원 통계를 출력함.

## 실행 방법

반드시 셸 내장 명령어가 아닌 풀 경로를 지정하여 실행해야 함.

```
/usr/bin/time -v ./your_executable
```

* 실행 예시: `/usr/bin/time -v ./release_bin/my_app`
* 결과 출력: 프로그램 종료 후 표준 에러(stderr)를 통해 보고서가 출력됨.


```
프로그램 실행 및 메모리 측정...
/usr/bin/time -v ./release_bin/my_app
	Command being timed: "./release_bin/my_app"
	User time (seconds): 0.07
	System time (seconds): 0.03
	Percent of CPU this job got: 98%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:00.11
	Average shared text size (kbytes): 0
	Average unshared data size (kbytes): 0
	Average stack size (kbytes): 0
	Average total size (kbytes): 0
	Maximum resident set size (kbytes): 49884    <=== 중요
	Average resident set size (kbytes): 0
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 18001
	Voluntary context switches: 1
	Involuntary context switches: 18
	Swaps: 0
	File system inputs: 0
	File system outputs: 0
	Socket messages sent: 0
	Socket messages received: 0
	Signals delivered: 0
	Page size (bytes): 4096
	Exit status: 0
```

## 핵심 지표 분석 (메모리 최적화 관련)

출력되는 여러 항목 중 메모리 경량화 루프에서 반드시 확인해야 할 지표임.

* Maximum resident set size (kbytes)
* 프로그램이 실행되는 동안 점유한 최대 물리 메모리(RAM) 크기임.
   * 가상 메모리가 아닌 실제 RAM 사용량을 나타내는 가장 객관적인 지표임.
* Average resident set size (kbytes)
* 실행 시간 동안의 평균 물리 메모리 점유량임.
* Major/Minor page faults
* 메모리 할당 및 접근 시 발생하는 페이지 폴트 횟수임. 빈번한 동적 할당 여부를 유추할 수 있음.

## 자동화 루프 활용 팁

파이썬 스크립트에서 이 데이터를 파싱하여 최적화 성공 여부를 판단함.

* 데이터 추출: subprocess.run 실행 시 stderr를 캡처한 후 정규표현식으로 숫자를 추출함.
* 성공 판정: 이전 회차의 Maximum resident set size보다 현재 수치가 감소했다면 최적화 성공으로 간주함.
* 주의 사항: 결과가 stderr로 나오므로 파이썬에서 capture_output=True 설정 시 result.stderr를 확인해야 함.

## 요약

* 메모리 경량화의 척도는 Maximum resident set size임.
* AI가 구조체를 최적화하거나 불필요한 복사를 줄이면 이 수치가 즉각적으로 감소함.
* 환경 변수나 추가 라이브러리 설치 없이 리눅스 표준 도구만으로 측정 가능한 가장 효율적인 방법임.


## 용어

* RSS: Resident Set Size
* Page Fault: 페이지 부재
* System Call: 시스템 호출 분석



