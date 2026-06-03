
### grpc를 ubuntu 22.04에 설치하기

```
# GCC 11 이상 및 최신 G++ 설치
sudo apt update
sudo apt install gcc-11 g++-11

# 다시 gRPC 빌드 디렉토리로 이동하여 캐시 삭제
cd ~/work/github/grpc/cmake/build
rm -rf *

# 새 컴파일러를 명시하여 CMake 다시 실행
cmake -DCMAKE_C_COMPILER=gcc-11 -DCMAKE_CXX_COMPILER=g++-11 -DCMAKE_CXX_STANDARD=17 ../..

# root dir에서 실행
cmake --build .
```
