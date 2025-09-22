---
title: "Unreal Interface TIL (0922)"
date : "2025-09-22 15:00:00 +0900"
last_modified_at: "2025-09-22T15:00:00"
categories:
  - Unreal
  - C++
tags:
  - Interface
---

## **인터페이스 이해하기**

###  인터페이스란?

- **인터페이스 (Interface)** 란 클래스 (또는 오브젝트)가 **반드시 구현**해야 할 **함수 목록**만을 미리 정의하고, 실제 동작(구현 내용)은 해당 클래스를 상속받거나 구현하는 쪽에서 자유롭게 작성할 수 있도록 하는 일종의 **계약서**<br>
- C++에서는 `UInterface`를 상속받아 `IItemInterface` 같은 인터페이스를 제작 가능하고, 언리얼 블루프린트에서도 “블루프린트 인터페이스”를 통해 비슷한 개념을 구현 가능<br>

### 상속 (Inheritance)과의 차이점?

- **상속**
    - 부모 클래스의 모든 속성과 기능을 자식 클래스가 물려받는 구조<br>
    - 부모 클래스에 구현된 로직을 자식 클래스가 그대로 사용 가능, 필요하다면 자식 클래스에서 **재정의(오버라이딩)**<br>
- **인터페이스**
    - **인터페이스**는 “이 함수를 반드시 만들어야 한다”라는 **함수 원형 (함수 시그니처)** 만을 정의<br>
    - 실제 함수가 **어떻게 동작**할지는 각 자식 (또는 구현 클래스)에서 자유롭게 작성<br>
- **상속**은 부모의 실제 구현을 가져다 쓰는 반면, **인터페이스**는 “함수의 틀”만 빌려 쓰고, 그 안에 담길 코드는 직접 작성<br>

### 인터페이스를 사용하면 좋은 점?

- **결합도 (Coupling) 감소**
    - 클래스 간 구체적인 구현 내용을 공유 x, 필요한 **함수 목록**만 약속하므로 클래스 간 의존도가 감소<br>
    - 즉, 다른 클래스 내부가 어떻게 돌아가는지 몰라도, “이 함수를 이렇게 호출하면 된다” 정도만 알면 됨<br>
- **확장성 (Extensibility) 향상**
    - 새로운 아이템 클래스를 만들 때, 이미 정의된 인터페이스를 **구현**하기만 하면 기존 시스템에 쉽게 편입 가능<br>
- **다형성 (Polymorphism) 극대화**
    - `TArray<IItemInterface*> Items;`와 같은 인터페이스 포인터 배열로 관리 시, 아이템 종류가 무엇이든 **같은 함수를 호출**하여 다룰 수 있음<br>

## 언리얼의 인터페이스 형태 (C++)

### `ItemInterface` 인터페이스 정의

- **공통적으로 필요한 함수**를 인터페이스로 묶어두면, 다른 아이템들도 쉽게 동일한 규칙 (함수 시그니처)을 갖출 수 있음<br>

```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "IItemInterface.generated.h"

// 인터페이스를 UObject 시스템에서 사용하기 위한 기본 매크로
UINTERFACE(MinimalAPI)
class UItemInterface : public UInterface
{
    GENERATED_BODY()
};

// 실제 C++ 레벨에서 사용할 함수 원형(시그니처)를 정의
class SPARTAPROJECT_API IItemInterface
{
    GENERATED_BODY()

public:
    // 플레이어가 이 아이템의 범위에 들어왔을 때 호출
    virtual void OnItemOverlap(AActor* OverlapActor) = 0;
    // 플레이어가 이 아이템의 범위를 벗어났을 때 호출
    virtual void OnItemEndOverlap(AActor* OverlapActor) = 0;
    // 아이템이 사용되었을 때 호출
    virtual void ActivateItem(AActor* Activator) = 0;
    // 이 아이템의 유형(타입)을 반환 (예: "Coin", "Mine" 등)
    virtual FName GetItemType() const = 0;
};
```

### 구조

- `UINTERFACE(MinimalAPI)`<br>
    - 언리얼 엔진의 리플렉션 시스템 (Reflection)을 위해 사용하는 매크로<br>
    - 이렇게 선언해야 블루프린트나 다른 모듈에서도 해당 인터페이스를 인식하고 사용 가능<br>
- `class UItemInterface : public UInterface`<br>
    - 실제 객체(클래스) 관리를 위한 언리얼 측 클래스, C++의 `IItemInterface`와 구분해 사용<br>
