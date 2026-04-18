---
title: Axum로 터미널 에뮬레이션 - Docker 연결
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
  - pty
  - xterm.js
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 작업 내용

- axum으로 만든 server에서 xterm.js로 UI를 구성하고, docker에서 띄우는 bash를 연결함
- root 계정으로 사용됨, non-root account로 변경할 필요가 있음
- white list를 일단 끔


### 테스트

<img width="738" height="472" alt="Image" src="https://github.com/user-attachments/assets/0b4ed145-16be-43b6-ae55-d2048728cc54" />

<img width="740" height="475" alt="Image" src="https://github.com/user-attachments/assets/30da51d0-c704-4935-a655-ab89a07067b3" />

<img width="741" height="473" alt="Image" src="https://github.com/user-attachments/assets/66d1a335-1c12-4bb6-8cc2-529dc130301f" />

### 도커 이미지 준비

``` bash
~/work/docker_prep/ubuntu2204_gcc$ docker images
REPOSITORY                      TAG       IMAGE ID       CREATED          SIZE
my-gcc-ubuntu                   latest    a64a4b4eacb6   12 minutes ago   352MB
```

### cargo build & run
``` bash
axum_bash_cli$ cargo build
   Compiling axum-bash-cli v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.93s
axum_bash_cli$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/axum-bash-cli`
2026-04-04T14:32:54.336600Z  INFO axum_bash_cli: listening on 127.0.0.1:8080
```

### 작업 경로
- https://github.com/fall20style/rust_prep/tree/main/05_gemini_harness_axum_bash_cli
- https://github.com/fall20style/rust_prep/commit/c8dd22d1aad104f561f2e97946475a11dd1b96d5

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
