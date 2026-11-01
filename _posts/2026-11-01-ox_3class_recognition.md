---
title: O / X / 분류 불가능 실시간 인식 AI 앱
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

# 손으로 그리는 O / X / 분류 불가능 - 전이학습 & 액티브 러닝

- 딥러닝 컴퓨터 비전 기술을 활용 토이 프로젝트 
- **마우스로 O, X 또는 정체 모를 낙서를 그리면 AI가 실시간으로 판별**
- 데이터가 14장밖에 없는 환경에서 **90% 이상의 높은 정확도**를 내고 틀린 결과를 수집해 스스로 진화
- **액티브 러닝(Active Learning) 파이프라인**

<!-- 1x3 실행 화면 캡처 이미지 레이아웃 추가 -->
<p align="center">
  <img width="32%" alt="그리기 화면 및 예측" src="https://github.com/user-attachments/assets/b873c830-9381-48d9-8fdd-b4097566fc8b" />
  <img width="32%" alt="진짜 정답 피드백 팝업" src="https://github.com/user-attachments/assets/57887d76-e50a-4514-a2fc-6ca9ab6bb347" />
  <img width="32%" alt="분류 불가능 판정 결과" src="https://github.com/user-attachments/assets/3fa4d8f6-7de6-473f-b646-07f43d2b0a5a" />
</p>
<p align="center">
  <em>▲ 1. 마우스 드로잉 및 실시간 예측 &nbsp;&nbsp;|&nbsp;&nbsp; 2. 오답 정정 피드백 팝업 &nbsp;&nbsp;|&nbsp;&nbsp; 3. '분류 불가능' 판정 예시</em>
</p>

- 처음에 X를 분류 불가능으로 봤지만, 다시 학습시키면, 맞게 X로 분류함.

<img width="282" height="412" alt="Image" src="https://github.com/user-attachments/assets/464c9eda-302f-4064-9f37-60c519a2475f" />

---

## 🚀 프로젝트 핵심 요약

* **프레임워크:** PyTorch (파이토치)
* **기반 모델:** MobileNetV3-Small (사전 학습된 가벼운 고성능 CNN 모델)
* **분류 클래스 (3-Class):** 
  1. `O` (동그라미)
  2. `X` (가위표)
  3. `분류 불가능 (Unclassified)` (애매한 낙서, 세모, 빈 화면 등)
* **핵심 기능:** 
  * 단 14장(클래스당 7장)으로 학습 가능한 **강력한 데이터 증강(Data Augmentation)**
  * 주피터 노트북 보안 샌드박스를 우회하는 **Tkinter 기반 독립형 그림판 GUI 팝업**
  * AI가 오판했을 때 원클릭으로 정답 폴더에 격리 수집하는 **데이터 피드백 파이프라인**

---

## 📁 1. 데이터 세팅 및 폴더 구조

```text
📁 내프로젝트폴더/
│
├── 📄 ox_3class_recognition.ipynb  <-- 주피터 노트북 파일
└── 📁 dataset/
    └── 📁 train/
        ├── 📁 O/            --> 동그라미 png 파일 (7장)
        ├── 📁 X/            --> 가위표 png 파일 (7장)
        └── 📁 Unclassified/ --> 낙서, 세모, 빈 도화지 등 (7장)
```
*💡 **Tip:** PNG 파일의 투명 배경(Alpha 채널) 오류를 막기 위해 코드 내부에서 `convert('RGB')` 전처리를 강제 적용*

---

## 🧠 2. 딥러닝 모델 설계: 왜 MobileNetV3 인가?

데이터가 턱없이 부족할 때 밑바닥부터 CNN 레이어를 쌓으면 모델이 이미지를 통째로 외워버리는 **과적합(Overfitting)**이 100% 발생.

이를 해결하기 위해 이미 수백만 장의 이미지(ImageNet)를 학습하여 선, 곡선, 형태적 특징을 마스터한 **MobileNetV3 Small**을 가져오기. (이를 **전이 학습, Transfer Learning**이라고 합니다.) 

수천 개의 사물을 맞추던 기존 최종 출력층(Classifier)을 과감히 떼어내고, 우리가 설계한 **3가지 클래스(출력 노드 3개)** 구조의 선형 레이어로 교체.

---

## 🎨 3. 실시간 마우스 드로잉 인터페이스 (Tkinter GUI)

Jupyter Notebook의 최신 보안 정책은 셀 내부에서 자바스크립트나 HTML5 캔버스를 동적으로 렌더링하는 것을 종종 차단. 

이 문제를 해결하기 위해 파이썬 내장 GUI 모듈인 **Tkinter**를 채택했습니다.

코드를 실행하면 주피터 브라우저 밖으로 **독립된 윈도우 창이 팝업**되어 100% 안정적으로 작동.

### 🕹️ 주요 화면 미리보기 및 기능
1. **검은 캔버스 영역:** 마우스 왼쪽 버튼을 드래그하여 흰색 펜으로 그림.
2. **🎯 예측하기 버튼:** AI가 현재 그림을 분석하여 `[ O ]`, `[ X ]`, `[ 분류 불가능 ]` 결과와 함께 **확신도(%)**를 보여줌.
3. **💾 틀린결과 정정 및 저장 버튼 (핵심):** 만약 AI가 내가 그린 동그라미를 X로 오판했다면 이 버튼을 누름. **"원래 [ O ] 임"**을 선택하면 즉시 오답 수집 폴더로 이미지가 자동 격리 저장!

---

## 🔄 4. 스스로 진화하는 액티브 러닝 파이프라인

이 앱의 특징: 사용자와 상호작용하며 데이터를 모으는 **피드백 파이프라인**.

사용자가 정정한 오답 데이터들은 아래의 체크 폴더에 쌓이게 됨.
* `dataset/check/should_o` (O인데 틀린 것)
* `dataset/check/should_x` (X인데 틀린 것)
* `dataset/check/should_unclassified` (분류 불가능인데 틀린 것)

- 노트북의 가장 마지막 셀을 실행하면, 이 체크 폴더에 쌓인 파일들을 기존의 학습 폴더(`dataset/train/...`)로 **자동으로 이동(shutil.move)**시켜 줍니다.
- 파일명이 겹치더라도 유실되지 않도록 자동으로 이름 뒤에 `_copy`를 붙여주는 안전장치도 포함되어 있습니다.

> **작업 루프:** 그림 그리기 ➡️ 오답 정정 및 수집 ➡️ 자동 이동 셀 실행 ➡️ 상단 데이터 로더 및 모델 학습 재실행(Retraining) ➡️ AI 개선

---

## 🎬 마치며

- **소량의 데이터로 학습을 시작해 사용자의 피드백을 받아 데이터셋을 확장하고 모델을 고도화하는 현대 AI 서비스의 핵심 주기(Data Flywheel)**를 직접 구현
- 파이토치의 직관적인 제어 능력과 파이썬 기본 GUI의 강력한 결합을 경험

- dl_ox_classification
