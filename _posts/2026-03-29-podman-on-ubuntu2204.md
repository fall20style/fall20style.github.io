## Ubuntu 20.04 Podman 설치하기

### Kubic 프로젝트 저장소 추가

```
echo "deb https://download.opensuse.org{VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
```

### GPG 보안 키 등록
패키지 변조 방지를 위해 인증 키를 추가함.
```
curl -L "https://download.opensuse.org{VERSION_ID}/Release.key" | sudo apt-key add -
```

### 패키지 목록 업데이트 및 설치
저장소 정보를 갱신하고 Podman 설치.

```
sudo apt-get update
sudo apt-get -y install podman
```

### 설치 확인
버전 정보를 출력하여 정상 설치 여부를 확인함.

```
podman --version
```
