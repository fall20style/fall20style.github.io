---
title: node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - NanoClaw
  - Docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행
- 흉내 낸 최소 기능 예제(MVP).

### 1. 실습용 프로젝트 준비
먼저 실습할 폴더를 만들고 필요한 파일을 준비함.

```
mkdir nanoclaw-mini && cd nanoclaw-mini
touch index.js
mkdir workspace
```

### 2. Node.js 코드 작성 (index.js)

child_process의 spawn을 사용하여 Docker와 실시간으로 통신하는 구조임.

``` javascript
const { spawn } = require('child_process');const readline = require('readline');const path = require('path');
// 1. 키보드 입력을 받기 위한 인터페이스 설정const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
const runAgent = (command) => {
  console.log(`\n[Host] 에이전트에게 명령 전달: "${command}"`);

  // 2. NanoClaw 방식의 Docker 실행 옵션 조립
  const hostPath = path.resolve(__dirname, 'workspace');
  const containerPath = '/app';
  
  const docker = spawn('docker', [
    'run', '--rm',
    '-i',                             // 입출력을 위한 대화형 모드
    '-v', `${hostPath}:${containerPath}`, // 볼륨 마운트 (격리)
    '-w', containerPath,              // 작업 디렉토리
    '--user', `${process.getuid()}:${process.getgid()}`, // 권한 매핑
    'node:20-slim',                   // 실행 환경 (이미지)
    'sh', '-c', command               // 전달받은 명령 실행
  ]);

  // 3. 에이전트의 출력(Stdout)을 호스트 터미널에 표시
  docker.stdout.on('data', (data) => {
    process.stdout.write(`[Agent Out]: ${data}`);
  });

  // 4. 에러 발생 시 출력
  docker.stderr.on('data', (data) => {
    process.stderr.write(`[Agent Error]: ${data}`);
  });

  // 5. 종료 처리
  docker.on('close', (code) => {
    console.log(`[Host] 에이전트 작업 종료 (Exit Code: ${code})`);
    askCommand(); // 다시 입력 대기
  });
};
const askCommand = () => {
  rl.question('\n명령을 입력하세요 (예: touch hello.txt / ls / pwd): ', (answer) => {
    if (answer.toLowerCase() === 'exit') return rl.close();
    runAgent(answer);
  });
};

console.log("--- NanoClaw Mini 시작 ---");
askCommand();
```

### 3. 코드 포인트 분석

* readline: 사용자로부터 명령을 입력받는 채팅 인터페이스 역할을 수행함.
* spawn: exec와 달리 프로세스를 비동기 스트림으로 제어함. 에이전트가 긴 작업을 할 때 실시간 로그를 보여주기에 적합함.
* -i 옵션: 컨테이너와 표준 입출력을 유지하여 Node.js가 에이전트의 대답을 가로챌 수 있게 함.
* 권한 매핑: 앞서 실습한 대로 호스트 파일 시스템 접근 에러를 방지함.

## 4. 실행 및 테스트

```
node index.js
```

실행 후 다음 명령어들을 입력해보며 workspace 폴더의 변화를 관찰함.

1. `touch test.log` (파일 생성 확인)
2. `ls -l` (권한 확인)
3. `node -e "console.log('Hello from Inside!')"` (내부 런타임 확인)

이 구조를 확장하면 사용자 입력 대신 AI(Claude)의 응답을 command 자리에 넣어 NanoClaw를 완성할 수 있음.


## 에이전트 작업 종료 직후 공유 폴더(workspace)를 스캔, 새로 생긴 파일을 찾아내는 기능을 추가
fs.readdirSync를 활용해 작업 전후의 파일 목록을 비교하는 방식이 가장 확실함.

### 수정된 index.js (파일 감지 로직 추가)

