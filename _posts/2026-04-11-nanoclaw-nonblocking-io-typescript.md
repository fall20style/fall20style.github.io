---
title: JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용
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


- Node.js 환경의 논블로킹(Non-blocking) I/O를 활용
- 적은 리소스로도 다수의 AI 에이전트 세션을 효율적으로 관리.

------------------------------
## 1. 논블로킹 I/O의 핵심 원리

* 멈춤 없는 작업: 파일 읽기, 네트워크 요청 등 시간이 걸리는 작업(I/O)을 운영체제에 맡기고, 메인 스레드는 즉시 다음 코드를 실행함.
* 이벤트 루프(Event Loop): 작업이 완료되면 이벤트 루프가 이를 감지하여 등록된 콜백 함수를 실행하는 구조.


## 2. NanoClaw에서의 실제 활용 사례

* 다중 채널 수신: WhatsApp, Telegram 등 여러 채널에서 동시에 메시지가 들어와도 지연 없이 DB에 기록함.
* 샌드박스 동시 실행: 특정 에이전트가 Docker 내에서 무거운 작업을 수행하는 동안, 시스템은 멈추지 않고 다른 사용자의 트리거 워드를 감지함.
* 비동기 워크플로우: async/await 문법을 사용하여 복잡한 에이전트 실행 순서를 가독성 있게 제어함.

## 3. 주요 장점 (Efficiency)

* 저사양 최적화: 멀티 스레드 언어처럼 요청마다 메모리를 크게 점유하지 않아 초경량 실행이 가능함.
* 확장성: 단일 프로세스로도 수많은 동시 연결을 매끄럽게 처리할 수 있음.
* 병목 현상 방지: 연산이 집약적인 작업은 Docker(별도 프로세스)로 분리하고, 메인 로직은 I/O 제어에만 집중.




## Related Posts
- [gemini CLI를 특정 시간에 실행]({% link _posts/2026-04-04-geminiCLI-run-on-schedule.md %})
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- [프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
- [Hyper는 프론트엔드 기술로 데스크톱 앱을 만드는 Electron 프레임워크]({% link _posts/2026-04-12-typescript-hyper-project.md %})
- [NanoClaw 아키텍처 설계 및 구성 요소 정리]({% link _posts/2026-04-25-nanoclaw-arch-desc.md %})
- [nanoclow code분석-1 src/index.ts]({% link _posts/2026-04-25-nanoclaw-code-reading.md %})
