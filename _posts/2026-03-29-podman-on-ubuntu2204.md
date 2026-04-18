---
title: Podman 설치 (Ubuntu 20.04)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - 컨테이너기술
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## Ubuntu 20.04 Podman 설치하기

### Kubic 프로젝트 저장소 추가

```
echo "deb https://download.opensuse.org{VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
```

### GPG 보안 키 등록
패키지 변조 방지를 위해 인증 키를 추가함.
```
curl -L "https://download.opensuse.org{VERSION_ID}/Release.key" | sudo apt-key add -
```

### 패키지 목록 업데이트 및 설치
저장소 정보를 갱신하고 Podman 설치.

```
sudo apt-get update
sudo apt-get -y install podman
```

### 설치 확인
버전 정보를 출력하여 정상 설치 여부를 확인함.

```
podman --version
```

## Related Posts
- [Docker Nginx 실습]({% link _posts/2026-03-29-nginx-run-on-docker.md %})
- [Podman Nginx 실습]({% link _posts/2026-03-29-podman-nginx-ubuntu20.04.md %})
- [docker의 container image 미사용 한번에 지우기]({% link _posts/2026-04-08-docker-remove-img-container-at-once.md %})
- [Podman 기본 명령어]({% link _posts/2026-04-08-podman-command-list.md %})
- [1개를 제외한 모든 docker image 삭제하기]({% link _posts/2026-04-18-remove-all-docker-img-except-one.md %})