```
const { spawn } = require('child_process');const readline = require('readline');const path = require('path');const fs = require('fs'); // 파일 시스템 모듈 추가
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
const hostPath = path.resolve(__dirname, 'workspace');
const runAgent = (command) => {
  // 1. 작업 시작 전 파일 목록 저장
  const filesBefore = new Set(fs.readdirSync(hostPath));
  
  const docker = spawn('docker', [
    'run', '--rm', '-i',
    '-v', `${hostPath}:/app`,
    '-w', '/app',
    '--user', `${process.getuid()}:${process.getgid()}`,
    'node:20-slim',
    'sh', '-c', command
  ]);

  docker.stdout.on('data', (data) => process.stdout.write(`[Agent]: ${data}`));

  docker.on('close', (code) => {
    console.log(`\n[Host] 작업 완료. 신규 파일 스캔 중...`);

    // 2. 작업 종료 후 파일 목록 비교
    const filesAfter = fs.readdirSync(hostPath);
    const newFiles = filesAfter.filter(file => !filesBefore.has(file));

    // 3. 결과 출력
    if (newFiles.length > 0) {
      console.log(`새 파일 발견: ${newFiles.join(', ')}`);
    } else {
      console.log(`새로 생성된 파일 없음.`);
    }

    askCommand();
  });
};
const askCommand = () => {
  rl.question('\n명령 입력: ', (answer) => {
    if (answer.toLowerCase() === 'exit') return rl.close();
    runAgent(answer);
  });
};
// workspace 폴더가 없으면 생성 if (!fs.existsSync(hostPath)) fs.mkdirSync(hostPath);

console.log("--- NanoClaw Mini (File Detector) 시작 ---");
askCommand();
```

### 구현 포인트

* Set(filesBefore): 비교 속도를 높이고 중복 체크를 쉽게 하기 위해 Set 자료구조를 사용함.
* filter 루프: 종료 시점의 폴더 상태를 다시 읽어(readdirSync), 이전 목록에 없던 파일만 골라냄.
* 실습 팁: `touch memo.txt`나 `node -e "require('fs').writeFileSync('data.json', '{}')"` 등을 입력해서 감지되는지 확인해 볼 수 있음.


## "생성 → 실행 → 삭제"가 NanoClaw와 최신 AI 에이전트들이 보안을 지키는 핵심 메커니즘

### 1. 생성 (Spawn)

* 사용자의 명령이 들어오는 순간, 미리 정의된 이미지(node:20-slim)를 기반으로 독립된 OS 환경을 즉시 가동함.
* 이때 호스트의 특정 폴더(workspace)만 살짝 연결해 줌.

### 2. 실행 (Execute)

* 컨테이너 안에서 에이전트가 touch helloworld.txt 같은 명령을 수행함.
* 파일은 실시간으로 공유 폴더에 기록되지만, 그 외의 시스템 변경사항(패키지 설치, 설정 변경 등)은 컨테이너 내부 메모리에만 머물게 됨.

### 3. 삭제 (Destroy)

* 명령 실행이 끝나 프로세스가 종료되면, Docker가 --rm 옵션에 따라 컨테이너를 흔적도 없이 통째로 삭제함.
* 에이전트가 무슨 짓을 했든 상관없이 호스트 시스템은 항상 깨끗한 상태를 유지함.

------------------------------
### 이 구조가 중요한 이유
만약 컨테이너를 삭제하지 않고 계속 띄워둔다면, 에이전트가 이전 작업에서 저지른 실수가 다음 작업에 꼬이거나, 보안 취약점이 누적될 수 있음. NanoClaw는 이를 "일회용 방진복"처럼 사용하여 매번 완벽하게 깨끗한 환경을 보장.




## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- [프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
- [NanoClaw 아키텍처 설계 및 구성 요소 정리]({% link _posts/2026-04-25-nanoclaw-arch-desc.md %})
- [nanoclow code분석-1 src/index.ts]({% link _posts/2026-04-25-nanoclaw-code-reading.md %})
