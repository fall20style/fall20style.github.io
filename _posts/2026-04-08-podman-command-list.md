---
title: Podman 기본 명령어
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Podman
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- Podman의 기본적인 이미지 내려받기, 컨테이너 생성 및 사용

## 1. 이미지 내려받기 (Pull)
저장소(Registry)에서 컨테이너 이미지를 로컬로 가져오는 단계임.

* Nginx 최신 이미지 내려받기

```
podman pull docker.io/library/nginx:latest
```

* 로컬 이미지 목록 확인

```
podman images
```

## 2. 컨테이너 생성 및 실행 (Run)
내려받은 이미지를 바탕으로 컨테이너를 만들고 실행함.

* -d: 백그라운드 실행
* --name: 컨테이너 이름 지정
* -p: 호스트와 컨테이너 포트 연결 (예: 호스트 8080 → 컨테이너 80)

# 'my-web'이라는 이름으로 Nginx 컨테이너 실행

```
podman run -d --name my-web -p 8080:80 docker.io/library/nginx:latest
```

# 실행 중인 컨테이너 확인

```
podman ps
```

## 3. 컨테이너 사용 및 조작 (Exec / Logs)
실행 중인 컨테이너 내부를 확인하거나 명령어를 전달함.

# 컨테이너 내부 쉘(bash) 접속

```
podman exec -it my-web /bin/bash
```

# 컨테이너 로그 확인

```
podman logs my-web
```

## 4. 컨테이너 중지 및 삭제 (Stop / Rm)
사용이 끝난 컨테이너를 멈추고 시스템에서 제거함.

# 컨테이너 중지

```
podman stop my-web
```

# 중지된 컨테이너 삭제 (완전히 제거)

```
podman rm my-web
```

# 모든 중지된 컨테이너 일괄 삭제

```
podman container prune
```

## Daemonless 구조
Podman은 데몬리스(Daemonless) 구조이므로 sudo 권한 없이도 명령어들을 그대로 사용


## 컨테이너 강제 삭제 (가장 빠른 방법)
실행 중인지 확인하고 중지한 뒤 삭제하는 과정을 한 번에 수행함.

```
podman rm -f my-web
```

## 단계별 삭제 (상태 확인 후 삭제)
실행 중인 상태를 보면서 안전하게 지우고 싶을 때 사용함.

* 1. 실행 중인 컨테이너 확인 (목록에 있는지 확인)

```
podman ps -a | grep my-web
```

* 2. 컨테이너 중지

```
podman stop my-web
```

* 3. 컨테이너 삭제

```
podman rm my-web
```

## 일일이 지우기 귀찮을 때 rm 옵션
- 만약 테스트용으로 쓰고 버릴 컨테이너라면, 다음부터 run 할 때 --rm 옵션을 붙여주면 좋음.
- 컨테이너를 중지하자마자 자동으로 삭제됨.

```
podman run -d --name my-web --rm -p 8080:80 docker.io/library/nginx:latest
```

## Related Posts
- [Docker Nginx 실습]({% link _posts/2026-03-29-nginx-run-on-docker.md %})
- [Podman Nginx 실습]({% link _posts/2026-03-29-podman-nginx-ubuntu20.04.md %})
- [Podman 설치 (Ubuntu 20.04)]({% link _posts/2026-03-29-podman-on-ubuntu2204.md %})
- [docker의 container image 미사용 한번에 지우기]({% link _posts/2026-04-08-docker-remove-img-container-at-once.md %})
- [1개를 제외한 모든 docker image 삭제하기]({% link _posts/2026-04-18-remove-all-docker-img-except-one.md %})

