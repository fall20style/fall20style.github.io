---
title: NanoClaw 아키텍처 설계 및 구성 요소 정리
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - nanoclaw
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## NanoClaw 아키텍처 설계 (요약)

``` mermaid
graph TD
    subgraph "External"
        Cloud[LLM Cloud]
        Sensors[Hardware Sensors]
    end

    subgraph "NanoClaw Kernel"
        direction TB
        Task[Task Scheduler]
        Mem[Linear Memory]
        Link[API Linker]
    end

    Sensors --> Task
    Task <--> Mem
    Task --> Link
    Link --> Cloud
```

## 계층별 핵심 기능

## 1. 인터페이스 계층 (Interface Layer)

* 이벤트 트리거: 센서 신호나 타이머 인터럽트를 에이전트 작업으로 전환함.
* 경량 프로토콜: MQTT, CoAP 등 임베디드 특화 통신 규격 사용함.

## 2. 코어 엔진 (Core Engine)

* 미니 플래너: 복잡한 추론 대신 사전에 정의된 '태스크 그래프' 기반으로 동작함.
* 문맥 최적화: 토큰 사용량을 줄이기 위해 필수 메타데이터만 램(RAM)에 적재함.
* 함수 매핑: AI의 요청을 하드웨어 제어 함수(GPIO, I2C 등)와 직접 연결함.

## 3. 자원 관리 (Resource Management)

* 정적 메모리 할당: 런타임 오류 방지를 위해 컴파일 시점에 메모리 크기 확정함.
* 보안 격리: API 키를 하드웨어 고유 식별자와 결합하여 암호화 저장함.


## 설계 특징

* 최소 의존성: 외부 라이브러리 사용을 배제하고 Rust 표준 라이브러리(no_std) 위주로 구성함.
* 저전력 모드: 작업이 없는 유휴 상태에서는 CPU를 슬립(Sleep) 모드로 전환하여 전력 소모 최소화함.

