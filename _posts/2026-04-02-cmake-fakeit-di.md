---
title: CMake로 FakeIt 사용 (의존성 주입)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - SWPattern
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 1. CMakeLists.txt 설정
REAL_IMPL 옵션이 ON일 때만 USE_REAL_IMPL 매크로를 정의하도록 구성함.

```
cmake_minimum_required(VERSION 3.10)
project(FakeItExample)
set(CMAKE_CXX_STANDARD 11)
```

### 1. REAL_IMPL 옵션 추가 (기본값 OFF)

```
option(REAL_IMPL "Use real implementation instead of mock" OFF)
add_executable(app main.cpp)
```

### 2. 옵션에 따라 전처리 지시자(USE_REAL_IMPL) 설정함if(REAL_IMPL)
```
    target_compile_definitions(app PRIVATE USE_REAL_IMPL)
    message(STATUS ">> Build Mode: REAL Implementation")else()
    message(STATUS ">> Build Mode: MOCK (FakeIt)")endif()
target_include_directories(app PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
```

## main.cpp 코드 적용 예시
소스 코드 내에서 #ifdef를 사용하여 실제 객체와 Mock 객체 사용 구간을 나눔.

``` c++
#include <iostream>
#include <string>

#ifdef USE_REAL_IMPL

    // 실제 객체 사용 시
#else
    #include <fakeit.hpp>
    using namespace fakeit;
#endif

struct MessageService {
    virtual bool send(std::string msg) = 0;
    virtual ~MessageService() = default;
};

// 실제 객체 구현
class RealMessageService : public MessageService {
public:
    bool send(std::string msg) override {
        std::cout << "[Real] " << msg << std::endl;
        return true;
    }
};
int main() {
#ifdef USE_REAL_IMPL
    // 1. 실제 객체 직접 사용함
    RealMessageService service;
    service.send("This is REAL");
#else
    // 2. FakeIt Mock 객체 사용함
    Mock<MessageService> mock;
    When(Method(mock, send)).Return(true);
    
    MessageService &service = mock.get();
    service.send("This is MOCK");
    
    Verify(Method(mock, send)).Exactly(1);
#endif
    return 0;
}
```

## 빌드 방법
터미널에서 REAL_IMPL 옵션을 넘겨 빌드 모드를 선택함.

* Mock 모드 (기본): cmake ..
* 실제 객체 모드: cmake -DREAL_IMPL=ON ..


## 코드 흐름 정리
제공된 main.cpp의 구조와 흐름을 정리함.
### 1. 전처리부 (Conditional Compilation)

* 컴파일 옵션 체크: CMakeLists.txt에서 넘겨준 REAL_IMPL 값에 따라 분기함.
* 헤더 제어: OFF일 때만 fakeit.hpp를 포함하여 빌드 가벼움을 유지함.

### 2. 구조부 (Interface & Class)

* 인터페이스(MessageService): 추상 메서드 send를 가진 부모 클래스임. Mock과 실제 객체 모두 이를 따름.
* 구현체(RealMessageService): 실제 콘솔 출력을 담당하는 로직이 들어있음.

### 3. 흐름 제어 (Main Function)

* 객체 생성 분기: #ifdef를 통해 빌드 시점에 어떤 객체를 생성할지 결정함.
* 실제 모드: RealMessageService 인스턴스를 직접 생성함.
   * Mock 모드: Mock<T>를 통해 가짜 객체를 만들고, When으로 가짜 동작을 주입함.
* 추상화 활용: 어떤 모드든 MessageService &service라는 인터페이스 참조를 통해 동일한 방식으로 send()를 호출함.
* 사후 검증(Mock 전용): Verify를 통해 실제 호출 여부를 체크하며 테스트 무결성을 확보함.

### 4. 핵심 요약

* 유연성: 코드 수정 없이 CMake 명령어 하나로 테스트용과 배포용 빌드를 전환함.
* 의존성 분리: 테스트 시 실제 출력 로직을 타지 않고도 send 호출 여부를 완벽히 확인 가능함.


## 의존성 주입(DI)

- 의존성 주입(DI)을 더 깔끔하게 처리하는 팩토리 패턴 적용
- 객체가 사용할 대상을 내부에서 직접 생성하지 않고 외부에서 넣어주는 방식.
- 이를 위해 MessageService를 사용하는 클라이언트 클래스를 추가하여 구조를 잡음.

### 1. 의존성 주입(DI) 구조의 main.cpp
App 클래스는 구체적인 클래스(Real/Mock)를 몰라도 인터페이스만 알면 작동함.

```c++
#include <iostream>
#include <string>
#include <memory>

#ifndef USE_REAL_IMPL
    #include <fakeit.hpp>
    using namespace fakeit;#endif
// 1. 인터페이스struct MessageService {
    virtual bool send(std::string msg) = 0;
    virtual ~MessageService() = default;
};
// 2. 실제 구현체
class RealMessageService : public MessageService {
public:
    bool send(std::string msg) override {
        std::cout << "[Real] " << msg << std::endl;
        return true;
    }
};
// 3. 의존성 주입을 받는 클라이언트 클래스 (핵심)
class App {
    MessageService& service; // 구체적인 구현이 아닌 인터페이스에 의존함
    public:
    // 생성자를 통해 외부에서 '주입'받음
    App(MessageService& svc) : service(svc) {}

    void run() {
        service.send("Hello DI World!");
    }
};
int main() {

#ifdef USE_REAL_IMPL
    // [실제 모드] 실제 객체를 생성하여 주입함
    RealMessageService realSvc;
    App app(realSvc); 
    app.run();
#else
    // [테스트 모드] Mock 객체를 생성하여 주입함
    Mock<MessageService> mock;
    When(Method(mock, send)).Return(true);

    App app(mock.get()); // Mock을 App에 주입함
    app.run();

    Verify(Method(mock, send)).Exactly(1); // App이 내부에서 호출했는지 검증함#endif
    return 0;
}
```

### 2. DI 적용 시 장점

* 결합도 낮춤: App 클래스는 RealMessageService가 없어도 빌드와 테스트가 가능함.
* 코드 재사용: 동일한 App 로직을 네트워크 전송용, 로그 출력용 등 다양한 서비스와 조합할 수 있음.
* 테스트 용이성: main 함수에서 보듯, 테스트 시점에만 간단히 Mock으로 바꿔 끼워 App의 로직만 검증할 수 있음.

### 3. 구조 요약

* 주체(App): "나는 send 기능이 필요해!"
* 주입자(main): "자, 여기 Real 혹은 Mock 객체야. 이걸 써!"
* 결과: App은 내부 로직 수정 없이 상황에 맞는 객체를 사용하게 됨.

## Related Posts
- [C++ 코딩 테스트용 기본 템플릿]({% link _posts/2026-04-01-cp-template-cpp.md %})
- [push_back보다 emplace_back이 효율적인 이유]({% link _posts/2026-04-01-emplace_back-effective.md %})
- [C++17 Structured Bindings의 활용]({% link _posts/2026-04-01-Structured_Bindings.md %})
- [FakeIt을 이용한 C++ 모의 객체(Mock) 생성]({% link _posts/2026-04-02-mock-object-with-fakeit.md %})
- [Protobuf를 활용한 Single Source of Truth(SSOT)]({% link _posts/2026-04-15-ssot-cpp-protobuf.md %})
