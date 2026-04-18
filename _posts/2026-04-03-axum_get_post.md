---
title: axum으로 GET/POST 예제
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

### 프로젝트 준비
```bash
cargo new axum_example
cd axum_example
cargo add axum
cargo add tokio --features full
cargo add serde --features derive
```


### Cargo.toml
```toml
[package]
name = "axum_example"
version = "0.1.0"
edition = "2024"

[dependencies]
axum = "0.8.8"
serde = { version = "1.0.228", features = ["derive"] }
tokio = { version = "1.51.0", features = ["full"] }
```

### src/main.rs
```rust
use ax_um::{
    routing::{get, post},
    Json, Router,
};
use serde::{Deserialize, Serialize};

// POST 요청에서 받을 데이터 구조체
#[derive(Deserialize)]
struct CreateUser {
    username: String,
}

// 응답으로 보낼 JSON 구조체
#[derive(Serialize)]
struct User {
    id: u64,
    username: String,
}

#[tokio::main]
async fn main() {
    // 1. 라우터 설정
    let app = Router::new()
        // GET 요청: 단순 문자열 응답
        .route("/", get(handler_hello))
        // POST 요청: JSON 데이터 처리
        .route("/users", post(create_user));

    // 2. 서버 실행 주소 설정 (localhost:3000)
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await.unwrap();
    println!("서버가 http://127.0.0.1:3000 에서 실행 중입니다.");
    
    axum::serve(listener, app).await.unwrap();
}

// GET 핸들러: 단순 문자열 반환
async fn handler_hello() -> &'static str {
    "Hello, Axum GET Request!"
}

// POST 핸들러: JSON 입력받아 JSON으로 응답
async fn create_user(Json(payload): Json<CreateUser>) -> Json<User> {
    let user = User {
        id: 1337,
        username: payload.username,
    };

    // Json()으로 감싸서 반환하면 Content-Type이 application/json으로 설정됩니다.
    Json(user)
}

```

### POST로 시험하기
```bash
curl -X POST http://localhost:8080/users  -H "Content-Type: application/json"  -d '{"username": "rust_ace"}'
```

## Related Posts
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
