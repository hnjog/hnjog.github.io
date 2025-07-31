---
title: "Interface"
last_modified_at: "2025-07-31T16:00:00"
categories:
  - C++
tags:
  - Interface
---

## Interface
오늘은 TIL을 좀 더 가볍게 다뤄보려고 한다<br>
기존의 포스팅은 작성에 몇시간이 걸리고<br>
정리에 시간이 걸렸지만<br>
오늘은 몸 상태가 좋지 않아 Interface 위주로 다루려 한다<br>

### 개념
'공통된 기능'을 '명세'만 하며<br>
실제 구현은 각 클래스가 하도록 하는 추상화 방식<br>

C++에서 Interface라는 '키워드'는 없지만<br>
'순수 가상 함수'를 통해 인터페이스를 구현할 수 있음<br>

```
class IInteractable { // I 를 붙여 Interface용 클래스 임을 쉽게 구분
public:
    virtual void Interact() = 0; // 순수 가상 함수 → 구현 없음
    virtual ~IInteractable() {}  // 가상 소멸자 권장
};
```

- 순수 가상 함수를 가진 클래스는 '직접 생성'될 수 없음<br>
- 순수 가상 함수를 가진 클래스를 상속 받은 경우<br>
  자식에서 객체 생성을 위해 해당 함수를 '반드시' 구현해야 함<br>

상속받으면 '이 함수를 반드시 구현해줘'라는<br>
일종의 '가이드라인'이다<br>

### 사용 이유
'공통된 기능'에 대한 약속<br>

요번에 구현하였던<br>
'문', '불타는제단','포탈' 등은<br>
전부 '플레이어'와 '상호작용'할 요소가 필요가 있다<br>

그런데 이들은 각각 '다른' 성격을 가진 존재들이다<br>

- 문 : 열리고 닫힘<br>
- 불타는 제단 : 파티클 이벤트 생성<br>
- 포탈 : 플레이어를 이동시킴<br>

그렇기에 '공통된 기능'인 '플레이어와 상호작용'만을 넣을 순 없을까?<br>
(플레이어가 이들을 타입 별로 구분하여 <br>
일일이 호출하는 것은 유지보수에 좋지 않으므로)<br>

이럴 때 필요한 것이 Interface!<br>

각각의 클래스들에서 특색에 맞게<br>
Interact 이벤트의 내용을 구현<br>

#### 문

<img width="1691" height="1087" alt="Image" src="https://github.com/user-attachments/assets/6dadf740-82ad-43af-acc7-94373933b2b8" /><br>

#### 제단

<img width="2015" height="1154" alt="Image" src="https://github.com/user-attachments/assets/462d7ce3-0ed3-48e7-9a20-66383b074b2b" /><br>

#### 포탈

<img width="2000" height="1030" alt="Image" src="https://github.com/user-attachments/assets/6d7660e7-4238-4dc7-a7d6-557290e3b219" /><br>


플레이어는 Interact 인터페이스를 상속받은 녀석에게 그저<br>
Interact 하라고 명령만 하면 된다!<br>
(객체지향의 다형성)<br>
(플레이어는 '반응'하라 하였고<br>
각각의 객체들은 각자의 '반응'을 보여준다)<br>

#### 플레이어

<img width="1276" height="349" alt="Image" src="https://github.com/user-attachments/assets/ad0f0819-5525-4ed3-9ad7-b01804f5bac5" /><br>

---

## 번외 : 공통된 부모 클래스로도 구현이 되잖아?
이론상 이 방식으로 '구현'은 가능하다<br>
다만 알아둘 점은<br>

- 부모 클래스를 통해 상속 받으면 'is-a' 관계<br>
- Interface는 'can-do'의 관계<br>

둘 다 '상속'의 기능을 사용하지만<br>
의도와 구조가 전혀 다른 방식들이다<br>

| 항목           | 공통 부모 클래스                      | Interface (C++ 추상 클래스 / BP Interface)    |
| ------------ | ------------------------------ | ---------------------------------------- |
| **역할**       | 공통 속성 및 기본 동작 제공               | 특정 "기능"만 명세 (구현은 개별 클래스가)                |
| **상속 의미**    | **"is-a"** (상속 구조상 동일 계층)      | **"can-do"**, "이 기능을 구현할 수 있음"           |
| **다중 상속**    | ❌ 제한적 (C++에서 위험, 단일 계층 추천)     | ✅ 안전한 다중 interface 구현 가능                 |
| **필수 구현 여부** | 구현 안 해도 됨 (가상함수 선택적)           | 전부 순수 가상 함수 (`= 0`) 강제 구현                |
| **예시**       | `class Actor : public UObject` | `class IDamageable` / `BP_IInteractable` |

'공통 부모'는 '강제로 그 타입'으로 묶인다는 점에 집중하자<br>

```
class InterActActor {
    FVector Position;
    virtual void Tick(float DeltaTime);
    virtual void Interact();
};

class Door : public InterActActor { ... };
class Portal : public InterActActor { ... };
```

Door 과 Portal 모두 'InterActActor'라는 공통 기반을 가지게 된다<br>
따라서 양 클래스는 부모 클래스의 '상태'와 '공통 기능'을 공유한다<br>

다만...<br>
- 공통된 기능이 더 필요할 경우, 부모 클래스가 커지게 되고<br>
  이후 추가하는 자식들은 필요하지 않은 기능도 상속받을 수 있음<br>

- 기본적으로 Interact() 자체가 virtual 이기에<br>
  '구현하지 않는' 휴먼에러의 발생 가능성 존재<br>
  (그렇다고 Interact()를 강제하면 Interact'만' 필요하지 않은<br>
  자식들은 강제로 구현하거나, 새로운 부모 클래스를 파야한다)<br>

- 클래스 다중 상속의 위험을 내포하고 있음<br>
  (diamond problem 등)<br>

그렇기에 '상속' 자체는 신중하게 고려해야 한다<br>
(정말 A를 B로 '말해도 되는가'?)<br>

Interface는 비교적 '가볍'기에<br>
위와 같은 고민사안에서 자유로운 편이다<br>
(물론 개념상 상속이긴 하지만<br>
 Interface는 '다중 상속'에 포함시키지는 않는 편)<br>

- 필요에 따라 '붙이는' 클래스이기에<br>
  개념이 서로 다른 클래스여도 '공통 함수'를 가질 수 있음<br>
  (ex : ISwimmable 을 통해 수영 가능 기능을 구현하도록 함)<br>
  반대로 같은 개념을 가지는 클래스 여도<br>
  차이점을 줄 수 있음<br>
  (ex : IShootable을 통해 같은 Character 여도<br>
   사격 기능의 존재를 판별 가능)<br>

- 순수 가상 함수를 '강제'하기에<br>
  반드시 구현해야 함<br>
  (애초에 그렇지 않으면 이 인터페이스를 상속받으면 안되는 클래스)<br>

- 그저 '공통 기능'만을 강조하기에<br>
  '인터페이스' 타입으로 업캐스팅하고<br>
  호출만 해주면 사용 클래스는 해당 클래스를 몰라도 됨<br>

상속은 '구조 공유'<br>
(공통적인 멤버 함수, 변수 등을 재활용)<br>
인터페이스는 '기능 공유'의 측면이 강하다<br>
(너도 그 기능이 있지?)<br>

## 정리
순수 가상 함수만 가진 추상 클래스를 이용해,<br>
"기능만 명세하고, 구현은 각 클래스가 다르게 하도록 강제"하는 개념<br>

서로 다른 구조의 객체들이 “같은 기능을 제공한다”는<br>
것을 표현하는 약속임을 알아두자!<br>