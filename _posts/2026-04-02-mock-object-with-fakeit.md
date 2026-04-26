---
title: Mock Object 패턴 (FakeIt 사용)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - SWPatterns
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

# Mock Object 패턴 (FakeIt 사용)

* 정의: 실제 객체를 사용하기 어려운 상황에서 이를 흉내 내는 '가짜 객체'를 만들어 테스트하는 기법임.
* 목적: 외부 의존성(DB, API 등)을 끊고 테스트하려는 로직에만 집중하기 위함임.
* 특징: 단순히 값을 반환하는 것을 넘어, 특정 메서드가 호출되었는지나 인자가 정확한지 등의 행위 검증에 초점을 맞춤.

## C++ FakeIt 라이브러리
- [FakeIt](https://github.com/eranpeer/FakeIt)
- C++11 기반의 헤더 전용(Header-only) 모킹 프레임워크
- 설정이 간편하고 직관적인 문법을 제공함.

## 예제 코드

```c++
#include <iostream>
#include <fakeit.hpp> 

using namespace fakeit;

// 1. 대상 인터페이스 정의struct MessageService {
    virtual bool send(std::string msg) = 0;
};

int main() {
    // 2. Mock 객체 생성
    Mock<MessageService> mock;

    // 3. 동작 설정 (Stubbing)
    // "hello"가 들어오면 true를 리턴하도록 설정함
    When(Method(mock, send).Using("hello")).Return(true);

    // 4. Mock 인스턴스 획득 및 사용
    MessageService &service = mock.get();
    bool result = service.send("hello");

    // 5. 검증 (Verification)
    // send 메서드가 정확히 1번 호출되었는지 확인함
    Verify(Method(mock, send)).Exactly(1);

    std::cout << "Result: " << std::boolalpha << result << std::endl;
    return 0;
}
```

## 주요 기능 요약

* Setup (When): 특정 호출에 대해 어떤 값을 반환할지 정의함.
* Verification (Verify): 예상한 대로 함수가 호출되었는지 사후 검증함.
* Matchers (Using): 특정 인자값이나 Any 같은 조건으로 호출을 제한할 수 있음.
* Spying: 실제 객체의 동작을 유지하면서 호출 기록만 추적하는 것도 가능.

## Related Posts
- [C++ 코딩 테스트용 기본 템플릿]({% link _posts/2026-04-01-cp-template-cpp.md %})
- [push_back보다 emplace_back이 효율적인 이유]({% link _posts/2026-04-01-emplace_back-effective.md %})
- [C++17 Structured Bindings의 활용]({% link _posts/2026-04-01-Structured_Bindings.md %})
- [CMake 환경에서 FakeIt과 의존성 주입(DI) 설정]({% link _posts/2026-04-02-cmake-fakeit-di.md %})
- [Protobuf를 활용한 Single Source of Truth(SSOT)]({% link _posts/2026-04-15-ssot-cpp-protobuf.md %})
