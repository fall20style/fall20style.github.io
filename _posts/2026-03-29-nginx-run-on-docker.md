---
title: Docker Nginx 실습
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

## Docker Nginx 실습

```
docker pull nginx
```
### 도커 허브에서 최신 Nginx 이미지를 로컬로 내려받음.

```
docker run --name my-nginx -d -p 8080:80 nginx
```
* my-nginx라는 이름의 컨테이너를 백그라운드(-d)로 실행함.
   * 호스트 8080 포트를 컨테이너 80 포트에 연결함.

### 컨테이너 상태 제어

```
docker stop my-nginx
```
* 실행 중인 my-nginx 컨테이너를 정지시킴.

```
docker start my-nginx
```
* 정지된 my-nginx 컨테이너를 다시 시작함.

### 볼륨 마운트 시도 및 충돌 관리

```
docker run --name my-nginx2 -d -p 8080:80 -v /home/USERNAME/work/docker_nginx:/usr/share/nginx/html nginx
```
* 로컬 폴더를 컨테이너 웹 경로에 연결(마운트)하여 실행 시도함.
   * 기존 컨테이너와 포트(8080)가 중복될 경우 에러 발생 가능성 있음.

```
docker stop my-nginx2
```
* 생성된 my-nginx2 컨테이너를 정지시킴.

### 컨테이너 정리 및 최종 실행

```
docker rm -f my-nginx
```
* 첫 번째 연습용 컨테이너를 강제 삭제하여 포트 점유를 해제함.

```
docker rm -f my-nginx2
```
* 설정이 중복되거나 잘못된 두 번째 컨테이너를 삭제함.

```
docker run --name my-nginx2 -d -p 8080:80 -v /home/USERNAME/work/docker_nginx:/usr/share/nginx/html nginx
```
* 최종 단계임. 포트 충돌 없이 내 로컬 폴더의 HTML 파일을 서비스하도록 재실행함.

------------------------------
브라우저에서 http://localhost:8080 접속 시 지정한 경로의 파일이 정상 노출됨.


### 실시간 로그 확인 (docker logs)
컨테이너에서 발생하는 표준 출력(로그)을 실시간으로 모니터링할 때 사용함.

```
docker logs -f my-nginx2
```
* my-nginx2 컨테이너의 로그를 실시간으로 출력함.
   * -f (follow) 옵션을 사용하여 새로운 로그가 쌓일 때마다 계속 보여줌.
   * 브라우저에서 접속 시 발생하는 HTTP 요청 로그를 확인할 수 있음.
   * Ctrl + C를 눌러서 로그 확인 모드에서 빠져나옴.

### 컨테이너 내부 접속 (docker exec)
실행 중인 컨테이너 안으로 들어가서 리눅스 명령어를 직접 입력할 때 사용함.

```
docker exec -it my-nginx2 /bin/bash
```
* my-nginx2 컨테이너 내부에 접속하여 터미널(bash)을 실행함.
   * -it (interactive + tty) 옵션으로 컨테이너 내부와 상호작용 가능한 터미널 환경을 만듦.
   * 접속 후 cd /usr/share/nginx/html 등으로 이동해 내부 파일을 직접 확인 가능함.
   * exit를 입력하면 컨테이너에서 빠져나오지만, 컨테이너는 종료되지 않고 계속 실행됨.

## Related Posts
- [Podman Nginx 실습]({% link _posts/2026-03-29-podman-nginx-ubuntu20.04.md %})
- [Podman 설치 (Ubuntu 20.04)]({% link _posts/2026-03-29-podman-on-ubuntu2204.md %})
- [docker의 container image 미사용 한번에 지우기]({% link _posts/2026-04-08-docker-remove-img-container-at-once.md %})
- [Podman 기본 명령어]({% link _posts/2026-04-08-podman-command-list.md %})
- [1개를 제외한 모든 docker image 삭제하기]({% link _posts/2026-04-18-remove-all-docker-img-except-one.md %})
