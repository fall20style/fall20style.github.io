---
title: duck-sim MCP 연동
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


## Qwen2.5-3B와 MCP로 가상 오리(duck-sim) 조종하기: LLM MCP 기반 로봇 제어

안녕하세요! 최근 LLM을 이용해 로봇이나 시뮬레이터를 제어하는 Embodied AI(체화된 인스턴스 AI) 분야가 뜨겁습니다.
오늘은 리눅스 환경에서 가상 biped 로봇 시뮬레이터인 duck-sim 위에, 최신 가성비 최강 소형 모델인 Qwen2.5-3B와 앤트로픽이 제안한 인프라 규격인 MCP(Model Context Protocol)를 얹어 AI 에이전트가 스스로 로봇을 제어하도록 구현한 성공 과정을 공유합니다.
HW 장비 없이 오직 리눅스 PC 환경(시뮬레이터)에서만 구현했기 때문에, 누구나 쉽게 따라 해볼 수 있는 아키텍처입니다!

## 🏗️ 전체 시스템 아키텍처 (System Architecture)

이번 프로젝트의 핵심은 "LLM이 어떻게 시뮬레이터의 내부 JSON-RPC 통신 규격을 이해하고 명령을 내릴 것인가?"였습니다. 저는 이를 위해 앤트로픽(Anthropic)이 오픈소스로 공개한 MCP(Model Context Protocol)를 브릿지로 활용했습니다.
• LLM Core: Qwen2.5-3B (경량 모델임에도 Tool Calling 및 구조화된 출력이 매우 뛰어남)
• Bridge (Orchestrator): MCP Server (로봇 제어용 JSON-RPC API를 LLM이 사용할 수 있는 'Tool' 형태로 변환 및 중계)
• Simulator: duck-sim (MuJoCo 기반 가상 가동 환경 + 실기 데몬 매핑)
데몬 간 통신 구조가 원래 JSON-RPC로 단일화되어 있어서, MCP Server를 구축하여 LLM에게 Tool 형태로 쥐여주기가 매우 수월했습니다.

## 🛠️ 핵심 구현 단계


### 1. duck-sim 가상 환경 준비 (Linux)

가장 먼저 리눅스 환경에서 실제 로봇 없이 제어 데몬들을 구동하는 duck-sim 환경을 세팅했습니다.

``` bash
# MuJoCo 기반의 시뮬레이터 구동 (실제 코어 데몬들과 가상 Body 연동)
./scripts/duck-sim
Use code with caution.
```
이 상태가 되면 백그라운드 소켓을 통해 robotd(50Hz 제어 루프), configd 등이 가상 환경에서 맞물려 돌아가기 시작합니다.

2. MCP(Model Context Protocol) 서버 구축

LLM이 robotctl이나 소켓 명령어 API 구조를 이해할 수 있도록 MCP Server를 래핑(Wrapping)했습니다. Qwen2.5 모델이 "오리에게 앞으로 한 걸음 가라고 해줘"라는 자연어를 들으면, MCP가 이를 인식해 적절한 도구(Tool)를 매핑해 줍니다.
• Exposed Tools via MCP:
	• walk(direction, speed)
	• grasp()
	• standup()
	• get_status() (시뮬레이터 상태 모니터링)

### 3. Qwen2.5-3B 연동 및 프롬프트 엔지니어링

3B라는 초경량 사이즈인 Qwen2.5-3B를 선택한 이유는 로컬 리눅스 환경에서 저지연(Low Latency)으로 실시간 피드백 루프를 돌리기 위함이었습니다.
LLM 시스템 프롬프트에 로봇의 상태 조건(예: 넘어짐, 앞에 물체가 있음)에 따라 어떤 MCP 도구를 호출해야 하는지 가이드라인을 제공했습니다.

## 🚀 결과: LLM Agent의 자율 제어 시나리오 테스트

연동 완료 후, LLM에게 가상의 상황을 부여하고 자율적으로 제어 루프를 수행하도록 테스트했습니다.

테스팅 시나리오: 물건을 집어오다가 넘어졌을 때 (Sim)

1. 상황 발생: 시뮬레이터 내부에서 오리가 물건을 집으려고 (grasp) 시도하다가 균형을 잃고 쓰러짐.
2. 상태 감지: MCP Server를 통해 LLM이 오리의 현재 상태 정보를 실시간으로 받아옴 (robotctl health 및 오도메트리 센서값 파싱).
3. LLM 추론: Qwen2.5-3B 모델이 상태 코드(기울어짐 확인)를 분석한 뒤, "먼저 스스로 일어나야(Standup) 다음 동작이 가능하다"고 판단함.
4. 도구 호출 및 실행: LLM이 MCP 인터페이스를 통해 자동으로 standup() 툴을 실행하고, 오리가 가상 환경에서 정상적으로 일어선 것을 확인한 후 다시 목적지까지 walk() 명령을 수행함.

