---
title: pyenv + venv 설정
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - pyenv
  - venv
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---



## 1. 개요

* pyenv: 특정 파이썬 버전 자체를 설치하고 관리함.
* venv: 설치된 파이썬 버전 내에서 프로젝트별 라이브러리를 격리함.

## 2. 세팅 순서

### 파이썬 버전 설치 (pyenv)
원하는 파이썬 버전을 시스템에 설치함.

```
pyenv install 3.11.5
```

### 프로젝트 폴더 생성 및 이동
새 프로젝트를 위한 디렉토리를 만들고 진입함.

```
mkdir my-app && cd my-app
```

### 해당 폴더의 파이썬 버전 고정 (pyenv local)
해당 폴더에서만 특정 버전을 사용하도록 설정함.

```
pyenv local 3.11.5
```

### 가상환경 생성 (venv)
현재 설정된 파이썬(3.11.5)을 기반으로 독립된 가상환경(venv)을 만듦.

```
python -m venv .venv
```

## 가상환경 활성화
가상환경을 켜서 라이브러리 설치 준비를 마침.

# Linux

```
source .venv/bin/activate
```

## 3. 주요 확인 사항

* which python 입력 시 프로젝트 폴더 내 .venv/bin/python 경로가 나오면 성공임.
* 가상환경 종료는 deactivate 명령어를 사용함.
* 프로젝트 루트의 .python-version 파일은 pyenv가 버전을 기억하는 용도임.

.
의존성 패키지 목록을 requirements.txt로 관리하는 방법도 알려줄까?

## Related Posts
- [Ubuntu 20.04에서 python 3.14.3 설치]({% link _posts/2026-03-29-python3.14.3-on-ubuntu2004.md %})
- [Python 3.14.3 venv 구축 및 fastmcp 패키지 설치]({% link _posts/2026-03-29-python3.14.3-pyenv-fastmcp.md %})
- [Python 3.15-dev 및 fastmcp 환경 구축]({% link _posts/2026-03-29-python3.15dev-pyenv-fastmcp.md %})
- [Python 3.14.0 + pyenv local + venv + 머신러닝 패키지 설치]({% link _posts/2026-04-10-pyenv-local-venv-requirements.md %})
- [Ubuntu 22.04에서 IoTivity 개발을 위한 Python 2.7 환경 구축 (pyenv & Docker)]({% link _posts/2026-04-11-python2.7-pyenv-scons-iotivity.md %})


