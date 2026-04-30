---
title: "Git이 특정 폴더를 서브모듈로 인식하는 이유와 해결 방법"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - git
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "Git이 특정 폴더를 서브모듈로 인식하는 이유와 해결 방법"
---

## 서브모듈로 인식되는 이유

* .git 폴더의 존재: `gdb_py_pipeline/project` 폴더 내부에 별도의 .git 폴더가 들어있기 때문임.
* 외부 저장소 복제: 외부 프로젝트를 git clone으로 가져와 메인 저장소 안에 넣었을 때 발생함.
* 개별 초기화: 해당 폴더에서 실수로 git init을 실행하여 독립적인 저장소로 설정된 경우임.
* 상위 Git의 판단: 상위 폴더의 Git은 내부에 .git을 가진 폴더를 만나면 이를 별개의 프로젝트(서브모듈)로 판단하고 추적을 중단함.


## 해결 방법

### 1. 하나의 저장소로 합치기 (권장)
복잡한 서브모듈 구조 대신 모든 파일을 메인 저장소에서 관리하고 싶을 때 사용함.

* 서브모듈 캐시 제거: `git rm --cached gdb_py_pipeline/project` 실행 (폴더 끝에 /를 붙이지 않도록 주의함).
* 내부 Git 폴더 삭제: `rm -rf gdb_py_pipeline/project/.git` 명령으로 독립 저장소 정보를 지움.
* 파일 다시 추가: `git add gdb_py_pipeline/project/` 실행 후 커밋함.

### 2. 정식 서브모듈로 등록하기
해당 폴더를 외부 저장소와 연결된 상태로 유지하고 싶을 때 사용함.

* 서브모듈 설정: .gitmodules 파일에 해당 경로를 정식 등록함.
* 명령어 사용: `git submodule add <저장소 주소> gdb_py_pipeline/project`를 실행함.


## 요약

* 현재 발생한 `fatal: Pathspec... is in submodule` 에러는 대부분 1번 방법(캐시 삭제 후 .git 제거)으로 해결됨.
* 작업 전 `ls -a gdb_py_pipeline/project`를 입력하여 .git 폴더가 있는지 먼저 확인하는 것이 좋음.

