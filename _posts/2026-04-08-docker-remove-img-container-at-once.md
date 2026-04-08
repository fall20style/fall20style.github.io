---
title: docker의 container image 미사용 한번에 지우기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


- prune 명령어는 사용하지 않는 리소스를 한꺼번에 정리할 때 유용

## 1. Container 관련

* 중지된 모든 컨테이너 삭제

```
docker container prune
```

* 실행 중이지 않은(Exited) 컨테이너만 골라 삭제함.

## 2. Image 관련

* 댕글링(Dangling) 이미지 삭제

```
docker image prune
```

* 이름이나 태그가 <none>으로 표시되는 이미지들만 삭제함.
* 사용되지 않는 모든 이미지 삭제

```
docker image prune -a
```

* 컨테이너에 할당되지 않은 모든 이미지를 삭제하여 용량을 대폭 확보함.

## 3. 공통 팁 (참고용)

* 확인 절차 생략 (강제 실행)

```
docker [대상] prune -f
```

* 삭제 전 묻는 "y/n" 단계를 건너뛰고 바로 실행함.
* 시스템 전체 통합 정리

```
docker system prune
```

* 컨테이너, 이미지, 네트워크를 한 번에 정리함. (--volumes 추가 시 볼륨도 포함)


## 실습

### docker 컨테이너 미사용 일괄 삭제
```
~/work/rust_prep$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                   CREATED      STATUS                     PORTS                                         NAMES
8d289187f9cc   35d1cc154555   "/usr/local/bin/my-d…"   2 days ago   Exited (255) 2 hours ago   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   my-web-terminal
0a2e5c14ee31   ubuntu         "sleep infinity"          2 days ago   Exited (255) 2 hours ago                                                 my_terminal
85ce72505741   a64a4b4eacb6   "/bin/bash"               3 days ago   Exited (129) 3 days ago                                                  jolly_payne
074954f1befe   a64a4b4eacb6   "/bin/bash"               3 days ago   Exited (0) 3 days ago                                                    determined_dhawan

~/work/rust_prep$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
8d289187f9cccb69198ea3cab267c44931f47d0846a8dc099468686598009075
0a2e5c14ee319b2b16ac524a99d011bd779b337b326e0b6cb5c5afb4d076267d
85ce72505741b3e30e9fec51fd1e9e45450fc87e5bf863c6e6e1b48feece5bed
074954f1befe9fad43cf7afa2ed49a7ddeda63071a0783047ef6c55823f36b19

Total reclaimed space: 50B
~/work/rust_prep$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### docker image 미사용 일괄 삭제
```
~/work/rust_prep$ docker images
REPOSITORY                      TAG          IMAGE ID       CREATED          SIZE
ghcr.io/fall20style/rust_prep   latest       06a201ced3fd   44 minutes ago   109MB
ghcr.io/fall20style/rust_prep   <none>       35d1cc154555   2 days ago       103MB
ghcr.io/fall20style/rust_prep   <none>       46b5bf97a7d1   2 days ago       96.8MB
ghcr.io/fall20style/rust_prep   <none>       50d82d3c1e85   2 days ago       96.8MB
catthehacker/ubuntu             act-latest   afa00835d9fb   6 days ago       1.64GB
ubuntu                          latest       f794f40ddfff   6 weeks ago      78.1MB

~/work/rust_prep$ docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Deleted Images:
...
Total reclaimed space: 1.71GB
~/work/rust_prep$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

