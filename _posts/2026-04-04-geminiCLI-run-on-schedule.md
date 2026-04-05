---
title: gemini CLI를 특정 시간에 실행
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - geminiCLI
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 매일 아침 8시에 뉴스 요약하고 저장
```
0 8 * * * /usr/local/bin/gemini "오늘의 주요 뉴스 요약해줘" >> ~/daily_news.txt
```
- linux의 경우, crontab을 이용함.


### 생각해볼 일
- 다른 CLI 기반 도구도 같은 방식으로 할 수 있겠다.
- 하네스 엔지니어링 설정이 된 dir에서 작업하기 (예를 들어 blog를 쓰고 git push)