## 💡 이번 연동을 통해 느낀 점 (Takeaways)

• MCP의 강력함: 예전 같았으면 LLM의 텍스트 출력을 정규식으로 복잡하게 파싱해서 가상 환경 스크립트에 파이프라인으로 쏴줘야 했을 텐데, MCP 규격을 쓰니 LLM 친화적으로 API가 깔끔하게 중계되어 개발 공수가 대폭 줄었습니다.
• Qwen2.5-3B의 놀라운 성능: 3B 수준의 가벼운 모델임에도 불구하고 MCP 스키마를 정확히 이해하고 상황에 맞는 Tool Calling을 실수 없이 수행해 냈습니다. 로컬 로봇 에이전트용 백엔드로 충분한 가능성을 보였습니다.
• Sim 단계의 메리트: 하드웨어 고장이나 모터 파손 걱정 없이 리눅스 환경 안에서 LLM의 무한한 에이전트 루프 테스트(RL 피드백 수집 등)를 안전하게 진행할 수 있어 대단히 효율적이었습니다.

## 🔮 MoJoCo


MuJoCo(Multi-Joint dynamics with Contact)는 로봇 공학, 생체 역학, 인공지능(특히 강화학습) 연구에서 표준처럼 사용하는 고성능 물리 엔진(Physics Engine)입니다.
원래는 모션 연구소(Motion Geometry)에서 개발한 유료 상용 소프트웨어였으나, 2021년 DeepMind(구글 딥마인드)가 회사를 인수한 후 완전히 오픈소스로 전환하여 현재는 전 세계 연구자들이 무료로 널리 사용하고 있습니다.

💡 아주 쉬운 비유: 로봇 공학자들을 위한 '현실적인 가상현실'

게임에서 자동차가 벽에 부딪히면 튕겨 나가거나 부서지는 연출을 하려면 물리 엔진이 필요합니다.
• 일반적인 게임 엔진(Unity, Unreal 등)은 시각적인 화려함과 실시간 구동 속도를 중시하기 때문에 물리적 계산을 적당히 '타협'합니다.
• 반면 MuJoCo는 현실 세계의 물리 법칙을 소수점 아래 세 자릿수까지 정밀하게 계산하는 데 초점을 맞춥니다. 모터의 토크, 관절의 마찰력, 지면과의 접촉 역학 등을 정확하게 모방하므로, 여기서 학습한 로봇 제어 코드(microduck 등)를 실제 로봇에 올려도 오차 없이 그대로 작동(Sim-to-Real)하게 됩니다.

## 🚀 MuJoCo의 핵심 특징

• 정밀한 접촉(Contact) 처리: 로봇의 발이 땅에 닿을 때 발생하는 미끄러짐, 부리나 손으로 물건을 집을 때(grasp)의 마찰력과 반발력을 매우 정교하게 계산합니다.
• 멀티 조인트(Multi-Joint) 최적화: 인간의 관절이나 오리 로봇처럼 여러 개의 다관절 구조(Articulated Structure)가 얽혀 있는 복잡한 시스템을 계산하는 속도가 현존하는 물리 엔진 중 가장 빠른 편에 속합니다.
• 강화학습과의 찰떡궁합: 인공지능이 "이렇게 움직이면 넘어지는구나", "저렇게 움직이면 앞으로 가는구나"를 수백만 번 반복하며 학습해야 하므로 시뮬레이션 속도가 빨라야 합니다. MuJoCo는 속도가 압도적으로 빨라 AI 학습 환경으로 가장 선호됩니다.
• XML 기반 모델링 (MJCF): 로봇의 크기, 무게, 모터 성능, 관절 위치 등을 .xml 파일 형태의 자체 규격(MJCF)으로 쉽게 기술하고 편집할 수 있습니다.

## 🦆 이번 프로젝트(duck-sim)에서의 역할

./scripts/duck-sim 환경에서, 눈에 보이는 가상 오리의 몸체와 오리가 딛고 있는 바닥, 그리고 오리가 넘어지거나 물건을 집는 물리적인 상호작용 전체를 계산해 주던 무대 뒷편의 진짜 핵심 엔진이 바로 이 MuJoCo였습니다.
이 가상 공간 덕분에 값비싼 실제 하드웨어가 부서질 걱정 없이 Qwen2.5 모델과 MCP를 붙여 마음껏 에이전트 자율 제어 테스트를 하실 수 있었던 것입니다.
