---
title: Modern C++ CP 골격 코드
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Modern C++
  - CP
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


## 1. 기본 템플릿
fastio는 필수.
bits/stdc++.h로 한 번에 해결.

```c++
#include <bits/stdc++.h>
using namespace std;
#define fastio ios_base::sync_with_stdio(0); cin.tie(0);
#define all(v) (v).begin(), (v).end()
typedef long long ll;

void solve() {
    // 알고리즘 본체
}
int main() {
    fastio;
    int t = 1; 
    // cin >> t; // 테케 여러 개일 때만 활성화
    while(t--) solve();
}
```

## 2. 모던 C++ 팁

* Structured Bindings (C++17): pair나 tuple 분해할 때 auto [a, b] = p; 처럼 쓰면 가독성 좋음.
* Lambda (C++11): sort 커스텀 비교나 함수 안의 재귀 만들 때 좋음.
* Range-based for: for(auto &x : v)로 컨테이너 순회 쉬움.
* emplace_back: 객체 생성 비용 줄여줘서 push_back보다 효율적.
* std::gcd, lcm (C++17): <numeric>에 이미 있음.

## 3. 주의사항
* int 범위 초과 방지를 위해 웬만하면 long long 쓰는 게 Good.
* endl 대신 '\n' 써야 시간 초과(TLE) 안 남.


## Related Posts
- [push_back보다 emplace_back이 효율적인 이유]({% link _posts/2026-04-01-emplace_back-effective.md %})
- [C++17 Structured Bindings의 활용]({% link _posts/2026-04-01-Structured_Bindings.md %})
- [CMake로 FakeIt 사용 (의존성 주입)]({% link _posts/2026-04-02-cmake-fakeit-di.md %})
- [Mock Object 패턴 (FakeIt 사용)]({% link _posts/2026-04-02-mock-object-with-fakeit.md %})
- [Protobuf를 활용한 Single Source of Truth(SSOT)]({% link _posts/2026-04-15-ssot-cpp-protobuf.md %})
