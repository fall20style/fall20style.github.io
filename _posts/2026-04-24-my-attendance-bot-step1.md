---
title: 디스코드 출석체크 봇 1 - 초기설정, 로그인 기능
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - bun
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- 디스코드 출석체크 봇의 초기 환경 설정
- 로그인 구현 내용

## 🤖 Discord 출석체크 봇 프로젝트 (TypeScript)

## 1. 프로젝트 개요

* 언어: TypeScript
* 런타임: Bun (또는 Node.js)
* 라이브러리: discord.js
* 목적: 서버 내 슬래시 명령어(/출석)를 통한 출석체크 시스템 구축

## 2. 초기 환경 설정

## 필요한 패키지 설치

* 프로젝트 초기화

```
bun init -y
```
* 라이브러리 설치

```
bun add discord.js dotenv
```

* 개발 도구 설치

```
bun add -d typescript @types/node
```

## .env 설정
보안이 필요한 토큰과 ID 정보를 파일로 분리하여 관리함.

```
DISCORD_TOKEN=your_bot_token_here
CLIENT_ID=your_application_id_here
```


## 3. 핵심 SW 구조

## 인터페이스 레이어 (Discord API)

* Client: 디스코드 서버와 통신하는 메인 객체임.

* Intents: 봇이 사용할 권한(Guilds 등)을 명시함.

* REST & Routes: 슬래시 명령어를 디스코드 서버에 등록하기 위해 사용함.

## 어플리케이션 레이어 (Event Handler)

* ClientReady: 봇 로그인 시 최초 1회 실행되며 명령어 등록을 수행함.

* InteractionCreate: 유저가 /출석 입력 시 이를 감지하고 응답 로직을 실행함.


## 4. 구현 코드 (index.ts)

``` typescript
import { 
  Client, 
  GatewayIntentBits, 
  Events, 
  SlashCommandBuilder, 
  REST, 
  Routes 
} from 'discord.js';
// 환경 변수 로드
const TOKEN = process.env.DISCORD_TOKEN;
const CLIENT_ID = process.env.CLIENT_ID;

if (!TOKEN || !CLIENT_ID) {
  console.error("❌ 설정 에러: .env 파일을 확인해야 함.");
  process.exit(1);
}

// 1. 명령어 정의const commands = [
  new SlashCommandBuilder()
    .setName('출석')
    .setDescription('오늘의 출석을 기록함!')
    .toJSON(),
];

const client = new Client({ intents: [GatewayIntentBits.Guilds] });

// 2. 봇 준비 및 명령어 등록
client.once(Events.ClientReady, async (readyClient) => {
  const rest = new REST({ version: '10' }).setToken(TOKEN);
  try {
    console.log('⏳ 명령어 등록 중...');
    await rest.put(Routes.applicationCommands(CLIENT_ID), { body: commands });
    console.log(`✅ 로그인 성공: ${readyClient.user.tag}`);
  } catch (error) {
    console.error('❌ 등록 실패:', error);
  }
});

// 3. 인터랙션 처리
client.on(Events.InteractionCreate, async (interaction) => {
  if (!interaction.isChatInputCommand()) return;

  if (interaction.commandName === '출석') {
    await interaction.reply({
      content: `✅ **${interaction.user.username}**님 출석 완료됨!`,
    });
  }
});

client.login(TOKEN);
```

## 5. 실행 및 결과

* 실행 명령어: `bun run index.ts`
* 동작 확인: 디스코드 채팅창에 / 입력 시 출석 명령어가 노출됨.
* 응답 확인: 유저 클릭 시 봇이 "출석 완료됨!" 메시지를 전송함.

