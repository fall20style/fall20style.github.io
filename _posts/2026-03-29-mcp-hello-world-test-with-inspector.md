---
title: MCP Inspector와 MCP hello world 예제 테스트
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - MCP기술
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 1. 테스트 환경 준비

* 사전 설치: Python 3.10 이상, Node.js(npx 사용) 설치 필요.
* 라이브러리: mcp, httpx 등 필요한 패키지 설치 필요.
* 서버 코드: FastMCP 등으로 작성된 server.py 파일 준비. (Gemini에 질문하면 코드 나옴)

### 2. 테스트 실행 순서

1. 터미널 실행: 프로젝트 루트 폴더에서 터미널 오픈.
2. 인스펙터 구동: 아래 명령어 입력.

```
npx @modelcontextprotocol/inspector python server.py
```

3. 브라우저 접속: 터미널에 출력된 URL(예: http://localhost:5173)로 접속.
4. 서버 연결: 화면 왼쪽 하단의 Connect 버튼 클릭.
5. 기능 확인: 상단 탭에서 Tools, Resources, Prompts 목록이 정상적으로 올라오는지 확인.
6. 도구 호출: 실행하려는 도구의 인자(Arguments) 값을 입력하고 Run Tool 클릭 후 결과값 검증.

### 3. Client (Inspector) vs Server (server.py) 흐름

| 단계 | 주체 | 동작 내용 | 통신 방식 (Transport) |
|---|---|---|---|
| 1. 구동 | Client | python server.py 프로세스를 자식 프로세스로 실행(Spawn) | OS Process 생성 |
| 2. 초기화 | Client | initialize 요청 전송 (지원 기능 및 버전 확인) | stdin (JSON-RPC) |
| 3. 기능 조회 | Client | list_tools, list_resources 등 목록 요청 | stdin (JSON-RPC) |
| 4. 목록 응답 | Server | @mcp.tool 등으로 정의된 기능 리스트 반환 | stdout (JSON-RPC) |
| 5. 실행 요청 | Client | 사용자가 입력한 파라미터와 함께 도구 호출 요청 | stdin (JSON-RPC) |
| 6. 결과 반환 | Server | 실제 파이썬 함수 실행 후 최종 결과값 응답 | stdout (JSON-RPC) |

### 4. 주요 특징 요약

* 주도권: 클라이언트(Inspector/Claude)가 항상 먼저 요청을 보내는 Request-Response 구조.
* 상태: 서버는 기본적으로 수동적이며, 클라이언트의 요청이 올 때까지 표준 입력(stdin)을 대기(Listening).

