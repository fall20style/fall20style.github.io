---
title: 원본 저장소에서 Fork 없이 바로 PR을 만드는 방법
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

## 1. 전제 조건

* 해당 저장소에 Collaborator(협업자) 권한이 있어야 함
* 권한이 없다면 본인 계정으로 Fork를 해야만 수정이 가능함

## 2. PR 생성 단계
권한이 있다면 아래 순서대로 진행하면 됨

1. 원본 저장소 클론 (Clone)

``` bash
git clone [원본_저장소_URL]
cd [저장소_이름]
```

2. 새 브랜치 생성 및 이동 (Branch)

``` bash
git checkout -b [새_브랜치_이름]
```

3. 코드 수정 및 커밋 (Commit)

``` bash
git add .
git commit -m "작업 내용 요약"
```

4. 원본 저장소로 푸시 (Push)

``` bash
git push origin [새_브랜치_이름]
```

5. GitHub에서 PR 생성
* 저장소 상단의 'Compare & pull request' 클릭
  * 내용 작성 후 제출하면 끝

## 3. 요약 비교

| 구분 | Fork 방식 | 직접 Push 방식 |
|---|---|---|
| 저장소 | 내 계정으로 복사된 곳 | 원본 저장소 그 자체 |
| 권한 | 누구나 가능 | 쓰기 권한 필수 |
| 용도 | 오픈 소스 기여 시 | 팀 프로젝트 협업 시 |
