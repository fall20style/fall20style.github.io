---
title: jsonrpc로 web서버와 python-app 통신 POC
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Flask
categories:
  - 습작만들기
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## Client SW 구조 (Web Server ↔ Python Client)

### 1. 시스템 아키텍처

* Server (Web App): HTTP 엔드포인트를 열고 JSON-RPC 요청을 받아 로직을 수행함.
* Client (Python App): tkinter 기반 GUI를 통해 사용자 입력을 받고 서버에 데이터를 요청함.
* Protocol: HTTP POST 전송, 데이터는 JSON-RPC 2.0 표준 규격을 따름.


### 2. Python Client 구성

| 구분 | 컴포넌트 | 주요 역할 |
|---|---|---|
| Server | RPC Handler | method에 따라 실제 함수(더하기 등)를 실행하고 결과 반환 |
| (Web) | CORS Setting | 외부 파이썬 클라이언트의 접근을 허용하도록 설정 |
| Client | tkinter (tk) | 메인 윈도우 생성 및 텍스트 박스, 버튼 등 UI 배치 |
| (Python) | messagebox | 서버 연결 실패나 입력 오류 발생 시 팝업 알림 표시 |
| | requests | 서버 URL로 JSON 데이터를 담아 HTTP POST 요청 전송 |
| | json | 파이썬 딕셔너리를 JSON 문자열로 변환하거나 서버 응답 파싱 |


### 3. 데이터 흐름 (Data Flow)

1. 입력: 사용자가 tk.Entry에 숫자 입력 후 tk.Button 클릭.
2. 생성: json 패키지를 이용해 서버가 이해할 수 있는 RPC 객체 생성.
* `{"jsonrpc": "2.0", "method": "add", "params": [1, 2], "id": 1}`
3. 전송: requests.post()를 사용하여 Web 서버 주소로 전송.
4. 처리: Web 서버가 계산 후 결과값을 JSON 형태로 리턴.
5. 출력: Python 앱이 응답을 받아 messagebox나 라벨에 결과 표시.


### 4. Python Client 임포트 구조

``` python
import tkinter as tk           # GUI 라이브러리 (기본 창, 레이아웃)
from tkinter import messagebox # 에러/알림 팝업창
import requests                # 서버 통신용 HTTP 라이브러리
import json                    # JSON 데이터 처리
```

## Server SW 구조 (Web Server ↔ Python Client)

- Flask를 사용하여 Web 서버 역할을 수행하는 구조
- 이 구조에서 Flask는 API를 제공하는 서버
- tkinter 앱이 이를 호출하는 방식


### Web 서버 구현 (Flask)
Flask 서버는 클라이언트(Python GUI)가 보낸 JSON 데이터를 해석하고 결과값을 돌려주는 역할을 해.
## 1. 서버 코드 (web_server.py)
이전 예제에서 배운 Success 래핑 방식을 적용한 완성형 코드.

``` python
from flask import Flask, request, Responsefrom flask_cors import CORSfrom jsonrpcserver import dispatch, method, Success
app = Flask(__name__)
CORS(app) # 다른 프로세스(GUI 앱)의 접근을 허용함

@methoddef add(a, b):
    """더하기 로직 수행"""
    print(f"서버 수신: a={a}, b={b}") # 서버 로그 출력
    return Success(a + b)

@app.route("/rpc", methods=["POST"]) # 엔드포인트를 /rpc로 설정def index():
    # 클라이언트가 보낸 JSON 데이터를 처리함
    response = dispatch(request.get_data().decode())
    return Response(str(response), mimetype="application/json")
if __name__ == "__main__":
    print("Flask JSON-RPC 서버 시작 (Port: 5000)")
    app.run(port=5000)
```


### SW 구조 업데이트 (Flask 기반)

