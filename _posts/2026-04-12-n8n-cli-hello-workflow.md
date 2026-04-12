---
title: n8n CLI를 설치하고, workflow 시험하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - n8n
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- 리눅스 환경에서 Docker를 이용해 n8n을 설치
- CLI를 통해 첫 자동화 워크플로우(helloworld.txt 기록)를 성공시키기

------------------------------
## 1. 환경 구축 및 권한 설정
n8n은 내부적으로 node(UID 1000) 계정을 사용하므로 호스트 폴더 권한 설정이 필수임.

* 작업 디렉토리 생성: `mkdir -p ~/.n8n`
* 권한 부여: `sudo chown -R 1000:1000 ~/.n8n` 및 `sudo chmod -R 777 ~/.n8n` 실행함.
* 이유: 권한이 맞지 않으면 EACCES: permission denied 에러가 발생하며 설정을 저장하지 못함.

------------------------------
## 2. n8n 서버 실행 (Docker)
컨테이너를 띄워 웹 UI 접속 및 DB 파일 생성을 확인함.

* 실행 명령어:

```
docker run -d --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

* 상태 확인: ls -la ~/.n8n을 통해 database.sqlite 생성을 확인함.

------------------------------
## 3. 워크플로우 JSON 작성 (test-now.json)
CLI에서 즉시 실행 가능한 수동 트리거(Manual Trigger) 방식의 JSON을 작성함.

* 핵심 노드: Manual Trigger → Execute Command
* 실행 명령: `echo "CLI Test: $(date)" >> /home/node/.n8n/helloworld.txt`
* 중요: 최신 버전 n8n 정책에 따라 JSON 상단에 "id": 1을 명시해야 DB 임포트 시 SQLITE_CONSTRAINT 에러를 피할 수 있음.

------------------------------
## 4. CLI를 이용한 워크플로우 등록 및 실행
서버 중지 후 CLI 전용 컨테이너를 활용해 정공법으로 진행함.

* 서버 일시 중지: docker stop n8n (DB 잠금 및 포트 충돌 방지)
* 워크플로우 임포트(Import):

```
docker run --rm -v ~/.n8n:/home/node/.n8n -v $(pwd)/test-now.json:/tmp/test-now.json n8nio/n8n import:workflow --input=/tmp/test-now.json
```

* 즉시 실행(Execute):

```
docker run --rm -v ~/.n8n:/home/node/.n8n n8nio/n8n execute --id 1
```

------------------------------
## 5. 트러블슈팅 요약

* SQLITE 에러: JSON에 id값이 없어서 발생함. 수동으로 id: 1을 부여해 해결함.
* --file 옵션 미지원: 최신 CLI는 파일을 직접 실행하지 못함. 반드시 import 후 id로 실행해야 함.
* 파일 미생성: 실행은 성공했으나 파일이 안 보인다면 /home/node/.n8n 경로 오타나 폴더 권한(777)을 다시 체크해야 함.

------------------------------
## 6. 최종 결과 확인
호스트 리눅스에서 텍스트 파일에 기록된 내용을 확인함.

* 확인 명령어: `cat ~/.n8n/helloworld.txt`
* 결과: `"CLI Test: 2026. 04. 12. ..."` 형태의 로그가 찍힘.

결론
- Docker 볼륨과 n8n CLI의 작동 원리를 이해함.

