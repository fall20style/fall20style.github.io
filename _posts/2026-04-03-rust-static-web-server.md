---
title: static web page server
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

### 구조
```
./Cargo.toml
./src
./src/main.rs
./static
./static/index.html
```

### src/main.rs

```rust
use axum::Router; // Note: check your import, usually 'axum'
use tower_http::services::ServeDir;
use std::net::SocketAddr;
use tokio::net::TcpListener;

#[tokio::main]
async fn main() {
    // FIX: Use fallback_service instead of nest_service for the root "/"
    let app = Router::new().fallback_service(ServeDir::new("./static"));

    let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
    let listener = TcpListener::bind(&addr).await.expect("Failed to bind address");
    
    println!("Listening on http://{}", addr);

    axum::serve(listener, app).await.expect("Server failed");
}
```

### 만들기 순서
```
$ cd axum_websvr
$ cd src                                                                                      
$ gedit main.rs                                                                               
$ mkdir static
$ vi index.html
$ cd ..

$ cargo add tower_http --features "fs"
$ cargo add tokio --features "full"
$ cargo build
$ cargo run
```

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
