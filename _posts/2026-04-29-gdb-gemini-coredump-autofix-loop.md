---
title: "GDB와 Gemini AI를 활용한 코어덤프 디버깅 루프 POC"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - GDB
  - Gemini
  - LangChain
  - Python
  - C++
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: GDB Python 스크립트와 Gemini AI를 결합하여 C++ 코어 덤프를 분석하는 디버깅 루프
---

## AI 기반 디버깅 루프 POC
- 코어 덤프 검토, 코드 수정 및 재빌드 자동화
- AI(Gemini 1.5 Flash)와 Python을 활용

## 주요 기술 스택

* Language: Python, C++
* Debugger: GDB (systemd-coredump 연동)
* AI: Google Gemini 1.5 Flash (via LangChain)
* Tools: difflib (정밀 패치 생성), Makefile (자동 빌드)

## 전체 프로세스

1. 실행 및 크래시 감지: run_app.py가 앱 실행 중 SIGSEGV 같은 크래시를 감지하면 루프가 시작.
2. 딥 디버깅 데이터 추출: extract_core.py가 코어 덤프를 생성하고, GDB 내장 파이썬 스크립트가 백트레이스와 변수 정보를 crash_context.json으로 추출.
3. AI 분석 및 처방: Gemini가 GDB 로그와 소스 코드를 분석하여 크래시 원인을 파악하고, 수정된 코드를 생성.
4. 정밀 패치 생성: AI의 오류를 방지하기 위해 difflib이 원본과 비교하여 output.diff를 생성.
5. 자동 반영 및 재빌드: apply_patch.py가 코드를 수정하고 make를 통해 빌드 성공 여부를 확인.
6. 최종 검증: 수정된 앱을 재실행하여 정상 종료(Exit Code 0)와 방어 로직 작동 여부를 확인하고 리포트를 생성.

## 핵심 포인트

* 정확성: difflib을 사용하여 물리적인 Diff 파일을 생성함으로써 patch 명령어와의 호환성을 보장.
* 자율성: 사람이 개입하지 않고 크래시 분석부터 코드 수정, 재컴파일 후 정상 작동 확인까지 자동화.
* 확장성: Python 스크립트 구조로, C/C++ 프로젝트에 쉽게 적용.

## 데이터 파이프라인 요약

```
App Crash
   → Core Dump
     → GDB 분석(JSON)
       → Gemini AI
         → difflib(Diff 생성)
           → Patch & Build
             → Success
```

## Code
- https://github.com/fall20style/docker_prep/tree/main/gdb_py_llm_automation/project
