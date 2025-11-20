---
title: "Class vs Struct"
date : "2025-11-18 14:00:00 +0900"
last_modified_at: "2025-11-18T14:00:00"
categories:
  - C++
tags:
  - C++
  - Class vs Struct
  - OOP
  - 모의 면접
---

## 클래스 vs 구조체?

기본적으로 거의 동일함<br>
차이점이 있다면<br>

- 기본 접근 제어자의 차이<br>
  - Class : Private (외부 접근 불가)<br>
  - Struct : Public (외부 접근 가능)<br>

- 기본 상속 지정자의 차이<br>
  - Class : Private로 상속(부모의 Public 요소들을 Private 지정자로 변경)<br>
  - Struct : Public으로 상속(부모의 접근 제어 상태를 유지)<br>

### 접근 제어자

| 접근 제어자        | 클래스 내부에서 접근 | 자식 클래스에서 접근 | 외부에서 접근 | 설명              |
| ------------- | ----------- | ----------- | ------- | --------------- |
| **public**    | ✔ 가능        | ✔ 가능        | ✔ 가능    | 모든 곳에서 접근 가능    |
| **protected** | ✔ 가능        | ✔ 가능        | ❌ 불가    | 외부는 불가, 자식은 가능  |
| **private**   | ✔ 가능        | ❌ 불가        | ❌ 불가    | 클래스 내부에서만 접근 가능 |


### 상속 지정자

| 상속 지정자           | 부모의 public 멤버 → 자식 | 부모의 protected 멤버 → 자식 | 의미                              |
| ---------------- | ------------------ | --------------------- | ------------------------------- |
| **public 상속**    | public             | protected             | 부모의 접근성을 그대로 유지                 |
| **protected 상속** | protected          | protected             | 외부에서 부모 인터페이스를 감추고, 자식 내부에서만 사용 |
| **private 상속**   | private            | private               | 부모의 모든 공개 인터페이스를 자식 내부로만 감춤     |


### TMI : 클래스를 구조체가, 구조체를 클래스가 상속 가능한가?

- 가능함<br>
  - 두 기능의 차이는 위에서 말한 2개의 차이만 존재하고<br>
    기타 기능은 동일하기에 가능<br>

- struct 역시 virtual, 상속, 다형성, template 등의 기능을 사용 가능함<br>

### 코드 예시 1 - 구조체가 클래스를 상속 받음

```cpp
class a
{
	int aa;
};

struct b : a
{
	int bb;
};

int main()
{
	b bt;
	bt.bb;

	return 0;
}
```

- 구조체 b의 상속 지정자가 public 이지만<br>
  클래스 a의 접근 제어자는 private이기에<br>
  b에서 aa를 접근하지는 못함<br>

### 코드 예시 2 - 클래스가 구조체를 상속 받음

```cpp
struct a
{
	int aa;
};

class b : a
{
public:
	int bb;
};

int main()
{
	b bt;
	bt.bb;

	return 0;
}
```

- 클래스 b의 기본 상속 지정자가 private이기에<br>
  구조체 a의 기본 접근 제어자가 public 임에도<br>
  private로 변경되어 bt 에서 aa에 대한 접근이 불가한 상태<br>

- 이렇기에 보통 '상속' 자체는 public 상속 지정자를 사용하여<br>
  상속의 코드 재사용성을 높이는 편<br>

### TMI : OOP의 4대 요소

- 추상화<br>
  : 필요한 정보만을 드러내고, 불필요한 내부 구현 숨기기<br>
    복잡한 시스템을 '단순'한 모델로 표현 가능해짐<br>

```cpp
class Character {
public:
    void Move(); // 움직일께!
private:
    void CalculatePath();  // 길 계산은 외부 객체가 알 필요 없음
};
```

- 캡슐화<br>
  : 데이터와 기능을 하나로 묶고, 외부의 잘못된 접근을 막는 것<br>
    멤버의 접근 지정자를 통해 외부 접근 제어<br>
    객체의 상태 보호 및 변경의 영향을 최소화함<br>

```cpp
class Player {
private:
    int Health; // 체력은 Player만의 것

public:
    void SetHealth(int NewHealth) { // 변경 시, Setter를 통해 안전한 변경을!
        if (NewHealth >= 0) Health = NewHealth;
    }
};
```

- 상속<br>
  : 부모의 속성과 기능을 자식이 물려 받는 것<br>
    코드 재사용성 증가<br>
    다형성의 기반이 됨<br>
    공통 기능을 부모가, 자식은 확장 기능으로<br>

```cpp
class Animal {
public:
    void Eat(); // 모든 동물은 먹는 기능 포함
};

class Dog : public Animal {
public:
    void Bark(); // 개는 '짖는' 기능 추가!
};
```

- 다형성<br>
  : 같은 기능 호출이지만, 실제 동작은 객체마다 다르게<br>
   '가상 함수'를 재정의 함으로서 구현<br>
   객체에 따라 호출 결과가 달라짐!<br>

```cpp
class Animal {
public:
    virtual void Sound() { cout << "???" << endl; }
};

class Dog : public Animal {
public:
    void Sound() override { cout << "멍!" << endl; }
};

Animal* a = new Dog();
a->Sound();  // 출력: "멍!"
```

- 접근 제어자는<br>
  추상화/캡슐화와 연관된 기능<br>

- 상속 제어자는<br>
  상속/다형성과 연관된 기능<br>

#### 요약 표

| 원칙      | 개념                  | 핵심 목적      |
| ------- | ------------------- | ---------- |
| **추상화** | 불필요한 구현 숨기고 핵심만 표현  | 복잡성 감소     |
| **캡슐화** | 데이터 보호 + 인터페이스 제공   | 안정성, 유지보수성 |
| **상속**  | 부모 기능을 자식이 물려받음     | 코드 재사용     |
| **다형성** | 같은 호출이 객체 따라 다르게 동작 | 확장성과 유연성   |

