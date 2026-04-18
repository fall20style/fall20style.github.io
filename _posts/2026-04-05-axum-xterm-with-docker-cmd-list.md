---
title: Axum-Docker docker 명령어 정리와 run.sh
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

### 1. 대상 컨테이너(접속용) 관리
웹 터미널이 접속하여 bash를 실행할 대상 컨테이너 관련 명령임.

* 컨테이너 생성 및 실행:

```
docker run -d --name my_terminal ubuntu sleep infinity
```

* `-d`: 백그라운드 실행.
   * sleep infinity: 컨테이너가 바로 종료되지 않고 계속 떠 있게 함.
* 컨테이너 상태 확인:

```
docker ps -f "name=my_terminal"
```

* 컨테이너 삭제:

```
docker rm -f my_terminal
```

### 2. 웹 터미널 서버(Axum) 실행 및 관리
작성한 Rust 코드가 담긴 서버 컨테이너 관련 명령임.

* 최신 이미지 다운로드 (Pull):

```
docker pull ghcr.io/fall20style/rust_prep:latest
```

* 서버 실행 (Run):

```
docker run -d \
  -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name my-web-terminal \
  ghcr.io/fall20style/rust_prep:latest
```

* `-p 3000:3000`: 호스트의 3000번 포트로 접속 허용.
   * -v /var/run/docker.sock...: 가장 중요. 호스트 도커 제어 권한 부여.
* 서버 로그 확인 (디버깅):

```
docker logs -f my-web-terminal
```

* 서버 강제 종료 및 삭제:

```
docker rm -f my-web-terminal
```

### 3. 이미지 및 시스템 환경 설정
빌드 결과물 확인 및 권한 설정 관련 명령임.

* 도커 소켓 권한 개방 (필수):

```
sudo chmod 666 /var/run/docker.sock
```

* 서버 컨테이너가 호스트 도커 엔진에 접근하기 위해 반드시 필요함.
* 빌드된 이미지 내부 파일 확인:

```
docker run --rm ghcr.io/fall20style/rust_prep:latest ls -R /usr/local/bin/
```

* 이미지 라이브러리 의존성 체크:

```
docker run --rm ghcr.io/fall20style/rust_prep:latest ldd /usr/local/bin/my-docker-term
```

* GitHub 컨테이너 레지스트리 로그인:

```
echo $TOKEN | docker login ghcr.io -u <ID> --password-stdin
```

### 4. 기타 유용한 확인 명령

* 실행 중인 모든 컨테이너 목록: `docker ps`
* 종료된 컨테이너 포함 전체 목록: `docker ps -a`
* 도커 이미지 목록: `docker images`


### run.sh - 자동화 script 

``` bash
#!/bin/bash

# 1. 기존 컨테이너 깔끔하게 정리함
echo "기존 컨테이너 정리 중..."
docker rm -f my_terminal my-web-terminal 2>/dev/null

# 2. 도커 소켓 권한 부여함 (비밀번호 입력 필요할 수 있음)
echo "도커 소켓 권한 설정 중..."
sudo chmod 666 /var/run/docker.sock

# 3. 대상 컨테이너(Ubuntu) 실행함
echo "대상 컨테이너(my_terminal) 실행 중..."
docker run -d --name my_terminal ubuntu sleep infinity

# 4. 최신 서버 이미지 가져옴
echo "최신 서버 이미지 다운로드 중..."
docker pull ghcr.io/fall20style/rust_prep:latest

# 5. 웹 터미널 서버 실행함
echo "웹 터미널 서버(my-web-terminal) 가동 중..."
docker run -d \
  -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name my-web-terminal \
  ghcr.io/fall20style/rust_prep:latest

# 6. 결과 확인
echo "모든 준비 완료!"
echo "접속 주소: http://localhost:3000"
docker ps
```

## Related Posts
- [axum으로 GET/POST 예제]({% link _posts/2026-04-03-axum_get_post.md %})
- [package axum-core v0.5.6 cannot be built]({% link _posts/2026-04-03-cargo-run-error.md %})
- [static web page server]({% link _posts/2026-04-03-rust-static-web-server.md %})
- [Rust Axum으로 Unit Test하고 실행]({% link _posts/2026-04-04-rust-calc-rest-api-unittest.md %})
- [Axum과 WebSockets를 이용한 터미널 에뮬레이션]({% link _posts/2026-04-04-simple-terminal-emulation-axum.md %})
- [Axum로 터미널 에뮬레이션 - Docker 연결]({% link _posts/2026-04-04-terminal-emul-axum-docker.md %})
- [Axum-Docker 웹 터미널 최종 가이드]({% link _posts/2026-04-05-axum-xterm-with-docker-cicd-debug.md %})
- [Axum-xterm.js 기반 Docker Web Terminal]({% link _posts/2026-04-05-axum-xterm-with-docker.md %})
- [CI/CD 파이프라인 상세 설계 (Sub-directory 기반)]({% link _posts/2026-04-05-cicd-axum-bollard-term.md %})
- [GitHub Actions 로컬 실행(act) 및 Rust 프로젝트 CI/CD 정리]({% link _posts/2026-04-08-axum-xterm-with-docker-cicd-local.md %})
