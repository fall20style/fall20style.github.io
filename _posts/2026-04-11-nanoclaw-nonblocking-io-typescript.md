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


