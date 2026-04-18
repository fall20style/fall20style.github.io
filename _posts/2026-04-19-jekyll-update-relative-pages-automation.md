---
title: Related Post를 자동으로 생성
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Gemini
  - Jekyll
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### gemini CLI를 사용
- post 아래의 page에 Related Posts를 생성해준다.

### AGENT.md 파일

``` markdown
## Requirement

1. `_post` dir has a lot of markdown pages.
2. I hope to create hyperlinks among them.



### Link example
this is an example.
copy the way it uses.


* Foramt
[Name]({% link _post/yyyy-mm-dd-word1-...-wordN.md %})

* Example
[HelloWorld]({% link _posts/2023-10-27-my-post.md %})


### Note
-. this `._post` directory has Jekyll based posts in .md

```

