---
title: package axum-core v0.5.6 cannot be built
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

cargo run error - `package `axum-core v0.5.6` cannot be built`

### 증상
```
axum_example$ cargo run 

error: package `axum-core v0.5.6` cannot be built because it requires rustc 1.78 or newer, while the cu
rrently active rustc version is 1.75.0
Either upgrade to rustc 1.78 or newer, or use
cargo update axum-core@0.5.6 --precise ver
where `ver` is the latest version of `axum-core` supporting rustc 1.75.0
...
```


### 원인
현재 설치된 Rust 컴파일러(rustc) 버전이 1.75.0, 사용하려는 axum-core v0.5.6 패키지는 최소 1.78 이상의 버전을 요구하기 때문에 발생


### 해결방법
```
axum_example$ rustc --version                   
rustc 1.75.0 (82e1608df 2023-12-21) (built from a source tarball)  
...
axum_example$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh   
...
axum_example$ rustup update stable
rustup: command not found

axum_example$ source $HOME/.cargo/env
axum_example$ rustup update stable
info: syncing channel updates for stable-x86_64-unknown-linux-gnu

  stable-x86_64-unknown-linux-gnu unchanged - rustc 1.94.1 (e408947bf 2026-03-25)

info: checking for self-update (current version: 1.29.0)
axum_example$ rustc --version
rustc 1.94.1 (e408947bf 2026-03-25)
```

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
