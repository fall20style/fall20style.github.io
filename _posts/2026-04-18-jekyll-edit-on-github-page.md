---
title: Minimal Mistakes 테마에 GitHub 수정 링크를 추가
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Jekyll
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

## 1. _config.yml 설정

* repository: "계정명/저장소명" 설정 확인 필요함.
* git_branch: main (또는 master) 추가 권장함.

## 2. 코드 구현 (수정 위치 선택)## 옵션 A: 포스트 하단에 추가

* `_includes/post_pagination.html` 파일 수정함.
* 적절한 위치에 아래 코드 삽입함.

``` html
<p>
  <a href="https://github.com{{ site.repository }}/edit/{{ site.git_branch | default: 'main' }}/{{ page.path }}" 
     class="btn btn--light-outline">
     <i class="fas fa-fw fa-edit"></i> Edit on GitHub
  </a>
</p>
```
- master 대신 main으로 변경

## 옵션 B: 사이드바에 추가

* `_includes/sidebar.html` 파일 수정함.
* 저자 프로필 하단에 아래 코드 삽입함.

``` html
<a href="https://github.com{{ site.repository }}/edit/{{ site.git_branch | default: 'main' }}/{{ page.path }}" 
   class="btn btn--info btn--small">
   <i class="fab fa-fw fa-github"></i> Edit on GitHub
</a>
```
- master 대신 main으로 변경

## 핵심 포인트

* `page.path`: 현재 문서의 경로를 자동으로 매칭함.
* `site.repository`: 저장소 경로를 동적으로 가져옴.
* `Font Awesome`: 테마 내장 아이콘 활용 가능함.