| 계층 | 구성 요소 | 역할 |
|---|---|---|
| Server Layer | Flask App | HTTP POST 요청 대기 및 라우팅 (/rpc) |
| (Web Server) | jsonrpcserver | 요청된 메서드 호출 및 JSON-RPC 표준 응답 생성 |
| Network Layer | HTTP / JSON | Content-Type: application/json으로 데이터 교환 |
| Client Layer | requests | 서버로 JSON 데이터 전송 및 응답 수신 |
| (Python App) | tkinter | 사용자 입력(Entry) 및 결과 출력(Label/Messagebox) |


### Python Client 연동 로직 (client.py)
tkinter 앱 내에서 서버를 호출 부분.

``` python
def call_server():
    url = "http://localhost:5000/rpc" # 서버 주소 및 경로
    
    # 서버에 보낼 JSON-RPC 객체
    payload = {
        "jsonrpc": "2.0",
        "method": "add",
        "params": {"a": 10, "b": 20},
        "id": 1
    }
    
    try:
        # requests 패키지를 사용해 서버 호출
        response = requests.post(url, json=payload)
        result = response.json()
        
        if "result" in result:
            print(f"서버 응답: {result['result']}")
        else:
            print(f"에러 발생: {result['error']}")
            
    except requests.exceptions.ConnectionError:
        print("서버가 꺼져 있어!")
```

### 실행 순서 (중요)

1. 서버 실행: 터미널에서 python web_server.py를 먼저 실행.
2. 클라이언트 실행: 새 터미널을 열고 python gui_app.py를 실행.
3. 동작 확인: GUI에서 숫자를 넣고 버튼을 누르면 서버 터미널에 로그가 찍히고 GUI에 결과 나옴.


만약 웹 브라우저(HTML/JS)에서도 이 서버를 같이 쓰고 싶다면, 동일한 http://localhost:5000/rpc 주소로 fetch 요청을 보내면 됨.


## curl로 호출하기

- 터미널에서 서버로 데이터를 보내는 도구
- 파이썬 코드나 웹 브라우저 없이도 서버가 잘 작동하는지 확인


### curl 명령어로 호출하기
터미널(CMD 또는 Bash)에 아래 명령어를 복사해서 붙여넣어봐.

``` bash
curl -X POST http://localhost:5000 \
     -H "Content-Type: application/json" \
     -d '{"jsonrpc": "2.0", "method": "add", "params": {"a": 10, "b": 20}, "id": 1}'
```


### 명령어 옵션 설명

* -X POST: 데이터를 보낼 때 POST 방식을 사용하겠다는 뜻임.
* -H "Content-Type: application/json": 서버에게 "내가 지금 보내는 데이터는 JSON 형식이야"라고 알려주는 헤더. (이게 없으면 서버가 데이터를 읽지 못할 수 있음)
* -d: 실제 보낼 데이터(Payload) 본문임. 우리가 약속한 JSON-RPC 규격의 객체를 그대로 적어주면 됨.

### curl을 쓰는 이유

1. 디버깅: 파이썬 GUI 앱에서 에러가 날 때, 이게 서버 문제인지 앱 문제인지 분별할 때 사용. curl로 잘 되면 서버는 정상.
2. 간편함: 복잡한 UI 코드 짤 필요 없이 명령어 한 줄로 기능을 테스트할 수 있음.
3. 자동화: 나중에 리눅스 스크립트(.sh) 같은 곳에 넣어서 자동으로 서버에 명령을 내릴 때도 유용.



### Code
- [web-py-jsonrpc-poc.zip](https://github.com/user-attachments/files/26854349/web-py-jsonrpc-poc.zip)

### 테스트 - server 플라스크

<img width="781" height="136" alt="Image" src="https://github.com/user-attachments/assets/70471bbc-7134-43a0-8034-ea142edc0baa" />

### 테스트 - python app

<img width="298" height="285" alt="Image" src="https://github.com/user-attachments/assets/3d0934d1-2242-437a-b87e-a75a003be441" />

### 테스트 - curl

``` bash
curl -X POST http://localhost:5000/      -H "Content-Type: application/json"      -d '{"jsonrpc": "2.0", "method": "add", "params": {"a": 10, "b": 20}, "id": 1}'
```

``` bash
{"jsonrpc": "2.0", "result": 30, "id": 1}
```

