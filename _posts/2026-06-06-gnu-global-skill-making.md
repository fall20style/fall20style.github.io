---
title: "gnu global을 이용한 skill만들기"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - "GNU global"
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "gnu global을 이용한 skill만들기"
---

## Gemini에서 grep을 이용해서 코드를 분석함

- GNU Global을 사용하도록 함

```
~/.gemini/skills/gnu-global-nav$ tree
.
└── SKILL.md

0 directories, 1 file
```

### SKILL.md 
```
---
name: gnu-global-nav
description: High-speed code navigation using GNU Global (gtags). Use this skill to find symbol definitions, references, and navigate large C/C++ or Java codebases more efficiently than grep. It provides precise symbol matching and reduces token usage by targeting specific lines.
---

# GNU Global Navigation Skill

This skill prioritizes the use of GNU Global (`gtags`) for precise and fast code navigation, falling back to `grep_search` only when necessary.

## Workflow

### 1. Index Check & Initialization
Before searching, check if the GNU Global index exists:
- Look for `GTAGS`, `GRTAGS`, and `GPATH` files in the project root.
- If they don't exist, run `gtags` to initialize the index.
- If the codebase has changed significantly, run `global -u` to update the index.

### 2. Precise Symbol Search
Use `global` for the following tasks to minimize token usage and turns:

- **Find Definition**: `global -d <symbol>`
  - Use this to jump directly to where a class, function, or variable is defined.
- **Find References**: `global -r <symbol>`
  - Use this to find all call sites or usages of a symbol.
- **Find Other Symbols**: `global -s <symbol>`
  - Use this for symbols not defined in GTAGS (e.g., external headers, some macros).
- **Find Files**: `global -P <pattern>`
  - Find pathnames that match the pattern.
- **List Symbols in File**: `global -f <file>`
  - Get an overview of all symbols defined in a specific file.

### 3. Fast String Search (Grep using Index)
- **Search Pattern**: `global -g <pattern>`
  - This is often faster than `grep_search` because it uses the GTAGS index for pre-filtering.

### 4. Handling Results
- **Single Result**: Use the line number provided by `global` to perform a surgical `read_file` with `start_line` and `end_line`.
- **Multiple Results**: If `global` returns many matches, analyze the file paths and line contents provided in the output to select the most relevant one before reading.
- **No Results**: If `global` finds nothing, fall back to `grep_search` for a more exhaustive (but slower) search.

## When to Fall Back to `grep_search`
- When searching for literal strings that are not symbols (e.g., log messages, comments).
- When complex regular expressions are required that `global -g` cannot handle efficiently.
- When searching in file types not supported or indexed by your current `gtags` configuration.

## Example
Instead of:
`grep_search(pattern="class MyClassName")`

Use:
`run_shell_command(command="global -d MyClassName")`
```


## gRPC HelloWorld 예제 Call Graph 상세 분석

이 문서는 `examples/cpp/helloworld`에 위치한 gRPC C++ 예제의 함수 호출 흐름과 각 호출의 역할을 설명합니다.

### 1. Client Side (`greeter_client.cc`)

클라이언트는 서버에 연결하고 `SayHello` RPC를 요청하는 과정을 거칩니다.

| 함수 호출 | 설명 |
| :--- | :--- |
| **`main()`** | 프로그램의 진입점입니다. |
| ├── `absl::ParseCommandLine()` | 명령줄 인자(예: `--target`)를 파싱합니다. |
| ├── `grpc::InsecureChannelCredentials()` | 보안 인증이 없는(Insecure) 채널 자격 증명을 생성합니다. |
| ├── `grpc::CreateChannel(target, ...)` | 지정된 주소로 gRPC 채널을 생성하여 연결을 시도합니다. |
| ├── **`GreeterClient` 생성자** | 채널을 받아 서비스와 통신할 Stub을 초기화합니다. |
| │   └── `helloworld::Greeter::NewStub()` | 프로토콜 버퍼로 생성된 코드를 사용하여 서비스용 Stub 객체를 생성합니다. |
| └── **`GreeterClient::SayHello(user)`** | 사용자 이름을 인자로 받아 실제 RPC 호출을 수행하는 고수준 메서드입니다. |
|     ├── `helloworld::HelloRequest::set_name()` | 요청(Request) 메시지에 사용자 이름을 설정합니다. |
|     ├── `ClientContext` 생성 | RPC 호출 시 메타데이터나 타임아웃 등을 제어하기 위한 컨텍스트입니다. |
|     ├── **`stub_->SayHello(&context, ...)`** | **[핵심]** 서버로 동기식 RPC 호출을 보냅니다. 응답이 올 때까지 대기합니다. |
|     └── `helloworld::HelloReply::message()` | 서버로부터 받은 응답 메시지에서 텍스트를 추출합니다. |

---

### 2. Server Side (`greeter_server.cc`)

서버는 특정 포트에서 대기하며 클라이언트의 요청을 처리할 서비스를 등록합니다.

| 함수 호출 | 설명 |
| :--- | :--- |
| **`main()`** | 프로그램의 진입점입니다. |
| ├── `absl::InitializeLog()` | Abseil 로깅 라이브러리를 초기화합니다. |
| └── **`RunServer(port)`** | 서버를 구성하고 실행하는 주 함수입니다. |
|     ├── `ServerBuilder` 객체 생성 | gRPC 서버 설정을 조립하기 위한 빌더입니다. |
|     ├── `builder.AddListeningPort()` | 서버가 바인딩할 주소와 포트, 보안 설정을 지정합니다. |
|     ├── `builder.RegisterService(&service)` | `GreeterServiceImpl` 인스턴스를 서비스 처리기로 등록합니다. |
|     ├── `builder.BuildAndStart()` | 설정을 기반으로 서버를 빌드하고 네트워크 리스닝을 시작합니다. |
|     └── `server->Wait()` | 서버가 종료될 때까지 메인 스레드를 블로킹 상태로 유지합니다. |

#### RPC 이벤트 발생 시 처리 흐름 (Server Implementation)

클라이언트의 요청이 도달하면 gRPC 런타임은 등록된 서비스 메서드를 호출합니다.

| 함수 호출 | 설명 |
| :--- | :--- |
| **`GreeterServiceImpl::SayHello()`** | `Greeter::Service`를 상속받아 오버라이드한 실제 비즈니스 로직입니다. |
| ├── `request->name()` | 클라이언트가 보낸 요청 메시지에서 데이터를 읽습니다. |
| ├── `reply->set_message()` | 응답 메시지에 "Hello " 접두사와 함께 데이터를 채웁니다. |
| └── `grpc::Status::OK` | 처리가 성공했음을 나타내는 상태 코드를 반환합니다. |

---

### 3. 요약: gRPC 통신 메커니즘
1.  **IDL (helloworld.proto)**: 서비스와 메시지 규격을 정의합니다.
2.  **Stub**: 클라이언트 측의 대리자 객체로, 네트워크 통신 복잡성을 캡슐화합니다.
3.  **Service Interface**: 서버 측에서 구현해야 하는 인터페이스입니다.
4.  **Synchronous Model**: 본 예제는 호출 결과가 돌아올 때까지 대기하는 동기식 모델을 사용합니다.



