---
title: Protobuf를 활용한 Single Source of Truth(SSOT)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - SSOT
  - Protobuf
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## 1. 개요

* SSOT (Single Source of Truth): 데이터 구조, API 명세, 설정값 등을 단 한 곳의 정의(Schema)로 관리하는 전략.
* C/C++의 한계: 정적 컴파일 언어 특성상 런타임 리플렉션이 부족하여, 스키마 변경 시 관련 코드(구조체, 직렬화, CLI)를 수동으로 수정해야 하는 번거로움.
* Protobuf의 역할: 언어 독립적인 스키마를 통해 C++ 코드를 자동 생성함으로써 이 문제를 해결.

## 2. Protobuf 기반 SSOT 워크플로우

1. .proto 파일 작성: 데이터 구조와 서비스 인터페이스를 정의함 (진실의 유일한 근원).
2. 코드 생성 (protoc): 정의된 스키마를 바탕으로 C++ 헤더(*.pb.h)와 소스(*.pb.cc) 파일을 자동 생성함.
3. 비즈니스 로직 결합: 생성된 클래스를 사용하여 API 통신, 데이터 저장, CLI 처리를 수행함.

## 3. 주요 구현 요소

### ① 스키마 정의 (user.proto)

```
syntax = "proto3";
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

### ② 자동 생성된 C++ 활용

- 데이터 구조: User 클래스가 생성되어 set_name(), name() 등의 메서드 제공함.
- 직렬화(Serialization): SerializeToString() 등을 통해 바이너리나 JSON으로 즉시 변환 가능함.
- 일관성: 스키마에서 필드 타입을 바꾸면 컴파일 타임에 모든 관련 로직에서 에러가 발생하여 동기화를 강제함.

## 4. SSOT 도입 시 장점 (C/C++ 기준)

| 구분 | 수동 관리 (AS-IS) | Protobuf SSOT (TO-BE) |
|---|---|---|
| 변경 관리 | 구조체, 파싱 로직, 문서 각각 수정 | .proto 수정 후 컴파일하면 끝 |
| 타입 안전성 | 런타임 캐스팅 에러 위험 | 컴파일 타임에 타입 체크 완료 |
| 호환성 | 버전 관리 및 하위 호환 구현 어려움 | 필드 번호를 통한 자동 하위 호환 지원 |
| 다국어 지원 | 타 언어 연동 시 규격 재정의 필요 | Python, Go, Java 등과 동일 스키마 공유 |

## 5. 요약 및 결론

- C/C++ 프로젝트에서 Protobuf는 단순한 직렬화 도구를 넘어, 시스템 전체의 데이터 규격을 정의하는 중심점 역할을 함.
- Single Source of Truth를 실현함으로써 대규모 프로젝트의 유지보수 비용을 획기적으로 줄이고, 데이터 불일치로 인한 런타임 버그를 예방할 수 있음.

## Related Posts
- [HTTP/2 Multiplexing과 gRPC의 관계]({% link _posts/2026-03-30-http2-multi-flexing-gRPC.md %})
- [Protobuf와 gRPC의 핵심 원리 이해]({% link _posts/2026-03-30-protobuf-and-gRPC.md %})
