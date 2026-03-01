---
title: Windows용 Emacs 설치 환경설정
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


### tango-dark theme 선택
```
ctrl-e load-theme <RET>
tango-dark <RET>
```

``` elisp
(load-theme 'tango-dark t)
```

### 이맥스 windows에서 설치하기
- https://ftp.gnu.org/gnu/emacs/windows/emacs-30/

### Windows config 파일 위치
```
C:\Users\fall2\AppData\Roaming\.emacs.d\init.el
```

### 이맥스 script 실행하기
``` elisp
(if nil
  (message "true");;여기 닫는 괄호에서 실행하면 "true"가 출력됩니다.
  (message "false")
);; 이 식은 실행하면 "false"만 출력합니다.
```
- ctrx-X ctrl-E 를 true 뒤에서 누르면 실행된다.

### Inconsolata font 지정하기
``` elisp
; (set-face-attribute 'default nil :font "Consolas") ; 영문 폰트
(set-face-attribute 'default nil :font "Inconsolata") ; 영문 폰트
(set-fontset-font "fontset-default" 'hangul (font-spec :family "NanumGothicCoding" :size 16)) ; 한글 폰트
#+end_src
```
