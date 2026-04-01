---
title: emplace_back이 더 효과적인 이유
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Modern C++
  - CP
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

push_back과 emplace_back의 차이는 객체를 만드는 위치.

## 1. push_back (복사/이동 발생)

* 임시 객체를 먼저 만든 뒤, 그걸 컨테이너 안으로 복사하거나 이동시킴.
* v.push_back(pair<int, int>(1, 2)); -> 객체 생성 후 컨테이너로 옮김 (2단계).

## 2. emplace_back (현지 생성)

* 생성자에 필요한 인자만 넘기면, 컨테이너가 가진 메모리 공간에서 직접 객체를 생성함.
* v.emplace_back(1, 2); -> 내부에서 바로 생성 (1단계). 불필요한 복사/이동 생성자 호출이 없음.

## 요약

* 불필요한 임시 객체 생성이 없음.
* 특히 pair, tuple, 또는 구조체(struct)를 다룰 때 성능 이득이 큼.
* 단순 int 같은 기본 타입은 차이 없음.


