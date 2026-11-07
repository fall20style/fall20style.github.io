---
title: 4GB VRAM 노트북(Intel Arc GPU)으로 나만의 로컬 AI 코딩 에이전트 구축
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

# 4GB VRAM 노트북(Intel Arc GPU)으로 나만의 로컬 AI 코딩 에이전트 구축하기 (`Qwen2.5-Coder-3B` + `my_agent.py`)

클라우드 API 비용이나 코드 보안 걱정 없이 내 노트북 안에서 직접 파일을 작성하고 실행하는 **로컬 코딩 에이전트**를 구축할 수 있을까요? 

이번 포스팅에서는 고가의 엔비디아(NVIDIA) 그래픽 카드가 아닌, **일반 노트북 사양(Windows + Intel Arc A350M VRAM 4GB + 16GB RAM)** 환경에서 **`Qwen2.5-Coder-3B`** 모델을 올리고, 초경량 자율 에이전트 **`my_agent.py`**를 직접 구현하며 겪은 핵심 인사이트 3가지를 공유합니다.

---

## 1. Intel GPU 환경에서 4GB VRAM의 한계 돌파하기

노트북에서 로컬 LLM을 돌릴 때 가장 먼저 마주하는 벽은 **하드웨어 구조와 메모리 한계**입니다.

### ① 내장 그래픽(GPU 1) vs 외장 그래픽(GPU 0)의 진실
작업 관리자를 열어보면 내장 그래픽(GPU 1)의 메모리가 7.8GB로 표시되어 AI 모델을 여기에 올리고 싶은 유혹에 빠지기 쉽습니다. 하지만 이는 일반 시스템 RAM을 빌려 쓰는 **공유 메모리(Shared Memory)**일 뿐이며, AI 행렬 연산을 가속하는 전용 **XMX 코어**가 없습니다. 따라서 진짜 전용 비디오 메모리(4GB)와 AI 가속 엔진을 품은 **외장 그래픽(`GPU 0`, Intel Arc A350M)**을 타겟팅해야 합니다.

### ② DLL 지옥을 피해 공식 Vulkan 바이너리로 정착하기
Windows 환경에서 인텔 GPU 가속을 쓰기 위해 `llama-cpp-python`을 직접 컴파일하거나 `ipex-llm`을 사용하면 Visual Studio C++ 컴파일러 부재나 `sycl7.dll` 런타임 버전 불일치(`Exit Code -1073741515`) 오류를 만나기 쉽습니다. 가장 깔끔한 해법은 **`llama.cpp` 공식 릴리즈의 Windows Vulkan 사전 빌드 바이너리(`llama-server.exe`)**를 사용하는 것입니다.

> **💡 핵심 팁:** Windows 작업 관리자의 `GPU 0`(Intel Arc A350M)은 Vulkan 드라이버에서 **`Vulkan1`**으로 매핑되는 경우가 많습니다. `--list-devices` 옵션으로 장치 번호를 확인한 뒤 `--device Vulkan1`을 명시해야 외장 GPU가 제대로 일합니다.

### ③ 레이어 분할 적재(Offloading)로 OOM 방지
`Qwen2.5-Coder-3B` (4비트 양자화 `Q4_K_M`, 약 2.2GB) 모델의 전체 36개 레이어를 4GB VRAM에 모두 올리면 대화 기억 공간(KV 캐시) 때문에 메모리 부족(`Out of Memory`)으로 튕깁니다. 전체 레이어 중 **14개만 GPU에 적재(`-ngl 14`)하고 나머지는 시스템 RAM으로 분산**시키면 안정성과 속도를 모두 잡을 수 있습니다.

```powershell
# Intel Arc GPU(Vulkan1)에 최적화된 로컬 API 서버 실행 명령어
.\llama-server.exe -m "./qwen2.5-coder-3b-instruct-q4_k_m.gguf" --device Vulkan1 --jinja -ngl 14 -c 16384 --temp 0.2 --top-p 0.8 --min-p 0.05 --repeat-penalty 1.15 --host 127.0.0.1 --port 8000
```

---

## 2. Qwen2.5-Coder-3B: 작지만 강한 코딩 특화 모델의 명과 암

알리바바의 **Qwen2.5-Coder-3B-Instruct**는 30억 개(3B) 파라미터라는 가벼운 체급에도 불구하고 코딩 벤치마크에서 구형 7B~13B 범용 모델을 뛰어넘는 뛰어난 코드 작성 능력을 보여줍니다.

