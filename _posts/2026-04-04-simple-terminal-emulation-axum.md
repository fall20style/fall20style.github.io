---
title: Axum과 WebSockets를 이용한 터미널 에뮬레이션
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - axum
  - Rust
  - pty
  - xterm.js
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

Rust의 Axum 프레임워크와 WebSockets, 그리고 xterm.js를 사용하여 웹 브라우저용 터미널 에뮬레이터를 구축하는 하네스 엔지니어링 과정을 설명.

## 하네스 엔지니어링 (Harness Engineering) 과정

이 프로젝트의 핵심은 **웹 클라이언트(xterm.js)**와 **서버 사이드 프로세스(Bash shell)** 사이의 실시간 양방향 데이터 흐름을 안전하고 효율적으로 연결하는 것.

1.  **PTY(Pseudo-Terminal) 통합:** `portable-pty` 크레이트를 사용하여 서버 환경에 종속되지 않는 가상 터미널 환경을 구축했습니다. 이는 실제 쉘(`bash`)과 상호작용하는 물리적 터미널의 동작을 추상화.
2.  **비동기 스트리밍:** Tokio 런타임을 기반으로 Axum의 WebSocket 핸들러를 구현했습니다. PTY에서 발생하는 출력(stdout/stderr)과 사용자의 입력(keystroke)을 비동기 태스크로 분리하여 처리함으로써, 명령 실행 중에도 입력을 받을 수 있는 인터랙티브한 환경을 보장.
3.  **보안 제어 (Command Filtering):** 단순한 터미널 제공을 넘어, `COMMAND_WHITELIST`를 도입하여 허용된 명령(`ls`, `vim`, `cat` 등)만 실행될 수 있도록 기본적인 필터링 로직을 설계.

## 아키텍처 (Architecture)

애플리케이션은 다음과 같은 레이어로 구성:

-   **Frontend (Browser):** [xterm.js](https://xtermjs.org/) 라이브러리를 통해 터미널 화면을 렌더링하고, 사용자 입력을 캡처하여 WebSocket으로 전송.
-   **Communication Layer:** HTTP 업그레이드를 통해 생성된 WebSocket 채널을 사용하여 바이너리 및 텍스트 데이터를 실시간으로 교환.
-   **Backend (Rust/Axum):** WebSocket 연결을 관리하고, PTY 마스터 측의 입출력을 중계.
-   **Execution Layer (PTY/Bash):** PTY 슬레이브 측에서 실제 `bash` 쉘이 실행되며, 사용자로부터 전달된 명령을 수행.

## 소프트웨어 요구사항 (SW Requirements)

이 시스템을 구동하고 개발하기 위해 필요한 주요 구성 요소:

1.  **개발 언어 및 런타임:**
    -   **Rust 1.75+:** 최신 비동기 문법 및 Axum 0.7 기능을 활용.
    -   **Tokio:** 고성능 비동기 I/O 처리를 위한 런타임.
2.  **주요 라이브러리 (Dependencies):**
    -   `axum` (v0.7): 웹 서버 및 WebSocket 라우팅.
    -   `portable-pty`: 크로스 플랫폼 가상 터미널 지원.
    -   `tower-http`: 정적 파일 서빙 및 미들웨어 관리.
    -   `serde`: JSON 데이터 직렬화.
3.  **프론트엔드 도구:**
    -   **xterm.js (v5.3+):** 브라우저 기반 터미널 에뮬레이터 UI.
4.  **운영 체제:**
    -   Linux 또는 macOS (PTY 지원이 원활한 환경 권장).
