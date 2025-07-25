---
title: "C++ 이것저것 2"
last_modified_at: "2025-07-25T16:00:00"
categories:
  - C++
tags:
  - 동적 메모리
  - 주소값
  - 캐스팅
  - 업/다운 캐스팅
  - 깊은/얕은 복사
  - 댕글리 포인터
  - 스마트 포인터
---

## 포인터(주소값)와 동적 메모리, 가상 메모리 영역
C++이 하드웨어와 밀접한 연관이 있는 이유들은<br>
'포인터'와 '주소값'에 대한 연산, 메모리에 대한 '직접 제어' 등이 가능한 점을 꼽는다<br>
(C#,Java,Python 등은 해당 개념을 추상화하여 지원)<br>


### 동적 메모리 할당
이전 포스팅의 ['포인터'](https://hnjog.github.io/c++/C++_Bases/)는 <br>
포인터 자체의 개념과 const pointer를 다루었으니<br>
포인터에 대한 더 연관된 개념들을 다룰 예정이다<br>

포인터를 사용하는 가장 큰 이유 중 하나는<br>
'동적 메모리 할당'을 하기 위함<br>
(가상 메모리 영역에 필요한 메모리 공간을 할당하고<br>
그 주소값을 받아 사용)<br>

먼저, 동적 메모리 할당을 사용하는 이유는 뭘까?<br>

1. 변수에 필요한 크기를 컴파일 타임에 알 수 없음<br>
  : 그렇기에 런타임중에 데이터를 유동적으로 다루려면<br>
   동적인 메모리 할당이 필요<br>
   (Vector<>가 사랑받는 이유는 편리한 동적 배열이기 때문 아닐까)<br>

2. 함수 호출 간에도 데이터를 유지 가능<br>
  : 지역 변수는 scope를 지나면 사라지지만<br>
    동적 할당한 변수는 주소값만 알면 여전히 해당 위치에 존재<br>

3. 스택의 용량을 넘어서는 배열이나 객체를 생성하기 위함<br>
   : 스택의 크기는 별도로 설정하지 않는 경우 약 2MB 가 한계<br>
    따라서 이것보다 더 큰 배열이나 객체를 생성하려면<br>
    동적 메모리 할당이 필수<br>

등의 이유가 존재한다<br>

이러한 동적 메모리 할당은 앞서 말한<br>
'가상 메모리 영역'의 'heap' 영역에 할당된다<br>

### 가상 메모리 영역 구조

```
주소 ↓ (낮음)
+-------------------------+
| Code 영역 (.text)      |  ← 실행 코드 저장
+-------------------------+
| Data 영역 (.data/.bss) |  ← 전역/정적 변수
+-------------------------+
| Heap 영역              |  ← 동적 메모리 (new/malloc)
|                         |  ↑ 위쪽으로 확장
+-------------------------+
| 미사용 영역            |  ← 가드 페이지 등
+-------------------------+
| Stack 영역             |  ← 지역 변수, 함수 콜
|                         |  ↓ 아래로 확장
+-------------------------+
주소 ↑ (높음)

```

- 이러한 '가상 메모리'는 OS가 각 프로세스에게<br>
  '독립적'인 공간을 제공하기 위한 것이다<br>
   (각 프로세스는 자신만이 0x... ~ 0x... 까지 독점한다 여김)<br>

- 실제 물리 메모리는 RAM과 매핑되어 있고<br>
  이는 Page Table과 TLB 등이 관리해준다<br>

- Heap에 실제 공간을 할당해주는 것은 OS에게 요청하여<br>
  우리가 원하는 new의 결과로 할당한 공간의 주소값을 받음<br>

가상 메모리에 대한 추가적인 내용은<br>
C++보다는 OS에 가까워지므로 이전 포스팅한 것을 남긴다<br>
[OS강의1](https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/cs/os/OSS1/) , [OS강의2](https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/cs/os/OSS2/)<br>

다시 원래 C++로 돌아오자면<br>

```
int* ptr = new int(10); // OS에게 int 크기 만큼의 동적 메모리 할당을 요청

delete ptr; // Os에게 해당 영역이 더 이상 필요하지 않음을 알림
```

new 를 통하여 메모리를 할당<br>
delete로 메모리를 반환<br>
한다는 점을 명심하자<br>
(new와 delete는 일종의 연산자이기에 '클래스의 연산자 재정의(오버로딩)'이 가능하단 점도 알아는 두자)<br>

## 캐스팅
'자료형 변환' 으로<br>
  특정 타입을 '다른 타입'으로 읽는 것을 말한다<br>
  C++에선 4가지 명시적 캐스팅 연산자를 지원한다<br>
  (static_cast, dynamic_cast,const_cast, reinterpret_cast)<br>

- static_cast<><br>
 : 정적 캐스팅(컴파일 타임에 검사)<br>
 기본 자료형의 형변환이나, '상속 관계'에서의 '업캐스팅' 같은 곳에 사용<br>
 (기본적으로 꽤 자주 사용하는 편, 컴파일 시간에만 검사하기에<br>
 자식 타입을 부모 타입으로 검사하는 업캐스팅은 문제없이 가능하나<br>
 부모 타입에서 자식 타입으로 캐스팅하는 다운 캐스팅은 불명확하다)<br>
 (다만 일부러 static_cast를 사용하여 assert나 크래시를 발생시켜<br>
 디버깅을 하는 경우도 있으니 회사의 코드 규칙을 확인할 것)<br>

- dynamic_cast<><br>
 : 동적 캐스팅(런타임 타입 확인 가능)<br>
 RTTI(Run-Time Type Information)을 사용하여 타입 검사<br>
 '다운캐스팅'을 안전하게 사용 가능하다<br>
 (포인터의 캐스팅 실패 시, nullptr을 반환<br>
 참조의 경우는 std::bad_cast 예외 발생)<br>
 해당되는 클래스에 virtual 키워드의 존재 필요<br>
 -> virtual 키워드가 있어야 컴파일러가<br>
 vtable : 가상 함수 주소 목록<br>
 vptr : 각 객체가 가리키는 테이블 포인터<br>
 RTTI 정보 구조체 : 타입 이름,계층관계<br>
 등을 생성하는데<br>
 이 중 RTTI 정보가 필요하기 때문<br>
 
  dynamic_cast를 사용하지 않는 프로젝트도 존재한다<br>
  dynamic_cast는 런타임에 RTTI를 탐색하기에 추가적인 오버헤드가 발생하며<br>
  RTTI 자체도 추가적인 메모리를 소모하는 요소가 있어서 주의가 필요<br>

- const_cast<><br>
 : const, volatile 속성을 제거하거나 추가하는데 사용<br>
  (기본적으로는 '제거'하는데 사용, 추가할땐 const 키워드를 붙여 사용하는것이 가능하므로)<br>

  기본적으로 잘 사용하지 않는 캐스팅<br>
  'const'가 붙은 이유 자체가 '수정하지 말 것'을 의미하는데<br>
  const_cast를 사용해야 한다면 애초에 const 키워드를 붙이지 않는 것이 올바른 코드<br>

  다만, 외부 코드(라이브러리 등) 같이<br>
  const 키워드를 컨트롤하지 못하는 경우,<br>
  매우 신중하게 사용할 순 있다<br>
  (라이브러리 return 값이 const 여서 건드리거나 수정할 수 없는 경우 등)<br>

  volatile? : 컴파일러 최적화에서 제외하는 키워드<br>
  (멀티 스레드 환경 등에서 컴파일러가 개입하여 값이 변하는걸 원치 않을때 사용)<br>

- reinterpret_cast<><br>
 : 비트 수준의 재해석 캐스팅<br>
  강제로 해당 타입으로 캐스팅하므로<br>
  일반적으로는 사용하지 않음<br>
  주로 '메모리' 상의 raw byte를 특정 타입으로 해석하는데 사용한다<br>
  (네트워크나 파일 쪽의 바이너리 데이터를 '특정한 타입'으로 읽어 해석하는 방식)<br>

| 캐스트 종류             | 목적                       | 검사 시점 | 주요 용도                 | 안전성     |
| ------------------ | ------------------------ | ----- | --------------------- | ------- |
| `static_cast`      | 컴파일 시 타입 변환              | 컴파일타임 | 기본형, 업캐스팅, 명시적 변환     | 중간      |
| `dynamic_cast`     | 다운캐스팅 + RTTI 검사          | 런타임   | 부모 → 자식 안전한 다운캐스팅     | 높음      |
| `const_cast`       | `const`/`volatile` 제거/부여 | 컴파일타임 | const 제거, API 호환용     | 중간\~위험  |
| `reinterpret_cast` | 비트 수준의 변환 (무조건 변환)       | 컴파일타임 | 포인터 ↔ 정수, 포인터 ↔ 포인터 등 | 낮음 (위험) |



## 업/다운 캐스팅
'상속'과 연관된 포인터 기능이자 '캐스팅'<br>

### 업캐스팅
자식 클래스 -> 부모 클래스 로의 형변환<br>

```
class Animal {
public:
    void speak() { std::cout << "Animal\n"; }
    virtual void Run() { std::cout << "동물이 달린다\n"; }
};

class Dog : public Animal {
public:
    void speak() { std::cout << "Dog\n"; }
    void Run() override { std::cout << "개가 달린다\n"; }
};

int main() {
    Dog d;
    Animal* a = &d;   // ✅ 업캐스팅
    a->speak();       // Animal::speak() 호출 (정적 바인딩)
    a->Run();         // Dog::Run() 호출 (동적 바인딩)
}

```

부모 타입으로 동작하므로<br>
부모에 정의된 멤버만 접근 가능<br>
(virtual 로 선언해야 '동적 바인딩'으로 호출됨)<br>



### 다운캐스팅
부모 클래스 -> 자식 클래스로의 변환<br>

```
Animal* a = new Dog();        // 실제는 Dog이지만 타입은 Animal*
Dog* d = (Dog*)a;             // ⚠️ 다운캐스팅 (명시적 필요)

Dog* safe_d = dynamic_cast<Dog*>(a);  // 안전한 다운캐스팅 (RTTI 필요)

```

주의할 점<br>

```
Animal* a = new Animal();  // 실제로는 Animal
Dog* d = (Dog*)a;          // ⚠️ 위험한 다운캐스팅
d->speak();                // 정의되지 않은 동작(UB)!
```

이런 식의 코드는 매우 위험<br>
상속 받은 클래스는 더 큰 크기의 클래스일 가능성이 높기에<br>
'할당되지 않은 영역'의 메모리에 접근하여 위험<br>
+ d->speak()는 Animal의 vtable을 확인하여 호출하는 점도 위험<br>

그래도 잘 사용하는 경우<br>
다양한 자식 클래스들의 다형성과 분기문 등을 실행할 수 있음<br>

```
// 베이스 클래스
class Animal {
public:
    virtual void speak() { cout << "Some animal sound\n"; }
    virtual ~Animal() = default;
};

// 파생 클래스 1
class Dog : public Animal {
public:
    void speak() override { cout << "Woof!\n"; }
    void bark() { cout << "Barking loudly!\n"; }
};

// 파생 클래스 2
class Cat : public Animal {
public:
    void speak() override { cout << "Meow!\n"; }
};

// 파생 클래스 3
class Duck : public Animal {
public:
    void speak() override { cout << "Quack!\n"; }
};

vector<Animal*> zoo;
zoo.push_back(new Dog());
zoo.push_back(new Cat());
zoo.push_back(new Duck());
zoo.push_back(new Dog());

// 모두 speak 호출, Dog만 bark도 호출
for (Animal* a : zoo) {
    a->speak();  // 다형성으로 동작

    // 다운캐스팅을 시도 (Dog 타입인지 확인)
    if (Dog* d = dynamic_cast<Dog*>(a)) {
        d->bark();  // Dog만 실행
    }
}

```

---

## 깊은/얕은 복사
동적 메모리 할당과 연관된 중요한 개념으로<br>
해당 메모리 공간을 '공유'할지, 메모리 공간까지 복사하여 독립시킬지에 대한<br>
개념이다<br>

### 얕은 복사(Shallow Copy)
객체의 '포인터'만 복사하여 '복사한 원본'과 '같은' 메모리 위치를 갖는다<br>

```
#include <iostream>
#include <cstring>
using namespace std;

class Shallow {
public:
    char* name;

    Shallow(const char* str) {
        name = new char[strlen(str) + 1];
        strcpy(name, str);
    }

    // 디폴트 복사 생성자 = 얕은 복사!
    // Shallow(const Shallow& other) = default;

    ~Shallow() {
        delete[] name;
    }
};

int main() {
    Shallow a("Dog");
    Shallow b = a;  // ⚠️ 얕은 복사

    b.name[0] = 'L';  // b만 바꿨지만 a도 바뀜
    cout << a.name << endl;  // 출력: "Log" (의도치 않음)
}
```

예시를 보자면<br>
b는 a의 'Dog' 글자를 얕은 복사로 copy함<br>
(사실상 b.name = a.name으로 포인터 주소를 복사)<br>

b.name을 바꾸게 되면<br>
a와 b가 공유하고 있는 주소값을 찾아가<br>
'메모리 영역' 내부에 있는 "Dog"을 수정하게 된다<br>

---

### 깊은 복사(Deep Copy)

```
class Deep {
public:
    char* name;

    Deep(const char* str) {
        name = new char[strlen(str) + 1];
        strcpy(name, str);
    }

    // 깊은 복사 생성자 구현
    Deep(const Deep& other) {
        name = new char[strlen(other.name) + 1];
        strcpy(name, other.name);
    }

    ~Deep() {
        delete[] name;
    }
};

```

위 예시에서 복사 생성자를 구현하여<br>
"Dog"을 카피할 때, 별도의 메모리 영역을 잡은 후,<br>
그 내용물을 카피하게 됨<br>

따라서 a와 b는 별도의 메모리 공간을 가지게 되며<br>
상호 독립적이게 된다<br>

- Rule of Five?<br>
 포인터 멤버를 가진 클래스는 아래의 5가지를 직접 구현해야 안전하는 규칙<br>
 1. 소멸자 (~Class)
 2. 복사 생성자(Class(const Class&))
 3. 복사 대입 연산자(operator=())
 4. 이동 생성자(Class(Class&&))
 5. 이동 대입 연산자(operator=(MyClass&&))<br>
 ('이동'과 'rvalue'는 당장 다루지는 않을 예정)<br>


정리<br>

| 구분     | 얕은 복사         | 깊은 복사           |
| ------ | ------------- | --------------- |
| 메모리 공유 | ✅ 공유함 (같은 주소) | ❌ 공유 안함 (새 메모리) |
| 안전성    | ⚠️ 이중 해제 위험   | ✅ 안전            |
| 사용 목적  | 단순 구조         | 리소스 소유 객체       |
| 디폴트 동작 | 컴파일러 자동 생성    | 직접 구현 필요        |

- 무조건 깊은 복사가 좋은 것만은 아니다<br>
  선언한 '메모리'가 매우 큰 경우<br>
  shared_ptr, 읽기 전용 등으로 메모리를 공유하는 것이<br>
  메모리 효율이 높아지며, 불필요한 메모리 할당을 하지 않아도 됨<br>
  (다만 메모리 해제는 반드시 원본 포인터 쪽에서 처리하거나<br>
  안전하게 스마트 포인터를 사용하자)<br>
  (물론 스마트 포인터를 사용하더라도 완벽히 메모리 누수를 막을 수 있는것은<br>
  아니므로 주의하자 ex : 순환참조)<br>
  
---

## 댕글리 포인터(dangling pointer)
이미 '해제된 메모리'를 가리키는 포인터<br>

포인터가 어떠한 주소값을 가지고 있는데<br>
해당 위치에 가보니 이미 메모리가 해제되어 있어서<br>
'어떠한 값'이 있는지를 모르는 경우를 뜻한다<br>

```
int* p = new int(10);
delete p;        // 메모리 해제됨
*p = 20;         // ❌ 댕글링 포인터 사용 → 정의되지 않은 동작 (UB)
```

이러한 댕글리 포인터는 갑자기 프로그램이 다운되거나 크래시를 발생시키는 원인이 되므로<br>
반드시 막아야 한다<br>

| 방법                      | 설명                                                 |
| ----------------------- | -------------------------------------------------- |
| **delete 후 nullptr 대입** | `delete p; p = nullptr;` → 더 이상 잘못된 접근 방지          |
| **RAII 사용 (스마트 포인터)**   | `std::unique_ptr`, `std::shared_ptr`로 생명 주기를 자동 관리 |
| **지역 변수 주소를 반환하지 않기**   | 대신 `new`, 혹은 매개변수 참조로 전달                           |
| **범위 내에서만 포인터 사용**      | 외부에서 참조하지 않도록 구조 설계                                |

##  RAII (Resource Acquisition Is Initialization)
'자원 획득'을 객체의 초기화와 동시에 하며<br>
'자원 해제'를 객체의 소멸과 같이 한다는 의미<br>

C++의 자원 관리 방식이며 '기능'은 아니지만<br>
생성자/소멸자 구조로부터 시작한 '자원 관리 방식'이다<br>
(패러다임)<br>

```
class FileGuard {
    FILE* fp;
public:
    FileGuard(const char* path) {
        fp = fopen(path, "r");
    }

    ~FileGuard() {
        if (fp) fclose(fp);
    }
};

```

예시의 FileGuard 클래스는<br>
1. 생성 시, fopen()을 통해 자원 획득<br>
2. scope를 벗어나는 경우, fileGuard가 호출되어 자원을 해제<br>
3. 예외가 발생하여도 C++이 지역 객체를 파괴하며 stack을 종료시키기에 자원 해제는 정상적으로 일어남<br>
   (예외 발생시, 지역 객체의 클래스 소멸자가 호출되기에)<br>

C++은 객체의 소멸 시점이 명확하기에 'RAII'가 가능<br>
(delete 호출, 지역 변수의 scope 벗어남 등)<br>

다만 C#,Java 나 Unreal 같은 게임 엔진은<br>
자체적으로 Garbage Collection이 존재하기에<br>
RAII가 적용된다고 보긴 힘들 수 있다<br>
(Unreal GC와 Shared_ptr의 비교는 아래쪽에서)

## 스마트 포인터(Smart Pointer)
일반 포인터처럼 동작하지만, 메모리를 자동으로 관리해주는 클래스 템플릿<br>
(RAII 기반으로 '스코프'를 벗어나면 자동으로 자원을 해제)<br>

- 기존 raw pointer(생 포인터)를 통한 문제와 스마트 포인터 사용을 통한 개선<br>

| 기존 raw pointer 문제              | 스마트 포인터의 해결              |
| ------------------------------ | ------------------------ |
| `delete` 누락 → 메모리 누수           | 스코프 벗어나면 자동 해제           |
| 예외 시 `delete` 실행 안 됨           | 소멸자에서 자동 처리됨             |
| 다중 포인터가 같은 메모리를 가리킴 → 이중 삭제 위험 | `shared_ptr`의 참조 카운트로 해결 |

<br><br>

- 스마트 포인터의 종류

| 타입                   | 소유권 모델         | 특징                             |
| -------------------- | -------------- | ------------------------------ |
| `std::unique_ptr<T>` | 단일 소유 (복사 불가)  | 가장 단순, 가장 빠름                   |
| `std::shared_ptr<T>` | 공유 소유 (참조 카운트) | 여러 곳에서 공유 가능                   |
| `std::weak_ptr<T>`   | 비소유 참조         | 순환 참조 방지용 (shared\_ptr과 함께 사용) |

<br><br>

1. unique_ptr<br>
  : 한 객체의 소유자는 '하나'라는 개념<br>
  복사 불가 하며, '이동'만 가능<br>

```
std::unique_ptr<int> p1 = std::make_unique<int>(10);
// std::unique_ptr<int> p2 = p1;        // ❌ 복사 불가
std::unique_ptr<int> p2 = std::move(p1);  // ✅ 소유권 이동
```
<br>
2. shared_ptr<br>
 : 여러 shared_ptr 들이 '동일 객체'를 소유<br>
   '참조 카운트(ref Count)'를 내부적으로 사용하여<br>
   소유 객체를 '얼마나 참조'하는지 파악<br>
   마지막 shared_ptr의 파괴로(ref Count == 0)<br>
   객체를 삭제<br>
  
<br>

```
#include <memory>
#include <iostream>

struct MyObject {
    MyObject() { std::cout << "생성\n"; }
    ~MyObject() { std::cout << "소멸\n"; }
};

int main() {
    std::shared_ptr<MyObject> p1 = std::make_shared<MyObject>();
    std::shared_ptr<MyObject> p2 = p1;  // 참조 카운트 증가
}  // 마지막 shared_ptr 파괴 → 소멸자 호출

```

<br>

3. weak_ptr<br>
 : shared_ptr의 참조 카운트에 영향을 주지 않는 '비소유' 참조<br>
   shared_ptr 끼리의 순환 참조를 '방지'하기 위함<br>
   (-> shared_ptr을 통해 서로를 참조하는 객체가 있으면<br>
   refCount가 0이 되지 않기 때문...)<br>
   이미 참조하는 대상이 없을 수 있기에 .lock() 등을 통해<br>
   접근해야 안전<br>

<br>

```
#include <memory>

std::shared_ptr<int> p = std::make_shared<int>(42);
std::weak_ptr<int> wp = p;

if (auto sp = wp.lock()) {
    std::cout << *sp << std::endl;  // 안전하게 접근
}
```

<br>

- 스마트 포인터 정리

| 항목       | `unique_ptr` | `shared_ptr` | `weak_ptr` |
| -------- | ------------ | ------------ | ---------- |
| 소유권      | 단일 소유        | 공유 소유        | 소유 안 함     |
| 복사       | ❌ 금지         | ✅ 허용         | ✅ 가능       |
| 이동       | ✅ 가능         | ✅ 가능         | ✅ 가능       |
| 참조 카운트   | 없음           | 있음           | 있음 (비가산)   |
| 순환 참조 방지 | 해당 없음        | ❌            | ✅ 필수       |

<br><br>

---

### Shared_ptr은 RAII? Unreal GC는 RAII가 아닌가?
Shared_ptr은 참조카운트(refCount) 기반이며, 동시에 '스코프'기반 객체<br>
(refCount가 0인 시점에서 바로 자원 해제)<br>

Unreal GC는 '객체'의 직접적인 delete 호출을 할 수 없고<br>
GC 내부 루틴에 따라서 제거하기에 '제거 시점'이 불명확하다<br>

정리<br>

| 항목             | `shared_ptr` (RAII)              | Unreal GC (추적 기반)                     |
| -------------- | -------------------------------- | ------------------------------------- |
| RAII 적용 여부   | ✅ YES (소유권 스코프 기반 해제) | ❌ NO (GC가 소멸 시점 결정) |
| 참조 카운트 기반?     | ✅ 예 (ref count == 0 → 즉시 delete) | ❌ 아니오 (ref count는 추적에 도움만 줄 뿐)        |
| 자동 해제 시점       | 정확 (스코프 벗어날 때)                   | 불명확 (GC 루틴 실행 시점)                     |
| 직접 delete 가능?  | ✅ 내부에서 delete 호출                 | ❌ 절대 금지 (Unreal 관리 대상)                |
| 개발자가 해제 제어 가능? | ✅ 가능 (`reset()`, 스코프 종료 등)       | ❌ 직접 제어 불가. UPROPERTY로 간접 제어          |
| 예외 안전성         | ✅ 뛰어남                            | ❌ 제한적 (에디터에서도 GC 돌리거나 수동으로 Clear해야 함) |
| ref count 사용 | ✅ 중심 로직               | 🔄 일부 사용 (보조적 추적용)  |
| 자동 삭제 시점     | **즉시, 예측 가능**         | **비동기, 예측 불가**      |


