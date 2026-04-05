---
title: Axum-Docker 웹 터미널 최종 가이드
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

### 1. 프로젝트 구조 (최종)
모든 소스와 리소스는 07_axum_bollard_term 폴더 내에 위치함.

```
07_axum_bollard_term/
├── Cargo.toml
├── Dockerfile           # nightly 이미지 및 경로 수정됨
└── src/
    ├── main.rs          # 0.0.0.0 바인딩 수정됨
    ├── index.html       # xterm.js 로컬 호출 수정됨
    ├── xterm.js         # wget으로 다운로드 완료함
    └── xterm.css        # wget으로 다운로드 완료함
```

### 2. 주요 코드 수정 사항

### 2.1. Backend (src/main.rs)

* 네트워크 바인딩: 도커 내부에서 외부 접속을 허용하도록 127.0.0.1에서 0.0.0.0으로 변경함.

```
// [수정] 127.0.0.1 -> 0.0.0.0let addr = std::net::SocketAddr::from(([0, 0, 0, 0], 3000));
```

* 리소스 포함: include_str! 매크로를 사용하여 HTML/JS/CSS를 바이너리에 직접 내장함.

### 2.2. Frontend (src/index.html)

* 라이브러리 로드: CDN 타임아웃 문제를 해결하기 위해 로컬 경로(/xterm.js)로 호출함.
* 초기 프롬프트: 접속 즉시 \n(Enter)을 전송하여 bash 프롬프트가 즉시 뜨도록 보정함.

### 3. Docker 환경 최적화 (Dockerfile)

* Build Stage: rustlang/rust:nightly-bookworm을 사용하여 최신 Cargo.lock (v4) 및 Edition 2024 이슈 해결함.
* Run Stage: debian:bookworm-slim을 사용하여 실행 파일명을 my-docker-term으로 정확히 일치시켜 복사함.
* 권한 부여: chmod +x 및 절대 경로 ENTRYPOINT로 실행 안정성 확보함.

### 4. CI/CD 파이프라인 (ci-cd.yml)

* 서브 디렉터리 타겟팅: working-directory와 context를 07_axum_bollard_term으로 고정하여 빌드 오류 방지함.
* GitHub Packages (GHCR): 빌드 성공 시 자동으로 도커 이미지를 업로드하도록 구성함.

### 5. 최종 실행 프로세스 (중요)

서버를 띄울 때 아래 순서를 지켜야 함.

1. 대상 컨테이너 가동: `docker run -d --name my_terminal ubuntu sleep infinity`
2. 도커 소켓 권한: `sudo chmod 666 /var/run/docker.sock`
3. 이미지 갱신: `docker pull ghcr.io/fall20style/rust_prep:latest`
4. 서버 컨테이너 실행:

``` bash
   docker run -d -p 3000:3000 \
     -v /var/run/docker.sock:/var/run/docker.sock \
     --name my-web-terminal \
     ghcr.io/fall20style/rust_prep:latest
```

## Test 절차

### 1단계: 대상 컨테이너 실행 (Terminal용)
웹 터미널로 접속할 실제 우분투 환경을 먼저 만듦.

* 이미 존재한다면 지우고 새로 만드는 것이 깔끔함

``` bash
docker rm -f my_terminal 2>/dev/null
docker run -d --name my_terminal ubuntu sleep infinity
```

### 2단계: 도커 소켓 권한 개방 (중요)

Axum 서버 컨테이너가 호스트의 도커 엔진에 접근할 수 있게 권한을 열어줌.

``` bash
sudo chmod 666 /var/run/docker.sock
```

### 3단계: 최신 서버 이미지 다운로드 (Pull)

CI/CD로 빌드된 최신 이미지를 GitHub에서 가져옴.
``` bash
docker pull ghcr.io/fall20style/rust_prep:latest
```

## 4단계: 웹 터미널 서버 실행 (Run)

호스트 포트 3000번과 도커 소켓을 연결하여 서버를 띄움.

# 기존 서버 컨테이너가 있다면 삭제함

``` bash
docker rm -f my-web-terminal 2>/dev/null
```

# 서버 실행 (마지막에 이미지 주소 확인 필수)

``` bash
docker run -d \
  -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name my-web-terminal \
  ghcr.io/fall20style/rust_prep:latest
```

## 5단계: 상태 확인 및 접속
서버가 Up 상태인지 확인하고 브라우저로 접속함.

# 두 컨테이너가 모두 'Up'인지 확인함

``` bash
docker ps
```
# 서버 로그에 에러가 없는지 확인함

``` bash
docker logs my-web-terminal
```

## 6단계: 브라우저 확인
주소창에 입력함.

* `http://localhost:3000` 접속
* 검은 화면 클릭 후 Enter 키 입력하여 프롬프트 확인

## 요약

1. my_terminal (대상) 실행함.
2. chmod 666 (권한) 설정함.
3. my-web-terminal (서버) 실행함.
4. localhost:3000 접속함.


## Log 확인
```
$ sudo chmod 666 /var/run/docker.sock
```

```
$ docker pull ghcr.io/fall20style/rust_prep:latestlatest: Pulling from fall20style/rust_prep
77dab3318d44: Already exists
c166f9789bb8: Pull complete
051c2c6b6bdb: Pull complete
4f4fb700ef54: Pull complete
Digest: sha256:f5df18d651b8c06082a63d2e83307f7573164cb77fa8a32dfb2e6ec12e448fde
Status: Downloaded newer image for ghcr.io/fall20style/rust_prep:latest
ghcr.io/fall20style/rust_prep:latest
```

```
$ docker rm -f my-web-terminal 2>/dev/null
my-web-terminal
```


```
$ docker run -d   -p 3000:3000   -v /var/run/docker.sock:/var/run/docker.sock   --name my-web-terminal   ghcr.io/fall20style/rust_prep:latest
8d289187f9cccb69198ea3cab267c44931f47d0846a8dc099468686598009075
```

```
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                   CREATED         STATUS         PORTS                                         NAMES
8d289187f9cc   ghcr.io/fall20style/rust_prep:latest   "/usr/local/bin/my-d…"   5 seconds ago   Up 2 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   my-web-terminal
0a2e5c14ee31   ubuntu                                 "sleep infinity"          7 minutes ago   Up 7 minutes                                                 my_terminal
```

```
$ docker logs my-web-terminal
서버 오픈: http://localhost:3000
```


