---
title: "LazyVim에서 TypeScript용 LSP를 설정하는 방법"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - nvim
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: "LazyVim에서 TypeScript용 LSP를 설정하는 방법"
---

## LazyExtras 사용
LazyVim 내장 기능을 활용하는 방식.

* Neovim 실행 후 `:LazyExtras` 입력함.
* lang.typescript 항목을 찾아 x 키를 눌러 활성화함.
* LSP(vtsls 또는 tsserver), 포맷터(prettier), 린터(eslint)가 한 번에 세팅됨.

* 서버 종류: 최근 LazyVim은 기존 tsserver 대신 성능이 개선된 vtsls를 기본으로 채택함.
* 동작 확인: 설정 후 :LspInfo를 쳐서 vtsls가 실행 중인지 확인함.
* 포맷팅: lang.typescript 엑스트라를 켜면 Prettier 설정도 같이 따라옴.


