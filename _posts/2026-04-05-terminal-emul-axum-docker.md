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
