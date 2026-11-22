---
title: Python으로 Gmail, Google Calendar MCP 서버 구축함 (Mock 모드 포함)
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

# Python으로 Gmail, Google Calendar MCP 서버 구축함 (Mock 모드 포함)

직접 만든 Python 에이전트에 연동하기 위해 Gmail과 Google Calendar를 제어하는 MCP(Model Context Protocol) 서버를 구현함. OAuth 인증 없이 에이전트 도구 호출 루프를 테스트할 수 있도록 인메모리 Mock 백엔드를 통합함.

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

## 이중 백엔드 아키텍처 (Real / Mock 전환)

에이전트 개발 및 테스트 시 실제 메일이 발송되거나 캘린더 일정이 오염되는 문제를 막기 위해 환경변수(`USE_MOCK`)로 동작 모드를 분리함.

- **인터페이스 유지**: `server.py`가 노출하는 도구 이름, 파라미터 스키마, 반환 문자열 포맷을 두 모드에서 동일하게 유지함.
- **Stateful Mock (`mock_backend.py`)**: `MockGoogleWorkspace` 클래스가 메모리 리스트(`emails`, `events`)를 유지함. `send_email`이나 `create_calendar_event` 호출 시 가상 ID를 발급해 메모리에 추가하며, 직후 조회 도구 호출 시 변경된 상태가 그대로 반영됨.

```text
USE_MOCK="false" (기본값) ──▶ Google Workspace Cloud API (실제 연동)
USE_MOCK="true"            ──▶ MockGoogleWorkspace (인메모리 테스트)
```

## 진행 과정 및 트러블슈팅

### 1. SDK v2.x 변경 사항 반영
- `mcp` 2.x 버전이 설치되면서 기존 `FastMCP`가 `MCPServer`로 변경됨.
- import 경로를 `from mcp.server.mcpserver import MCPServer`로 수정함.
- 클라이언트 스키마 조회 속성도 `camelCase`(`inputSchema`)에서 `snake_case`(`input_schema`)로 변경됨.

### 2. OAuth 인증 및 실행 환경 분리
- `uv` 가상환경(`.venv`)에서 패키지를 설치했으므로 전역 파이썬 대신 `uv run python`으로 실행해야 모듈을 인식함.
- `stdio` 모드는 백그라운드 입력 대기 상태이므로 브라우저 인증 창이 자동으로 열리지 않음. 별도 명령어로 `get_credentials()`를 1회 실행해 `token.json`을 미리 생성함.
- `SCOPES` 설정 시 도메인 뒤에 `/auth/gmail.modify`, `/auth/calendar` 권한 경로를 명시해야 함.

### 3. 서브프로세스 환경변수 전파
- 클라이언트가 `StdioServerParameters`로 `server.py`를 서브프로세스로 실행할 때 부모 프로세스의 환경변수(`USE_MOCK`)가 자동 상속되지 않는 문제가 발생함.
- 클라이언트 설정에 `env=dict(os.environ)`을 명시해 환경변수가 서버 프로세스로 전달되도록 해결함.

## 검증 결과
- **Real 모드**: 실제 Gmail 검색, 본문 디코딩, 메일 발송, 캘린더 일정 등록 및 재조회를 검증하고 `v1.0.0` 릴리즈를 발행함.
- **Mock 모드**: `\$env:USE_MOCK="true"` 설정 후 인증 파일 없이 5종 도구의 상태 변화(`mock_msg_001` 본문 조회, 메일 발송 후 재검색, 일정 추가 후 최상단 반영)를 검증하고 `v1.1.1` 및 아키텍처 문서를 정리함.
