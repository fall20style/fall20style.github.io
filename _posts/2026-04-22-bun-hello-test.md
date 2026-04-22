---
title: bun - hello test 작성
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

## Bun이란?

[Bun](https://bun.sh)은 빠르고 모든 기능을 갖춘 JavaScript 런타임, 패키지 관리자, 테스트 러너 및 번들러

Zig 프로그래밍 언어와 JavaScriptCore 엔진을 사용하여 처음부터 구축되었으며, 속도와 개발자 경험에 중점을 둔 Node.js의 드롭인(drop-in) 대체품으로 설계.

### 주요 기능
- **매우 빠름:** Node.js보다 최대 4배 더 빠르게 시작하며 npm보다 훨씬 빠르게 패키지를 설치.
- **올인원 (All-in-One):** Node.js, npm/yarn/pnpm, Webpack/Vite, Jest/Vitest를 대체.
- **네이티브 TypeScript 및 JSX 지원:** 별도의 트랜스파일 과정 없이 `.ts` 및 `.tsx` 파일을 직접 실행.
- **Node.js 호환성:** 대부분의 Node.js API를 구현하고 `node_modules`를 지원.
- **웹 표준 API:** `fetch`, `WebSocket` 등을 내장 지원.

## 설치 방법

```bash
curl -fsSL https://bun.sh/install | bash
```

## Bun 사용 방법

### 스크립트 실행

JavaScript 또는 TypeScript 파일을 직접 실행:

```bash
bun run hello.ts
# 또는 간단하게
bun hello.ts
```

### 패키지 관리

Bun은 매우 빠른 패키지 관리자:

```bash
bun install          # 의존성 설치
bun add <package>    # 패키지 추가
```

### 테스트
Bun에는 빌트인 테스트 러너가 포함되어 있음:
```bash
bun test
```

## 프로젝트 예제

### 1. hello.ts

```typescript
// hello.ts
console.log("Hello, Bun! 🚀");
```

**스크립트 실행:**

```bash
~/work/bun_test$ bun run hello.ts
Hello, Bun! 🚀
```

### 2. server.ts

`Bun.serve`를 사용한 고성능 HTTP 서버.

```typescript
// server.ts
const server = Bun.serve({
  port: 3000,
  fetch(request) {
    return new Response("Welcome to Bun!");
  },
});

console.log(`Listening on http://localhost:${server.port}...`);
```

**서버 실행:**
```bash
~/work/bun_test$ bun run server.ts
Listening on http://localhost:3000...
```
