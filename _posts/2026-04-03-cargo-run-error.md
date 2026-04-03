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
