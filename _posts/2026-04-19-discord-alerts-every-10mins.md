---
title: Discord 매 10분 자동 메시지 전송 기능 구현
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

## 필수 설치 및 환경 구성
프로젝트 폴더에서 아래 명령어 순서대로 실행.

### 1. npm 프로젝트 초기화

```
npm init -y
```

### 2. 필요한 라이브러리 설치 (discord.js, node-cron)

```
npm install discord.js node-cron
```

### 3. 개발용 라이브러리 및 타입 정의 설치

```
npm install -D typescript ts-node @types/node @types/node-cron
```

### 4. TypeScript 설정 파일 생성

```
npx tsc --init
```

## 전체 소스 코드 (discord-bot-gemini.ts)

``` typescript
import { Client, GatewayIntentBits, TextChannel } from 'discord.js';
import cron from 'node-cron';

// 1. 봇 클라이언트 설정const client = new Client({
  intents: [GatewayIntentBits.Guilds],
});

// 설정값 
const TOKEN = 'YOUR_BOT_TOKEN_HERE';const CHANNEL_ID = 'YOUR_CHANNEL_ID_HERE';

client.once('ready', () => {
  console.log(`Bot 로그인 성공: ${client.user?.tag}`);

  // 2. 10분 간격 스케줄링 설정 (*/10 * * * *)
  cron.schedule('*/10 * * * *', async () => {
    try {
      const channel = await client.channels.fetch(CHANNEL_ID);

      if (channel instanceof TextChannel) {
        const now = new Date();
        const hours = now.getHours();
        const minutes = now.getMinutes().toString().padStart(2, '0');

        // 메시지 전송
        await channel.send(`현재 시간은 ${hours}:${minutes}임! (10분 간격 알림)`);
        
        // 터미널 로그 출력
        console.log(`[알림] ${hours}:${minutes} 메시지 전송 완료`);
      }
    } catch (error) {
      console.error('메시지 전송 중 오류 발생:', error);
    }
  });
});

client.login(TOKEN);
```

## 실행 방법
터미널에서 아래 명령어로 봇 실행.

```
npx ts-node discord-bot-gemini.ts
```

## 사전 준비 사항

* 개발자 모드 활성화: 디스코드 설정 > 고급 > 개발자 모드 켜기.
* 채널 ID 복사: 대상 채널 우클릭 > 채널 ID 복사 후 코드의 CHANNEL_ID에 붙여넣기.
* 봇 토큰: 디스코드 개발자 포털에서 봇 생성 후 토큰 복사하여 TOKEN에 붙여넣기.
* 봇 초대: 서버에 봇을 초대하고 해당 채널에 메시지 보내기 권한 부여 확인.

## Screenshot 

<img width="578" height="308" alt="Image" src="https://github.com/user-attachments/assets/b06dd6b5-fa83-4dde-a62f-178341b154bc" />
