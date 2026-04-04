## Anthropic의 하네스(Harness) 구조 분석

### 1. 분리 이유 (배경)

* 긴 문서 요약이나 복잡한 코딩 시, 생성과 검증을 동시에 하면 실수 발견이 어려움.
* '작성자'와 '검토자'를 나누어 상호보완하는 체계를 구축함.

### 2. 주요 구성 요소

* 생성기(Generator): 사용자 요청에 맞춰 코드나 초안 등 결과물을 생성함.
* 평가기(Evaluator): 정의된 기준(정확성, 스타일 등)에 따라 결과물을 엄격히 심사함.
* 루프(Loop): 평가기를 통과할 때까지 피드백과 수정을 무한 반복함.

### 3. 하네스 구조의 장점

* 품질 보증: 반복 확인을 통해 최종 결과물의 신뢰도를 극대화함.
* 전문성 강화: 생성기는 창의성에, 평가기는 논리적 검증에만 집중함.
* 환각 감소: 평가기가 사실 관계를 대조하여 거짓 정보를 효과적으로 걸러냄.

### 4. 작동 방식 예시

   1. 입력: 대규모 보고서 요약 요청함.
   2. 생성: 생성기가 요약본을 작성함.
   3. 평가: 평가기가 누락된 수치나 왜곡된 내용을 체크함.
   4. 수정: 통과 못 하면 평가기가 피드백을 주고, 생성기가 이를 반영해 다시 작성함.

이 구조는 장시간 실행(Long-running)이 필요한 복잡한 작업에서 AI가 일관성을 유지하도록 돕는 핵심 프레임워크.


## 코딩 프로젝트에 하네스(Harness) 구조를 적용

### 1. 시나리오: "복잡한 API 연동 모듈 작성"

사용자가 "특정 서비스의 API를 사용하여 데이터를 가져오고 가공하는 파이썬 스크립트를 짜줘"라고 요청한 상황.

### 2. 하네스 구조 작동 방식

* 생성기 (Generator) - "코딩 담당"
* 사용자 요구사항을 읽고 전체 코드 구조를 설계함.
   * API 문서를 참고해 인증 로직, 데이터 수집 함수, 가공 로직을 작성함.
   * 결과물로 api_client.py 초안을 내놓음.
* 평가기 (Evaluator) - "코드 리뷰어"
* 정확성 체크: "API 키가 하드코딩되지 않았는가?", "예외 처리(Error Handling)가 되어 있는가?" 확인함.
   * 보안/효율성: "비효율적인 루프가 있는가?", "보안 취약점은 없는가?" 검사함.
   * 테스트 실행: 코드를 가상 환경에서 돌려보고 에러 발생 시 로그를 수집함.
* 루프 (Loop) - "수정 및 보완"
* 평가기가 "API 호출 실패 시 재시도(Retry) 로직이 빠졌음"이라는 피드백을 생성기에 던짐.
   * 생성기는 피드백을 바탕으로 코드를 수정함.
   * 평가기가 "통과"를 외칠 때까지 이 과정을 반복함.

### 3. 코딩 프로젝트에서의 장점

* 버그 감소: 사람이 직접 디버깅하기 전에 AI가 스스로 오류를 잡아내서 완성도가 높음.
* 일관성 유지: 수천 줄의 대규모 프로젝트에서도 평가기가 전체 스타일(Lint)과 규칙을 감시함.
* 엣지 케이스 대응: "네트워크가 끊기면 어떻게 돼?" 같은 질문을 평가기가 던져서 예외 상황까지 대비함.

* 평가기에게 유닛 테스트(Unit Test) 코드를 짜게 시키고, 그걸 직접 실행해서 통과 여부를 따지게 하면 더 좋음.



## 생성기, 평가기 -> 에이전트 역할
- 생성기와 평가기 각각이 독립적인 에이전트(Agent) 역할을 수행.
- 특정 목적을 가진 '두 명의 AI 전문가'가 협업하는 구조임.

### 1. Why

* 독립적 사고: 생성기는 '창작'에, 평가기는 '비판'에 최적화된 서로 다른 프롬프트(지시문)를 가짐.
* 도구 사용: 평가기 에이전트는 코드를 직접 실행하거나, 외부 문서(API)를 검색하는 등 능동적으로 움직임.
* 자율적 루프: 사람이 개입하지 않아도 두 에이전트가 서로 대화하며 결과물을 완성할 때까지 반복함.

### 2. 멀티 에이전트 하네스의 핵심

