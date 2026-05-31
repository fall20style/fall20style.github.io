---
title: "Gemini Hook으로 작업 자동화: 시간 추적 및 요약 기록"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - gemini-cli
  - hooks
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "Gemini Hook으로 작업 자동화: 시간 추적 및 요약 기록"
---

- Hook을 활용하여 개발 워크플로우를 자동화
- 프롬프트 요청의 실행 시간과 내용을 자동으로 기록

## Gemini CLI Hook 시스템 구축

Gemini CLI의 Hook 기능을 활용하여 프롬프트 요청이 끝날 때마다 자동으로 정보를 기록하는 시스템을 구축.

### 1. Hook 이란?

Gemini CLI의 Hook은 특정 이벤트(예: `BeforeAgent`, `AfterAgent`)가 발생했을 때 사용자 정의 스크립트를 실행할 수 있도록 하는 메커니즘. 

이를 통해 Gemini CLI의 동작을 확장하거나 사용자 정의 워크플로우를 자동화.

### 2. 구현 상세

**a) 기능**:
구현된 Hook 시스템은 각 프롬프트 요청이 완료될 때마다 다음 정보를 `/home/mjpark/work/gemini_log/CHECKLIST.csv` 파일에 자동으로 기록.
*   **타임스탬프**: `YYYY-MM-DD-HH:mm:ss` 형식의 실행 시간
*   **요약**: 프롬프트의 첫 줄을 최대 50자까지 요약 (쉼표 제거)
*   **소요 시간**: 작업에 걸린 시간 (예: `1.07s`)

**b) Hook 스크립트**:
두 가지 종류의 Hook 스크립트를 사용했습니다.
*   `before-agent.js`: `BeforeAgent` 이벤트 발생 시, 현재 시간을 밀리초 단위로 `.gemini/tmp/` 디렉토리에 임시 저장.
*   `after-agent.js`: `AfterAgent` 이벤트 발생 시, 저장된 시작 시간을 읽어와 현재 시간과의 차이로 소요 시간을 계산. 이후, 프롬프트 내용을 요약, 형식화된 타임스탬프와 함께 `CHECKLIST.csv` 파일에 모든 정보를 추가.

**c) Node.js 선택 이유**:
*   **표준 환경 및 일관성**: Gemini CLI 자체가 Node.js 환경에서 구동되므로, Node.js를 사용하는 것이 가장 호환성이 높음.
*   **풍부한 표준 라이브러리**: 파일 시스템(`fs`), 경로 처리(`path`), JSON 파싱 등 Hook에서 필요한 기능들을 별도 라이브러리 없이 Node.js 내장 모듈만으로 간결하게 구현 가능.
*   **JSON 처리 최적화**: Gemini CLI가 Hook에 데이터를 JSON 형식으로 전달, Node.js는 이를 쉽게 파싱하고 처리할 수 있음.
*   **확장성**: 향후 API 호출 등 더 복잡한 기능이 필요할 경우, NPM 생태계를 통해 쉽게 기능을 확장할 수 있음.d

**d) 설정**:
이 Hook들은 프로젝트 디렉토리(`.gemini/settings.json`)에 다음과 같이 등록되어, 해당 프로젝트 내에서 Gemini CLI 사용 시 자동으로 실행.

```json
{
  "hooks": {
    "BeforeAgent": [
      {
        "matcher": "*",
        "hooks": [
          {
            "name": "before-agent-logger",
            "type": "command",
            "command": "node .gemini/hooks/before-agent.js"
          }
        ]
      }
    ],
    "AfterAgent": [
      {
        "matcher": "*",
        "hooks": [
          {
            "name": "after-agent-logger",
            "type": "command",
            "command": "node .gemini/hooks/after-agent.js"
          }
        ]
      }
    ]
  }
}
```

### 3. 파일 구조

구현된 Hook 시스템은 다음과 같은 파일 구조를 가짐

