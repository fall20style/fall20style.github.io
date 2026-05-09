---
title: "Emacs 설정 및 업그레이드 (26.3 -> 30.0)"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - skills
  - gemini
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "Emacs 설정 및 업그레이드 (26.3 -> 30.0)"
---

## 1. Emacs 버전 업그레이드

* 현재 상태: GNU Emacs 26.3 사용 중임.
* 업그레이드 필요성: Tree-sitter, Native Comp 등 최신 기능을 활용하기 위해 29.x 버전 권장함.
* 방법:

```
* Linux: PPA(ppa:ubuntuhandbook1/emacs) 추가 후 apt install 수행함.
   * macOS: Homebrew 사용(brew install emacs-plus@29)함.
```

## 2. 패키지 저장소(MELPA) 설정

* 목적: Markdown-mode 등 외부 확장을 설치하기 위함임.
* 설정: init.el 상단에 아래 코드를 추가함.

```
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org") t)
(package-initialize)
```

## 3. Markdown 모드 설치 및 설정

* 패키지 이름: markdown-mode임.
* 설치 방법: M-x package-install -> markdown-mode 입력함.
* 주요 설정:
* .md 파일 오픈 시 자동 실행되도록 설정함.
   * 코드 블록(Code Block) 내 문법 강조를 위해 `markdown-fontify-code-blocks-natively`를 t로 설정함.

## 4. 에러 해결 (ispell-menu-map-needed)

* 원인: Emacs 업그레이드 후 오래된 패키지가 삭제된 내부 변수를 참조하여 발생함.
* 조치 방법:
* M-x package-list-packages에서 패키지 전체 업데이트 수행함.
   * 해결 안 될 경우 init.el에 `(setq ispell-menu-map-needed nil)` 추가함.




