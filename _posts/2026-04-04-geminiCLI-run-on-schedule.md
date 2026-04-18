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



## Related Posts
- [NanoClaw의 결과 반환 구조 연습]({% link _posts/2026-04-11-nanoclaw-docker-sandboxing-test.md %})
- ["프로젝트 구상 NanoClaw-Clone (AI 샌드박스 오케스트레이터 구현)"]({% link _posts/2026-04-11-nanoclaw-mini-ai-commander-plan.md %})
- [node.js 입력 대기 → 자식 프로세스 생성 → Docker 격리 실행]({% link _posts/2026-04-11-nanoclaw-nodejs-child_process-poc.md %})
- [nanoClaw에서 사용하는 node.js 기능]({% link _posts/2026-04-11-nanoclaw-nodejs-functions.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
- [Gemini CLI Docker 실행 가이드]({% link _posts/2026-04-16-run-gemini-cli-on-docker.md %})
- [Gemini CLI로 gdb bt 결과를 분석하기]({% link _posts/2026-04-17-gemini-cli-analyze-gdb-bt.md %})
- [Gemini를 포함하는 Dockerfile]({% link _posts/2026-04-17-gemini-cli-dockerfile.md %})
- [ZeroClaw 정보]({% link _posts/2026-04-17-zeroclaw-info.md %})
- [discord bot에서 gemini bot으로 동작하기]({% link _posts/2026-04-18-claw-poc-run-gemini-on-docker.md %})
- [Docker로 Gemini CLI 돌리는 법]({% link _posts/2026-04-18-how-to-run-gemini-on-docker.md %})
