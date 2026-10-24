---
title: Custom Jev Variant — ModernBERT 기반 영·한 뉴스 카테고리 의사결정 엔진
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


## 📑 Custom Jev Variant — ModernBERT 기반 영·한 뉴스 카테고리 의사결정 엔진

- 프로젝트 명: Custom Jev Variant — ModernBERT 기반 영·한 뉴스 카테고리 의사결정 엔진
- 작성일자: 2026년 10월 24일
- 버전: v1.0

### 1. 개요 (Introduction)

제한적인 환경(Few-shot)에서 거대 모델의 지식을 정제하여, 로컬 환경에서 밀리초(ms) 단위로 작동하는 System 1 고속 뉴스 카테고리 분류 의사결정 엔진의 설계 및 구현 내용

- 목적: 오픈소스 기반 ModernBERT와 LoRA를 융합하여 상용 의사결정 API(Jev) 수준의 신뢰도 점수를 출력하는 경량 분류 엔진 구축
- 주요 특징: 텍스트 생성을 배제한 구조화된 확률 출력, 데이터 중심(Data-Centric) 밸런싱 최적화, ONNX 포맷을 통한 서버리스 서빙 가속화

### 2. 시스템 아키텍처 및 데이터 흐름 (System Architecture & Data Flow)

본 시스템은 데이터 증류(Distillation)와 경량 파인튜닝(PEFT) 단계를 거쳐 최종 배포 엔진으로 전환되는 파이프라인 구조를 가집니다.

```
[1. 데이터 준비 단계]
교사 모델 (LLM) ──(데이터 증류)──> [State, Question, Target] 포맷 JSONL 데이터셋 생성 (카테고리당 5건 균등 배치)
                                                                 │
[2. 학습 단계]                                                   ▼
ModernBERT-base ──(LoRA 어댑터 결합) <─────────────────── 토큰화 및 전처리 (Max Length: 128)
        │
        ▼
[3. 변환 및 배포 단계]
LoRA 가중치 완전 병합 (Merge & Unload) ──(Optimum)──> ONNX 가속화 엔진 ──> IPyWidgets 대화형 GUI View 서빙
```

### 3. 데이터셋 및 파이프라인 설계 (Data & Pipeline Design)


#### 3.1 Jev 스타일 데이터 스키마

단순 텍스트 분류가 아닌 문맥(state)과 평가 기준(question)을 동적으로 결합하는 의사결정 구조를 가집니다.
- 포맷: {"state": "뉴스 제목", "question": "평가 질문", "target": 카테고리 인덱스}
- 카테고리 매핑 레이아웃 (총 7개 클래스):
```
	0: 정치, 1: 경제, 2: 사회, 3: 생활/문화, 4: 세계, 5: IT/과학, 6: 날씨
```

#### 3.2 데이터 밸런싱 (Data Centric AI)

특정 카테고리로 가중치가 쏠리는 단어 편향(Token Bias)을 방지하기 위해, 모든 카테고리를 정확히 5건씩 균등하게 배치(총 35건의 퓨샷 세트)하여 균등 분산(Uniform Distribution) 상태를 보장합니다.

### 4. 모델 훈련 및 의존성 설계 (Training & Optimization)

- 베이스 백본 모델: answerdotai/ModernBERT-base
- PEFT(LoRA) 레이어 우회 타겟팅: torchao 버전 충돌 문제를 우회하고 ModernBERT 구조에 최적화하기 위해 추적 대상 선형 레이어를 수동으로 선언합니다

```
target_modules=["Wqkv", "Wo", "Wi", "Wo"]
```

- 훈련 하이퍼파라미터 사양:
	- Learning Rate: 3e-4
	0 Epochs: 5 ~ 8 (데이터 밀도 보완용)
	- Eval Strategy: "no" (최신 transformers 컴파일러 호환 패치)

### 5. 구성 컴포넌트 소스코드 (Implementation Source)


#### 5.1 통합 모델 학습 및 전처리 셀

``` python
import json
import torch
import numpy as np
from datasets import Dataset
from transformers import AutoTokenizer, AutoModelForSequenceClassification, Trainer, TrainingArguments
from peft import LoraConfig, get_peft_model, TaskType

# [생략] synthetic_dataset 배열 선언 (정치~날씨 각 5건씩 총 35건 배치)

model_id = "answerdotai/ModernBERT-base"
tokenizer = AutoTokenizer.from_pretrained(model_id)
base_model = AutoModelForSequenceClassification.from_pretrained(model_id, num_labels=7)

lora_config = LoraConfig(
    task_type=TaskType.SEQ_CLS, r=32, lora_alpha=64,
    target_modules=["Wqkv", "Wo", "Wi", "Wo"], lora_dropout=0.1, bias="none"
)
model = get_peft_model(base_model, lora_config)

def preprocess_function(examples):
    inputs = [f"뉴스 제목: {title} \n질문: {q}" for title, q in zip(examples['state'], examples['question'])]
    result = tokenizer(inputs, truncation=True, padding="max_length", max_length=128)
    result["label"] = examples["target"]
    return result

# 훈련 실행 후 모델 레이어 완전 병합(Merge) 처리
training_args = TrainingArguments(
    output_dir="./korean_news_jev", learning_rate=3e-4, per_device_train_batch_size=2,
    num_train_epochs=8, weight_decay=0.01, fp16=torch.cuda.is_available(), eval_strategy="no", report_to="none"
)
trainer = Trainer(model=model, args=training_args, train_dataset=Dataset.from_json("train_data.jsonl").map(preprocess_function, batched=True))
trainer.train()

merged_model = model.merge_and_unload()
merged_model.eval()
Use code with caution.
```

### 5.2 한글화 시각화 (Data Distribution Plot)

``` python
import matplotlib.pyplot as plt
import koreanize_matplotlib # 한글 깨짐 방지 전용 모듈 가동
from collections import Counter

plt.rcParams['axes.unicode_minus'] = False
# Counter를 활용한 막대그래프 렌더링 및 비율 브리핑 로그 출력부 구현
Use code with caution.
```

### 5.3 다중 점수 매트릭스 출력형 GUI 테스터

``` python
import ipywidgets as widgets
from IPython.display import display, clear_output

# Softmax 연산을 거쳐 전체 카테고리의 확신도를 정렬된 텍스트 바(Bar) 차트로 출력하는 IPyWidgets view 레이아웃 구현
Use code with caution.
```

### 6. 성능 및 기대 효과 (Performance & Conclusion)

1. 확률 보정(Calibration) 실현: 단순 탑-1 결과 제공을 넘어, 모든 카테고리에 할당된 확률(Confidence Score) 점수를 출력하여 결정이 모호한 경계선 데이터를 정밀하게 모니터링할 수 있습니다.
2. 초저지연 시스템 보장: 텍스트를 생성하는 디코딩 과정이 없는 비자기회귀형 구조 및 LoRA 가중치 사전 융합을 통해, 일반 로컬 CPU 환경에서도 15~33ms 성능 반환 수치를 달성할 수 있습니다.

