---
title: Python 3.14.0 + pyenv local + venv + 머신러닝 패키지 설치
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
  - requirements.txt
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


### 1. 버전 확인 및 로컬 설정
먼저 설치된 버전을 확인하고 해당 디렉토리의 파이썬 버전을 고정함.

* 설치된 리스트 확인 (3.14.0이 없으면 pyenv install 3.14.0 선행)

```
pyenv versions
```

* 현재 디렉토리에 3.14.0 적용

```
pyenv local 3.14.0
```
* 버전 적용 확인

```
python --version
```

### 2. venv 가상환경 생성 및 활성화
프로젝트 전용 가상환경을 만들고 활성화함.

* 가상환경 생성 (이름: my_local_py_3.14.0)
```
python -m venv my_local_py_3.14.0
```
* 가상환경 활성화 (macOS/Linux 기준)
```
source my_local_py_3.14.0/bin/activate
```
### 3. 머신러닝 패키지 설치
requirements.txt 파일을 작성한 후, pip를 통해 한꺼번에 설치함.

```
numpy
pandas
scipy
matplotlib
seaborn
scikit-learn
xgboost
lightgbm
catboost
jupyterlab
```

* 설치 명령어

* 패키지 일괄 설치
```
pip install -r requirements.txt
```
* 설치된 리스트 확인
```
pip list
```

## Related Posts
- [Ubuntu 20.04에서 python 3.14.3 설치]({% link _posts/2026-03-29-python3.14.3-on-ubuntu2004.md %})
- [Python 3.14.3 venv 구축 및 fastmcp 패키지 설치]({% link _posts/2026-03-29-python3.14.3-pyenv-fastmcp.md %})
- [Python 3.15-dev 및 fastmcp 환경 구축]({% link _posts/2026-03-29-python3.15dev-pyenv-fastmcp.md %})
- [pyenv + venv 설정]({% link _posts/2026-04-09-pyenv-venv-setting.md %})
- [Ubuntu 22.04에서 IoTivity 개발을 위한 Python 2.7 환경 구축 (pyenv & Docker)]({% link _posts/2026-04-11-python2.7-pyenv-scons-iotivity.md %})

