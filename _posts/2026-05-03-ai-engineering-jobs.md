---
title: "AI 엔지니어링 직군 정리"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - math
toc: true
toc_sticky: true
toc_label: 목차
description: "AI 엔지니어링 직군 정리"
---

AI 엔지니어링 직군은 최근 기술 고도화에 따라 역할 세분화
 

## AI 에이전트 엔지니어 (AI Agent Engineer)

목표: 스스로 판단하고 도구를 사용하여 목표를 완수하는 '지능형 에이전트' 구축

* 워크플로우 설계: LLM이 복잡한 작업을 단계별로 추론(Reasoning)하고 계획하도록 설계
* 도구 연동 (Tool Use): API 호출, 웹 검색, 데이터베이스 조회 등 외부 도구와의 연결 구현
* 자율성 부여: ReAct, Plan-and-Solve 등 프롬프트 기법을 활용해 실행 결과에 따른 피드백 루프 구축

## 도메인 LLM 엔지니어 (Domain LLM Engineer)

목표: 특정 산업(의료, 법률, 금융 등)에 특화된 고성능 언어 모델 최적화

* 특화 학습: 특정 도메인의 전문 데이터를 활용한 SFT(지도 미세 조정) 및 DPO/PPO(정렬 학습) 수행
* 데이터 거버넌스: 도메인 전문 지식이 반영된 고품질 학습 데이터셋 구축 및 정제
* 평가 지표 수립: 해당 산업군에서 필요한 정확도와 신뢰성을 측정할 수 있는 벤치마크 개발

## LLMOps 엔지니어 (LLMOps Engineer)

목표: LLM 서비스의 생애주기(학습, 배포, 모니터링) 자동화 및 안정화

* 파이프라인 구축: 데이터 수집부터 모델 배포까지의 전 과정을 자동화(CI/CD)
* 실시간 모니터링: 모델의 답변 품질(Hallucination), 비용, 지연 시간(Latency) 추적 및 관리
* 인프라 관리: GPU 리소스 효율화 및 데이터 베이스(Vector DB) 운영 최적화

## 추론 최적화 엔지니어 (Inference Optimization Engineer)

목표: 모델의 답변 속도를 높이고 운영 비용(Hwang)을 최소화

* 경량화 기술: 양자화(Quantization), 지식 증류(Distillation), 가지치기(Pruning) 적용
* 엔진 최적화: vLLM, TensorRT-LLM 등을 활용해 하드웨어 성능 극한까지 활용
* 처리량 개선: 다수 사용자가 접속해도 빠르게 응답할 수 있도록 동적 배칭(Batching) 등 기술 구현

