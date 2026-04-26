---
title: typescript에서 Promise 의미
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

## Promise를 가진 코드 분석하기

``` typescript
export async function getAttendanceCount(userId: string): Promise<number> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
  });

  return user ? user.attendanceCount : 0;
}

```

TypeScript에서 Promise는 "당장 결과는 없지만, 미래에 값을 주겠다는 약속"을 의미함.
코드가 실행되는 동안 멈추지 않고 비동기적으로 작업을 처리할 때 사용함.


## 1. 비동기 작업의 영수증

* `prisma.user.findUnique`는 데이터베이스(DB)에 접근함.
* DB에서 데이터를 가져오는 건 시간이 걸리는 작업임.
* 이때 프로그램이 멈추지 않도록, 일단 "나중에 값을 채워줄게"라는 의미로 Promise라는 객체를 먼저 반환함.

## 2. Promise<number>의 의미

* Promise: "이 함수는 비동기로 동작함."
* `<number>`: "약속이 지켜졌을 때(성공했을 때) 최종적으로 돌려줄 값은 숫자(number)임."
* 즉, getAttendanceCount는 호출되자마자 숫자를 주는 게 아니라, 숫자를 담을 박스(Promise)를 먼저 주는 것임.

## 3. async와 await의 관계

* `async`: 이 키워드가 붙은 함수는 무조건 Promise를 반환함.
* `await`: Promise라는 박스가 열릴 때까지(데이터가 올 때까지) 기다렸다가 안의 내용물을 꺼내겠다는 뜻임.
* `const user = await prisma...` 부분에서 DB 작업이 끝날 때까지 기다린 후 실제 user 데이터를 변수에 담음.

## 일상생활 비유

* 주문: 카페에서 커피를 주문함.
* Promise 받음: 점원이 준 진동벨이 Promise임. (커피는 아니지만, 커피를 줄 거라는 약속임)
* 비동기 작업: 커피가 만들어지는 동안 나는 자리에 앉아 휴대폰을 봄 (멈추지 않음).
* await: 진동벨이 울리면(작업 완료) 가서 커피(number 값)를 받아옴.

## 요약

* Promise는 비동기 작업의 상태와 결과를 담는 객체임.
* 함수 타입에 `Promise<number>`라고 적힌 건, 이 함수를 다 쓰고 나면 결국 숫자 데이터를 얻게 될 것임을 보장함.

