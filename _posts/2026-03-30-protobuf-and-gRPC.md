## Protobuf와 gRPC의 관계

protoc (Protocol Buffer Compiler)가 .proto 파일을 읽어서 통신에 필요한 복잡한 로직을 담은 파이썬 코드를 자동으로 생성해 줌.
개발자는 생성된 코드를 가져다 쓰기만 하면 됨. 구체적인 과정은 다음과 같음.

### 1. protoc가 생성해주는 두 종류의 파일
파이썬의 경우, grpcio-tools를 통해 다음 두 파일을 얻게 됨.

   1. _pb2.py (메시지 파일):
   * 데이터 구조(클래스)가 정의되어 있음.
      * 데이터를 이진(Binary)으로 바꾸고 다시 읽는 직렬화/역직렬화 로직이 들어있음.
   2. _pb2_grpc.py (서비스 파일):
   * 클라이언트용(Stub): 서버에 있는 함수를 마치 내 컴퓨터에 있는 함수처럼 호출하게 해주는 코드.
      * 서버용(Servicer): 서버 기능을 구현할 때 상속받아야 하는 베이스 클래스 코드.
   
------------------------------
### 2. 생성 및 사용 흐름 (요약)

① .proto 정의 (설계도)

```
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

② protoc 실행 (공장 가동)

```
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. chat.proto
```

③ 생성된 코드 활용 (조립)

* 서버: _pb2_grpc.GreeterServicer를 상속받아 SayHello 함수 내용만 채움.
* 클라이언트: _pb2_grpc.GreeterStub을 생성하여 SayHello()를 호출함.

------------------------------
### 3. 이렇게 하는 이유 (효과)

* 노가다 방지: HTTP/2 연결 관리, 헤더 압축, 데이터 파싱 코드를 직접 짤 필요가 없음.
* 언어 간 호환성: 똑같은 .proto 파일로 Java 서버와 Python 클라이언트를 만들 수 있음.
  - 각 언어용 protoc가 해당 언어에 맞는 코드를 알아서 뽑아주기 때문임.
* 타입 체크: IDE(PyCharm, VS Code 등)에서 자동 완성이 지원되어 오타나 타입 실수를 사전에 방지함.

결론: protoc는 gRPC 통신의 "핵심 엔진과 배선"을 자동으로 깔아주는 도구임.


