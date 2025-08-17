---
title: "디자인 패턴"
last_modified_at: "2025-08-17T14:00:00"
categories:
  - Design Pattern
tags:
  - 싱글톤
  - 데코레이터
  - 옵저버
---

## 디자인 패턴
개발 시 반복적으로 등장하는 문제를<br>
해결하기 위한 일반화된 풀이 방법<br>

개발자들이 다른 개발자들을 위해<br>
'공통적'으로 고민하는 문제를 정리하여 만든 패턴<br>
('설계 원칙'에 가깝다)<br>

![Image](https://github.com/user-attachments/assets/c01f26e3-0478-4f3d-a71b-c711385f2163)<br>

### 생성 패턴(Creational Patterns)
객체 생성 과정에서의 '유연성'과 '재사용성'을 높이는 패턴<br>
객체 '생성'의 '책임과 과정'을 캡슐화 함<br>

- 싱글톤(Singleton): 객체를 단 하나만 생성해 전역적으로 접근 (ex: 게임 매니저, Logger)<br>
- 팩토리 메서드(Factory Method): 객체 생성을 서브클래스에서 결정 (ex: 무기 클래스 → SwordFactory, GunFactory)<br>
- 추상 팩토리(Abstract Factory): 관련 객체들을 하나의 “팩토리”로 묶어 생성 (ex: UI Theme Factory → Button, CheckBox 생성)<br>
- 빌더(Builder): 복잡한 객체를 단계별로 조립 (ex: 캐릭터 생성기)<br>
- 프로토타입(Prototype): 기존 객체를 복사(clone)해서 생성 (ex: 아이템 복제)<br>

#### 싱글톤 패턴

하나의 인스턴스만이 존재하며<br>
'전역 접근'을 제공하는 패턴<br>

- 전역 상태를 통해 접근이 쉬워지지만<br>
  해당 클래스에 대한 '결합도'가 올라갈 수 있으니 주의가 필요<br>

![Image](https://github.com/user-attachments/assets/b69f42b0-0402-44d9-b6d2-37e819f8386b)<br>

예시상황)<br>
게임에서 '비행기'가 '반드시' 하나만 존재해야 하는 경우<br>

![Image](https://github.com/user-attachments/assets/0e9ebbe3-351d-43cc-ab8d-8f6e2450a193)<br>


생성자를 private로<br>
복사 생성자 및 대입 연산자를 '삭제'하여 객체가 늘어나는 경우를 방지<br>

이후, GetInstance() 함수를 통하여 객체 생성 및<br>
'기존 객체'를 반환<br>

코드)<br>

```
class Airplane {
private:
    static Airplane* instance; // 유일한 비행기 객체를 가리킬 정적 포인터
    int positionX;             // 비행기의 X 위치
    int positionY;             // 비행기의 Y 위치

    // private 생성자: 외부에서 객체 생성 금지
    Airplane() : positionX(0), positionY(0) {
        cout << "Airplane Created at (" << positionX << ", " << positionY << ")" << endl;
    }

public:
    // 복사 생성자와 대입 연산자를 삭제하여 복사 방지
    Airplane(const Airplane&) = delete;
    Airplane& operator=(const Airplane&) = delete;

    // 정적 메서드: 유일한 비행기 인스턴스를 반환
    static Airplane* getInstance() {
        if (instance == nullptr) {
            instance = new Airplane();
        }
        return instance;
    }

    // 비행기 위치 이동
    void move(int deltaX, int deltaY) {
        positionX += deltaX;
        positionY += deltaY;
        cout << "Airplane moved to (" << positionX << ", " << positionY << ")" << endl;
    }

    // 현재 위치 출력
    void getPosition() const {
        cout << "Airplane Position: (" << positionX << ", " << positionY << ")" << endl;
    }
};
```

##### Unreal에선?

보통 Subsystem 이 이에 가까움<br>
(다만 수명 관리는 엔진 내부에서 해줌)<br>

그 외에도 AssetManager 등이 '전역 접근'을 허용함<br>

##### 생각해볼 점

- 프로세스의 전반적으로 접근할 필요가 있는 시스템 구조나<br>
  '리소스,캐시 매니저'처럼 '반드시 하나'여야 하는 경우에<br>
  적합한 디자인 패턴이다<br>

- 다만, 편하다는 의미로 막 사용할만한 패턴은 아님<br>
 (전역 변수,함수 등의 남용은 디버깅 난이도를 높인다)<br>

- 멀티 스레드 환경이면 '반드시' 하나인 싱글톤 객체에 접근할 때<br>
  유의할 것(제대로 lock을 걸어야 함)<br>

---

### 구조 패턴(Structual Patterns)

클래스, 객체 의 구조를 조직화 하여 더 큰 구조를 만드는 패턴<br>
객체 간의 '관계와 구조'를 다룬다<br>

- 브리지(Bridge): 구현과 추상을 분리해 독립적으로 확장 (ex: 무기 클래스와 캐릭터 클래스 분리)<br>
- 컴포지트(Composite): 트리 구조를 구성, 부분-전체 계층을 동일하게 다룸 (ex: 씬 그래프, UI 위젯 트리)<br>
- 데코레이터(Decorator): 객체에 기능을 동적으로 추가 (ex: 무기 + 불속성 효과)<br>
- 플라이웨이트(Flyweight): 공통 데이터를 공유해 메모리 절약 (ex: 수많은 총알, 나무 인스턴스)<br>
- 프록시(Proxy): 대리 객체가 접근 제어/캐싱/로깅 등을 수행 (ex: 네트워크 요청 지연 로딩)<br>


#### 데코레이터 패턴

런타임 중, '기능'을 덧붙이기 위한 합성 패턴<br>
상속 없이 객체의 '행위'를 감싸서 기능 확장<br>

![Image](https://github.com/user-attachments/assets/52131c8f-b749-49fb-833e-6c52db5d5c5a)<br>

예시상황)<br>
피자를 만듦<br>
'기본 베이스'가 존재하며,<br>
토핑을 원하는 대로 추가하며 동적으로 피자를 만듦<br>

![Image](https://github.com/user-attachments/assets/8e46af40-e625-4224-8c81-04cc2c0a2829)<br>

데코레이터 클래스를 통하여<br>
Basic 피자 객체를 감싸며<br>
신규 기능을 제공<br>

코드)<br>

```
#include <iostream>
#include <string>

using namespace std;

// **추상 컴포넌트 (Component): Pizza**
// - 피자 객체의 기본 구조를 정의하는 인터페이스입니다.
// - 모든 피자는 이름(`getName`)과 가격(`getPrice`)을 가져야 합니다.
class Pizza {
public:
    virtual ~Pizza() {}
    virtual string getName() const = 0;  // 피자의 이름 반환
    virtual double getPrice() const = 0; // 피자의 가격 반환
};

// **구체 컴포넌트 (Concrete Component): BasicPizza**
// - 기본 피자 클래스입니다.
// - 피자의 기본 베이스(이름과 가격)를 구현합니다.
class BasicPizza : public Pizza {
public:
    string getName() const {
        return "Basic Pizza"; // 기본 피자의 이름
    }
    double getPrice() const {
        return 5.0; // 기본 피자의 가격
    }
};

// **데코레이터 추상 클래스 (Decorator): PizzaDecorator**
// - 기존 피자의 기능을 확장하기 위한 데코레이터의 기본 구조를 정의합니다.
// - 내부적으로 `Pizza` 객체를 감싸며, 이름과 가격에 추가적인 기능을 제공합니다.
class PizzaDecorator : public Pizza {
protected:
    Pizza* pizza; // 기존의 피자 객체를 참조합니다.
public:
    // 데코레이터는 피자 객체를 받아서 감쌉니다.
    PizzaDecorator(Pizza* p) : pizza(p) {}
    
    // 소멸자에서 내부 피자 객체를 삭제합니다.
    virtual ~PizzaDecorator() {
        delete pizza;
    }
};

// **구체 데코레이터 (Concrete Decorators): Cheese, Pepperoni, Olive**
// - 각각의 토핑 데코레이터는 `PizzaDecorator`를 상속받아 이름과 가격을 확장합니다.

// 치즈 토핑 데코레이터
class CheeseDecorator : public PizzaDecorator {
public:
    CheeseDecorator(Pizza* p) : PizzaDecorator(p) {}
    string getName() const {
        // 기존 피자의 이름에 " + Cheese"를 추가
        return pizza->getName() + " + Cheese";
    }
    double getPrice() const {
        // 기존 피자의 가격에 치즈 추가 비용 1.5를 더함
        return pizza->getPrice() + 1.5;
    }
};

// 페퍼로니 토핑 데코레이터
class PepperoniDecorator : public PizzaDecorator {
public:
    PepperoniDecorator(Pizza* p) : PizzaDecorator(p) {}
    string getName() const {
        // 기존 피자의 이름에 " + Pepperoni"를 추가
        return pizza->getName() + " + Pepperoni";
    }
    double getPrice() const {
        // 기존 피자의 가격에 페퍼로니 추가 비용 2.0을 더함
        return pizza->getPrice() + 2.0;
    }
};

// 올리브 토핑 데코레이터
class OliveDecorator : public PizzaDecorator {
public:
    OliveDecorator(Pizza* p) : PizzaDecorator(p) {}
    string getName() const {
        // 기존 피자의 이름에 " + Olive"를 추가
        return pizza->getName() + " + Olive";
    }
    double getPrice() const {
        // 기존 피자의 가격에 올리브 추가 비용 0.7을 더함
        return pizza->getPrice() + 0.7;
    }
};

// **클라이언트 코드**
// - 피자와 데코레이터를 조합하여 최종 피자를 생성하고, 정보를 출력합니다.
int main() {
    // 1. 기본 피자를 생성합니다.
    Pizza* pizza = new BasicPizza();

    // 2. 치즈 토핑을 추가합니다.
    pizza = new CheeseDecorator(pizza);

    // 3. 페퍼로니 토핑을 추가합니다.
    pizza = new PepperoniDecorator(pizza);

    // 4. 올리브 토핑을 추가합니다.
    pizza = new OliveDecorator(pizza);

    // 5. 최종 피자 정보 출력
    cout << "Pizza: " << pizza->getName() << endl; // 피자의 이름 출력
    cout << "Price: $" << pizza->getPrice() << endl; // 피자의 가격 출력

    // 6. 메모리 해제
    delete pizza;

    return 0;
}
```

- 내부에서 소멸자로 pizza를 제거하기에<br>
  main에서 저렇게 new를 많이 생성해 보여도<br>
  leak 가 발생 x<br>
  피자 포인터에 기존 피자를 inner로 가진 새로운 피자를 담는 방식임<br>
  
- 여러 클래스를 정의해두고<br>
  런타임 중, 해당 '옵션'을 추가한 새로운 클래스를 생성<br>
  (일반 피자에 치즈, 페퍼로니, 올리브를 추가하듯)<br>


##### Unreal에선?

여러 데코레이션 컴포넌트를 Actor에게 붙이고 On/Off<br>

무기와 같은 클래스에 다양한 옵션을 주고 싶을때도 사용 가능<br>

```
// 공통 훅
struct FShotContext { float Damage; TArray<AActor*> Hits; /* etc. */ };

class IWeaponModifier {
public:
  virtual ~IWeaponModifier() = default;
  virtual int32 Priority() const { return 0; }            // 적용 순서
  virtual void PreFire(FShotContext& Ctx) {}              // 전처리
  virtual void PostHit(FShotContext& Ctx) {}              // 후처리
};

// 예시 모듈들
class FireMod   : public IWeaponModifier { /* DoT, Ignite 등 */ };
class PierceMod : public IWeaponModifier { /* 관통 로직 */ };
class CritMod   : public IWeaponModifier { /* 크리 계산 */ };

class Weapon {
  TArray<TUniquePtr<IWeaponModifier>> Mods;  // 조합
public:
  void Fire(){
    FShotContext Ctx{BaseDmg};
    Mods.StableSort([](auto& L, auto& R){ return L->Priority() < R->Priority(); });
    for (auto& M : Mods) M->PreFire(Ctx);
    // 실제 발사 & 히트 수집…
    for (auto& M : Mods) M->PostHit(Ctx);
  }
};
```

- 기본적으로 이러한 방식은 '단일 책임 원칙'을 지키기 쉬워짐<br>
- 다양한 버프 / 디버프 구현 등을 자연스럽게 구현 가능<br>

'다양한 옵션'은 하나의 클래스 내부의 '변수'를 생성함으로도 가능<br>
(구현이 빠르고, 캐시 친화적)<br>
(단, 옵션이 늘수록 클래스의 역할이 커짐)<br>

양쪽 다 상황에 맞게 사용하는 걸 권장<br>

##### 생각해볼 점

- 기본 클래스에 여러가지 '옵션'을 주고 싶을때<br>
  고려 가능한 패턴<br>

- 데코레이션의 '단계'가 많아지면 '흐름 파악'이 어려울 수 있기에<br>
  네이밍 규칙이나 시각화가 필요<br>
  (비직관적)<br>

- 데코레이터들이 하나의 '상태'를 공유하기 시작하면<br>
  결합도가 올라가므로 입력/출력 에 대한 인터페이스를 명확히 설정할 것을 권장<br>

---

### 행동 패턴(Behavioral Patterns)
객체 간의 책임 분배와 상호작용을 정의하는 패턴<br>
객체 간 협력을 통해 문제를 해결하는 방식에 집중<br>

- 옵저버(Observer): 상태 변화를 여러 객체에 통지. (ex: 이벤트 브로드캐스트)<br>
- 전략(Strategy): 알고리즘을 캡슐화해 교체 가능. (ex: AI 이동 방식: 추적/회피/순찰 교체)<br>
- 커맨드(Command): 요청을 객체로 캡슐화. (ex: 실행 취소/재실행, Input 시스템)<br>
- 상태(State): 객체의 상태에 따라 행동을 변경. (ex: 플레이어 Idle/Run/Attack 상태머신)<br>
- 템플릿 메서드(Template Method): 알고리즘의 골격 정의, 세부 구현은 서브클래스에서. (ex: 추상 게임 루프)<br>
- 이터레이터(Iterator): 집합 객체를 순차적으로 접근. (ex: STL vector iterator)<br>
- 중재자(Mediator): 객체 간의 복잡한 의존성을 중재. (ex: UI 위젯 상호작용 관리)<br>
- 책임 연쇄(Chain of Responsibility): 요청을 처리할 수 있는 객체가 나올 때까지 전달. (ex: UI 이벤트 전파)<br>
- 메멘토(Memento): 객체 상태를 캡슐화해 저장/복원. (ex: 게임 세이브/로드)<br>
- 인터프리터(Interpreter): 언어 문법을 해석. (ex: 스크립트 파서)<br>

#### 옵저버 패턴
'주체(Subject)'의 상태 변화를 다수의 '구독자(Observer)'에게<br>
자동으로 통지한다<br>

1:N 이벤트 BroadCast<br>


![Image](https://github.com/user-attachments/assets/a3eb1df5-ca23-4b04-ae4f-dc6f3d152309)<br>

예시상황)<br>
엑셀 프로그램에서 데이터 값 변경 시<br>
해당 데이터를 이용하는 모든 차트들에게 반영되게 하고 싶음<br>

![Image](https://github.com/user-attachments/assets/83e22d05-3008-485c-8fd5-9091fe535579)<br>

주체인 엑셀 시트 클래스가<br>
특정한 값이 바뀐 경우<br>
notify() 클래스를 호출하여<br>
등록된 옵저버들에게 Update() 함수를 호출 시킴<br>

코드)<br>

```
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// Observer 인터페이스
// - Observer 패턴에서 상태 변화를 알림받는 객체들의 공통 인터페이스
// - Observer들은 이 인터페이스를 구현하여 `update` 메서드를 통해 데이터를 전달받음
class Observer {
public:
    virtual ~Observer() = default;               // 가상 소멸자
    virtual void update(int data) = 0;           // 데이터 업데이트 메서드 (순수 가상 함수)
};

// Subject 클래스 (엑셀 시트 역할)
// - 데이터의 상태 변화를 관리하며, 모든 등록된 Observer들에게 변경 사항을 알림
class ExcelSheet {
private:
    vector<Observer*> observers;                 // Observer들을 저장하는 리스트
    int data;                                    // 현재 데이터 상태

public:
    ExcelSheet() : data(0) {}                    // 생성자: 초기 데이터 값은 0

    // Observer 등록 메서드
    // - 새로운 Observer를 등록하여 변경 사항 알림을 받을 수 있도록 추가
    void attach(Observer* observer) {
        observers.push_back(observer);
    }

    // 데이터 변경 알림 메서드
    // - 등록된 모든 Observer들의 `update` 메서드를 호출하여 데이터 변경 사항을 알림
    void notify() {
        for (Observer* observer : observers) {
            observer->update(data);              // 각 Observer에게 데이터를 전달
        }
    }

    // 데이터 설정 메서드
    // - 데이터를 변경하고 변경 사항을 모든 Observer에게 알림
    void setData(int newData) {
        data = newData;                          // 새로운 데이터로 갱신
        cout << "ExcelSheet: Data updated to " << data << endl;
        notify();                                // Observer들에게 알림
    }
};

// 구체적인 Observer 클래스: BarChart (막대 차트)
// - 데이터를 막대 그래프로 표현
class BarChart : public Observer {
public:
    void update(int data) {                      // 데이터 업데이트 시 호출됨
        cout << "BarChart: Displaying data as vertical bars: ";
        for (int i = 0; i < data; ++i) {
            cout << "|";                         // 데이터 값만큼 막대 출력
        }
        cout << " (" << data << ")" << endl;
    }
};

// 구체적인 Observer 클래스: LineChart (라인 차트)
// - 데이터를 선형 그래프로 표현
class LineChart : public Observer {
public:
    void update(int data) {                      // 데이터 업데이트 시 호출됨
        cout << "LineChart: Plotting data as a line: ";
        for (int i = 0; i < data; ++i) {
            cout << "-";                         // 데이터 값만큼 선 출력
        }
        cout << " (" << data << ")" << endl;
    }
};

// 구체적인 Observer 클래스: PieChart (파이 차트)
// - 데이터를 파이 그래프로 표현
class PieChart : public Observer {
public:
    void update(int data) {                      // 데이터 업데이트 시 호출됨
        cout << "PieChart: Displaying data as a pie chart slice: ";
        cout << "Pie [" << data << "%]" << endl; // 데이터 값 출력 (가정: % 비율로 표현)
    }
};

// 메인 함수
int main() {
    // Subject 생성
    ExcelSheet excelSheet;                       // 데이터를 관리하는 엑셀 시트 객체 생성

    // Observer 객체 생성 (각 차트 객체)
    BarChart* barChart = new BarChart();         // 막대 차트 생성
    LineChart* lineChart = new LineChart();      // 라인 차트 생성
    PieChart* pieChart = new PieChart();         // 파이 차트 생성

    // Observer 등록
    // - 각 차트(Observer)를 엑셀 시트(Subject)에 등록
    excelSheet.attach(barChart);
    excelSheet.attach(lineChart);
    excelSheet.attach(pieChart);

    // 데이터 변경 테스트
    // - 데이터를 변경하면 등록된 모든 Observer들이 알림을 받고 화면에 갱신
    excelSheet.setData(5);                       // 데이터 변경: 5
    excelSheet.setData(10);                      // 데이터 변경: 10

    // 메모리 해제
    // - 동적 할당된 Observer(차트) 객체 삭제
    delete barChart;
    delete lineChart;
    delete pieChart;

    return 0;
}
```

##### Unreal에선?

DECLARE_MULTICAST_DELEGATE_* 를 통해 완벽한 구현 가능<br>
특정한 상태 변화 시, Delegate에 등록한 객체들에게<br>
broadCast 함으로서 이벤트를 발생시킨다<br>

##### 생각해볼 점

- 여러 시스템이 '동일한 이벤트'에 반응할 때<br>
  고려 가능한 패턴<br>

- 구독 해제는 필수<br>
- 일부 Delegate는 옵저버가 살아있는지를 확인해야 함<br>
  (생명 주기 문제)<br>

- broadCast 호출 순서는 비결정적임<br>
  (순서와 관련되었다면 다른 패턴이나 추가적인 자료구조를 고려)<br>
  ('중재자 패턴')<br>

- 멀티스레드에서 활용시, 동기화 전략이 별도로 필요<br>

