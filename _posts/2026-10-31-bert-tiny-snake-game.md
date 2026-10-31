---
title: BERT-Tiny로 스네이크 게임 플레이
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - AI
  - BERT
  - pygame
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: 10×10 스네이크를 BERT-Tiny로 플레이시키고, 전문가 증류와 DAgger 루프, pygame GUI까지 정리한 과제.
---

# BERT-Tiny로 스네이크 게임 플레이시키기

## 어떤 과제였나

- 10×10 격자 스네이크를 BERT-Tiny + 선형 분류기* 플레이시키는 과제.
- 매 스텝 상태를 문장으로 만들고, 모델이 `좌회전 / 직진 / 우회전` 세 가지 행동 중 하나를 선택.
- 강화학습 대신 규칙 기반 전문가(greedy teacher)의 플레이를 지도학습으로 증류.

```text
danger 0 1 0 food fwd 4 side -1 dist 5 tail dx 0 dy 2 len 3 area 97 free 0.97 age 0
```

위험한 방향 3개, 진행 방향 기준 음식 위치(`forward`, `side`, `dist`), 꼬리 상대 좌표, 길이, 도달 가능한 빈칸, 마지막 음식 이후 스텝을 담기.

## 만든 것

| 구성 | 내용 |
|---|---|
| 환경 | 10×10, 상대 행동 3개, 시드 고정 |
| 모델 | `bert-tiny` + `Linear(128 → 3)`, full fine-tuning |
| 학습 데이터 | 5,000 / 15,000 샘플 (전문가 90% + 무작위 10%) |
| 개선 루프 | DAgger / Expert Iteration |
| 데모 | pygame GUI, 체크포인트 나란히 비교, mp4 녹화 |

꼬리 칸으로 이동할 수 있게 하고, 음식 뒤쪽 blind spot을 없애고, 보드가 가득 차면 승리로 끝나게 하고, 시드로 판을 재현.

### DAgger 루프

전문가의 1스텝 greedy 라벨만으로는 모델이 전문가의 실수를 그대로 배웁니다. 그래서 라운드마다 다음을 반복.

1. 현재 best 정책과 전문가를 `β = max(0.1, 1/(r+1))` 비율로 섞어 플레이
2. 방문한 상태에 전문가 라벨을 붙여 데이터로 저장
3. 최근 2라운드 데이터로 모델을 다시 학습
4. 고정 시드로 평가하고, `best + 0.2`를 넘으면 승격

## 결과

| 에이전트 | 평가 | 평균 점수 |
|---|---:|---:|
| 전문가 (greedy) | 500판 | 19.27 |
| `loop_best.pt` | 100판 | 19.95 |
| 5k 체크포인트 | 10판 | 11.10 |
| `loop_best.pt` | 10판 | 23.20 |

- 전문가는 자기 몸에 갇히는 종료가 91%였고, 그 한계가 그대로 점수 상한이 됨.
- `loop_best`는 100판에서 19.95로 전문가 수준에 도달.

## 한계

- 교사가 1스텝 greedy라 정책의 상한도 전문가 수준.
- 관측이 보드 전체가 아니라 손으로 설계한 스칼라 특징.
- 모델은 `argmax`라 결정적. 같은 시드면 항상 같은 판.

## 코드

- 저장소: <https://github.com/fall20style/bert-tiny-snake-game>
- 구현 설명: `TECHNICAL.md`
- 루프 로그 분석: `LOOP_ANALYSIS.md`
