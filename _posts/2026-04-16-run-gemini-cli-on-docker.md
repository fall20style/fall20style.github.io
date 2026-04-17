---
title: Gemini CLI Docker 실행 가이드
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

## 1. 개요

* Gemini CLI를 Docker에서 실행하는 방법임.
* 호스트 환경을 깔끔하게 유지 가능함.
* AI 개발 에이전트 테스트에 최적임.


## 2. 실행 방법

## npx로 즉시 실행

* 빌드 과정이 귀찮을 때 사용함.
* 일회성 확인용으로 매우 편리함.

```
docker run -it --rm node:20-slim npx @google/gemini-cli
```

## Dockerfile 빌드 (권장)

* 자주 사용할 경우 이미지 만드는 것이 좋음.
* 다음 내용을 Dockerfile로 저장하면 됨.

```
FROM node:20-slim
RUN npm install -g @google/gemini-cli
WORKDIR /app
ENTRYPOINT ["gemini"]
```

* 빌드: `docker build -t gemini-cli .` 입력함.
* 실행: `docker run -it gemini-cli` 입력함.


## 3. 설정 유지 (볼륨 마운트)

* 컨테이너가 종료되어도 로그인을 유지하려면 필수임.
* 호스트 폴더와 연결하는 작업임.

```
docker run -it \
  -v ~/.gemini:/root/.gemini \
  -v $(pwd):/app \
  gemini-cli
```

* 인증 유지: `~/.gemini` 경로를 마운트하여 로그인 정보를 저장함.
* 파일 연동: 현재 디렉토리(/app)를 연결하여 AI가 코드를 읽도록 함.


## 4. 주의사항

* API KEY: 환경 변수로 전달하려면 -e GEMINI_API_KEY=값을 사용함.
* 인증: 터미널에 나타나는 URL을 복사하여 호스트 브라우저에서 승인함.
* 권한: 컨테이너가 생성한 파일의 권한 문제가 발생할 수 있으므로 확인해야 함
