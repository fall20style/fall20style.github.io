---
title: discord bot에서 gemini bot으로 동작하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Gemini CLI
  - Discord
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 한 일
gemini cli를 포함하는 docker image 작성
- discord 봇으로 docker를 띄워서 그 안에서 gemini를 실행하는 샌드박싱을 함

### gemini cli를 포함하는 docker image 작성
- https://github.com/fall20style/docker_prep/tree/main/gemini-cli-docker

### discord 연동하는 echo bot 변경
- https://github.com/fall20style/docker_prep/tree/main/discord_echo_bot


### Gemini-Integrated Bot

This version uses the `gemini-cli` Docker image to run commands via an LLM agent.

### Environment Setup

```bash
export GEMINI_API_KEY=your_gemini_api_key_here
```

**Run Bot:**
```bash
npx ts-node discord-bot-gemini.ts
```

### Screenshot 1
<img width="951" height="751" alt="Image" src="https://github.com/user-attachments/assets/2389bb59-93a1-4fb3-9401-2566ec86760b" />

### Screenshot 2
<img width="1080" height="2066" alt="Image" src="https://github.com/user-attachments/assets/ba8f0cc1-201d-40a6-920c-adcf741a7146" />

### Screenshot 3
<img width="897" height="194" alt="Image" src="https://github.com/user-attachments/assets/0f6b85c6-c8d7-46b6-a71e-372f71b93178" />
