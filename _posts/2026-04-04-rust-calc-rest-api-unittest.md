---
title: Rust Axum으로 Unit Test하고 실행
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - axum
  - Rust
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 코드 위치
https://github.com/fall20style/rust_prep/tree/main/01_axum_test_demo

데이터 구조, 비즈니스 로직(핸들러), 라우팅 설정, 그리고 테스트의 4개 부분.

## 분석

### 1. 데이터 구조 (Models)

#[derive(Serialize, Deserialize, Debug, PartialEq)]pub struct User { ... }

#[derive(Deserialize)]pub struct CalcInput { a: i32, b: i32 }

#[derive(Serialize)]pub struct CalcOutput { result: i32 }


* #[derive(Serialize, Deserialize)]: Rust의 데이터를 JSON으로 변환(Serialize)하거나, 반대로 JSON을 Rust 구조체로 변환(Deserialize)
* CalcInput은 사용자가 보내는 숫자 두 개를 받고, CalcOutput은 계산 결과를 담아 응답할 때 사용

### 2. 비즈니스 로직 (Handlers)

```
async fn compute(Json(payload): Json<CalcInput>) -> Json<CalcOutput> {
    let sum = payload.a + payload.b;
    Json(CalcOutput { result: sum })
}
```


* 핸들러: 특정 URL로 요청이 들어왔을 때 실행되는 함수.
* Json(payload): Axum의 'Extractor' 기능. HTTP 요청의 바디(Body)에 담긴 JSON을 자동으로 CalcInput 구조체로 변환.
* 반환값인 Json(CalcOutput)은 결과를 다시 JSON 형식으로 응답하겠다는 뜻.

### 3. 라우팅 설정 (App Router)

```
pub fn app() -> Router {
    Router::new()
        .route("/user", get(|| async { ... }))
        .route("/calc", post(compute))
}
```


* Router: 어떤 경로(URL)로 들어온 요청을 어떤 핸들러에 연결할지 결정하는 지도 역할.
* .route("/calc", post(compute)): /calc 주소로 POST 요청이 오면 compute 함수를 실행하라는 설정.
* app() 함수를 그대로 가져다 쓰기 위해 별도의 함수로 분리.

### 4. 서버 실행 (Main)

```
#[tokio::main]async fn main() {
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app()).await.unwrap();
}
```


* tokio::main: 비동기 실행 환경을 만들어주는 진입점.
* TcpListener: 3000번 포트를 열고 클라이언트의 접속을 기다림.

### 5. 테스트 모듈 (Tests)

```
#[cfg(test)]mod tests { ... }
```

* oneshot: 실제 서버를 구동(네트워크 연결)하지 않고, 메모리 상에서 가상의 요청을 라우터에 던져 응답을 바로 확인하는 방식.
* assert_eq!: 예상한 결과(예: 30)와 실제 응답값이 일치하는지 검증.



## 빌드 명령 정리
### 1. 프로젝트 생성 및 이동

새 프로젝트 만들고 폴더로 들어감
```
cargo new axum_test_demo
cd axum_test_demo
```

### 2. 라이브러리 설치 (의존성 추가)

서버랑 JSON 처리에 필요한 애들임
```
cargo add axum@0.7
cargo add serde@1.0 --features derive
cargo add serde_json@1.0
cargo add tokio@1.0 --features full
# 테스트할 때만 쓰는 도구들임 (--dev 옵션)
cargo add tower@0.4 --features util --dev
cargo add http@1.0 --dev
cargo add mime@0.3 --dev
cargo add http-body-util@0.1 --dev
```

### 3. 코드 검사 및 빌드

코드에 문법 에러 없는지 빠르게 확인함
```
cargo check
```
전체 프로젝트 컴파일해서 실행 파일 만듦
``
cargo build
```

### 4. 테스트 및 실행

작성한 테스트 케이스(GET, POST, 계산기) 다 돌려봄
```
cargo test
```
실제 서버 띄워서 대기시킴 (http://localhost:3000)
```
cargo run
```

### 5. 코드 깔끔하게 정리

들여쓰기랑 줄바꿈 알아서 예쁘게 맞춰줌

```
cargo fmt
```
