---
title: 디스코드 출석체크 봇 2 - Event Driven Architecture
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - discord
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


디스코드 봇은 전형적인 이벤트 기반 아키텍처(Event-Driven Architecture)로 동작함.
봇 서버가 먼저 뭘 하는 게 아니라, 디스코드 서버에서 발생하는 '사건'을 기다렸다가 반응하는 구조임.

## 1. 리액티브(Reactive)한 동작 방식
봇은 실행되는 동안 웹소켓(WebSocket) 연결을 유지함.

디스코드에서 발생하는 모든 일(메시지, 버튼 클릭, 입장/퇴장 등)은 '게이트웨이'를 통해 봇에게 패킷(Packet) 형태로 전달됨.

* 대기: 봇은 CPU를 거의 쓰지 않고 입력을 기다림.
* 이벤트 발생: 유저가 /출석 입력.
* 콜백 실행: 봇 프로그램 내의 `client.on('interactionCreate', ...)` 로직이 깨어나서 돌아감.

## 2. 주요 이벤트 카테고리
네가 말한 '대응'은 크게 3가지 상황으로 나뉨.

| 구분 | 설명 | 예시 |
|---|---|---|
| 명령어 대응 | 유저가 의도적으로 봇을 부를 때 | /출석, /순위, /도움말 |
| 상태 변화 대응 | 서버의 환경이 변할 때 | 유저 입장(환영 인사), 채널 삭제 |
| 데이터 감시 | 모든 메시지를 읽고 필터링할 때 | 비속어 차단, 자동 응답 (봇 권한 필요) |

## 3. 예외적인 '능동적' 동작 (Scheduled Task)
채팅 메시지 응답 외에 봇이 먼저 움직이는 경우도 있긴 함. 보통 스케줄러(Cron)를 사용함.

* 예시: "매일 아침 9시에 출석하지 않은 인원에게 알림 보내기", "매 정시마다 주식 시세 전송"
* 이때는 이벤트 대기와 별개로 서버 자체 타이머에 의해 코드가 실행됨.

## 소프트웨어 구조 관점에서의 팁
봇이 커질수록 index.ts에 모든 이벤트를 다 넣으면 관리가 안 됨.
그래서 보통 아래와 같은 구조로 설계함.

* Handlers: 이벤트(Event)와 명령어(Command)를 분리해서 로딩하는 로직.
* Services: DB 처리나 외부 API 연동 등 핵심 비즈니스 로직.
* Utils: 시간 계산, 마크다운 포맷팅 등 공통 함수.

## 결론
봇은 "이벤트 수신기"임.
그래서 개발할 때 가장 중요한 건 "이 이벤트가 왔을 때, 유저에게 어떤 결과를 '어떻게(임베드, 버튼 등)' 보여줄 것인가"와 "그 과정에서 데이터를 '어떻게(DB)' 저장할 것인가" 이 두 가지 핵심 설계에 집중 필요

## Related Posts
- [discord-bot-gemini 4.22 수정]({% link _posts/2026-04-22-discord-bot-gemini-fix.md %})
- [discord.js의 Interaction 타입]({% link _posts/2026-04-25-discord-js-interacraction-button.md %})
- [discord.js의 Client 이벤트 목록]({% link _posts/2026-04-25-discord-js-event-list.md %})
- [Docker sandbox 실행 + Discord Bot 통합]({% link _posts/2026-04-12-docker-sandboxing-discord-bot.md %})
- [Discord Echo 봇 환경 구축 및 실행]({% link _posts/2026-04-12-typescript-discord-echo-bot.md %})
- [Discord 매 10분 자동 메시지 전송 기능 구현]({% link _posts/2026-04-19-discord-alerts-every-10mins.md %})
- [Discord bot 환경 변수 적용 및 타입 에러를 수정]({% link _posts/2026-04-19-discord-bot-env-in-code.md %})
- [디스코드 봇 개발 주제 모음]({% link _posts/2026-04-19-discord-bot-toy-project-list.md %})
- [디스코드 출석체크 봇 1 - 초기설정, 로그인 기능]({% link _posts/2026-04-24-my-attendance-bot-step1.md %})
- [디스코드 출석체크 봇 4 - db기능 통합하고 모듈화]({% link _posts/2026-04-26-prisma-db-handling.md %})
- [디스코드 출석체크 봇 3 - db기능만 떼어서 실험]({% link _posts/2026-04-26-prisma-db-init-test.md %})
