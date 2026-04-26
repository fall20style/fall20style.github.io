---
title: NanoClaw의 결과 반환 구조 연습
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

- NanoClaw는 명령 수신 시 호스트와 격리된 Docker Sandbox에서 AI 에이전트를 실행함.
- 사용자 ID 매핑(UID/GID) 방식을 통해 권한 문제 없이 안전하게 파일을 수정하고 결과를 반환하는 과정.

## 1. 격리된 실행 환경 구축 (Build)

- AI 에이전트 전용 독립 환경(이미지)을 빌드함.
- 일반 사용자 계정을 포함해 보안을 강화하는 단계.

* Dockerfile 폴더에서 실행

``` bash
docker build -t my-sandbox .
```

### Dockerfile

``` Dockerfile
# 1. 가벼운 배포용 베이스 이미지 선택
FROM node:20-slim

# 2. 보안을 위해 root가 아닌 일반 사용자 생성 (AI 에이전트 권한 제한)
RUN useradd -m agentuser
USER agentuser

# 3. 작업 디렉토리 설정
WORKDIR /home/agentuser/app

# 4. 의존성 파일 복사 및 설치 (캐싱 활용)
COPY --chown=agentuser:agentuser package*.json ./
RUN npm install --production

# 5. 소스 코드 복사 (호스트와 연결될 chats 폴더 제외)
COPY --chown=agentuser:agentuser . .

# 6. AI 에이전트 실행 명령
CMD ["node", "dist/index.js"]

```

## 2. 안전한 통로 연결 및 권한 위임 (Run)
호스트 폴더를 컨테이너와 연결함. --user $(id -u):$(id -g) 옵션으로 현재 사용자 권한을 에이전트에게 그대로 전달하는 것이 핵심임.

``` bash
docker run -it --rm \
  -v /home/mjpark/work/docker_sandbox/my-project:/app/workspace \
  -w /app/workspace \
  --user $(id -u):$(id -g) \
  my-sandbox bash
```

* 볼륨 마운트: 특정 폴더만 에이전트에게 공유함.
* 권한 매핑: 호스트와 컨테이너의 ID를 일치시켜 Permission denied 에러를 방지함.

## 3. 에이전트 작업 수행 (Agent Action)
샌드박스 내부에서 에이전트가 부여받은 권한으로 파일을 생성하거나 코드를 실행함.

* 컨테이너 내부 실행 (성공)
``` bash
$ touch test_from_container.txt
$ exit
```

## 4. 결과 확인 및 자동 정리 (Cleanup)

작업 종료 시 컨테이너는 즉시 파기(--rm)되지만, 결과물은 호스트 폴더에 보존됨.

* 결과 확인: 호스트에서 test_from_container.txt 파일 생성 확인됨.
* 보안 효과: 에이전트가 호스트 권한 범위를 벗어나는 행위를 할 수 없음.




## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
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
