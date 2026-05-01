---
title: "디스코드 스타일 진화 TODO List"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - phoenix
categories:
  - TodoList
toc: true
toc_sticky: true
toc_label: 목차
description: "phoenix를 이용한 프로젝트 할 일 정리"
---

## 디스코드 스타일 진화 TODO List

### 1단계: Foundation (현재 완료 & 기반 다지기)

- [x] LiveView Hello World: 웹소켓 통신의 기본 이해
- [x] 기초 채팅 구현: PubSub을 이용한 실시간 메시지 전달
- [x] 닉네임 및 세션 관리: PID 대신 사용자가 설정한 이름이나 랜덤 익명 이름 부여
- [ ] Tailwind UI 입히기: 디스코드 특유의 어두운 테마(Dark Mode) 레이아웃 적용

### 2단계: Persistence (데이터의 영속성)

- [ ] 데이터베이스(PostgreSQL) 연동: 서버를 껐다 켜도 채팅 기록이 남도록 설정
- [ ] 사용자 인증(Auth): phx.gen.auth를 사용하여 실제 회원가입 및 로그인 구현
- [ ] 메시지 히스토리: 채널에 접속했을 때 과거 메시지 50개를 불러오는 로직 구현

### 3단계: Real-time Presence (누가 온라인인가?)

- [ ] Phoenix Presence 도입: 현재 접속 중인 사용자 목록 실시간 표시 (디스코드의 오른쪽 사용자 목록 기능)
- [ ] 입력 중...(Typing...) 표시: 누군가 타자를 치면 상단에 알림 표시
- [ ] 상태 표시: 온라인, 자리비움, 방해금지 등 사용자 상태 동기화

### 4단계: Structure & Scale (방대한 커뮤니티로)

- [ ] 다중 채널/서버(Guild): 여러 개의 채팅방(Topic)을 만들고 사용자가 선택해서 들어갈 수 있는 구조 설계
- [ ] 파일 및 이미지 전송: 채팅창에 이미지 업로드 및 미리보기 기능 추가
- [ ] 분산 서버 설정: libcluster를 사용하여 여러 대의 Elixir 서버 간에 메시지 동기화 테스트

### 5단계: Performance & Advanced (디스코드급 최적화)

- [ ] 데이터 압축: zstd를 활용하여 웹소켓 트래픽 최적화
- [ ] 성능 벤치마크: Locust나 Tsung 같은 도구로 동시 접속자 수천 명 가상 테스트
- [ ] Rust 연동: 성능이 극도로 필요한 로직을 Rust(NIFs)로 작성하여 Elixir에 연결
