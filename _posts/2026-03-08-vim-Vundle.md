---
title: Vim Vundle로 플러그인 설치
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Git
  - Vim
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### vim Vundle 설치
```
cd ~/.vim
git clone https://github.com/VundleVim/Vundle.vim.git
mkdir bundle
mv Vundle.vim/ bundle/
```

### vim Vundle로 플러그인 설치

vi 켜고 다음 명령을 입력하기
```
:PluginUpdate
```
