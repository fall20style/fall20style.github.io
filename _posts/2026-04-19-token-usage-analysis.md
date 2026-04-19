---
title: "Gemini CLI 토큰 소모량 측정"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Gemini CLI
  - Analytics
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

# 토큰 소모 방법론 분석

Gemini CLI 환경 내 일일 토큰 사용량 파악을 위한 연구 및 분석 내용임.

## 1. 목적
2026-04-19 세션 동안 "소모된"(과금 대상과 동일한 사용량) 총 토큰 수를 실증적으로 검증함.

## 2. 연구 방법론

### A. 공식 CLI 도구
Gemini CLI 도움말 시스템과 내부 문서 확인을 통해 조사 시작.
- **확인된 명령:** `/stats model`
- **유용성:** 세션 사용량 및 일일 할당량 제한에 대한 실시간 피드백 제공.
- **한계:** 사용자 모니터링에는 유용하나, 외부 처리를 위한 원시 로그를 수동 조작 없이 추출하기는 어려움.

### B. 파일 시스템 포렌식
영구적인 텔레메트리 및 세션 상태 확인을 위해 로컬 환경 검사.
- **세션 경로:** `/home/mjpark/.gemini/tmp/posts/chats/`
- **데이터 형식:** 구조화된 `messages` 배열을 포함하는 JSON 파일.
- **메타데이터 구조:** 모델 생성 메시지에 아래 스키마의 `tokens` 객체 포함됨.
  ```json
  "tokens": {
    "input": <integer>,
    "output": <integer>,
    "cached": <integer>,
    "thoughts": <integer>,
    "tool": <integer>,
    "total": <integer>
  }
  ```

## 3. 계산 로직
실질적 토큰 소모량 계산을 위해 세션 기록의 각 턴(turn)에 아래 공식 적용.

**소모량 = (입력 토큰 - 캐시된 토큰) + 출력 토큰 + 생각(Thought) 토큰**

- **입력 - 캐시:** 컨텍스트 캐시에 없는, 모델이 새로 처리한 신규 컨텍스트량.
- **출력:** 최종 응답 생성 토큰.
- **생각(Thoughts):** 모델 내부 추론 과정에서 생성된 토큰.

## 4. 실증 데이터 (2026-04-19 세션)

기본 세션 기록(`session-2026-04-19T01-57-607b9722.json`) 내 9개 턴 합산 결과임.

| 지표 | 토큰 수 |
| :--- | :--- |
| **(입력 - 캐시) 합계** | 35,062 |
| **출력 토큰 합계** | 504 |
| **생각 토큰 합계** | 2,511 |
| **오늘 총 소모량** | **38,077** |

## 5. 결론

Gemini CLI는 로컬 임시 디렉토리에 정밀한 사용 데이터를 유지함. 입력과 캐시의 차이 및 생성된 출력(생각 포함)을 집계하여 프로그래밍 방식으로 정확한 소모량 측정이 가능함.

지속적인 모니터링을 위해 `/stats model` 명령으로 남은 일일 요청 할당량(일반 계정 기준 하루 1,000회) 확인 권장.
