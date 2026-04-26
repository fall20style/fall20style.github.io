---
title: '데이터가 버퍼를 타고 흐른다'는 SW적 의미
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - 파이프라인 패턴
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## 데이터가 버퍼를 타고 흐르는 구조

GStreamer나 소프트웨어 설계에서 '데이터가 버퍼를 타고 흐르는 구조'를 만들려면,
데이터의 생산부터 소비까지를 연결하는 연결 통로(Interface)와 흐름 제어(Flow Control) 메커니즘을 설계해야 함.

가장 표준적인 3단계 구성 방법

## 1. 노드(Element)와 통로(Pad) 정의
데이터가 머무는 곳(노드)과 이동하는 구멍(패드)을 설계합니다.

* Source (생산자): 데이터를 생성하여 버퍼에 담아 보냅니다. (예: 카메라, 파일 읽기)
* Transform (가공자): 버퍼를 받아 내용을 수정하거나 변환한 뒤 다음으로 넘깁니다. (예: 필터, 인코딩)
* Sink (소비자): 최종 데이터를 처리합니다. (예: 화면 출력, 파일 저장)


## 2. 데이터 전달 방식 선택 (Push vs Pull)
버퍼가 흐르는 '동력'을 누가 제공하느냐를 결정해야 합니다.

* Push Model (GStreamer 기본): 상류(Upstream)에서 데이터가 준비되면 하류(Downstream)로 밀어넣는 방식입니다. 실시간 스트리밍에 적합합니다.
* Pull Model: 하류(소비자)에서 "데이터 더 줘"라고 요청하면 상류에서 데이터를 보내는 방식입니다. 파일 편집처럼 정해진 데이터를 빠르게 처리할 때 유리합니다.


## 3. 핵심 메커니즘 구현 (코드 관점)

실제로 데이터가 흐르게 하려면 다음 요소들이 필요합니다.

## ① 공통 버퍼 구조체 (Buffer Structure)

단순 데이터뿐만 아니라, 메타데이터(타임스탬프, 데이터 크기 등)를 포함해야 합니다.

``` C
struct Buffer {
    void* data;      // 실제 데이터 주소
    size_t size;     // 데이터 크기
    long timestamp;  // 재생 시간 정보
};
```

## ② 큐(Queue)와 스레드 분리

흐름이 막히지 않으려면 각 단계 사이에 큐(Queue)를 두어 완충 지대를 만듭니다.

* Thread 1: 카메라에서 데이터를 읽어 Queue에 넣음
* Thread 2: Queue에서 꺼내와서 화면에 출력

## ③ 흐름 제어 (Backpressure)

소비자가 너무 느리면 생산자가 속도를 줄여야 합니다. 큐가 가득 차면 생산자를 잠시 멈추는(Blocking) 로직이 반드시 필요합니다.

## GStreamer 방식의 예시 코드 (개념적)

만약 직접 파이프라인을 코드로 짠다면 이런 흐름이 됩니다.

# 1. 엘리먼트 생성

```
source = create_element("filesrc")
filter = create_element("capsfilter")
sink = create_element("autovideosink")
```

# 2. 파이프라인(컨테이너) 구축

```
pipeline = Pipeline()
pipeline.add(source, filter, sink)
```

# 3. '패드'를 통해 연결 (데이터 통로 개설)

```
source.link(filter)
filter.link(sink)
```

# 4. 상태를 'PLAYING'으로 변경하여 버퍼 흐름 시작

```
pipeline.set_state(PLAYING)
```


## 구현 시 주의할 점

* 메모리 관리: 버퍼가 파이프라인을 통과할 때, 누가 메모리를 해제할지 정해야 함. (참조 횟수 계산 방식 권장).
* 동기화: 비디오와 오디오가 따로 흐를 때 타임스탬프를 보고 싱크를 맞추는 로직이 핵심.



데이터가 흐르는 구조를 만드는 방식

데이터가 흐르는 구조를 만들 때 Queue(FIFO)가 가장 직관적이지만,
시스템의 목적(지연 시간, 데이터 크기, 처리 방식)에 따라 다른 메커니즘을 사용하기도 함.


## 1. 링 버퍼 (Ring Buffer / Circular Buffer)

고정된 크기의 메모리 공간을 원형으로 연결하여 사용하는 방식.

* 작동 방식: 쓰기 포인터와 읽기 포인터가 원을 그리며 회전.
* Queue와 차이: 새로운 메모리 할당/해제가 반복되지 않아 성능이 매우 빠르고 지연 시간이 짧음.
* 사용 사례: 오디오 드라이버, 임베디드 시스템, 고성능 네트워크 카드 등 실시간성이 극도로 중요한 곳.


## 2. 제로 카피 공유 메모리 (Zero-copy Shared Memory)
데이터를 한 곳에서 다른 곳으로 복사하지 않고, 메모리 주소값만 주고받는 방식.

* 작동 방식: 생산자가 메모리 영역에 데이터를 쓰고 "여기 다 썼어"라고 신호(Signal)만 보냄. 소비자는 그 주소에 직접 접근해 읽음.
* Queue와 차이: 대용량 버퍼를 실제로 이동(Copy)시키지 않으므로 CPU 부하가 거의 없음.
* 사용 사례: 4K/8K 고해상도 영상 처리, 가상 머신 간 데이터 통신.


## 3. 발행-구독 모델 (Pub/Sub via Event Bus)
특정 대상을 지정해 버퍼를 주는 게 아니라, 관심 있는 모두에게 뿌리는 방식.

* 작동 방식: 생산자는 'Topic'에 데이터를 던지고(Publish), 필요한 소비자들은 그 Topic을 구독(Subscribe)해서 가져감.
* Queue와 차이: 1:1이 아닌 1:N 통신이 가능하며, 시스템 간 결합도가 가장 낮음.
* 사용 사례: 마이크로서비스 아키텍처(MSA), 분산 로그 시스템 (Kafka, RabbitMQ 등).


## 4. 리액티브 스트림 (Reactive Streams / Observable)
데이터가 준비되었을 때만 이벤트를 발생시켜 흐름을 제어하는 방식.

* 작동 방식: 데이터가 흐르는 통로 자체를 하나의 '강(Stream)'으로 보고, 연산자(Map, Filter 등)를 체인처럼 엮음.
* Queue와 차이: 명시적으로 저장소에서 꺼내오는 느낌보다는 데이터가 발생할 때마다 반응(React)하는 함수형 스타일.
* 사용 사례: RxJava, WebFlux, 비동기 UI 이벤트 처리.


## 비교 요약

| 방식 | 핵심 장점 | 추천 상황 |
|---|---|---|
| Queue | 구현이 쉬움 | 일반적인 비동기 처리 |
| Ring Buffer | 속도가 빠름, 메모리 고정 | 실시간 제어, 하드웨어 통신 |
| Zero-copy | 대용량 처리 효율 | 고화질 영상, 빅데이터 처리 |
| Pub/Sub | 유연한 확장성 (1:N) | 시스템 간 연동, 로그 수집 |
| Reactive | 복잡한 가공 로직 | 비동기 함수형 프로그래밍 |


## GStreamer는 무엇을 쓸까?
GStreamer는 내부적으로 Queue 엘리먼트를 쓰기도 하지만, 기본적으로는 제로 카피(Zero-copy)를 지향하며 버퍼의 참조 횟수(Ref-counting)를 관리하는 방식을 혼합 사용.