- `class SPARTAPROJECT_API IItemInterface`<br>
    - 우리가 **직접 구현**해서 사용할 **인터페이스 함수**들을 정의<br>
    - `= 0;`으로 끝나는 순수 가상 함수(Pure Virtual Function) 형태로 지정하므로, **반드시** 이를 구현(Override)할 것<br>

- 이와 같이 언리얼에서 C++로 '인터페이스'를 구현할 때는<br>
  `UInterface + IInterface(C++)` 의 **'한 쌍'** 으로 만들어짐<br>

| 구분                            | 무엇인가                       | 주 역할                                        | 멤버 변수 가능 | UFUNCTION/UPROPERTY                                           |
| ----------------------------- | -------------------------- | ------------------------------------------- | -------- | ------------------------------------------------------------- |
| `UItemInterface : UInterface` | **UObject 기반의 “타입/메타 정보”** | 리플렉션, “이 타입의 인터페이스를 구현함”을 엔진/블루프린트가 알아차리게 함 | (의미상) X  | 여기서 **함수는 선언하지 않음** (보통 비워둠)                                  |
| `IItemInterface`              | **순수 C++ 인터페이스 본체**        | 실제로 호출할 **함수 시그니처**를 선언                     | X        | **여기에 `UFUNCTION`을 단다** (인터페이스 함수 노출은 여기서 함). `UPROPERTY`는 불가 |


### Unreal Interface 에 대한 설명

- U와 I 한쌍으로 '나누는 이유?'<br>
  : BP와 C++ 에서 Interface를 안전하게 접합하기 위하여 사용<br>
  - UInterface 는 `UClass`로서 리플렉션을 통해 '타입 표지자, 메타' 등을 제공<br>
  - IInterface 가 '호출 표면' (UFUNRCION)을 제공<br>

- 이 두 클래스들이 서로 '엮일 수 있는' 이유?<br>
  : GENERATE_BODY() 를 통해<br>
    UHT(Unreal Header Tool)가<br>
    UInterface::StaticClass() 같은 리플렉션 정보를 만들고<br>
    이후, IInterface 에 선언된 UFUNCTION 들을 '인터페이스 메시지'로 등록<br>
    이를 통해<br>
    - `obj->GetClass()->ImplementsInterface(UYourInterface::StaticClass())`로 인터페이스 상속 여부 체크<br>
       (**리플렉션** 정보 존재로 인하여 파악 가능)<br>
    - `IYourInterface::Execute_YourFunc(obj, args...)` 로 C++/블루프린트의 구현을 통일된 경로로 호출 가능<br>

- 기본적으로 '상태'를 표현하는 멤버 변수는<br>
  '인터페이스'가 아니라 실제 '구현 클래스'에서<br>
  선언하는 것이 정석<br>

### C++ 에서 다루는 방식

- BP / C++ 양쪽을 고려 호출 코드<br>

```cpp
void TryInteract(UObject* Target, AActor* InstigatorActor)
{
    if (Target && Target->GetClass()->ImplementsInterface(UInteractable::StaticClass()))
    {
        IInteractable::Execute_Interact(Target, InstigatorActor); // BP/C++ 둘 다 커버
    }
}
```

C++이란 것이 확실한 경우는 C++ Interface로 Cast 하여 사용도 가능하다<br>
(다만 이때는 BP면 nullptr을 return하므로 Execute_ 방식이 안전)<br>

- TScriptInterface 타입

```cpp
// UPROPERTY 참조를 통해서
UPROPERTY(EditAnywhere, BlueprintReadWrite)
TScriptInterface<UInteractable> InteractableRef; // U* 타입 사용 (UHT가 요구)

// 일반 C++ (파라미터 / 리턴)
TScriptInterface<IInteractable> InteractIt;
```

### 인터페이스와 관련된 팁들

- '상태'는 '구현 클래스'에 '행위'는 '인터페이스'에<br>
  : 인터페이스에 '데이터/로직'이 들어가지 말 것<br>

- `Execute_` 를 통한 호출이 안전<br>
  (BP를 커버)<br>

- 단순히 'Tag'만 필요하다면<br>
  '빈 인터페이스' 대신에 Gameplay Tag 나 마킹용 컴포넌트를 고려해볼 것<br>

- 인터페이스는 '결합도'와 연관<br>
  : 상호참조/의존이 심해질수록<br>
  **인터페이스를 통한 DI(Dependency Injection, 의존성 주입)** 패턴을 추천<br>
  (외부 클래스와 직접 연결되지 않는 방식)<br>

