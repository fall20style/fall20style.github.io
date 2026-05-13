
# Linux 환경에서 GCC 14를 사용하여 gRPC를 빌드하는 방법


gRPC는 현재 C++17 이상의 표준을 요구하며, GCC 14는 이를 완벽하게 지원합니다.

가장 권장되는 CMake 방식과 Bazel 방식 두 가지를 모두 설명합니다.



## Linux에서 GCC 14를 이용한 gRPC 빌드 가이드

## 1. 사전 필수 도구 설치
먼저 시스템에 GCC 14와 빌드에 필요한 도구들이 설치되어 있어야 합니다. (Ubuntu/Debian 기준)

```
sudo apt-get update
sudo apt-get install -y build-essential autoconf libtool pkg-config cmake git
```

# GCC 14 및 G++ 14 설치

```
sudo apt-get install -y gcc-14 g++-14
```

## 2. 소스 코드 준비
  gRPC 저장소를 클론하고 모든 서브모듈(Abseil, Protobuf 등)을 함께 가져옵니다.

```
git clone --recursive https://github.com/grpc/grpc
cd grpc
```

- 최신 안정화 버전을 사용하려면 태그를 체크아웃하세요 (예: v1.70.x)

```
git checkout v1.70.0
git submodule update --init --recursive
```

## 3. CMake를 이용한 빌드 (권장 방식)

CMake 실행 시 컴파일러를 gcc-14와 g++-14로 명시적으로 지정합니다.

```
# 빌드 디렉토리 생성 및 이동
mkdir -p cmake/build
cd cmake/build
```


# 컴파일러 지정 및 C++ 표준 설정

(gRPC 1.70Linux에서 GCC 14를 사용하여 gRPC를 빌드하는 방법을 정리한 가이드입니다.
이 가이드는 **CMake**와 **Bazel** 두 가지 주요 빌드 시스템을 모두 다룹니다.

```
9 # Linux에서 GCC 14를 이용한 gRPC 빌드 가이드
      10
      11 이 문서는 Linux 환경에서 최신 GCC 14 컴파일러를 사용하여 gRPC C++ 라이브러리를 소스에서 빌드하는 방법을 설명합니다.
      12
      13 ## 1. 필수 구성 요소 설치
      14
      15 먼저 GCC 14와 필요한 빌드 도구들을 설치해야 합니다. (Ubuntu/Debian 기준)
      시스템 업데이트
      sudo apt-get update

      GCC 14 설치
      sudo apt-get install -y gcc-14 g++-14

      기타 필수 빌드 도구 설치
      sudo apt-get install -y build-essential autoconf libtool pkg-config cmake git
```


## 2. 소스 코드 가져오기

gRPC는 많은 외부 라이브러리를 서브모듈로 사용하므로 `--recursive` 플래그를 사용하여 클론해야 합니다.
  git clone --recursive -b v1.70.0 https://github.com/grpc/grpc
  cd grpc
  이미 클론했다면 서브모듈 업데이트
  git submodule update --init --recursive

  1 *(참고: `v1.70.0` 이상의 버전은 C++17을 최소 사양으로 요구하며, GCC 14와 잘 호환됩니다.)*
  2
  3 ---
  4
## 3. CMake를 이용한 빌드
  6
  7 CMake는 환경 변수나 옵션을 통해 컴파일러를 지정하기 쉽습니다.
  8

### 환경 변수 설정
  10 터미널 세션에서 GCC 14를 기본 컴파일러로 인식하도록 설정합니다.
  export CC=gcc-14
  export CXX=g++-14
  1

### 빌드 및 설치
  mkdir -p cmake/build
  cd cmake/build

CMake 구성 (C++17 표준 강제)
  cmake -DCMAKE_BUILD_TYPE=Release \
                           -DCMAKE_CXX_STANDARD=17 \
                           -DCMAKE_C_COMPILER=gcc-14 \
                           -DCMAKE_CXX_COMPILER=g++-14 \
                           -DgRPC_INSTALL=ON \
                           -DgRPC_BUILD_TESTS=OFF \
                           ../..

  빌드 (코어 수에 맞춰 병렬 빌드)
make -j$(nproc)

설치 (선택 사항)
  sudo make install

## 4. Bazel을 이용한 빌드 (권장)

gRPC 개발팀이 주로 사용하는 방식입니다. Bazel은 자체적인 샌드박스 환경에서 빌드하므로 컴파일러 지정 시 주의가 필요합니다.

### 환경 변수 전달 빌드

Bazel이 설치된 환경에서 다음 명령어를 사용합니다.

GCC 14를 사용하도록 환경 변수 전달
CC=gcc-14 CXX=g++-14 bazel build --cxxopt="-std=c++17" :all

또는 `--repo_env` 옵션을 사용하여 명시적으로 전달할 수 있습니다.

```
bazel build --repo_env=CC=gcc-14 --repo_env=CXX=g++-14 :all
```

### Bazel 7 이상 사용 시 주의사항

현재 gRPC는 Bazel의 새로운 모듈 시스템인 `bzlmod`와 완전한 호환이 되지 않을 수 있습니다. 빌드 에러 발생 시 다음 플래그를
  추가하세요.
  bazel build --enable_bzlmod=false :all


## 5. GCC 14 빌드 시 주의사항 및 팁

1.  **Strict Warnings**: GCC 14는 이전 버전보다 경고(Warning) 처리가 엄격합니다. 만약 경고가 에러로 처리되어 빌드가
  중단된다면, CMake 구성 시 `-Wno-error` 관련 플래그를 추가하거나 소스 코드를 수정해야 할 수 있습니다.
2.  **C++ 표준**: gRPC 1.70 버전부터는 최소 **C++17**이 필요합니다. GCC 14는 기본적으로 C++17 이상을 지원하므로 문제가
  없으나, CMake 사용 시 `-DCMAKE_CXX_STANDARD=17`을 명시하는 것이 안전합니다.
3.  **의존성 라이브러리**: Abseil 등 내부 의존성 라이브러리들도 동일한 GCC 14 컴파일러와 C++ 표준으로 빌드되어야 링크
  에러를 방지할 수 있습니다. 위와 같이 gRPC 빌드 시 서브모듈을 포함하여 빌드하면 자동으로 해결됩니다.

## 6. 빌드 확인

빌드가 완료된 후 예제 코드를 통해 정상 작동 여부를 확인할 수 있습니다.

cd examples/cpp/helloworld
  mkdir -p cmake/build
  cd cmake/build
  cmake -DCMAKE_PREFIX_PATH=/path/to/installed/grpc \
                            -DCMAKE_C_COMPILER=gcc-14 \
                            -DCMAKE_CXX_COMPILER=g++-14 \
                            ..
                            make

