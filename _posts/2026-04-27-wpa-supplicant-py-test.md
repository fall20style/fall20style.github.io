---
title: wpa_supplicant 기능을 python 테스트
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - wpa_supplicant
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### 1. Unix Domain Socket 주요 명령 목록

`wpa_supplicant` 제어 인터페이스에서 지원하는 주요 명령

이 명령들은 Unix Domain Socket(`SOCK_DGRAM`)을 통해 전송됨.

*   **`PING`**: 서버가 응답하는지 확인. (응답: `PONG`)
*   **`STATUS`**: 현재 연결 상태(SSID, BSSID, IP 등)를 확인.
*   **`SCAN`**: 주변 와이파이 스캔을 시작.
*   **`SCAN_RESULTS`**: 마지막 스캔 결과를 가져옴.
*   **`LIST_NETWORKS`**: 저장된 네트워크 프로필 목록을 보여줌.
*   **`SIGNAL_POLL`**: 현재 연결된 신호의 세기(RSSI)와 속도를 확인함.
*   **`MIB`**: 상세한 MIB(Management Information Base) 통계 정보를 가져옴.
*   **`ADD_NETWORK`**: 새로운 네트워크 프로필을 생성함.
*   **`SET_NETWORK <id> <var> <val>`**: 특정 네트워크의 설정을 변경함 (예: `ssid`, `psk`).
*   **`ENABLE_NETWORK <id>`**: 특정 네트워크를 활성화함.
*   **`DISABLE_NETWORK <id>`**: 특정 네트워크를 비활성화함.
*   **`REMOVE_NETWORK <id>`**: 네트워크 프로필을 삭제함.
*   **`SAVE_CONFIG`**: 현재 설정을 파일에 저장함.

### 2. 예제 코드 설명

본 저장소에 포함된 예제 스크립트들은 Python의 `socket` 모듈을 사용하여 `wpa_supplicant`와 직접 통신.

*   **`ping.py`**: 가장 기본적인 동작 확인용 스크립트입니다. `PING` 명령을 보내고 `PONG` 응답을 받으면 성공.
*   **`status.py`**: 현재 연결된 네트워크의 상세 정보를 가져와 파싱합니다. `wpa_state`, `ssid`, `ip_address` 등을 확인.
*   **`list_network.py`**: 장치에 등록된 모든 네트워크 프로필의 ID, SSID, BSSID 정보를 출력.
*   **`scan.py`**: `SCAN` 명령을 내려 스캔을 유도하고, 일정 시간 대기 후 `SCAN_RESULTS`를 통해 주변 AP 목록을 가져옴.
*   **`signal_poll.py`**: 실시간으로 RSSI(신호 세기), LINKSPEED(전송 속도), FREQUENCY(채널 주파수) 정보를 가져옴.
*   **`mip.py`**: `MIB` 명령을 통해 하위 레벨의 드라이버 및 프로토콜 통계 데이터를 가져옴.

### 3. 결과 예시

#### `status.py` 실행 결과 예시:

```text
=== 현재 연결 상세 상태 ===
현재 상태: COMPLETED
연결된 SSID: My_Home_WiFi
IP 주소: 192.168.0.15
MAC 주소: 00:11:22:33:44:55
```

#### `scan.py` 실행 결과 예시:

```text
스캔 중... (5초만 대기)

=== 찾은 와이파이 목록 ===
bssid / frequency / signal level / flags / ssid
00:aa:bb:cc:dd:ee  2412  -45  [WPA2-PSK-CCMP][ESS]  Home_AP
11:22:33:44:55:66  5240  -67  [WPA2-PSK-CCMP][ESS]  Cafe_WiFi
```

#### `signal_poll.py` 실행 결과 예시:

```text
=== 실시간 신호 품질 분석 ===
신호 세기(RSSI): -52 dBm
링크 속도: 144 Mbps
노이즈: 9999
주파수: 2412 MHz
```


#### Code
https://github.com/fall20style/docker_prep/tree/main/wpa_supplicant_test
