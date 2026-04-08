---
title: GitHub Actions - run: | 문법
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - GitHub Actions
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- GitHub Actions에서 run: |은 여러 줄의 명령어를 순차 실행할 때 쓰는 문법.

## 1. 의미 및 특징

* 다중 명령 수행: 한 줄에 다 적기 힘든 복잡한 스크립트를 줄바꿈하며 작성 가능함.
* 가독성 향상: 터미널에 직접 코딩하듯 위에서 아래로 한 줄씩 실행되어 가독성이 좋음.
* 문자열 보존: YAML에서 | 기호는 줄바꿈을 포함한 문자열을 있는 그대로 인식함.

## 2. 예시 비교

* 사용 전 (불편함): `run: date; echo "완료" >> readme.txt` (한 줄에 몰아 써야 함)
* 사용 후 (깔끔함):

```
run: |
  date
  echo "완료" >> readme.txt
```

## 3. 주의사항

* 들여쓰기 필수: run: | 다음 줄부터는 반드시 일정한 간격으로 들여쓰기를 맞춰야 함.
* Bash 문법: 기본적으로 리눅스 쉘(Bash) 명령어를 여러 줄 적는 방식.

