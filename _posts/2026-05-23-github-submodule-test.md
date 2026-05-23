---
title: "github submodule 테스트"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - "github submodule"
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "github submodule 테스트"
---


## 연습 위치

- https://github.com/fall20style/parent1
- https://github.com/fall20style/child1
- https://github.com/fall20style/child2

### git을 clone하기 parent1, child1, child2
```
PS C:\Users\fall2\work\gitdir> git clone git@github.com:fall20style/parent1.git
Cloning into 'parent1'...
PS C:\Users\fall2\work\gitdir> git clone git@github.com:fall20style/child1.git
Cloning into 'child1'...
PS C:\Users\fall2\work\gitdir> git clone git@github.com:fall20style/child2.git
Cloning into 'child2'...
```

- 3개의 git을 clone하고 README.md 에 노트를 함

### child1, child2를 git add submodule하기

```
PS C:\Users\fall2\work\gitdir\parent1> git submodule add git@github.com:fall20style/child1.git
Cloning into 'C:/Users/fall2/work/gitdir/parent1/child1'...
PS C:\Users\fall2\work\gitdir\parent1> git submodule add git@github.com:fall20style/child2.git
Cloning into 'C:/Users/fall2/work/gitdir/parent1/child2'...
```

- parent1에서 child1, child2를 git submodule로 add

### .gitmodules 확인하기

```
PS C:\Users\fall2\work\gitdir\parent1> cat .\.gitmodules
[submodule "child1"]
        path = child1
        url = git@github.com:fall20style/child1.git
[submodule "child2"]
        path = child2
        url = git@github.com:fall20style/child2.git
PS C:\Users\fall2\work\gitdir\parent1>
```

- .gitmodule 파일에 위치가 등록되어 있음

### 파일 내용을 출력해 보기
```
PS C:\Users\fall2\work\gitdir\parent1> Get-Content .\README.md
# parent1
PS C:\Users\fall2\work\gitdir\parent1> Get-Content .\child1\README.md
# child1
PS C:\Users\fall2\work\gitdir\parent1> Get-Content .\child2\README.md
# child2
```

### child1에서 파일 내용변경

- git add / git commit / git push로 변경을 반영함

### parent에서 git pull하면 가져옴

```
git pull
```

### parent에서 submodule을 가져오기
```
git submodule update --init --recursive
```


----

## Git Submodule 업데이트 방법

외부에서 `child2` 저장소에 기능이 추가되었을 때, `parent1`에서 이를 반영하는 방법임.

### 1. 특정 서브모듈만 원격 최신 상태로 업데이트함
`parent1` 루트 디렉토리에서 아래 명령어를 실행하여 `child2`를 최신 커밋으로 업데이트함.
```bash
git submodule update --remote child2
```

### 2. 서브모듈 디렉토리에서 직접 가져옴
서브모듈 경로로 이동하여 일반적인 `git pull`을 수행하는 방식임.
```bash
cd child2
git pull origin main
cd ..
```

### 3. 부모 프로젝트에 변경 사항 반영함
서브모듈의 지점이 변경되었으므로, `parent1`에서도 이를 커밋하고 푸시해야 함.
```bash
git add child2
git commit -m "Update child2 to latest version"
git push origin main
```

### 참고사항
모든 서브모듈을 한꺼번에 최신 상태로 업데이트하려면 아래 명령어를 사용함.
```bash
git submodule update --remote
```


