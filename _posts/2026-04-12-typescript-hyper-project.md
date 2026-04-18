---
title: Hyper는 프론트엔드 기술로 데스크톱 앱을 만드는 Electron 프레임워크
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - typescript
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

- Hyper는 프론트엔드 기술로 데스크톱 앱을 만드는 Electron 프레임워크

## 1. 핵심 기술 스택

* 언어: TypeScript (안정성과 타입 체크를 위해 사용)
* 프레임워크: Electron (기반 환경) + React (UI 구성)
* 터미널 엔진: xterm.js (화면에 글자를 뿌리고 제어하는 핵심 엔진, VS Code 터미널과 동일함)
* 상태 관리: Redux (터미널의 설정이나 탭 상태 등을 관리함)

## 2. 프로세스 구조 (Electron 방식)

Hyper는 크게 두 가지 프로세스로 나뉘어 돌아감.

- Main Process (백엔드): Node.js 환경임. 창을 띄우고, 운영체제의 실제 셸(bash, zsh, powershell 등)과 통신하며, 플러그인을 로드함.
- Renderer Process (프론트엔드): 크롬 브라우저 환경임. React를 사용해 우리가 보는 터미널 화면(글자, 탭, 설정 창 등)을 그림.

## 3. 통신 방식 (RPC)

* 사용자가 키보드를 치면 Renderer가 이를 감지함.
* 이 신호를 Main 프로세스로 보냄.
* Main은 실제 OS의 셸에 명령을 전달하고, 돌아온 결과를 다시 Renderer에 전달해 화면에 출력함.

## 4. 플러그인 시스템

* Hyper의 가장 큰 특징임. 모든 기능이 플러그인으로 확장이 가능함.
* .hyper.js 설정 파일에 테마나 기능을 추가하면, 런타임에 JavaScript 모듈을 동적으로 로드해서 UI나 로직을 바꿈.

## 5. 디렉토리 구조 (주요 폴더)

* app/: Electron 메인 프로세스 로직이 담긴 곳임.
* lib/: React 컴포넌트와 Redux 액션 등 UI 관련 코드가 있음.
* bin/: 명령줄에서 hyper라고 쳤을 때 실행되는 스크립트가 들어있음.

결론: 웹 개발자라면 익숙한 React와 Node.js 지식만으로 터미널을 커스텀하거나 기능을 추가할 수 있는 구조임.
혹시 Hyper의 플러그인을 직접 만들어보는 방법이나 설정 파일 수정법이 궁금함?

핵심 기능과 기본 플러그인을 포함한 Hyper의 실제 코드 규모를 분석한 결과임. VS Code와 비교하면 훨씬 슬림해서 분석하기 딱 좋은 사이즈임.

## 1. 코드 라인 수 (LOC)
핵심 로직이 담긴 app과 lib 폴더 기준임.

* 전체 TypeScript/JavaScript: 약 3만 ~ 5만 라인 내외임. (주석 및 테스트 코드 포함)
* 핵심 로직: 실제 터미널 구동과 프로세스 통신을 담당하는 코드는 약 1만 라인 정도로 매우 응축되어 있음.
* 비교: VS Code가 수백만 라인인 것에 비하면 약 1/100 수준이라 개인이 전체 구조를 파악하기에 충분함.

## 2. 핵심 구성 요소 규모

* Renderer (UI/React): 코드의 약 60%를 차지함. 터미널의 탭 관리, 알림, 검색 바, 그리고 xterm.js를 화면에 렌더링하는 컴포넌트들임.
* Main (Node.js/Electron): 코드의 약 20%임. 설정 파일(hyper.js) 로드, 메뉴 바 생성, 업데이트 확인, 그리고 실제 셸(Shell) 세션을 생성하고 관리함.
* RPC & State (Redux): 코드의 약 20%임. 메인 프로세스와 렌더러 간의 데이터를 주고받고 터미널의 상태를 동기화함.

## 3. 핵심 플러그인 규모
Hyper는 기본적으로 "플러그인의 집합"처럼 설계됨. 기본 포함된 핵심 플러그인(Core Plugins)들은 놀라울 정도로 코드가 짧음.

* 테마 관련: 파일 하나(수십 줄)로 색상 값만 바꾸는 수준임.
* 기능 확장: 대부분 100~300라인 내외의 작은 모듈들임. (예: 특정 단축키 추가, UI 요소 숨기기 등)

## 4. 외부 의존성 (Heavy lifting)
Hyper가 작게 유지될 수 있는 이유는 무거운 작업을 외부 라이브러리에 맡기기 때문임.

* xterm.js: 터미널의 렌더링 성능과 글자 처리는 이 거대한 라이브러리가 다 함.
* node-pty: OS의 실제 터미널 프로세스를 띄우고 통신하는 복잡한 로직은 이 모듈이 처리함.

## 결론
- Hyper는 "가볍고 확장 가능한 껍데기"를 만드는 데 집중한 프로젝트.
- TypeScript로 작성된 React 컴포넌트와 Redux 상태 관리의 정석을 보기 좋음.

## Related Posts
- [Discord Echo 봇 환경 구축 및 실행]({% link _posts/2026-04-12-typescript-discord-echo-bot.md %})
- [JavaScript/TypeScript의 논블로킹 I/O 모델과 NanoClaw에서의 사용]({% link _posts/2026-04-11-nanoclaw-nonblocking-io-typescript.md %})
