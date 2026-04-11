---
title: nanoClaw에서 사용하는 node.js 기능
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

## 1. child_process (가장 핵심)

- 용도: Docker 컨테이너를 실행하고 제어함.
- 상세: spawn이나 exec 함수를 통해 터미널 명령어를 직접 던짐. 에이전트가 수행한 결과값(Stdout)과 에러(Stderr)를 실시간으로 받아오는 통로임.

## 2. fs (File System)

- 용도: 세션별 작업 폴더 관리 및 파일 전송.
- 상세: readdir, readFileSync, writeFileSync 등을 사용함. 에이전트가 생성한 이미지나 PDF를 읽어 사용자에게 전달하거나, CLAUDE.md 메모리 파일을 읽고 쓰는 데 필수임.

## 3. path

- 용도: 경로 보안 및 플랫폼 간 호환성 유지.
- 상세: 호스트와 컨테이너 간의 볼륨 마운트 경로를 절대 경로로 계산함. path.join이나 path.resolve를 사용해 상위 디렉토리로의 비정상적인 접근(Path Traversal 공격)을 방지함.

## 4. events (EventEmitter)

- 용도: 메시지 수신 및 작업 상태 알림.
- 상세: 채팅 채널에서 새 메시지가 오거나 Docker 작업이 끝났을 때 이벤트를 발생시켜 논블로킹 방식으로 로직을 처리함.

## 5. process

- 용도: 시스템 권한 및 환경 설정 관리.
- 상세: process.getuid()를 통해 사용자 ID(UID)를 파악하여 Docker 권한 문제를 해결하고, process.env로 API 키를 안전하게 로드함.

## 6. crypto (또는 외부 라이브러리 연동)

- 용도: 세션 보안 및 데이터 무결성.
- 상세: 각 세션이나 파일의 고유 식별자를 생성하거나 보안 토큰을 다루는 데 사용됨.

------------------------------
- NanoClaw는 Node.js를 운영체제와 AI 에이전트를 이어주는 강력한 오케스트레이터(Orchestrator)"로 사용.
