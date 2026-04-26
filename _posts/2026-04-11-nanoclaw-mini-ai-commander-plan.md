---
title: "프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - NanoClaw
  - Docker
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 프로젝트 계획안: NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)


```
https://github.com/fall20style/docker_prep/tree/main/nanoclaw-mini
```        

### 1단계: 핵심 환경 구축 (Foundation)
가장 먼저 Node.js와 Docker가 소통할 수 있는 기반을 다짐.

- "최소한의 코드로 구현하는 강력한 격리"를 증명하는 프로젝트 계획안

------------------------------

* 프로젝트 초기화: TypeScript 환경 설정(tsconfig.json) 및 필수 모듈(@types/node) 설치.
* Docker 베이스 이미지 선정: 에이전트가 사용할 가벼운 이미지(node:20-slim 또는 python:3.11-slim) 준비.
* 파일 시스템 설계: 세션별 격리 폴더(chats/{session_id}) 구조 생성 로직 구현.

### 2단계: 샌드박스 실행 엔진 구현 (Core Executor)
호스트의 명령을 컨테이너로 전달하고 결과를 수거하는 핵심 기능을 만듦.

- child_process 연동: spawn을 사용하여 docker run --rm 명령어를 동적으로 생성하는 함수 구현.
- 보안 옵션 적용: --user $(id -u), --network none, --memory 제한 등 NanoClaw의 핵심 보안 옵션 주입.
- 볼륨 마운트: 호스트의 작업 폴더를 컨테이너 내부로 연결하는 로직 완성.

### 3단계: 파일 감지 및 수거 시스템 (Data Egress)
에이전트가 작업한 결과물(파일)을 호스트가 인지하게 함.

- Diff 스캔 로직: 명령 실행 전/후의 파일 목록을 비교하여 신규 생성된 파일 리스트 추출.
- 미디어 처리: 특정 확장자(png, pdf 등)를 감지했을 때 별도의 로그를 남기거나 처리하는 핸들러 작성.

### 4단계: 메시징 인터페이스 시뮬레이션 (Interface)
실제 WhatsApp 연동 대신, 터미널 입력으로 메시징 채널을 흉내 냄.

- CLI 챗봇 구현: readline을 통해 @Andy 같은 트리거 워드를 감지하면 샌드박스를 가동하는 루프 생성.
- 상태 관리: 에이전트의 실행 상태(Running, Finished, Error)를 터미널에 시각적으로 표시.

### 5단계: 보안 및 예외 처리 고도화 (Hardening)
실제 운영 시 발생할 수 있는 위험 요소를 차단함.

- 타임아웃 설정: 에이전트가 무한 루프에 빠질 경우를 대비해 일정 시간 후 컨테이너를 강제 종료(kill)하는 로직 추가.
- 권한 세분화: 읽기 전용 마운트(:ro)와 읽기/쓰기 마운트(:rw)를 상황에 따라 분리하여 테스트.


## 과제 제출용 체크리스트

1. TypeScript로 작성되어 타입 안정성이 확보되었는가?
2. 실행 후 컨테이너가 자동으로 삭제(--rm)되는가?
3. 호스트 사용자의 UID/GID가 컨테이너에 올바르게 전달되는가?
4. 신규 생성된 파일이 호스트의 workspace 폴더에 잘 저장되는가?



## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
- [NanoClaw 아키텍처 설계 및 구성 요소 정리]({% link _posts/2026-04-25-nanoclaw-arch-desc.md %})
- [nanoclow code분석-1 src/index.ts]({% link _posts/2026-04-25-nanoclaw-code-reading.md %})
