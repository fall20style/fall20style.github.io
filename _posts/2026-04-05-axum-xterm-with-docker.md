---
title: Axum-xterm.js 기반 Docker Web Terminal
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
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- 비동기 이벤트 기반의 스트리밍 프록시 구조

### 시스템 배치도 (Deployment Diagram)

<img width="950" height="561" alt="Image" src="https://github.com/user-attachments/assets/6a469403-a83a-4970-a330-a2b7a560127b" />

### 1. 시스템 개요 (System Overview)
웹 브라우저에서 실행 중인 Docker 컨테이너의 셸(bash)에 직접 접속하여 명령어를 실행하고 결과를 실시간으로 확인할 수 있는 Web-based 원격 터미널.

### 2. 기술 스택 (Tech Stack)

* Frontend: xterm.js (터미널 렌더링), 브라우저 표준 WebSocket API
* Backend: Rust (Axum Framework), Tokio (비동기 런타임)
* Infrastructure: Docker (Target Container), Bollard (Docker API Client)
* Communication: WebSocket (Binary/Text Streaming)

### 3. 시스템 아키텍처 (Architecture)## 3.1. 논리적 계층 구조 (Logical Layers)

1. UI Layer (Browser): 사용자의 키 입력을 가로채 서버로 보내고, 서버에서 온 바이트 데이터를 ANSI 시퀀스로 해석하여 화면에 그림.
2. Streaming Layer (Axum WS): WebSocket 세션을 유지하며 브라우저와 서버 간의 전송 채널 제공.
3. Bridge Layer (Rust/Bollard): WebSocket의 입출력 스트림과 Docker Exec의 stdin/stdout 스트림을 상호 매핑(Forwarding).
4. Runtime Layer (Docker Engine): 실제 격리된 환경에서 bash 프로세스를 구동하고 TTY를 할당.

### 3.2. 데이터 흐름도 (Data Flow)

* Input: User Key → xterm.js → WebSocket (Binary) → Axum → Docker Stdin
* Output: Docker Stdout/err → Axum → WebSocket (Binary) → xterm.js → Screen Update

### 4. 상세 설계 (Detailed Design)## 4.1. WebSocket 핸들링 및 비동기 처리

* Task 분리: 하나의 WebSocket 연결에 대해 두 개의 비동기 Task(tokio::spawn)를 생성하여 전이중(Full-duplex) 통신을 구현함.
* docker_to_ws: 컨테이너의 출력을 계속 감시하며 브라우저로 쏴줌.
   * ws_to_docker: 브라우저의 입력을 컨테이너의 입력 스트림에 씀.
* 종료 처리: tokio::select! 매크로를 사용하여 한쪽 세션이 끊기면(브라우저를 닫거나 컨테이너가 죽으면) 즉시 리소스를 해제함.

### 4.2. TTY 및 환경 설정

* Interactive Mode: CreateExecOptions에서 tty: true, stdin: true를 설정하여 대화형 프롬프트 활성화.
* 컬러 및 인코딩: TERM=xterm-256color 환경 변수를 주입하여 리눅스 표준 컬러 출력을 지원함.

### 5. 주요 인터페이스 (API Interface)

| 구분 | 프로토콜 | 엔드포인트 | 역할 |
|---|---|---|---|
| Static | HTTP/GET | / | 터미널 클라이언트(index.html) 제공 |
| Assets | HTTP/GET | /xterm.js | xterm.js 라이브러리 서빙 |
| Stream | WebSocket | /ws | 실제 셸 데이터 스트리밍 |

### 6. 보안 및 고려사항 (Security & Considerations)

* 권한 관리: 현재 시스템은 /var/run/docker.sock에 직접 접근하므로, 서버 프로세스의 보안 권한 관리가 핵심임.
* 리소스 관리: 사용자가 접속할 때마다 docker exec 프로세스가 생성되므로, 비정상 종료 시에도 컨테이너 내 프로세스가 좀비가 되지 않도록 abort() 처리를 강화함.

### 디버깅
- 검은 화면에서 멈춤
  - xterm.js, xterm.css를 받아서 local에 저장함

### 스크린샷
<img width="430" height="319" alt="Image" src="https://github.com/user-attachments/assets/b320c230-b3d7-49d1-8948-c48c58187612" />

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
