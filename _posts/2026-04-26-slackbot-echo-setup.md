## Slack App 설정 (Web)

## 1. 앱 생성 및 토큰 발급

* App 생성: [Slack API](https://api.slack.com/apps)에서 From Scratch로 생성함.
* 봇 토큰(xoxb-): OAuth & Permissions에서 아래 권한 추가 후 Install to Workspace 눌러 발급받음.
* chat:write, app_mentions:read, im:history, message.channels
* 소켓 모드: Settings > Socket Mode를 On으로 켬.
* 앱 토큰(xapp-): 소켓 모드 켤 때 혹은 Basic Information 하단에서 connections:write 권한으로 생성함.

## 2. 이벤트 구독

* Event Subscriptions: 기능을 On으로 설정함.
* 이벤트 추가: Subscribe to bot events 섹션에 message.channels, message.im 추가함.
* 권한 갱신: 설정 변경 후 상단 노란 바의 reinstall your app을 눌러야 실제 작동함.

## 3. 앱 홈 설정

* App Home > Messages Tab에서 Allow users to send... 체크박스 켬. (봇에게 DM 보내기 위함)


## 💻 로컬 개발 환경 (Bun + TS)

## 1. 프로젝트 초기화

``` bash
mkdir slack-echo-bot && cd slack-echo-bot
bun init -y
bun add @slack/bolt
```

## 2. 환경 변수 설정 (.env)
보안을 위해 토큰을 파일로 관리함.

```
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-token
```

## 3. 에코 봇 코드 작성 (index.ts)

``` typescript
import { App } from "@slack/bolt";
const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  appToken: process.env.SLACK_APP_TOKEN,
  socketMode: true,
});

app.message(async ({ message, say }) => {
  if ("text" in message && message.text) {
    await say(`[Bun Echo]: ${message.text}`);
  }
});
await app.start();
console.log("⚡️ Slack Echo Bot 실행 중!");
```

## 실행 및 테스트

* 실행: `bun index.ts` 입력함.
* 초대: 사용할 채널에 /invite @봇이름으로 봇 초대함. (DM은 필요 없음)
* 확인: 메시지 보내서 봇이 대답하면 성공임.


## 코드 분석
### 1. app.message(...)

* 역할: 슬랙에서 발생하는 메시지 이벤트를 감지하는 리스너임.
* 작동: 채널이나 DM에 새로운 메시지가 올라오면 이 함수 내부의 코드가 실행됨.
* 첫 번째 인자 ("" 생략): 원래는 특정 단어를 넣어서 필터링할 수 있지만, 지금처럼 비워두면 모든 텍스트 메시지에 반응함.

### 2. async ({ message, say }) => { ... }

* async: 메시지 전송은 네트워크 통신이므로 비동기로 처리함.
* message: 사용자가 보낸 메시지의 상세 정보(텍스트, 보낸 사람, 시간 등)가 담긴 객체임.
* say: 해당 메시지가 올라온 채널에 답장을 보낼 수 있게 해주는 편리한 함수임.

### 3. if ("text" in message && message.text)

* 타입 가드(Type Guard): 슬랙 메시지 이벤트는 종류가 다양함(파일 업로드, 메시지 삭제 등).
* 역할: "이 이벤트가 글자(text)를 포함하고 있는가?"를 검사함. 텍스트가 있을 때만 코드가 실행되도록 해서 에러를 방지함.

### 4. await say(\[Bun Echo]: ${message.text}`);`

* 실제 응답: 사용자가 보낸 텍스트(message.text) 앞에 [Bun Echo]:를 붙여서 다시 채널에 전송함.
* await: 메시지가 안전하게 전송될 때까지 기다림.

