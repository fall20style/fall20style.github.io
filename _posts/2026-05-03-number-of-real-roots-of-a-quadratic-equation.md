---
title: "이차방정식의 실근 개수를 판단하는 판별식"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
use_math: true 
tags:
  - AI
toc: true
toc_sticky: true
toc_label: 목차
description: "이차방정식의 실근 개수를 판단하는 판별식"
---

## 이차방정식과 판별식

이차방정식 $ax^2 + bx + c = 0$ ($a \neq 0$)에서 근의 공식은 다음과 같음.


$$x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$ 

이때 루트 안에 있는 식 $b^2 - 4ac$를 판별식(Discriminant)이라 하며, 보통 $D$로 나타냄. [2] 


## 판별식 $D$의 값에 따른 근의 종류

판별식의 부호에 따라 실근의 개수와 종류가 결정됨. 

* $D > 0$ 인 경우: 서로 다른 두 실근을 가짐.
* $D = 0$ 인 경우: 중근(서로 같은 두 실근)을 가짐.
* $D < 0$ 인 경우: 서로 다른 두 허근을 가짐 (실근은 없음).



## 짝수 판별식

이차항의 계수 $b$가 짝수($b = 2b'$)일 때, 계산을 간편하게 하기 위해 $D/4$를 사용함.

$$D/4 = (b')^2 - ac$$ 

이 공식 또한 $D$와 동일한 기준으로 근의 종류를 판별함. 
