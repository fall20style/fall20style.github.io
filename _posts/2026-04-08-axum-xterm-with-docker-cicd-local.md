---
title: GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - act
  - docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 1. 오픈소스 act 소개 및 장점

* 정의: GitHub Actions 워크플로우를 로컬 Docker 환경에서 재현하는 도구임.
* 장점:
* git push 없이 즉각적인 피드백 가능함.
   * GitHub Actions 무료 빌드 시간을 절약함.
   * 로컬에서 시크릿 변수(-s) 주입 및 디버깅이 용이함.

## 2. 설치 및 기본 사용법

* 설치: 운영체제별 패키지 매니저 사용함 (brew, choco, scoop 등)
* 사전 요구사항: Docker가 반드시 설치되어 실행 중이어야 함.
* 주요 명령어:
* act: push 이벤트 워크플로우 실행함.
   * act -j [job_id]: 특정 작업만 골라서 실행함.
   * act -l: 실행 가능한 워크플로우 목록 확인함.

## 3. Rust 프로젝트 적용 시 이슈 해결

* Command not found: 기본 Docker 이미지에 Cargo가 없어서 발생함. 워크플로우에 dtolnay/rust-toolchain 단계를 추가하여 해결함.
* Password required: 로컬 act 환경에 GitHub Secrets가 없어서 발생함. .secrets 파일을 만들거나 -s GITHUB_TOKEN=값으로 주입해야 함.
* Push 동작: act 성공 시 실제로 설정된 레지스트리(GHCR 등)로 이미지가 업로드됨. 테스트 시에는 push: false 조건이나 --dryrun 모드 권장함.

## 4. GitHub Container Registry (GHCR) 접근 오류

* 현상: denied: denied 에러 발생하며 docker pull 실패함.
* 원인: 로그인 세션 만료, PAT 권한 부족, 또는 패키지가 Private 설정인 경우임.
* 해결:
* docker logout ghcr.io 후 PAT 토큰으로 재로그인함.
   * GitHub 패키지 설정에서 Visibility를 Public으로 변경함.
   * 사용 중인 토큰에 read:packages 권한이 있는지 확인함.

## 실습

### act 설치하기
```
~/work/act_test$ curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
nektos/act info checking GitHub for latest tag
nektos/act info found version: 0.2.87 for v0.2.87/Linux/x86_64
nektos/act info installed ./bin/act
```

### act 버전 확인
```
~/work/act_test$ bin/act --version
act version 0.2.87
```

### 대상 소스코드
```
~/work/rust_prep$ vi .github/workflows/CICD-07-axum-bollard-term.yml 
~/work/rust_prep$ 
```

### secret을 지정해서 act 호출
```
~/work/rust_prep$ act --secret-file .secrets
```
- .secrets에 github token id를 저장

### cargo 설치
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

```
. "$HOME/.cargo/env"
```
- 환경변수 설정하기

### act 빌드가 되면 내려받을 수 있음 (push on 속성)
```
~/work/rust_prep$ docker login ghcr.io -u fall20style
Password: 

~/work/rust_prep$ docker pull ghcr.io/fall20style/rust_prep:latest
latest: Pulling from fall20style/rust_prep
Digest: sha256:9bce0eda108c5e7491e9426f134d8f9cfc9604d5ab1035747cf91dfd31ac6c86
Status: Image is up to date for ghcr.io/fall20style/rust_prep:latest
ghcr.io/fall20style/rust_prep:latest

```

