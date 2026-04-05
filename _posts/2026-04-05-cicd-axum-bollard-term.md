
## CI/CD 파이프라인 상세 설계 (Sub-directory 기반)

### 1. 개요 및 목적

* 07_axum_bollard_term 폴더 내 Rust 소스 자동 빌드 및 배포 시스템임.
* Docker 이미지 생성 후 GitHub Container Registry(GHCR)에 업로드하는 것이 최종 목표임.

### 2. GitHub Actions 워크플로우 코드 (ci-cd.yml)
프로젝트 루트의 .github/workflows/ci-cd.yml에 작성함.

``` yml
name: Subdir Rust Docker CI/CD
on:
  push:
    branches: [ "main" ]
    paths:
      - '07_axum_bollard_term/**' # 1. 특정 폴더 변경 시에만 실행함
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  WORKING_DIRECTORY: 07_axum_bollard_term # 2. 서브 디렉터리 경로 변수화함
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    defaults:
      run:
        working-directory: ${{ env.WORKING_DIRECTORY }} # 3. 모든 명령어를 해당 폴더 내에서 실행함

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # 4. 의존성 라이브러리 캐싱으로 빌드 시간 대폭 단축함
      - name: Set up Rust cache
        uses: actions/cache@v3
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ${{ env.WORKING_DIRECTORY }}/target/
            key: ${{ runner.os }}-cargo-${{ hashFiles(format('{0}/Cargo.lock', env.WORKING_DIRECTORY)) }}

      - name: Build Binary
        run: cargo build --release

      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # 5. 빌드 컨텍스트를 서브 디렉터리로 지정하여 도커 이미지 제작함
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ${{ env.WORKING_DIRECTORY }}
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

### 3. 핵심 설정 항목 설명

* Selective Trigger (paths): 전체 레포지토리가 아닌 07_axum_bollard_term 폴더 내 파일이 바뀔 때만 빌드가 시작됨. 불필요한 서버 자원 낭비 방지함.
* Working Directory 설정: defaults.run을 통해 cargo build 등의 명령어가 서브 디렉터리 내부에서 실행되도록 강제함. 경로 에러 원천 차단함.
* Rust Cache Strategy: Cargo.lock 파일의 해시값을 키로 사용함. 라이브러리 구성이 바뀌지 않으면 이전 빌드 결과물을 재사용하여 속도가 매우 빠름.
* Docker Build Context: 도커 빌드 시 파일 복사(COPY)의 기준점을 서브 디렉터리로 고정함. Dockerfile 내의 경로 작성 편의성 제공함.

### 4. 최종 산출물 활용

* 저장소: GitHub 레포지토리의 Packages 탭에 저장됨.
* 실행: 서버에서 아래 명령으로 즉시 가동 가능함. 호스트 도커 소켓 공유 설정이 핵심임.

```
docker run -d -p 3000:3000 -v /var/run/docker.sock:/var/run/docker.sock <이미지_주소>
```

