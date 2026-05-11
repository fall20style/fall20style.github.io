---
title: 거대 코드베이스 분석을 위한 Graphify 활용법
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - graphify
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: 거대 코드베이스 분석을 위한 Graphify 활용법
---

## 1. 개요 및 장점

* 구조화: 수만 줄의 코드를 AI용 지식 그래프로 변환함.
* 효율성: 토큰 사용량을 최대 70배 이상 절감하여 비용을 아낌.
* 정확성: 코드와 문서를 연결해 AI의 환각(Hallucination)을 방지함.


## 2. 설치 및 준비

* `pip install graphifyy` 명령어로 라이브러리를 설치함.
* `graphify install`을 통해 AI 어시스턴트에 기능을 연동함.
* 프로젝트 루트 폴더에서 작업을 시작함.


## 3. 코드베이스 스캔

* `/graphify .` 명령어를 입력하여 전체 프로젝트를 분석함.
* 코드: AST 기반으로 함수, 클래스 간 의존 관계를 파악함.
* 멀티모달: 관련 PDF, 이미지, 영상 데이터까지 통합함.
* 결과: graphify-out/ 폴더에 시각화 파일과 리포트를 생성함.


## 4. 실전 분석 활용

* 영향도 파악: 특정 모듈 수정 시 연쇄적으로 영향을 받는 곳을 추적함.
* 설계 이해: 코드와 설계 문서를 매칭하여 개발 의도를 질문함.
* 시각화: graph.html을 브라우저로 열어 관계망을 직접 확인함.


## 5. 유지 관리

* `Git Hook`을 설정하여 코드가 바뀔 때마다 그래프를 자동 갱신함.
* 생성된 리포트의 추천 질문 기능을 활용해 분석 효율을 높임.



## Git Hook을 설정하여 코드가 바뀔 때마다 지식 그래프를 자동 갱신


### 1. 자동 설정 명령 실행

* 터미널에서 분석 중인 프로젝트 폴더로 이동함.
* graphify hook install 명령어를 입력함.
* 이 명령은 .git/hooks/post-commit 파일을 자동으로 생성하거나 기존 파일에 실행 스크립트를 추가함.

### 2. 작동 원리

* 트리거: git commit이 성공적으로 완료되는 즉시 실행됨.
* 동작: 변경된 파일 내용을 기반으로 기존 지식 그래프(graph.json)를 부분 업데이트함.
* 효과: AI 어시스턴트가 항상 최신 상태의 코드 구조를 바탕으로 답변할 수 있게 함.


### 3. 수동 설정 방법 (필요시)
명령어가 작동하지 않거나 커스텀이 필요한 경우 직접 작성함.

* .git/hooks/post-commit 파일을 열거나 생성함.
* 아래 내용을 파일 끝에 추가함:

```
#!/bin/sh
graphify . --update-only
```

* `chmod +x .git/hooks/post-commit` 명령으로 실행 권한을 부여함.


### 4. 주의 사항

* 커밋 속도: 코드베이스가 너무 크면 커밋 후 그래프 갱신에 약간의 시간이 소요될 수 있음.
* 성능 최적화: --update-only 옵션을 사용하면 전체를 다시 스캔하지 않고 변경분만 반영하여 속도를 높임.
* 환경 변수: API 키 등 필요한 환경 변수가 쉘 설정(~/.zshrc 등)에 등록되어 있어야 오류 없이 작동함.

