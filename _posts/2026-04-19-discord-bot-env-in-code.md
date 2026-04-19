---
title: Discord bot 환경 변수 적용 및 타입 에러를 수정
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - discord
  - node.js
  - typescript
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 추가 패키지 설치
환경 변수 로드를 위해 dotenv 설치 필수.

``` bash
npm install dotenv
npm install -D @types/dotenv
```

## .env 파일 설정
프로젝트 루트 폴더에 생성. 토큰 유출 방지를 위해 절대 공유 금지.

```
DISCORD_TOKEN=봇_토큰_입력
CHANNEL_ID=메시지_보낼_채널_ID_입력
```

## 최종 코드 (discord-bot-gemini.ts)
TypeScript 타입 에러(string | undefined) 해결 위해 논리적 단언(!) 및 체크 로직 추가.

``` typescript
// ...
import dotenv from 'dotenv';

// .env 파일 로드
dotenv.config();

// 환경 변수 할당
const TOKEN = process.env.DISCORD_TOKEN;
const CHANNEL_ID = process.env.CHANNEL_ID;

client.once('ready', () => {
  console.log(`Bot 로그인 성공: ${client.user?.tag}`);

  // 10분 간격 실행
  cron.schedule('*/10 * * * *', async () => {
    try {
      // CHANNEL_ID가 없을 경우 예외 처리
      if (!CHANNEL_ID) {
        console.error('설정된 채널 ID가 없음');
        return;
      }

      // 타입 에러 방지 위해 CHANNEL_ID 뒤에 ! 추가
      const channel = await client.channels.fetch(CHANNEL_ID!);
```

## 실행 및 주의사항

* 실행: `npx ts-node discord-bot-gemini.ts`
* 에러 해결: CHANNEL_ID! 구문은 해당 변수가 확실히 문자열임을 컴파일러에게 알림.
* 보안: .gitignore 파일에 .env를 추가해 원격 저장소 업로드 방지.

