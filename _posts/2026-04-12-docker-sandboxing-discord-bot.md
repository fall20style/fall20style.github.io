---
title: Docker sandbox 실행 + Discord Bot 통합
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - typescript
  - docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


- NanoClaw의 sandboxing 구조를 흉내내기
- 기존의 Docker 샌드박스 실행 로직(runAgent)을 디스코드 봇 인터페이스와 결합
- 외부에서 시스템을 제어하는 구조


------------------------------

## 1. 통합 시스템 구조

* 인터페이스: 디스코드 채팅창을 통해 사용자 명령 수신함.
* 컨트롤러: Node.js(discord.js)가 트리거 워드(@Andy) 감지 시 샌드박스 가동함.
* 실행기: Docker 컨테이너가 격리된 환경에서 Bash 명령을 수행함.
* 피드백: 작업 완료 후 생성된 파일 목록과 종료 코드를 디스코드에 답장함.

## 2. 핵심 코드 통합 (discord-bot.ts)

* 비동기 처리: async/await를 사용하여 Docker 작업이 끝날 때까지 디스코드 응답을 대기(Wait)함.
* 상태 알림: message.reply()로 작업 시작을 알리고, 완료 후 edit()을 통해 기존 메시지를 결과값으로 업데이트함.
* 권한 위임: `--user $(id -u):$(id -g)` 옵션으로 호스트와 컨테이너 간 파일 쓰기 권한 문제를 해결함.


``` typescript
import { Client, GatewayIntentBits, Message } from 'discord.js';
import { spawn } from 'child_process';
import * as path from 'path';
import * as fs from 'fs';

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
  ],
});

const DISCORD_TOKEN = 'YOUR_BOT_TOKEN';
const TRIGGER_WORD = '@Andy';
const hostPath = path.resolve(__dirname, 'workspace');

// 기존 Docker 실행 로직 (Promise 버전)
const runAgent = (command: string): Promise<{ code: number | null, newFiles: string[] }> => {
  return new Promise((resolve) => {
    const filesBefore = new Set(fs.readdirSync(hostPath));
    
    const docker = spawn('docker', [
      'run', '--rm', '-i',
      '-v', `${hostPath}:/app`,
      '-w', '/app',
      '--user', `${process.getuid?.()}:${process.getgid?.()}`,
      'node:20-slim',
      'sh', '-c', command
    ]);

    docker.on('close', (code) => {
      const filesAfter = fs.readdirSync(hostPath);
      const newFiles = filesAfter.filter(file => !filesBefore.has(file));
      resolve({ code, newFiles });
    });
  });
};

client.on('messageCreate', async (message: Message) => {
  if (message.author.bot) return;

  // 트리거 워드 확인 (@Andy)
  if (message.content.startsWith(TRIGGER_WORD)) {
    const userCommand = message.content.replace(TRIGGER_WORD, '').trim();
    
    if (!userCommand) {
      return message.reply('명령어를 입력해 주세요.');
    }

    const loadingMsg = await message.reply(`[NanoClaw] 샌드박스 가동 중: \`${userCommand}\`...`);

    try {
      // Docker 실행 및 결과 대기
      const result = await runAgent(userCommand);
      
      let response = `작업 종료 (Code: ${result.code})`;
      
      // 새로 생성된 파일이 있다면 알림
      if (result.newFiles.length > 0) {
        response += `\n생성된 파일: ${result.newFiles.join(', ')}`;
        
        // (선택 사항) 생성된 파일을 디스코드에 직접 업로드
        // await message.reply({ files: result.newFiles.map(f => path.join(hostPath, f)) });
      }

      await loadingMsg.edit(response); // 기존 메시지 수정하여 결과 표시
    } catch (error) {
      await loadingMsg.edit(`에러 발생: ${error}`);
    }
  }
});

if (!fs.existsSync(hostPath)) fs.mkdirSync(hostPath);
client.login(DISCORD_TOKEN);

```

## 3. 사전 환경 준비

* 이미지 확보: 실행 속도 향상을 위해 `docker pull node:20-slim` 명령어로 베이스 이미지를 미리 내려받음.
* 작업 폴더: 호스트 시스템에 `workspace` 폴더를 생성하여 컨테이너와 볼륨 마운트 통로를 마련함.
* 토큰 보안: 디스코드 개발자 포털에서 발급받은 봇 토큰을 client.login()에 정확히 입력함.

## 4. 트러블슈팅 및 주의사항

* 응답 지연: 최초 실행 시 이미지를 내려받는 과정에서 멈춘 것처럼 보일 수 있으니 `docker pull` 확인 필수임.
* 인텐트 설정: 디스코드 관리자 페이지에서 Message Content Intent가 켜져 있지 않으면 명령어를 인식하지 못함.
* 파일 감지: `fs.readdirSync`를 이용해 작업 전후 폴더 상태를 비교, 에이전트가 만든 결과물만 골라냄.


## 스크린샷

<img width="534" height="332" alt="Image" src="https://github.com/user-attachments/assets/d03f1132-6b45-4241-acb4-3f107cb0800a" />

<img width="491" height="36" alt="Image" src="https://github.com/user-attachments/assets/ad1f5e50-27cd-47cd-833a-5d0a4a9e20d4" />

## 향후 할 일
- 에이전트가 생성한 이미지나 PDF 파일을 채팅창에 직접 업로드하는 기능

