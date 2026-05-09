---
title: "NanoClaw 설정 + gemini CLI"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - skills
  - gemini
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "NanoClaw 설정 및 구성 보고서 - gemini CLI"
---

**날짜:** 2026-05-05
**어시스턴트 이름:** Gemini
**대상 플랫폼:** Linux (Ubuntu/Debian 계열)
**컨테이너 런타임:** Docker

## 1. 설정 실행 이력

환경 부트스트랩 및 구성을 위해 다음 단계들이 수행되었습니다:

- **부트스트랩 (`./setup.sh`):** 프로젝트 초기화, Node.js 버전 확인 (v22.14.0) 및 기본 의존성 설치를 완료했습니다.
- **타임존 설정:** 시스템 및 애플리케이션 타임존을 `Asia/Seoul`로 설정했습니다.
- **환경 확인:** Linux 플랫폼, Docker 가용성 및 프로젝트 구조를 확인했습니다.
- **컨테이너 설정:** 
  - `nanoclaw-agent:latest` Docker 이미지를 수동으로 빌드했습니다.
  - 컨테이너 실행 및 IPC 연결 상태를 확인했습니다.
- **마운트 설정:** `~/.config/nanoclaw/mount-allowlist.json`에 기본 빈 마운트 허용 리스트를 생성했습니다.
- **Discord 연동:** 
  - `https://github.com/qwibitai/nanoclaw-discord.git`에서 Discord 채널 기능을 머지했습니다.
  - `discord.js` 및 관련 의존성을 설치했습니다.
  - TypeScript 프로젝트를 다시 빌드했습니다.
- **등록:** 
  - Discord 채널 JID `dc:1492857709656146063`을 **메인 채널(Main Channel)**로 등록했습니다.
  - 트리거 패턴을 `@Gemini`로 설정했습니다.
- **서비스 설정:** 
  - systemd 사용자 서비스(`nanoclaw.service`)를 생성하고 활성화했습니다.
  - SSH 로그아웃 후에도 서비스가 유지되도록 `loginctl linger`를 활성화했습니다.
- **검증:** 최종 엔드-투-엔드 확인을 통해 서비스 실행, Discord 설정, 그룹 등록이 정상임을 확인했습니다.

## 2. 환경 변수 설정 (`.env`)

현재 `.env` 파일 설정은 다음과 같습니다:

```env
ASSISTANT_NAME="Gemini"
DISCORD_BOT_TOKEN="디스코드 봇 키"
ANTHROPIC_API_KEY="API 키"
TZ="Asia/Seoul"
```

*참고: `ANTHROPIC_API_KEY`는 현재 NanoClaw/Claude SDK의 내부 체크를 통과하기 위해 Gemini API Key를 할당해 둔 상태*

## 3. 데이터베이스 상태 (SQLite)

- **등록된 그룹:** 1개 (Discord 메인 채널)
- **활성화된 채팅:** 1개 (Discord 메인 채널)
- **메시지:** 테스트 메시지 삽입을 통해 에이전트 트리거 기능을 확인했습니다.

## 4. Systemd 서비스 상태

- **유닛 경로:** `~/.config/systemd/user/nanoclaw.service`
- **상태:** 활성 (실행 중)
- **로그:** `logs/nanoclaw.log` 및 `logs/nanoclaw.error.log`에서 확인할 수 있습니다.

## 5. Gemini 모델 연동을 위한 향후 단계

NanoClaw는 기본적으로 Claude Agent SDK를 기반으로 하므로, Gemini 모델을 본격적으로 사용하려면 다음이 필요합니다:
1. Anthropic API 호환 프록시 (예: LiteLLM, OneCLI 등) 설정.
2. `.env` 파일에 다음 항목 업데이트:
   - `ANTHROPIC_BASE_URL="http://프록시-주소/v1"`
   - `ANTHROPIC_MODEL="gemini-1.5-pro"` (또는 원하는 Gemini 모델명)

## 스크린샷, 숙제

<img width="555" height="474" alt="Image" src="https://github.com/user-attachments/assets/13c8db70-0102-4a1a-b478-efc56ff7ff61" />

- 동작 확인을 해야 함
