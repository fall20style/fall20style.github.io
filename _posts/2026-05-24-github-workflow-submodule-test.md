---
title: "GitHub Actions - 서브모듈(Submodule)을 불러오지 못하는 문제"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - "github submodule"
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "GitHub Actions에서 서브모듈(Submodule)을 불러오지 못하는 문제 해결"
---




## 

## 문제 상황

* GitHub Actions 워크플로우 실행 시, 메인 저장소 코드는 정상적으로 체크아웃됨.
* 하지만 프로젝트 내부의 서브모듈 폴더가 비어 있어 빌드나 테스트 과정에서 File Not Found 또는 라이브러리 참조 오류가 발생함.
* 기본 actions/checkout@v4 설정만 사용하면 서브모듈 내용을 자동으로 가져오지 않기 때문임.

------------------------------
##  해결 방법: submodules: recursive 옵션 추가
checkout 스텝에 submodules: recursive 설정을 추가하여 하위 저장소까지 모두 내려받도록 수정함.

jobs:
  check-submodules:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          # 서브모듈을 재귀적으로(recursive) 모두 가져오도록 설정
          submodules: recursive

## 📋 옵션 상세 설명

* false (기본값): 서브모듈을 무시함.
* true: 메인 저장소에 직속된 서브모듈만 가져옴.
* recursive: 서브모듈 안에 또 다른 서브모듈이 있는 중첩 구조까지 끝까지 추적해서 가져옴. 프로젝트 규모가 크다면 이 옵션이 가장 안전함.

------------------------------
## ⚠️ 주의사항: Private 서브모듈 권한 문제
서브모듈이 별도의 Private 저장소인 경우, 기본 GITHUB_TOKEN으로는 접근 권한 오류(Authentication failed)가 발생할 수 있음. 이럴 때는 별도의 PAT(Personal Access Token)를 생성하여 주입해야 함.

      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive
          token: ${{ secrets.MY_GITHUB_PAT }} # 서브모듈 접근 권한이 있는 토큰

------------------------------
## ✅ 요약

* GitHub Actions는 기본적으로 서브모듈을 가져오지 않음.
* submodules: recursive를 명시해야 하위 프로젝트 코드까지 정상적으로 로드됨.
* 로컬에서 git submodule update --init --recursive를 실행하는 것과 동일한 효과를 냄.

------------------------------
이 내용이 도움이 되었다면?

* Private 저장소 권한 설정 방법이 더 궁금하신가요?
* 서브모듈 특정 브랜치만 가져오는 법을 알고 싶으신가요?
* LFS(Large File Storage) 설정도 함께 적용해야 하나요?

추가로 필요한 설정이 있다면 말씀해 주시기 바람.

