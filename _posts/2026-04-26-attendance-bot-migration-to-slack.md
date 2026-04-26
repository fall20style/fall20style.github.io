---
title: Discord 출석체크 봇을 Slack으로 마이그레이션하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - slack
  - bun
  - prisma
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: 기존 Discord용으로 제작된 출석체크 봇을 Slack 환경으로 마이그레이션한 과정을 정리.
---

기존에 Discord용으로 제작했던 출석체크 봇의 핵심 로직을 유지하면서, 플랫폼을 Slack으로 전환하는 작업을 진행. 런타임은 동일하게 Bun을 사용하며, 데이터베이스 관리를 위해 Prisma를 도입함.

## 1. 프로젝트 개요

* **목적**: Discord 봇의 출석체크 기능을 Slack으로 이식
* **주요 기능**:
    * `/출석`: 사용자별 일일 출석 기록 및 누적 횟수 관리
    * `/버튼`: Slack의 Interactive Components(버튼) 동작 테스트
    * **Echo**: 일반 메시지에 대한 에코 응답 (기존 기능 유지)
* **기술 스택**: Bun, Slack Bolt (@slack/bolt), Prisma, SQLite

## 2. 주요 작업 내용

### Prisma를 이용한 DB 설계
기존 Discord ID를 사용하던 구조를 Slack User ID에 맞게 재사용함. SQLite를 사용하여 가볍게 데이터를 관리함.

```prisma
model User {
  id               String   @id
  lastAttendance   DateTime
  attendanceCount  Int      @default(0)
}
```

### Slack Bolt 프레임워크 적용
Discord.js와는 다른 Slack Bolt의 이벤트 처리 방식을 적용함. `app.command()`와 `app.action()`을 사용하여 슬래시 명령어와 버튼 클릭 이벤트를 처리함.

### 코드 구조 분리
* `index.ts`: 앱 초기화 및 이벤트 등록
* `bot-service.ts`: 비즈니스 로직 (명령어 및 액션 핸들러)
* `db-mod.ts`: 데이터베이스 접근 로직 (Prisma Client 사용)

## 3. 핵심 구현 코드

### 출석 체크 로직 (db-mod.ts)
중복 출석 체크를 위해 `lastAttendance` 날짜를 현재 날짜와 비교하고, 처음 출석하는 유저인 경우 `upsert`를 통해 데이터를 생성하거나 업데이트함.

```typescript
export async function processAttendance(userId: string) {
  const now = new Date();
  return await prisma.user.upsert({
    where: { id: userId },
    update: {
      lastAttendance: now,
      attendanceCount: { increment: 1 },
    },
    create: {
      id: userId,
      lastAttendance: now,
      attendanceCount: 1,
    },
  });
}
```

### Slack 인터랙션 (bot-service.ts)

Slack의 `blocks`를 활용하여 버튼을 전송하고, `action_id`를 통해 버튼 클릭을 감지.

```typescript
export async function handleButtonCommand({ command, ack, say }: any) {
    await ack();
    await say({
        blocks: [
            {
                type: "section",
                text: { type: "mrkdwn", text: "아래 버튼을 눌러보셈!" }
            },
            {
                type: "actions",
                elements: [
                    {
                        type: "button",
                        text: { type: "plain_text", text: "여기를 누르셈!" },
                        action_id: "confirm_button"
                    }
                ]
            }
        ]
    });
}
```

## 4. 실행 및 설정

1. **의존성 설치**: `bun install`
2. **DB 초기화**: `bunx prisma db push`
3. **환경 변수**: `.env` 파일에 `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN` 설정
4. **실행**: `bun run index.ts`

## 5. 마치며

Slack은 Discord에 비해 Block Kit을 통한 UI 구성이 매우 유연하다는 장점이 있음.

- 플랫폼 간의 이벤트 처리 방식 차이를 이해하고,
- Prisma를 통해 타입 안정성이 확보된 DB 레이어를 구축할 수 있었음.

## Related Posts
- [디스코드 출석체크 봇 1 - 초기설정, 로그인 기능]({% link _posts/2026-04-24-my-attendance-bot-step1.md %})
- [Prisma DB 핸들링 기초]({% link _posts/2026-04-26-prisma-db-handling.md %})
- [Slack Echo Bot 환경 구축]({% link _posts/2026-04-26-slackbot-echo-setup.md %})
---
