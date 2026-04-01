# Structured Bindings

C++17부터 도입된 Structured Bindings는 pair, tuple, 구조체, 배열의 값을 한 번에 여러 변수로 분해해서 받는 문법.

## 1. 기본 문법
auto [a, b, c] = 대상; 형태로 작성. 각 변수가 알아서 타입을 추론.

## 2. 주요 활용 케이스

* pair / tuple 분해 (가장 많이 씀)

```
pair<int, string> p = {1, "hello"};
auto [id, msg] = p; // id = 1, msg = "hello"
```

* map 순회할 때 (가독성)

```
map<int, int> m;
for (auto const& [key, val] : m) {
    // m.first, m.second 대신 key, val로 바로 접근 가능
}
```

* 구조체(struct) 분해

```
struct Point { int x, y; };
Point p = {10, 20};
auto [px, py] = p; // px = 10, py = 20
```

## 3. 장점

* p.first, get<0>(t) 처럼 복잡하게 안 써도 됨.
* 변수 이름을 직관적으로 지을 수 있어서 코드가 읽기 편해짐.
* const auto& [a, b] 처럼 써서 복사 비용도 아낄 수 있음.