* 페르소나 분리: 생성기는 "천재 개발자", 평가기는 "깐깐한 보안 전문가/QA" 역할을 맡음.
* 컨텍스트 공유: 두 에이전트는 같은 대화 기록이나 파일 시스템을 공유하며 정보를 주고받음.
* 종료 조건: 평가기 에이전트가 "승인(Approve)" 사인을 보낼 때까지 프로세스가 멈추지 않음.

### 3. 코딩 예시에서의 역할 분담

* 생성기 에이전트: "이 기능 구현을 위해 A 라이브러리를 쓰고, 구조는 이렇게 짬."
* 평가기 에이전트: "A 라이브러리는 보안 취약점이 있음. B로 바꾸고 다시 짜오셈."
* 생성기 에이전트: "알겠음. B로 바꾸고 테스트 코드도 추가했음."

결론적으로, 하네스는 '생성 에이전트'와 '평가 에이전트'가 서로 꼬리에 꼬리를 무는 멀티 에이전트 시스템의 전형적인 모델.




## Rust + Axum API 개발 하네스 구성

## 1. 생성기 에이전트 (Generator)

* 역할: 사용자의 요구사항을 바탕으로 Axum REST API 소스 코드 및 Cargo.toml 작성.
* 핵심: 아키텍처 설계 및 비즈니스 로직 구현.

## 2. 평가기 에이전트 (Evaluator)

* 역할: 생성된 코드의 정적 분석 및 Rust 컨벤션(Idiomatic Rust) 검토.
* 핵심: 보안 취약점, 에러 핸들링, State 관리의 적절성 심사.

## 3. 검증기 에이전트 (Verifier)

* 역할: 코드의 동적 검증 및 실행 가능성 확인.
* 작동 방식:
1. Unit Test 생성: 코드 로직에 맞는 #[cfg(test)] 모듈 및 통합 테스트 코드 작성.
   2. 컴파일 시뮬레이션: cargo build 및 cargo check 시 발생할 수 있는 소유권(Ownership) 문제나 타입 불일치 예측.
   3. 런타임 체크: cargo run 시 엔드포인트가 정상적으로 바인딩되는지, 패닉(Panic) 가능성이 없는지 검토.
* 출력: PASS / FAIL 리포트 및 상세 컴파일 에러 메시지(Mock-up).

------------------------------
## 프로젝트용 갱신 프롬프트 (System Prompt)

```
# Role: Rust Axum API Development & Verification Harness
## [Phase 1: Generator]- Objective: Generate production-ready Axum REST API code.
- Requirement: Provide `Cargo.toml` and structured `src/*.rs`.
- Target: Async/Await with `tokio`, error handling with `anyhow`.

## [Phase 2: Evaluator]- Objective: Static code review & Security audit.- Focus: Axum Extractor usage, Shared State thread-safety, API Spec matching.
- Output: Review summary with [CRITICAL/ADVISORY].

## [Phase 3: Verifier]- Objective: Dynamic validation via Testing.- Tasks:
  1. Generate Unit Tests for all endpoints and business logic.
  2. Simulate `cargo build` and `cargo run` execution.
  3. Verify HTTP status codes and JSON response structures.
- Output: `[VERIFICATION RESULT]` (Status: PASS/FAIL, Log: Test results).

## [Updated Execution Loop]1. **Input:** User provides API requirements.2. **Gen:** Generator creates initial code and project structure.3. **Eval:** Evaluator reviews the code. If failed, return to Gen.4. **Verify:** 
   - Verifier writes and "runs" unit tests.
   - If `cargo build` or tests fail, Verifier provides error logs to Gen.5. **Finalize:** Loop ends only when BOTH Evaluator and Verifier give "GREEN LIGHT (PASS)".
```

## 갱신된 루프의 핵심 (Feedback Cycle)

* 컴파일 기반 피드백: Verifier가 cargo build 실패를 시뮬레이션하면, 생성기는 단순히 코드를 고치는 게 아니라 Rust 컴파일러의 지시사항(Error E0xxx)을 반영하여 수정함.
* 테스트 주도 개발(TDD): Verifier가 미리 작성한 유닛 테스트를 통과해야만 최종 결과물로 인정되므로, API의 신뢰도가 비약적으로 향상됨.

## 느낀점
1. 스펙과 AGENT.md README.md를 주면, 아웃풋이 생성됨.
2. 프롬프팅보다 편리

## 위치
- https://github.com/fall20style/rust_prep/tree/main/04_gemini_harness_calc_api
