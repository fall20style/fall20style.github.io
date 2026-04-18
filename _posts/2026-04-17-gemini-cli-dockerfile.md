---
title: Gemini를 포함하는 Dockerfile
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Gemini CLI
  - Docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## Dockerfile 작성

- 베이스 이미지... 노드 사용...FROM node:20-slim
- CLI 설치...RUN npm install -g @google/gemini-cli
- 실행 경로...WORKDIR /app
- 기본 명령...ENTRYPOINT ["gemini"]


## 빌드 및 실행

   1. 이미지 만들기...
   * docker build -t gemini-cli .
   2. 컨테이너 돌리기...
   * API 키가 꼭 필요함...
      * docker run -it -e GEMINI_API_KEY="본인_키" gemini-cli
   

## 참고할 점

* API 키... 여기서 발급...
* 파일 연결... 로컬 파일 보려면 -v $(pwd):/app 옵션 추가...
* 설정 저장... 컨테이너 끄면 설정 사라짐... 볼륨 마운트 추천...


## 스크린샷

<img width="887" height="196" alt="Image" src="https://github.com/user-attachments/assets/456b61d8-b512-4df1-8b4f-e1590b369ee5" />


## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- ["프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)"]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
