---
title: 디스코드 출석체크 봇 3 - db기능만 떼어서 실험
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - discord
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---




## 목적
// # test-db-2.ts 가이드 문서
- `test-db-2.ts` 스크립트의 작동 방식과 데이터베이스 테스트 절차를 설명
- 이 스크립트는 Prisma ORM을 사용하여 SQLite 데이터베이스와 연동
- 유저의 출석 체크 로직을 시뮬레이션합니다.

## 1. 주요 기능
- **환경 변수 설정**: Bun 환경에 맞춰 `DATABASE_URL`을 동적으로 설정.
- **중복 출석 방지**: 당일 이미 출석한 기록이 있는지 확인.

- **Upsert 로직**: 유저가 없으면 새로 생성(`create`)하고, 있으면 정보(`lastAttendance`, `attendanceCount`)를 업데이트(`update`).
- **안전한 연결 종료**: 작업 완료 후 Prisma Client의 연결을 해제.


## 2. 코드 상세 설명

### Prisma 초기화

``` typescript
process.env.DATABASE_URL = "file:./prisma/dev.db";
const prisma = new PrismaClient();
```
- 프로젝트 내의 SQLite 파일 경로를 명시적으로 지정하여 Prisma Client를 생성.

### 중복 출석 체크

``` typescript
const lastDate = new Date(
  lastAttendance.getFullYear(),
  lastAttendance.getMonth(),
  lastAttendance.getDate()
);
if (lastDate.getTime() === today.getTime()) {
  console.log("❌ 결과: 이미 오늘 출석 기록이 존재함!");
  return;
}
```
- 기존 유저의 마지막 출석 날짜와 현재 날짜를 비교하여 날짜가 같으면 진행을 중단.

### 데이터 업데이트 (Upsert)

```typescript
const updatedUser = await prisma.user.upsert({
  where: { id: userId },
  update: {
    lastAttendance: now,
    attendanceCount: { increment: 1 },
  },
  create: {
    id: userId,
    lastAttendance: now,
    attendanceCount: 1,
  },
});
```
- `increment: 1`을 사용하여 누적 출석 횟수를 안전하게 증가.

## 3. 실행 방법

Bun을 사용하여 스크립트를 직접 실행.

```bash
bun run test-db-2.ts
```

### 1번 실행

```bash
~/work/docker_prep/my-attendance-bot$ bun run test-db-2.ts
[test_user_777] 유저로 DB 테스트 시작함...
결과: 출석 데이터 기록 성공!
상세: 누적 횟수 [1회], 마지막 시간 [Sun Apr 26 2026 00:15:49 GMT+0900 (Korean Standard Time)]
```

### 2번 실행

```bash
~/work/docker_prep/my-attendance-bot$ bun run test-db-2.ts
[test_user_777] 유저로 DB 테스트 시작함...
결과: 이미 오늘 출석 기록이 존재함! (중복 체크 성공)
```

## 4. 주의 사항
- `prisma/schema.prisma` 파일에 `User` 모델이 정의되어 있어야 합니다.

```
model User {
  id               String   @id
  lastAttendance   DateTime
  attendanceCount  Int      @default(0)
}
```

- 실행 전 `prisma generate`를 통해 Prisma Client가 최신 상태인지 확인하십시오.
- 데이터베이스 파일(`./prisma/dev.db`)이 존재하는지 확인하십시오.
