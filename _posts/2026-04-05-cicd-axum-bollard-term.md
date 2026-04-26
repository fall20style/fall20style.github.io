---
title: CI/CD 파이프라인 상세 설계 (Sub-directory 기반)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Docker
  - CI/CD
  - github workflow
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 1. 개요 및 목적

* 07_axum_bollard_term 폴더 내 Rust 소스 자동 빌드 및 배포 시스템임.
* Docker 이미지 생성 후 GitHub Container Registry(GHCR)에 업로드하는 것이 최종 목표임.

### 2. GitHub Actions 워크플로우 코드 (ci-cd.yml)
프로젝트 루트의 .github/workflows/ci-cd.yml에 작성함.

- https://github.com/fall20style/rust_prep/actions/workflows/CICD-07-axum-bollard-term.yml

### 3. 핵심 설정 항목 설명

* Selective Trigger (paths): 전체 레포지토리가 아닌 07_axum_bollard_term 폴더 내 파일이 바뀔 때만 빌드가 시작됨. 불필요한 서버 자원 낭비 방지함.
* Working Directory 설정: defaults.run을 통해 cargo build 등의 명령어가 서브 디렉터리 내부에서 실행되도록 강제함. 경로 에러 원천 차단함.
* Rust Cache Strategy: Cargo.lock 파일의 해시값을 키로 사용함. 라이브러리 구성이 바뀌지 않으면 이전 빌드 결과물을 재사용하여 속도가 매우 빠름.
* Docker Build Context: 도커 빌드 시 파일 복사(COPY)의 기준점을 서브 디렉터리로 고정함. Dockerfile 내의 경로 작성 편의성 제공함.

### 4. 최종 산출물 활용

* 저장소: GitHub 레포지토리의 Packages 탭에 저장됨.
* 실행: 서버에서 아래 명령으로 즉시 가동 가능함. 호스트 도커 소켓 공유 설정이 핵심임.

```
docker run -d -p 3000:3000 -v /var/run/docker.sock:/var/run/docker.sock <이미지_주소>
```

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-Docker docker 명령어 정리와 run.sh]({% link _posts/2026-04-05-axum-xterm-with-docker-cmd-list.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
