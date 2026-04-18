---
title: 1개를 제외한 모든 docker image 삭제하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Docker
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


### 이미지 상태 확인
``` bash
~/work/docker_prep/gemini-cli-docker$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
gemini-cli      latest    73625a3849a3   11 hours ago   376MB
<none>          <none>    21ad467bb55b   11 hours ago   376MB
<none>          <none>    238daa430cdc   11 hours ago   376MB
<none>          <none>    aa23e2eb9ee8   11 hours ago   376MB
<none>          <none>    d70d29e3df97   11 hours ago   376MB
<none>          <none>    70b347023366   11 hours ago   376MB
<none>          <none>    a95b84167377   11 hours ago   376MB
<none>          <none>    af547ac356a9   11 hours ago   376MB
<none>          <none>    efd07d0d0094   11 hours ago   357MB
bash            latest    5699da41bf65   2 days ago     15.6MB
my-python-27    latest    ba1b2f9f38c3   6 days ago     1.7GB
my-sandbox      latest    778cede88ae3   6 days ago     200MB
my-python-app   latest    31f316596795   6 days ago     1.56GB
n8nio/n8n       latest    3caff4df1693   7 days ago     1.21GB
node            20-slim   6d1685dbd5a2   11 days ago    200MB
```
- 많이 있음

### gemini-cli만 빼고, 나머지 이미지는 삭제
``` bash
~/work/docker_prep/gemini-cli-docker$ docker rmi $(docker images -q | grep -v 73625a3849a3)
Deleted: sha256:21ad467bb55b41ddcc604c992665980f84b4b4cc0965cf896b8acf4fd11e8966
...
...
...
Deleted: sha256:6d1685dbd5a2080706f76567afce903406efa4b2b556feaf9f7afb34a09541cd
```

### 결과 확인
``` bash
~/work/docker_prep/gemini-cli-docker$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
gemini-cli   latest    73625a3849a3   11 hours ago   376MB
```

