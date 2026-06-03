---
title: "grpc hello world 분석"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - gRPC
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "grpc hello world 분석"
---

## grpc의 예제 코드 동작시키기

```
cd /home/USER/work/github/grpc/examples/cpp/helloworld make clean
export PKG_CONFIG_PATH=/home/USER/work/github/grpc/third_party/re2:$PKG_CONFIG_PATH
make 2>&1 | tee ~/build3.log
./greeter_async_server
```

### gRPC 공식 GitHub 저장소(grpc/grpc)의 C++ Helloworld 예제

클라이언트가 서버에 이름을 보내면 서버가 인사를 반환하는 가장 기본적인 단방향(Unary) RPC 구조
- Protobuf 정의
- 서버 구현
- 클라이언트 구현


### 서비스 및 메시지 정의 (helloworld.proto)

- 빌드 시점에 이 .proto 파일을 컴파일러(protoc)로 실행하면 C++ 클래스 파일(helloworld.pb.h/.cc, helloworld.grpc.pb.h/.cc)이 자동으로 생성

### 서버 구현 (greeter_server.cc)

- 서버는 자동으로 생성된 base 클래스를 상속받아 실제 비즈니스 로직을 구현하고, 특정 포트를 열어 대기
- 서버는 Greeter::Service를 상속받아 SayHello 메서드에서 요청받은 이름에 "Hello "를 붙여 반환하는 비즈니스 로직을 구현
- ServerBuilder를 사용해 포트(50051)를 열고 등록된 서비스를 구동


### 클라이언트 구현 (greeter_client.cc)

- 클라이언트는 서버와 연결되는 채널(Channel)을 생성하고, 서버의 메서드를 원격 호출하는 스텁(Stub)을 이용해 통신
- 클라이언트는 Greeter::NewStub으로 스텁을 생성하여 서버와 연결(Channel)을 맺고, SayHello 메서드를 호출하여 원격 인사를 받음.

## gRPC C++의 동기(Sync) 예제와 비동기(Async) 예제

gRPC C++의 동기(Sync) 예제와 비동기(Async) 예제의 가장 큰 차이점은 스레드가 작업 완료를 기다리는 방식(Bloking vs Non-blocking)과 CompletionQueue(완료 큐)의 유무

### 작동방식 비교

#### 동기 방식

- 함수를 호출하면 결과가 올 때까지 스레드가 그 자리에 멈춰서(Blocking) 기다립니다.
- 특징: 코드가 직관적이고 읽기 쉽습니다. 하지만 동시 요청이 많아지면 요청마다 스레드를 새로 생성해야 하므로 CPU와 메모리 리소스 소모가 극심해집니다

#### 비동기 방식

- 개념: 함수를 호출하면 즉시 제어권을 반환(Non-blocking)하고 다른 작업을 수행합니다. 작업이 완료되면 알림을 받아 처리합니다.
- 특징: gRPC::CompletionQueue라는 이벤트 큐를 사용합니다. 하나의 스레드가 수천 개의 연결과 요청 완료 이벤트를 번갈아가며 처리할 수 있어 대규모 고성능 서버에 필수적입니다.


