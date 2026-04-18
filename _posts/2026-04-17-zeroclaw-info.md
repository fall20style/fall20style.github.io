---
title: ZeroClaw 정보
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Rust
  - ZeroClaw
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## 개요 및 특징

* ZeroClaw는 OpenClaw 기반의 초경량 AI 에이전트 런타임임.
* 저사양 기기나 임베디드 환경에서도 LLM 에이전트를 돌릴 수 있게 최적화됨.
* 극강의 가벼움: 5MB 미만의 RAM만 점유함.
* 초고속 실행: 스타트업 시간이 10ms 미만으로 매우 빠름.
* 비용 절감: 기존 대비 메모리 99%, 운영 비용 98%까지 아낄 수 있음.

## 컨테이너 기술 및 보안

* 내부적으로 선택적 컨테이너 기술을 사용함.
* Docker 런타임 어댑터: 에이전트의 작업이 호스트 OS를 건드리지 않게 격리함.
* 샌드박싱: "No host execution" 원칙으로 시스템 침해 위험을 방지함.
* 임베디드 등 컨테이너가 없는 환경은 소프트웨어 가드레일로 보안을 유지함.

## 기술 스택 (Rust)

* 100% Rust 언어로 작성됨.
* 기존 TypeScript 기반보다 훨씬 가볍고 빠름.
* 단일 바이너리: 약 3.4MB 크기의 파일 하나로 실행됨.
* Rust 특유의 메모리 안전성 덕분에 버그와 보안 취약점이 적음.

## 메신저 연동

* 디스코드, 슬랙, 텔레그램 등과 연동 가능함.
* API 우선 설계: 내장 HTTP 서버를 통해 외부 플랫폼과 쉽게 연결함.
* 활용: 디스코드 봇을 통해 서버를 관리하거나 실시간 명령을 수행할 수 있음.
* 저사양 클라우드나 라즈베리 파이에서도 원활하게 돌아감.


## 별도의 런타임이 필요 없음

Java나 Python, Node.js 같은 언어들과 Rust의 결정적인 차이점임.

## 런타임이 필요 없는 이유

* 네이티브 컴파일: Rust는 소스 코드를 기계어(Machine Code)로 직접 번역함.
* 단일 바이너리: 실행에 필요한 모든 라이브러리를 하나의 파일 안에 다 집어넣음.
* 가상 머신 없음: JVM(Java)이나 V8(Node.js) 같은 중간 매개체가 필요 없음.

## 언어별 비교 (실행 환경)

* Python: python 인터프리터가 설치되어 있어야 함.
* Java: JRE나 JDK가 설치되어 있어야 함.
* Node.js: node 엔진이 설치되어 있어야 함.
* Rust (ZeroClaw): 파일 딱 하나만 있으면 그냥 바로 실행됨.

## 개발과 실행의 차이

* 개발할 때: Cargo나 Rustc 같은 툴체인이 필요함.
* 배포/실행할 때: 결과물인 실행 파일(예: zeroclaw)만 있으면 끝임. 대상 기기에 Rust를 깔 필요가 전혀 없음.





## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- ["프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)"]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
