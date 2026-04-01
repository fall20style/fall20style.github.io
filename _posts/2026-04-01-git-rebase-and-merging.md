---
title: git rebase와 merging의 차이
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
description: desc가 여기에
---

- Git Rebase는 브랜치 기준점(Base)을 다시 설정(Re)하는 기능.
- 내가 작업한 브랜치를 대상 브랜치의 최신 커밋 위로 옮겨서 히스토리를 한 줄로 정렬.

## 1. 주요 특징

* 히스토리 정리: 머지 커밋 없이 커밋 로그를 깔끔하게 한 줄로 만듦.
* 최신 상태 유지: 메인 브랜치의 최신 변경 사항을 내 작업 위로 가져와서 충돌 미리 해결 가능함.
* 커밋 편집: rebase -i로 지저분한 커밋들을 하나로 합치거나 메시지 수정 가능함.

## 2. Merge vs Rebase

* Merge: 브랜치 갈래가 그대로 남고 새로운 '머지 커밋'이 생김. 히스토리가 복잡해질 수 있음.
* Rebase: 기존 커밋들을 복사해서 최신 위치에 재배치함. 히스토리가 단순해지지만 커밋 해시가 변함.

## 3. 주의사항

* 공유 금지: 이미 서버에 Push된 커밋은 Rebase하면 안 됨. 동료들의 로컬 히스토리와 꼬여서 난리 날 수 있음.
* 충돌 처리: 커밋마다 충돌을 해결해야 할 수도 있어 양이 많으면 번거로움.

## 4. 명령어 순서

   1. git checkout feature (내 브랜치 이동)
   2. git rebase main (메인 기준으로 재설정)
   3. (충돌 나면 수정 후) git add . -> git rebase --continue
   4. git push -f (히스토리가 바뀌었으므로 강제 푸시 필요함)

