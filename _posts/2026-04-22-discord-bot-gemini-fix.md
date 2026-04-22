---
title: discord-bot-gemini 4.22 수정
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - bun
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


### discord-bot-gemini 4.22 수정
1. command를 실행할 수 있도록 변경하기
2. bun으로 실행하게 변경

### 서버 실행
```
~/work/docker_prep/discord_echo_bot$ bun run discord-bot-gemini.ts
디스코드 연결 시도 중...
디스코드 봇 로그인 성공: Fall20StyleEcodBot#5832
[알림] 22:40 메시지 전송 완료
```

### Bun 설치하기
- https://bun.com/docs/installation

### Discord Screenshot

#### 날씨

<img width="604" height="161" alt="Image" src="https://github.com/user-attachments/assets/2b38aa54-0790-4259-979b-c61b7aba7c71" />

#### 샌드박싱 수정

<img width="572" height="536" alt="Image" src="https://github.com/user-attachments/assets/8c0fe7c5-44fc-49d9-ba4c-9264426fbff7" />

### 느낌

1. bun이 시작 시간이 node.js에 비해 빠르다.
2. typescript로 error를 수정함. 뿌듯했다.
