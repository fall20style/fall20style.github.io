---
title: Python으로 Gmail, Google Calendar MCP 서버 구축
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - AI
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

# Python으로 Gmail, Google Calendar MCP 서버 구축

직접 만든 Python 에이전트에 연동하기 위해 Gmail과 Google Calendar를 제어하는 MCP(Model Context Protocol) 서버를 구현함.

## 구현 환경
- **언어 및 패키지**: Python 3.10+, `uv`, `mcp` v2.x SDK, `google-api-python-client`
- **통신 방식**: `stdio` (에이전트에서 서브프로세스로 호출)
- **인증 방식**: Google Cloud OAuth 2.0 (`credentials.json`, `token.json`)

## 구현 기능 (Tools 5종)

### Gmail
- `search_emails`: 검색 쿼리 기반 메일 목록 및 메타데이터 조회
- `get_email_body`: 멀티파트 메시지 본문 파싱 및 텍스트 추출
- `send_email`: 새 이메일 작성 및 발송

### Google Calendar
- `list_upcoming_events`: 현재 시점 이후 예정된 일정 조회
- `create_calendar_event`: 한국 표준시(`Asia/Seoul`) 기준 새 일정 등록

<img width="605" height="387" alt="mcp_test_01" src="https://github.com/user-attachments/assets/984bdfd7-c85d-4c0e-851e-d002c3207955" />

## 진행 과정 및 트러블슈팅

### 1. SDK v2.x 변경 사항 반영
- `mcp` 2.x 버전이 설치되면서 기존 `FastMCP`가 `MCPServer`로 변경됨.
- import 경로를 `from mcp.server.mcpserver import MCPServer`로 수정함.
- 클라이언트 스키마 조회 속성도 `camelCase`(`inputSchema`)에서 `snake_case`(`input_schema`)로 변경됨.

### 2. OAuth 인증 및 실행 환경 분리
- `uv` 가상환경(`.venv`)에서 패키지를 설치했으므로 전역 파이썬 대신 `uv run python`으로 실행해야 모듈을 인식함.
- `stdio` 모드는 백그라운드 입력 대기 상태이므로 브라우저 인증 창이 자동으로 열리지 않음. 별도 명령어로 `get_credentials()`를 1회 실행해 `token.json`을 미리 생성함.
- `SCOPES` 설정 시 도메인 뒤에 `/auth/gmail.modify`, `/auth/calendar` 권한 경로를 명시해야 함.

## 검증 결과
- `test_client.py`를 작성해 `stdio` 세션 연결 및 도구 5종의 JSON Schema 노출을 확인함.
- 메일 검색 후 추출한 ID로 본문 조회, 테스트 메일 발송, 일정 등록 및 재조회까지 정상 작동을 확인함.
- 설치 가이드(`INSTALL.md`)와 명세(`README.md`)를 정리하고 GitHub `v1.0.0` 릴리즈를 발행함.
