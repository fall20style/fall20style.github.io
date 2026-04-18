---
title: Docker로 Gemini CLI 돌리는 법
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Gemini CLI
  - 습작만들기
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---





## Dockerfile

- git 경로

## 실행 및 인증

1. 이미지 빌드:

```
docker build -t gemini-cli .
```

2. 컨테이너 실행:

```
docker run -it --rm -e GEMINI_API_KEY="본인_키" -v /home/mjpark:/home/mjpark gemini-cli
```

3. 구글 계정 인증:
* 터미널에 표시되는 URL에 접속
* 구글 로그인 후 인증 코드 복사
* 터미널에 붙여넣기

## 문제 해결

* EACCES 에러: 호스트 폴더 권한 문제. -u root로 실행하거나 UID를 1000으로 설정.
* UID 1000 충돌: 기본 node 사용자가 1000번이라 발생. userdel로 삭제 후 새로 생성.
* 이미지 못 찾음: 명령어 순서가 잘못되면 도커가 옵션을 이미지 이름으로 착각.


## 주의사항

* API 키 보안: 노출된 키는 Google AI Studio에서 삭제하고 새로 발급.
* 설정 저장: -v 옵션으로 다음 실행 시 재인증 불필요.


## Screenshot

<img width="887" height="196" alt="Image" src="https://github.com/user-attachments/assets/cc2fee75-a9ce-46e0-848c-fb619abd626c" />


## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- ["프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)"]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
