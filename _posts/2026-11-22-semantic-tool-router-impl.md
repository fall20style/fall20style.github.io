---
title: 가용 RAM 1.4GB 리눅스 서버에서 Semantic Tool Router 구축
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - AI
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

# 가용 RAM 1.4GB 리눅스 서버에서 Semantic Tool Router 구축하기

가용 RAM 1.4GB(GPU 없음) Linux 서버 환경에서 사용자 의도를 분류하고 외부 도구(Function Calling)로 연결하기 위해 Cross-Encoder와 ONNX Runtime 기반의 Semantic Tool Router를 구축했음.

---

## 1. 배경

LLM에 Function Calling을 위임할 때 등록된 모든 함수의 JSON 스키마를 프롬프트에 포함하면 다음 문제가 발생함.

1. 매 요청마다 수천 토큰의 도구 명세를 전송해 비용과 응답 지연이 증가함.
2. 도구 개수가 늘어날수록 LLM의 함수 선택 정확도가 떨어짐.
3. 가용 RAM 1.4GB 환경에서는 7B급 Local LLM 구동 시 OOM(Out of Memory)이 발생함.

이를 해결하기 위해 **1단계에서 경량 Encoder 라우터가 필요한 도구만 선택하고, 2단계에서 후속 LLM이 선택된 1개 도구의 인자(JSON)만 추출하는 2단계 구조**로 설계했음.

---

## 2. 아키텍처

학습은 Google Colab(T4 GPU)에서 수행하고, 추론 서버(Linux CPU)에는 PyTorch 없이 `onnxruntime`만 배포함.

```text
[사용자 질문] ("어제 시킨 운동화 배송 어디쯤 왔어?")
      │
      ▼  Tokenizer (Batch Pairing: 질문 × 전체 도구)
[문장 쌍 생성] 
  ├─ [CLS] 질문 [SEP] Tool: get_weather | Desc: ... [SEP]
  ├─ [CLS] 질문 [SEP] Tool: search_order | Desc: ... [SEP]
  └─ [CLS] 질문 [SEP] Tool: cancel_subscription | Desc: ... [SEP]
      │
      ▼  ONNX Runtime (CPUExecutionProvider, ~247ms)
[Cross-Encoder 추론] ──▶ 도구별 독립 적합도 점수 산출 (0.0 ~ 1.0)
      │
      ├─▶ 모든 도구 < 0.55 ──▶ 일반 대화 응답
      └─▶ 도구 >= 0.55     ──▶ 선택된 도구 스키마만 LLM에 전달해 인자 추출 및 실행
```

### 주요 설계 방식
* **Cross-Encoder 적용:** 짧은 질문과 긴 도구 설명문은 텍스트 길이가 다른 비대칭 구조임. 코사인 유사도 기반 Bi-Encoder 대신 두 문장을 함께 입력받는 Cross-Encoder를 사용했음.
* **합성 데이터 생성:** 외부 API 없이 Colab에서 `Qwen2.5-7B-Instruct (4-bit)`를 구동해 `(질문 + 함수명 + 인자 JSON)` 데이터셋을 생성했음.
* **All-Pairs 페어링:** 질문 1개당 정답 도구(`1.0`)와 나머지 모든 오답 도구(`0.0`)를 1:N으로 매칭해 요청형 어미(`~해줘`, `~확인 좀`)에 편향되는 과적합을 방지했음.

---

## 3. 트러블슈팅

### ① Zero-shot Bi-Encoder 점수 변별력 부족
* **현상:** 사전학습 임베딩 모델로 코사인 유사도를 계산한 결과 모든 도구가 `0.73 ~ 0.85` 구간에 몰려 일반 대화까지 도구를 호출함.
* **조치:** `klue/roberta-base` 위에 단일 스칼라 출력 헤드(`num_labels=1`)를 추가해 Cross-Encoder로 파인튜닝했음. 학습 후 정답은 `1.00`, 오답은 `0.00`으로 분리됨.

### ② RoBERTa 문장 쌍 입력 시 CUDA assert 에러
* **현상:** 질문과 도구 설명을 쌍(Pair)으로 토크나이징해 학습할 때 `CUDA error: device-side assert`가 발생함.
* **조치:** RoBERTa의 `type_vocab_size`는 `1`(`0`만 허용)이므로 문장 쌍 입력 시 부여되는 `token_type_ids=1`이 인덱스를 초과함. 토크나이저에 `return_token_type_ids=False`를 지정해 해결했음.

### ③ `MSELoss` 모델에 `Sigmoid` 중복 적용
* **현상:** `Validation Loss`가 `0.000384`까지 수렴했으나 추론 점수가 `0.738`과 `0.498`로 출력됨.
* **조치:** `float` 라벨(`1.0`, `0.0`)로 학습된 `MSELoss` 모델의 출력 로짓 자체가 이미 `1.0`과 `0.0`에 수렴함. 중복 적용된 Sigmoid를 제거하고 `np.clip(logits, 0.0, 1.0)`으로 변경해 `1.00`과 `0.00`을 얻었음.

### ④ 추론 서버 메모리 최적화
* **현상:** 가용 RAM 1.4GB 서버에 PyTorch와 모델을 함께 로드하면 메모리 여유분이 부족함.
* **조치:** 모델을 `torch.onnx.export(..., dynamo=False)`로 변환하고 추론 서버에는 `onnxruntime`만 설치했음. 프로세스 메모리 점유율을 약 280MB로 낮췄음.

---

## 4. 벤치마크 결과

학습에 사용하지 않은 14개 테스트 케이스로 1단계 라우팅과 2단계 인자 추출을 평가했음.

```text
[02] ✅ O | 질문: 내일 서울 기온 몇 도까지 떨어져?
      정답: get_weather {'location': '서울', 'date': '내일'}
      예측: get_weather {'location': '서울', 'date': '내일'}
[05] ✅ O | 질문: 어제 시킨 운동화 배송 어디쯤 왔어?
      정답: search_order {'item_name': '운동화', 'order_id': None}
      예측: search_order {'item_name': '운동화', 'order_id': None}
[09] ✅ O | 질문: 유료 멤버십 오늘 바로 해지해 주세요
      정답: cancel_subscription {'plan_name': '유료 멤버십', 'immediate': True}
      예측: cancel_subscription {'plan_name': '유료 멤버십', 'immediate': True}
[12] ✅ O | 질문: 오늘 점심 메뉴 뭐 먹을까?
      정답: None {}
      예측: None {}

===========================================================================
📊 [최종 벤치마크]
 - 1단계 함수 라우팅 정확도 (Tool Accuracy) : 92.86% (13/14)
 - 2단계 함수+인자 최종 정확도 (E2E Accuracy): 71.43% (10/14)
 - 평균 ONNX 라우터 추론 속도 (Latency)    : 247.76 ms (CPU 기준)
===========================================================================
```

독립 스코어링 방식을 적용해 *"내일 서울 날씨 알려주고 내 택배 어디 왔는지도 봐줘"* 같은 복합 질문에서도 `get_weather`와 `search_order` 두 도구를 동시에 선택함.

---

## 5. 향후 작업

* FastAPI 기반 REST API 엔드포인트 추가
* 인자 추출 프롬프트에 Few-shot 예시를 추가해 2단계 E2E 정확도 개선
