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
