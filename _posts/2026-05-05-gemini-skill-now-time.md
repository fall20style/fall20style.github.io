---
title: "gemini CLI에서 now-time skill을 만들고 생성"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - skills
  - gemini
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "gemini CLI에서 now-time skill을 만들고 생성"
---


## 목적
now-time 스킬을 생성, 등록 및 사용하는 방법을 정리함.


### 1. 스킬 디렉토리 생성
스킬 파일들을 저장할 폴더를 만듦.
1 mkdir -p ~/.gemini/skills/now-time


### 2. SKILL.md 파일 작성
스킬의 정의와 지침을 담은 SKILL.md 파일을 해당 디렉토리에 생성함.

1 # Now Time Skill
2 현재 시간을 출력하기
3 - YYYY-MM-DD-HH:MM 포맷으로 적어주기
4 - 예제 시간 : 2026-05-05-16:50

### 3. 스킬 등록

gemini-cli 설정 파일(~/.gemini/config.yaml 등)이나 시스템 경로에 해당 스킬 위치를 등록함. (현재 환경에서는
    이미 /home/mjpark/.gemini/skills/now-time/SKILL.md에 위치해 있음.)

### 4. 스킬 활성화 및 사용
Gemini CLI 세션 내에서 activate_skill 도구를 사용하여 호출함.


* 활성화: activate_skill(name="now-time") 명령을 통해 스킬 지침을 로드함.
* 실행: 활성화된 스킬의 지침(YYYY-MM-DD-HH:MM 포맷 준수)에 따라 결과물을 출력함.

### 5. 사용 예시

- 사용자 입력: "지금 몇 시야?" 또는 "now-time 스킬 실행해 줘"
- Gemini CLI 응답: "2026-05-05-16:50" (지정된 포맷으로 출력함)

