---
title: "객체 복사 금지와 그 이유"
date : "2025-12-04 14:00:00 +0900"
last_modified_at: "2025-12-04T14:00:00"
categories:
  - C++
tags:
  - C++
  - 복사 생성자  
---

## 객체의 복사 금지 방법들

- 복사 생성자 / '=' 연산자 오버로딩에 대한 `= delete` 처리<br>
  - delete 는 new 와 같이 사용하는 '메모리 해제' 연산자로서의 기능도 있으나<br>
    함수 뒤에 '= delete'를 통해 원치 않은 함수를 삭제할 수 있음<br>
    (C++ 11)<br>
    - 컴파일러의 원치 않은 특정 함수들을 만드는 것을 예방함<br>
    - 해당 함수 사용 시도시, 컴파일 에러 발생<br>
      ('삭제된 함수' 사용 시도)<br>

```cpp
class FileHandle {
public:
    FileHandle(const char* path);
    ~FileHandle();

    FileHandle(const FileHandle&) = delete;            // 복사 생성 금지
    FileHandle& operator=(const FileHandle&) = delete; // 복사 대입 금지

    // 필요하다면 move만 허용
    FileHandle(FileHandle&&) noexcept;
    FileHandle& operator=(FileHandle&&) noexcept;
};
```

- 복사 생성자/대입에 private 처리하기<br>
  - C++ 11 이전에 사용 가능한 방식<br>
    외부에서 접근하려 하면 컴파일 에러 발생<br>
    ('private 접근' 시도)<br>

```cpp
class NonCopy {
public:
    NonCopy() = default;
private:
    NonCopy(const NonCopy&);            // 선언만 하고
    NonCopy& operator=(const NonCopy&); // 정의는 안 함
};
```

- Unique_ptr 같은 '복사 불가' 관련 멤버를 포함하기<br>
  - 컴파일러가 해당 클래스의 복사 연산들을 제거<br>

```cpp
class Owner {
    std::unique_ptr<int> Ptr; // unique_ptr 자체가 복사 불가

public:
    Owner() : Ptr(std::make_unique<int>(10)) {}
    // 별도 조치 안 해도 Owner는 복사 불가 타입이 됨
};
```

## 객체 복사를 막는 이유

- 소유권(OwnerShip) & 자원 관리<br>
- 안정성 과 성능<br>

### 이유 1. 소유권 (OwnerShip) & 자원

- '하나의 객체'만이 가져야 하는,<br>
   한 객체만 존재해야 하는 상황<br>
  - Thread 와 동기화 도구 쪽이 대표적인 예시<br>
  - 디자인 패턴 쪽에서는 '싱글톤'도 비슷한 경우가 될 수 있음<br>

- 이러한 경우는<br>
  객체의 '복사 생성/대입' 가능성을 막아<br>
  '소유권' 개념을 명확히!<br>

- 이전에 배웠던 RAII 와 관련됨<br>
  - '한 객체가 오로지 소유하는 것'이란 부분에서<br>
    `소유권`과 연관됨<br>
  - [RTTI? RAII?](https://hnjog.github.io/c++/RTTI_RAII/)<br>


### 이유 2. 안정성 과 성능

**안정성**<br>
- '얕은 복사' 문제를 해결<br>
   [깊은/얕은 복사?](https://hnjog.github.io/c++/C++_Pointers/#%EA%B9%8A%EC%9D%80%EC%96%95%EC%9D%80-%EB%B3%B5%EC%82%AC)<br>

- 단순한 생포인터를 그대로 복사하는 경우<br>
  서로 같은 메모리를 가리키기에<br>
  둘 중 하나만 delete 하더라도<br>
  남은 쪽이 댕글리 포인터가 되거나<br>
  중복 delete 문제가 발생<br>

- 그렇기에 Unique_Ptr 등으로 복사를 막거나<br>
  Shared_Ptr 사용을 권장하여 '참조'하도록 함<br>

**성능**
- 지나치게 큰 배열 이나 큰 크기의 리소스는<br>
  '값을 복사'하는 경우, 연산량과 메모리 소모를 증가<br>
  - 특히 '지역변수'로 먼저 만들고 복사하는 경우<br>
    이러한 문제가 두드러짐<br>

- 아예 복사를 막거나<br>
  Move를 통해 이미 만든 개체의 소유권을 이동시켜<br>
  성능을 개선할 수 있음<br>
