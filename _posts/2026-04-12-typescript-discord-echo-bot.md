---
title: Discord Echo 봇 환경 구축 및 실행
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

- 디스코드와 Node.js를 연동하여 사용자의 메시지를 그대로 받아치는(Echo) 기초 시스템 구축.

------------------------------

## 작업순서

### 1. 디스코드 개발자 설정 (Portal)
봇의 두뇌가 될 가상 계정을 생성하고 권한을 설정함.

* 애플리케이션 생성: Discord Developer Portal에서 New Application 생성함.
* 봇 토큰 발급: Bot 메뉴에서 Reset Token을 눌러 고유 토큰을 복사함. (유출 주의)
* 권한 활성화: Bot 메뉴 내 Message Content Intent를 반드시 ON으로 설정함. (메시지 읽기 필수)
* 서버 초대: OAuth2 -> URL Generator에서 bot 및 Administrator 체크 후 생성된 URL로 내 서버에 봇을 초대함.

### 2. Node.js 프로젝트 설정 (Ubuntu)
TypeScript 실행 환경을 CommonJS 방식으로 안정화함.

``` bash
# 1. package.json에서 "type": "module" 삭제 (에러의 근원 제거)
sed -i '/"type": "module"/d' package.json

# 2. tsconfig.json 새로 작성
cat <<EOF > tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "CommonJS",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "strict": true,
    "types": ["node"]
  }
}
EOF

```

* 패키지 설치: discord.js, ts-node, typescript, @types/node를 로컬에 설치함.
* CommonJS 전환: package.json에서 "type": "module"을 삭제하여 최신 Node.js(v22+)와의 충돌을 방지함.
* 설정 파일: tsconfig.json을 생성하여 Node.js 타입을 명시적으로 지정함.

### 3. Echo 봇 코드 구현 (echo.ts)
메시지 수신 이벤트를 처리하는 핵심 로직을 작성함.

``` typescript
import { Client, GatewayIntentBits } from 'discord.js';

// 1. 봇 클라이언트 설정 (어떤 이벤트를 들을지 결정)
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,           // 서버 접속용
    GatewayIntentBits.GuildMessages,    // 서버 메시지 수신용
    GatewayIntentBits.MessageContent,   // 메시지 내용 읽기용 (중요!)
  ],
});

// 2. 봇이 준비되었을 때 실행
client.once('ready', () => {
  console.log(`로그인 성공: ${client.user?.tag}`);
});

// 3. 메시지 수신 및 Echo 로직
client.on('messageCreate', (message) => {
  // 봇이 자기 자신의 메시지에 응답하는 무한 루프 방지
  if (message.author.bot) return;

  console.log(`[${message.author.username}]: ${message.content}`);

  // 사용자가 보낸 말을 그대로 따라 함
  message.reply(`에코: ${message.content}`);
});

// 4. 로그인
client.login('복사한_토큰');

```

* 인텐트 설정: 봇이 서버와 메시지 내용에 접근할 수 있도록 GatewayIntentBits를 구성함.
* 이벤트 리스너: messageCreate 이벤트를 통해 사용자 메시지를 감지함.
* 무한 루프 방지: message.author.bot 조건을 확인하여 봇 자신의 메시지는 무시하도록 처리함.
* 답장 실행: message.reply() 함수를 사용하여 수신된 텍스트를 그대로 반환함.

### 4. 실행 및 결과 확인

* 실행 명령어: `npx ts-node echo.ts`로 봇을 구동함.
* 로그인 확인: 터미널에 로그인 성공: 봇이름#번호 문구가 뜨는지 확인함.
* 동작 테스트: 디스코드 채팅창에 메시지 입력 시 봇이 즉각적으로 답장하는지 확인함.

------------------------------


## 트러블슈팅 목록

------------------------------
### Discord & Node.js 트러블슈팅 가이드## 1. Node.js 버전 및 모듈 충돌 (ERR_UNKNOWN_FILE_EXTENSION)

* 증상: .ts 파일을 실행할 때 "Unknown file extension" 에러 발생함.
* 원인: 최신 Node.js(v22+) 환경에서 ESM("type": "module") 설정과 ts-node가 충돌함.
* 해결: package.json에서 "type": "module" 줄을 삭제하고, tsconfig.json의 module 설정을 CommonJS로 변경하여 안정화함.

### 2. 권한 거부 에러 (EACCES)

* 증상: npm install -g 사용 시 시스템 폴더 접근 권한 에러 발생함.
* 원인: 우분투 시스템 폴더(/usr/local/bin)에 대한 쓰기 권한이 없음.
* 해결: -g(전역) 설치 대신 프로젝트 폴더 내 로컬 설치(npm install --save-dev)를 사용하고 npx로 실행함.

### 3. 인증 실패 에러 (401: Unauthorized / invalid Authorization header)

* 증상: npx ts-node echo.ts 실행 시 로그인 단계에서 튕김.
* 원인: 봇 토큰 값이 잘못되었거나 복사 과정에서 오타가 발생함.
* 해결: 개발자 포털의 Bot 메뉴에서 Reset Token을 눌러 새 토큰을 정확히 복사하여 client.login()에 붙여넣음. (OAuth2의 Client Secret과 혼동 금지)

### 4. 메시지 읽기 실패 (봇이 반응하지 않음)

* 증상: 로그인은 성공했으나 채팅을 쳐도 봇이 대답하지 않음.
* 원인: Message Content Intent 권한이 비활성화되어 메시지 본문을 읽지 못함.
* 해결: 개발자 포털 Bot 메뉴 하단에서 Message Content Intent를 ON으로 켜고 프로그램을 재시작함.

### 5. 봇 미표시 (서버 목록에 없음)

* 증상: 봇은 켜져 있는데 디스코드 서버 멤버 목록에 보이지 않음.
* 원인: 봇 계정만 만들고 실제 서버로 초대(Invite)하지 않음.
* 해결: OAuth2 -> URL Generator에서 bot 체크 후 생성된 링크를 브라우저에 입력하여 서버에 승인 절차를 거침.

## 스크린샷
<img width="811" height="883" alt="Image" src="https://github.com/user-attachments/assets/f398cb9e-a7e4-429e-9d63-e2d469312f91" />

<img width="872" height="123" alt="Image" src="https://github.com/user-attachments/assets/dff0e866-aa9d-4eab-9c8d-d86c1ff49887" />


## Related Posts
- [Docker sandbox 실행 + Discord Bot 통합]({% link _posts/2026-04-12-docker-sandboxing-discord-bot.md %})
- [Hyper는 프론트엔드 기술로 데스크톱 앱을 만드는 Electron 프레임워크]({% link _posts/2026-04-12-typescript-hyper-project.md %})