```
/home/mjpark/work/gemini_log/
├── CHECKLIST.csv
├── README.md
├── .gemini/
│   ├── settings.json
│   ├── hooks/
│   │   ├── before-agent.js
│   │   └── after-agent.js
│   └── tmp/
```

### 4. 사용 범위

현재 설정한 Hook은 `.gemini/settings.json` 파일이 위치한 `/home/mjpark/work/gemini_log/` 프로젝트 디렉토리 및 그 하위 경로에서만 동작. 이는 Gemini CLI가 해당 프로젝트의 설정을 우선적으로 사용하기 때문. 다른 디렉토리에서 Gemini CLI를 실행할 경우, 이 Hook들은 자동으로 실행되지 않음.

### 5. script로 test하기

```bash
~/work/gemini_log$ ./test_hooks.sh 
--- Hook 테스트 시작: Session ID: test-manual-1780240073 ---

[INFO] '.gemini/hooks/before-agent.js' 실행 중...
{}
[SUCCESS] 시작 파일이 성공적으로 생성되었습니다: .gemini/tmp/gemini_start_test-manual-1780240073.txt

[INFO] 작업 시간 시뮬레이션 (1초 대기)...
sleep 1

[INFO] '.gemini/hooks/after-agent.js' 실행 중...
{}

[INFO] 'CHECKLIST.csv' 파일 내용 확인:
2026-05-31-08:30:00, test prompt, 0.00s
2026-05-31-08:35:00, second test, 1.07s
2026-05-30-23:19:24, keep working, 0.00s
2026-05-30-23:25:05, 지금껏 작업 내용을 /home/mjpark/work/gemini_log에 README.md, 20.97s
2026-05-30-23:34:26, 등록된 hook이 뭐가 있는지 list up 해줘, 8.04s
2026-05-30-23:36:23, 훅은 이 dir 아래에서만 동작하는 건가?, 2.64s
2026-05-30-23:36:50, 위 설명을 README.md에 추가해 줘., 4.08s
2026-05-30-23:40:02, hook 작성하는 방법 적어줘. HOOK_GUIDE.md로 저장, 7.14s
2026-05-31-00:06:13, command를 node를 이용해서 했네. 특별한 이유가 잇어?, 5.20s
2026-05-31-00:06:31, 이 설명도 README.md에 적어줘, 5.08s
2026-05-31-00:10:58, 이 dir의 md파일을 모두 읽고  blog에 소개할 수 있도록 post를 작성해 줘. /, 15.25s
2026-05-31-00:11:53, 이 dir의 md파일을 모두 읽고  blog에 소개할 수 있도록 post를 작성해 줘., 36.03s
2026-05-31-21:23:46, s, 40.29s
2026-05-31-23:24:29, 프롬프트 호출 없이  지금 작성한 hook을 test하는 방법을 md파일로 작성해 줘, 19.00s
2026-05-31-23:25:03, 프롬프트 호출 없이  지금 작성한 hook을 test하는 방법을 md파일로 작성해 줘, 14.73s
2026-05-31-23:26:55, TEST_HOOK_GUIDE.md 에 저장해 줘, 17.27s
2026-06-01-00:06:43, TEST_HOOK_GUIDE.md를 읽고 bash script를 작성해 줘, 15.33s
2026-06-01-00:07:40, Manual test of hook sequence., 0.06s
2026-06-01-00:07:53, Manual test of hook sequence., 0.05s

[INFO] 임시 시작 파일 삭제 확인...
[SUCCESS] 임시 시작 파일이 성공적으로 삭제되었습니다.

--- Hook 테스트 완료 ---
~/work/gemini_log$ ls
CHECKLIST.csv  HOOK_GUIDE.md  README.md  TEST_HOOK_GUIDE.md  blog_post_about_hooks.md  test_hooks.sh
~/work/gemini_log$ vi test_hooks.sh 
```
