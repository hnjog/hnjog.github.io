---
title: "Lambda"
date : "2026-01-26 14:00:00 +0900"
last_modified_at: "2026-01-26T14:00:00"
categories:
  - C++
  - Lambda
tags:
  - C++ 11
  - Lambda
  - 람다식
---

## Lambda?

'이름 없는 함수'에 대한 일종의 표현식으로<br>
'익명 타입의 객체'(Closure)로 만들어지며<br>
해당 객체가 `operator()`를 통해 함수처럼 호출<br>

- C++ 11 부터 도입<br>
- '즉석'에서 만들어 사용하는 함수<br>
  (위에서 말하였듯, 정확히는 런타임 중 생성되는 객체)<br>

- 람다의 '타입'은 이름 없는 클래스 타입! (Closure)<br>

- 람다 표현식의 결과는 prvalue<br>
  - prvalue?<br>
    : Pure rvalue 로서<br>
      rvalue 중 '순수한 값'에 가까움<br>
      (리터럴 , x + y 같은 연산 결과, T{} 같은 임시객체 생성)<br>

## 람다의 특징

- 각 람다마다 'Closure' 타입이 생성<br>
  - 캡쳐의 여부에 따라 생성/대입 규칙이 달라질 수 있음<br>

- 호출 동작은 `operator()`를 통해<br>
  - 람다 호출 시, Closure object의 operator() 가 실행<br>
    - 값 캡쳐(copy)시엔 '캡처'된 복사본에 접근<br>
    - 참조 캡쳐(ref)시엔 '원본'에 접근<br>

- 캡처 없는 람다는 '함수 포인터'로 변환 가능함<br>

- '댕글링' 참조에 유의할 것<br>
  - [&] 캡쳐로 가져온 대상이 메모리 해제가 되어 있을 경우<br>
    '정의되지 않은 동작'이 발생 가능함<br>
    + this로 캡쳐한 경우도 '수명'에 주의할 것<br>


## 람다의 사용법

- 기본 문법<br>

```cpp
[capture](params) -> return_type { body }
```

- capture : 바깥 변수를 가져오는 방식<br>
- params : 함수 파라미터<br>
- return_type : 생략하면 자동 추론<br>
- body : 실행할 코드<br>

### 캡쳐

- 외부 변수를 어떻게 가져올지를 결정하는 방식

| 캡처        | 의미                  |
| --------- | ------------------- |
| `[]`      | 캡처 없음               |
| `[=]`     | 사용한 변수 전부 **값** 캡처  |
| `[&]`     | 사용한 변수 전부 **참조** 캡처 |
| `[x]`     | x만 값 캡처             |       
| `[&x]`    | x만 참조 캡처            |      
| `[x, &y]` | 혼합                  |         
| `[this]` | 클래스 내부 멤버 접근            |


- mutable 키워드<br>
  : '값 캡쳐'를 수정할 수 있도록 하는 키워드<br>

```cpp
int x = 5;

auto f = [x]() mutable {
    x += 10;     // 내부 복사본 변경
    return x;
};

int r = f(); // 15
// 바깥 x는 여전히 5
```

- init-capture<br>
  : 캡쳐 하면서 이름/값을 새로 지정<br>
    std::move를 사용한 '이동'도 가능함!<br>

```cpp
int base = 10;

auto f = [v = base * 2](int x) {
    return v + x;  // v는 20으로 저장됨
};

---

#include <memory>
auto p = std::make_unique<int>(7);

auto g = [q = std::move(p)]() {
    return *q;
};
```

### std::function

- 람다를 '함수 포인터' 처럼 저장하거나, 다른 함수의 인자로 넘길때 사용<br>
  - 보통 auto를 사용하는 편도 많음(단순 호출/전달 에 용이)<br>
  - 특정 '콜백'용 함수로 저장 / 교체 하는 경우, 표준으로 사용 가능<br>
    - 공통 래핑용 인터페이스이기에, '간접 호출' 오버헤드 + 캡쳐가 클 경우, 힙 할당 등의<br>
      성능 이슈 발생 가능<br>

```cpp
#include <functional>

std::function<int(int, int)> func = [](int a, int b) { return a + b; };
```

## 람다의 사용 이유

- 가독성 + 편리함<br>
  : 별도의 함수를 만들지 않고 '관련 로직' 근처의 인스턴스 함수를 만들어<br>
    의도를 이해하기 쉬운 편<br>
    - 특히 sort, find 같은 stl 알고리즘과 궁합이 좋은 편<br>

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v{5, 1, 4, 2, 3};

std::sort(v.begin(), v.end(), [](int a, int b) {
    return a < b;
});

int t = 3;
auto it = std::find_if(v.begin(), v.end(), [t](int x) {
    return x >= t;
});
```

## 참고 사이트

- [https://en.cppreference.com/w/cpp/language/lambda.html]<br>