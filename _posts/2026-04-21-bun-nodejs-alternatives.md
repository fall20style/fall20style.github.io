---
title: bun 설치 방법, 빠른 이유
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - bun
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## Bun 요약: 사용법 및 속도의 비밀

## 주요 사용법 (명령어 요약)

* 설치: curl -fsSL https://bun.sh | bash (macOS/Linux/WSL 기준임)
* 프로젝트 초기화: bun init (기본 설정 파일 생성함)
* 패키지 관리:
* 설치: bun install (npm보다 압도적으로 빠름)
   * 추가: bun add [패키지명]
   * 삭제: bun remove [패키지명]
* 실행:
* 파일 실행: bun index.ts (TS 컴파일러 없이 즉시 실행함)
   * 스크립트 실행: bun [스크립트명]
   * 워치 모드: bun --watch index.ts (코드 수정 시 자동 재시작함)
* 패키지 실행: bunx [패키지명] (npx 대체용임)

## Bun이 빠른 이유

* 엔진의 차이: V8(Node.js) 대신 Safari의 JavaScriptCore(JSC)를 사용함. 시작 속도가 빠르고 메모리 효율이 좋음.
* Zig 언어 활용: 시스템 언어인 Zig로 작성되어 저수준(low-level) 메모리 제어와 하드웨어 최적화가 정밀함.
* 통합 설계: 패키지 매니저, 번들러, 런타임이 하나로 합쳐져 있어 도구 간 데이터 전송 오버헤드가 없음.
* 네이티브 TS/JSX 지원: 별도의 변환(Transpile) 단계 없이 엔진이 직접 코드를 이해하고 실행함.
* 고성능 API: Bun.file(), Bun.serve() 등 자체 API가 운영체제 커널 수준에서 최적화되어 있음.
