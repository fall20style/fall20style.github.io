---
title: gRPC Git Submodule 구조 요약
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - grpc
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: gRPC Git Submodule 구조 요약
---

# gRPC Git Submodule 구조 및 의존성 가이드

gRPC C++ 코어 및 관련 라이브러리들은 성능과 이식성을 위해 다양한 외부 오픈소스 프로젝트를 `git submodule`로
포함하고 있습니다. 모든 서브모듈은 기본적으로 `third_party/` 디렉토리에 위치합니다.

## 1. 핵심 인프라 및 런타임 의존성

gRPC의 동작에 필수적인 라이브러리들입니다.

  *   **abseil-cpp**: Google의 표준 C++ 확장 라이브러리로, gRPC 내부에서 `std::` 타입을 보완하거나 대체하여
  사용합니다.
  *   **boringssl**: 암호화 및 TLS/SSL 통신을 위한 OpenSSL의 포크 버전입니다.
  *   **protobuf**: gRPC의 인터페이스 정의 언어(IDL)이자 데이터 직렬화 메이저 프레임워크입니다.
  *   **zlib**: HTTP/2 스트림 압축 및 데이터 압축을 위해 사용됩니다.
  *   **cares**: 비동기 DNS 조회를 위한 라이브러리입니다.
  *   **re2**: 효율적인 정규 표현식 처리를 위해 사용됩니다.

## 2. 프로토콜 및 부가 기능

  *   **upb**: C로 작성된 작고 빠른 프로토콜 버퍼 라이브러리로, gRPC Core 내부 및 Python 구현체에서 널리
  사용됩니다.
  *   **envoy-api / xds**: xDS 프로토콜(L7 로드 밸런싱 및 구성) 구현을 위한 API 정의 모음입니다.
  *   **opentelemetry-cpp**: 분산 트레이싱 및 메트릭 수집을 위한 Observability 라이브러리입니다.
  *   **opencensus-proto**: 통계 수집 및 분산 추적 프로토콜을 위한 정의입니다.

## 3. 테스트 및 도구 (Development Only)
  개발 및 검증 단계에서 주로 사용되는 모듈들입니다.

  *   **googletest (gtest/gmock)**: C++ 단위 테스트 및 모킹 프레임워크입니다.
  *   **benchmark**: 성능 측정을 위한 마이크로 벤치마크 프레임워크입니다.
  *   **fuzztest**: 퍼즈(Fuzz) 테스팅을 위한 도구입니다.
  *   **bloaty**: 바이너리 크기 분석 도구입니다.

## 4. 서브모듈 관리 방법

### 서브모듈 초기화 및 업데이트

  프로젝트를 처음 클론할gRPC 저장소의 `.gitmodules` 파일을 분석하여 git submodule 구조를 정리해 드립니다.

### gRPC Git Submodule 구조 정리

  gRPC는 다양한 외부 라이브러리를 의존성으로 가지며, 이를 `third_party` 디렉토리 하위의 git submodule로 관리하고
  있습니다. 주요 하위 모듈들은 다음과 같은 역할을 수행합니다.


#### 1. 핵심 의존성 (Core Dependencies)

  *   **[abseil-cpp](https://github.com/abseil/abseil-cpp):** C++ 표준 라이브러리를 보완하는 알고리즘, 컨테이너
  등을 제공합니다.
  *   **[protobuf](https://github.com/protocolbuffers/protobuf):** gRPC의 데이터 직렬화 및 인터페이스 정의
  언어(IDL)인 Protocol Buffers의 핵심 구현체입니다.
  *   **[boringssl](https://boringssl.googlesource.com/boringssl):** 보안 전송(TLS/SSL)을 위한 OpenSSL 기반의
  라이브러리입니다.
  *   **[zlib](https://github.com/madler/zlib):** 데이터 압축 및 해제를 위한 표준 라이브러리입니다.

#### 2. 성능 및 테스트 도구 (Performance & Testing)

  *   **[benchmark](https://github.com/google/benchmark):** 코드의 마이크로 벤치마크 측정을 위한
  라이브러리입니다.
  *   **[googletest](https://github.com/google/googletest):** 단위 테스트(Unit Test) 및 모킹(Mocking)
  프레임워크입니다.
  *   **[fuzztest](https://github.com/google/fuzztest):** 소프트웨어의 취약점을 찾기 위한 퍼즈 테스팅
  프레임워크입니다.

#### 3. 네트워크 및 인프라 (Network & Infrastructure)

  *   **[cares](https://github.com/c-ares/c-ares):** 비동기 DNS 요청을 처리하기 위한 C 라이브러리입니다.
  *   **[re2](https://github.com/google/re2):** 안전하고 효율적인 정규표현식 라이브러리입니다.
  *   **[envoy-api](https://github.com/envoyproxy/data-plane-api):** xDS와 같은 서비스 메시 제어 평면을 위한 API
  정의를 포함합니다.
  *   **[googleapis](https://github.com/googleapis/googleapis):** Google API들의 gRPC 인터페이스 정의(proto
  파일)를 포함합니다.

#### 4. 기타 유틸리티 (Other Utilities)

  *   **[bloaty](https://github.com/google/bloaty):** 바이너리 크기 분석 도구입니다.
  *   **[opentelemetry-cpp](https://github.com/open-telemetry/opentelemetry-cpp):** 분산 추적 및 메트릭 수집을
  위한 OpenTelemetry 구현체입니다.

  ---

### 서브모듈 관리 명령어 (참고)

gRPC를 처음 클론할 때 모든 의존성을 함께 가져오려면 다음 명령어를 사용합니다:

```
git clone --recursive https://github.com/grpc/grpc.git
```

이미 클론된 상태에서 서브모듈을 초기화하고 업데이트하려면:

```
git submodule update --init --recursive
```
