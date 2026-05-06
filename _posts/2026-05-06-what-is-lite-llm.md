---
title: "LiteLLM 용도"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
use_math: true 
tags:
  - litellm
toc: true
toc_sticky: true
toc_label: 목차
description: "LiteLLM이란?"
---


## LiteLLM 개요 및 역할

### 1. 정의 및 핵심 기능

* 여러 AI 모델(Gemini, GPT, Claude 등)의 호출 방식을 하나로 통합하는 AI 어댑터이다.
* 다양한 모델의 API 규격을 OpenAI 표준 형식으로 바꿔주는 중간 번역기 역할을 한다.

### 2. Nanoclaw 연동 원리

* 신호 변환: Nanoclaw가 보내는 'Claude 규격' 요청을 'Gemini 규격'으로 실시간 통번역한다.
* 경로 우회: Nanoclaw의 목적지를 Anthropic 서버가 아닌 LiteLLM 서버(localhost:4000)로 돌린다.
* 결과: Nanoclaw는 Claude와 대화한다고 생각하지만, 실제로는 Gemini가 응답한다.

### 3. 주요 장점

* 교체 용이: 코드 수정 없이 설정 파일만 바꿔서 Gemini 1.5 Pro, Flash 등으로 즉시 전환 가능하다.
* 비용 관리: 상대적으로 저렴한 Gemini API를 사용하여 비용을 절감할 수 있다.
* 확장성: 향후 출시될 새로운 모델들도 LiteLLM만 업데이트하면 바로 연동할 수 있다.

### 현재 상황 요약

* Nanoclaw는 TypeScript 기반 프로젝트이고, LiteLLM은 Python 기반 서버 도구이다.
* 현재 Python 3.14의 호환성 문제로 설치가 막힌 상태이므로, 안정적인 Python 3.12 환경을 구축하거나 Docker를 사용하는 것이 해결책이다.

### Screenshot 1
<img width="623" height="468" alt="Image" src="https://github.com/user-attachments/assets/772bb5fa-0209-453b-a623-8af00f16fc50" />

### Screenshot 2
<img width="898" height="882" alt="Image" src="https://github.com/user-attachments/assets/c85851da-c823-4d72-a489-06e34b6fef21" />
