---
title: Podman Nginx 실습
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

## Podman 실행 히스토리 분석
사용자의 터미널 히스토리 중 Podman 명령어 흐름과 의미를 정리함.

### 명령어 실행 흐름 요약

| 순번 | 실행 명령어 | 주요 내용 및 분석 |
|---|---|---|
| 1 | podman run -d -p 8080:80 --name my-web nginx | 최초 실행: nginx 컨테이너를 my-web 이름으로 백그라운드 구동함. |
| 2 | podman run -d -p 8080:80 --name my-web nginx | 중복 실행 시도: 동일 이름(my-web)이 이미 존재해 에러 발생했을 것으로 보임. |
| 3 | podman run -d -p 8080:80 --name my-web nginx | 재시도: 기존 컨테이너가 삭제되지 않아 여전히 이름 충돌 발생함. |
| 4 | podman run -d -p 8080:80 --name my-web2 nginx | 추가 생성: 이름을 my-web2로 변경해 새로운 컨테이너 실행 성공함. |

------------------------------

### 명령어 옵션 상세 분석

```
명령어: podman run -d -p 8080:80 --name my-web nginx
```

* run: 이미지를 기반으로 새 컨테이너를 생성 및 실행함.
* -d (detach): 컨테이너를 백그라운드 모드로 실행함.
* -p 8080:80: 호스트 8080 포트를 컨테이너 내부 80 포트로 연결함.
* --name my-web: 컨테이너에 고유 별칭을 부여함.
* nginx: 사용할 베이스 이미지 이름임.

------------------------------
### 특이사항

   1. 관리 주체 분리: docker rm으로는 Podman 컨테이너를 지울 수 없음. 반드시 podman rm을 사용해야 함.
   2. 이름 중복 주의: 동일 이름으로 다시 실행하려면 기존 컨테이너를 먼저 삭제(podman rm -f my-web)해야 함.
   3. 포트 충돌: my-web2 실행 시 8080 포트가 점유 중이면 -p 8081:80처럼 포트 번호를 바꿔야 함.

## Related Posts
- [Docker Nginx 실습]({% link _posts/2026-03-29-nginx-run-on-docker.md %})
- [Podman 설치 (Ubuntu 20.04)]({% link _posts/2026-03-29-podman-on-ubuntu2204.md %})
- [docker의 container image 미사용 한번에 지우기]({% link _posts/2026-04-08-docker-remove-img-container-at-once.md %})
- [Podman 기본 명령어]({% link _posts/2026-04-08-podman-command-list.md %})
- [1개를 제외한 모든 docker image 삭제하기]({% link _posts/2026-04-18-remove-all-docker-img-except-one.md %})