하지만 실제 에이전트 프레임워크(DeepSeek Harness 등)에 연동해 본 결과, **경량 모델 특유의 한계점**도 명확히 드러났습니다.

* **무거운 시스템 프롬프트 소화 불가:** 상용 에이전트 툴들은 수많은 도구 설명서를 담아 약 **10,000토큰**에 달하는 거대한 시스템 프롬프트를 매번 전송합니다. 3B 모델은 이 긴 프롬프트에 압도되어 첫 응답에만 2분 가까이 걸리거나, 학습 데이터에 있던 중국어 메타데이터를 환각(Hallucination)으로 내뱉는 현상을 보였습니다.
* **Tool Calling 규격 누락:** 정식 도구 호출 태그(`<tool_call>`)를 빼먹고 날것의 JSON 텍스트만 출력하거나, 존재하지 않는 파일(`.DSH/files.txt`)을 읽으려 시도하는 실수가 발생했습니다.

**결론은 명확했습니다.** 3B 모델의 잠재력을 100% 끌어내려면 1만 토큰짜리 무거운 기성 하네스 대신, **프롬프트를 250토큰 이내로 극단적으로 다이어트한 전용 에이전트**가 필요했습니다.

---

## 3. 초경량 자율 코딩 에이전트 `my_agent.py` 설계 포인트

3B 모델이 헷갈리지 않고 스스로 코드를 작성·실행·수정(Self-Correction)할 수 있도록 파이썬으로 직접 구현한 **`my_agent.py`**에는 4가지 실전 엔지니어링 기법이 적용되었습니다.

### ① JSON 대신 XML 태그 파싱 도입 (`<write>`, `<shell>`)
3B 모델이 파이썬 GUI(`tkinter` 등)처럼 줄바꿈과 따옴표가 많은 긴 코드를 JSON의 `"content"` 문자열 안에 넣으려 하면 이스케이프 문법 오류(`Invalid control character`)가 빈번하게 발생합니다. 이를 해결하기 위해 직관적인 XML 태그 구조를 도입했습니다.

```xml
<write file="gugudan.py">
for i in range(1, 10):
    print(f"3 x {i} = {3 * i}")
</write>
```

### ② 자가 수정 루프 (Self-Correction Loop)
에이전트가 파일을 생성하지 않고 실행부터 시도해 `FileNotFoundError`가 발생하거나 코드에 문법 오류가 있을 때, 루프를 종료하지 않고 **에러 로그를 다시 모델에게 피드백**합니다. 모델은 에러 원인을 파악해 `<write>` 태그로 코드를 스스로 고친 뒤 `<shell>` 태그로 재실행합니다.

### ③ 작업 단위 컨텍스트 초기화 (문맥 오염 방지)
경량 모델은 이전 대화에서 한 번 실수하거나 사과한 기록이 대화 히스토리(`messages`)에 남아 있으면, 다음 명령을 주어도 이전 사과 패턴을 그대로 반복하는 경향이 있습니다. 새로운 사용자 명령(`input()`)이 들어올 때마다 대화 기록을 초기화하여 항상 맑은 정신(?)으로 작업에 임하게 했습니다.

### ④ 한글 Windows 환경 및 GUI 비동기 처리
* **인코딩 충돌 해결:** 한글 Windows PowerShell의 기본 출력(`CP949`)과 파이썬(`UTF-8`) 충돌을 막기 위해 바이트 단위로 출력을 받아 순차 디코딩(`safe_decode`)합니다.
* **GUI 창 타임아웃 예외 처리:** `tkinter` 등으로 윈도우 창을 띄우면 창이 닫힐 때까지 프로세스가 대기하게 됩니다. `subprocess.TimeoutExpired` 발생 시 에러가 아닌 'GUI 실행 완료' 상태로 처리하여 에이전트가 자연스럽게 보고를 마치도록 설계했습니다.

---

## 마치며

비록 4GB VRAM이라는 제한된 노트북 환경이었지만, **하드웨어 특성에 맞춘 레이어 분할 적재(Vulkan)**와 **경량 모델에 최적화된 프롬프트 및 파싱 설계(`my_agent.py`)**를 결합하자 인터넷 연결 없이도 스스로 코드를 짜고 에러를 고치는 실용적인 로컬 코딩 에이전트를 완성할 수 있었습니다. 

고가의 GPU가 없더라도 로컬 AI 에이전트 아키텍처의 원리를 깊이 이해하고 직접 구현해 보고 싶은 분들께 이 기록이 좋은 길잡이가 되기를 바랍니다.
