---
title: "GDB와 Gemini AI를 활용한 코어 덤프 자동 분석"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - GDB
  - Gemini
  - LangChain
  - Python
  - C++
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: GDB Python 스크립트와 Gemini AI를 결합하여 C++ 코어 덤프를 자동으로 분석하는 방법
---

GDB의 Python 인터페이스와 Google의 Gemini AI를 활용하여, 크래시 발생 시 생성되는 코어 덤프(Core Dump)를 자동으로 분석하고 리포트를 생성하는 워크플로우를 만들어 봄.

## 1. 프로젝트 구조

전체 시스템은 다음과 같은 단계로 동작함.

1.  **크래시 유도**: `main.cpp`에서 의도적인 NULL 포인터 접근으로 코어 덤프 생성.
2.  **데이터 추출**: GDB 내부에서 `extract_crash.py`를 실행하여 스택 트레이스, 레지스터, 어셈블리 정보를 JSON으로 추출.
3.  **AI 분석**: `analyze_crash.py`가 LangChain을 통해 Gemini AI에게 추출된 데이터를 전달하고 디버깅 리포트를 요청.

## 2. 환경 설정

먼저 분석에 필요한 라이브러리와 API 키를 설정.

```bash
pip install langchain-google-genai
export GOOGLE_API_KEY='your-gemini-api-key'
```

시스템에서 코어 덤프가 생성되도록 설정을 확인.

```bash
ulimit -c unlimited
sudo sysctl -w kernel.core_pattern=core
```

## 3. 핵심 구현 내용

### GDB 데이터 추출 (extract_crash.py)

GDB의 Python API를 사용하면 디버깅 정보를 프로그래밍 방식으로 제어할 수 있음.

```python
import gdb

# 현재 스레드 정보 및 백트레이스 추출
frame = gdb.selected_frame()
pc = frame.pc()
# ... 레지스터 및 소스 코드 정보 수집 후 JSON 저장
```

### Gemini AI 분석 (analyze_crash.py)

추출된 JSON 데이터를 Gemini 모델에 전달하여 분석 리포트를 작성.

```python
model = ChatGoogleGenerativeAI(model="gemini-3-flash-preview")
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 시스템 소프트웨어 엔지니어이자 디버깅 전문가입니다."),
    ("user", "아래 데이터를 분석해줘: {data}")
])

chain = prompt | model | StrOutputParser()
response = chain.invoke({"data": crash_data})
```

## 4. 실행 결과

애플리케이션을 빌드하고 크래시를 발생시킨 뒤, 분석 스크립트를 실행.

```bash
make && ./release_bin/my_app
python3 analyze_crash.py
```

`output.md` 파일에 생성된 분석 결과의 예시.

```markdown
### 1. Crash 요약
- **Signal**: SIGSEGV (Segmentation fault)
- **발생 지점**: `main.cpp:10` (foo 함수 내부)

### 2. 원인 분석
- `$pc` 레지스터가 `0x0` 주소를 가리키는 `ptr`을 참조하려 함.
- 어셈블리 분석 결과 `mov (%rax), %eax` 명령에서 잘못된 메모리 참조 발생.

### 3. 해결 제안
- `foo` 함수 호출 전 포인터 유효성 검사 추가 필요.
- `if (ptr != nullptr) { ... }` 블록으로 보호.
```

## 마무리

GDB와 최신 LLM을 결합하면 단순한 백트레이스 확인을 넘어, 어셈블리 레벨의 정밀 분석과 구체적인 해결 방안까지 자동으로 도출할 수 있음.

---
#### 소스 코드

- https://github.com/fall20style/docker_prep/tree/main/gdb_py_llm_automation/project
