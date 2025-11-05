---
title: "김하연 튜터님 강의 - 'Component 기반 모듈러 캐릭터 시스템'"
date : "2025-11-03 12:00:00 +0900"
last_modified_at: "2025-11-03T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Component
  - Modular
---

# Component 기반 모듈러 캐릭터 시스템에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. Epic Games는 왜 Lyra를 만들었나? 🎮

Unreal의 철학을 배우는 챌린지 반<br>
그렇기에 Lyra 프로젝트는 아주 좋은 교재<br>
(그래도 무작정 보는 것보다 핵심을 짚어 효율적인 학습을 진행)<br>

## 1.1. Fortnite의 진화와 Epic의 해답

- **배틀로얄 → 플랫폼**

> 포트나이트는 하나의 게임이 여러 장르를 포함하는 플랫폼으로 진화
> 
- **Epic Games의 3대 질문**
    1. 확장 가능한 게임 구조는?
    2. UE5의 정석적 사용법은?
    3. AAA 게임 구조를 어떻게 가르칠까?
- **해답 → Lyra Starter Game**
    - 단순 데모 ❌
    - 실전 교과서 ✅

## 1.2. Modular Gameplay의 핵심

- 기존 방식의 문제
    
```cpp
class AFortniteCharacter : public ACharacter
{
    // 전투, 수영, 춤, 운전, 건축... 모든 기능이 한 곳에!
    // → 수천 줄의 괴물 클래스
};
```
    
- Lyra의 해결책
    
```cpp
class ALyraCharacter : public ACharacter
{
    // 거의 텅 빈 캐릭터!
};

// 기능은 Component로 분리
UBuildingComponent  // 건축할 때만
USwimmingComponent  // 물속일 때만
UVehicleComponent   // 차량 탑승 시
```
    
> 결과: 하나의 캐릭터가 상황에 따라 다른 게임처럼 동작
> 

컴포넌트에 기능을 부여하여<br>
런타임 상황에 따라 이러한 기능을 조립하여 사용<br>

## 1.3. Component 시스템의 실전 사례

- Overwatch - 상속의 한계
    - **문제 상황: 아나는 어떤 클래스?**
        - 저격 (DPS)
        - 힐 (Support)
        - 다중 역할 → 상속 구조 붕괴
    - **해결책: Component 기반**

```cpp
class HeroBase
{
    UHealthComponent* Health;
    UAbilityComponent* PrimaryFire;
    UAbilityComponent* SecondaryFire;
    // 기능 단위로 조립
};
```

컴포넌트는 단순히 코드를 '깔끔히'하는 용도로 끝나지 않음<br>
(다중 상속을 피하는 용도로 사용이 가능)<br>

컴포넌트 기반 구조로 작성하여<br>
기능 단위의 조립<br>

- GTA - 동적 차량 시스템
    - **문제: 수백 종류의 차량**
        
```cpp
// ❌ 나쁜 예
class AVehicle
{
    // 기본
    UEngineComponent* Engine;
    UWheelComponent* Wheels[4];
    
    USirenComponent* Siren;     // 경찰차만 필요
    UTurretComponent* Turret;   // 탱크만 필요
    UFlightComponent* Flight;   // 비행기만 필요
    // 대부분이 쓸모없는 기능
};
```

- **해결: 런타임 조합**
        
```cpp
// ✅ 좋은 예
void ConvertToPoliceVehicle(AVehicle* Vehicle)
{
    USirenComponent* Siren = NewObject<USirenComponent>(Vehicle);
    Vehicle->AddComponent(Siren);
}
```

- 언리얼 5의 구조 원칙<br>
    1. **Modular** – 기능은 분리하고, 조립은 자유롭게<br>
    2. **Scalable** – 소형 인디부터 대형 AAA까지 대응<br>
    3. **Collaborative** – 수십 명이 동시에 작업 가능<br>

런타임에 컴포넌트를 붙이는 방식으로<br>
필요한 기능만 붙일 수 있음<br>

모든 기능을 최대한 분리하여 조립<br>
이러한 방식을 통해 다양한 크기의 문제를 해결할 수 있음<br>
+ 협업에 특화<br>

# 2. Component 기초 개념 🧩

## 2.1. Component란?

- **행동은 컴포넌트가 한다**
    
```cpp
class AActor
{
    FTransform Transform;               // 위치
    TArray<UActorComponent*> Components; // 기능들
}
```
    
- Actor = 게임 세상에 존재하는 ‘물건’ 또는 ‘존재’<br>
- Component = 그 물건이 할 수 있는 ‘기능’<br>
- Component의 두 가지 종류<br>

- 1. `UActorComponent` – 기능만 있음 (비공간적)<br>
    - 위치 정보 없음<br>
    - 공간 상 존재하지 않음<br>
    - **로직만 처리**하는 뇌 같은 존재<br>
    - 체력, 인벤토리, AI 등 (로직 전용)<br>
        
```cpp
UHealthComponent        // 체력 계산
UInventoryComponent     // 아이템 보관
UAIPerceptionComponent  // AI 감지
```

- 2. `USceneComponent` – 위치가 있음 (공간적)<br>
    - 위치, 회전, 크기를 가짐 (`RelativeTransform`)<br>
    - 다른 컴포넌트에 **부착 가능**<br>
    - 게임 월드에서 실제로 **공간에 존재**<br>
    - 카메라, 메쉬, 충돌 등 (위치 필요 기능)<br>
        
```cpp
UCameraComponent        // 카메라 위치
UStaticMeshComponent    // 3D 모델
UCapsuleComponent       // 충돌 캡슐
```
        
- 둘의 차이점 요약<br>
        
| 항목 | `UActorComponent` | `USceneComponent` |
| --- | --- | --- |
| 위치 정보 | ❌ 없음 | 있음 (`RelativeTransform`) |
| 월드 배치 | ❌ 안 됨 | 씬 상 위치 존재 |
| 부착 관계 | ❌ 불가능 | 부모-자식 연결 가능 |
| RootComponent 가능 | ❌ 안 됨 | 가능 (Actor 위치 기준) |
| 렌더링/충돌 처리 | ❌ 불가 | 하위 타입만 가능 (`UPrimitiveComponent`) |
| 성능 부담 | 낮음 (가벼움) | 상대적 비용 있음 (Transform 계산 등) |


- Component 계층도 (요약)<br>
    
```
UActorComponent (논리형)
│   ├── UHealthComponent
│   ├── UInventoryComponent
│   └── UAIPerceptionComponent
│
└── USceneComponent (위치형)
    ├── UCameraComponent
    ├── UAudioComponent
    └── UPrimitiveComponent
        ├── UStaticMeshComponent
        ├── USkeletalMeshComponent
        └── UCapsuleComponent
```

USceneComponent의 하위 PrimitiveComponent가<br>
Scene에서 '렌더링'되는 녀석들의 기반<br>

'RootComponent'가 곧 'Actor'의 기준점인 역할<br>
(Actor가 World에 위치되기 위하여 반드시 RootComponent - 위치 배치용이 필요)<br>

## 2.2. Component 생성 방법

## 방법 1: 블루프린트 에디터

```
1. 블루프린트 열기
2. Components 탭 → "+" 클릭
3. 원하는 Component 선택
4. 자동으로 RootComponent에 부착
```

### ✅ 장점

- **빠르고 시각적**이다 (디자이너도 쉽게 조작 가능)<br>
- 구조가 바로 에디터에서 눈에 보여서 초보자에게 친숙<br>
- 반복적으로 붙는 컴포넌트에 적합 (UI, Audio 등)<br>

작은 프로젝트라면 80%를 이방식으로 작성해도 문제 x<br>

### ❌ 단점

- **조건부 생성 불가능** (게임 중에는 못 바꿈)<br>
- 동적으로 붙거나 제거되는 구조는 구현 어려움<br>
- 클래스에 따라 자동 세팅하고 싶은 경우 제약이 많음<br>

아이템을 먹었을때, 컴포넌트를 부착하는 등의 방식은 불가<br>
+ 해당 컴포넌트들이 고정됨<br>
(유지보수 x)<br>

코드와의 연결도 힘들어짐<br>
(다만 여전히 직관적인 점 - 디자이너가 제작 쉬움)<br>

## 방법 2: C++ 생성자

```cpp
AMyCharacter::AMyCharacter()
{
    // 논리 컴포넌트
    HealthComponent = CreateDefaultSubobject<UHealthComponent>(TEXT("Health"));

    // 위치 컴포넌트 (부착 필요)
    SpringArmComponent = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
    SpringArmComponent->SetupAttachment(RootComponent);
}
```

### ✅ 장점

- 컴포넌트가 항상 Actor에 붙어 있음 (신뢰성 ↑)<br>
- **퍼포먼스 최적화**: 게임 시작 전에 모든 세팅이 완료됨<br>
- 블루프린트에서 **값만 덮어쓰기 가능** (Override friendly)<br>

퍼포먼스 면에서 좋음<br>
안정적이고, BP 쪽에 커스터마이징 가능<br>

(공통적으로 가져야 하는 경우는 이 방식이 안정적이고 성능상 좋다)<br>

### ❌ 단점

- 조건적 생성 어려움 (if문 같은 로직 불가)<br>
- 생성자 안에서는 **런타임 정보 접근 불가** (월드 상태 등)<br>
- 너무 많은 걸 붙이면 **유연성 떨어짐**<br>

여전히 고정적임<br>
(생성자에서 런타임 정보를 고려하기 힘듦)<br>

## 방법 3: 런타임 동적 생성

```cpp
void AMyCharacter::AddAbility(TSubclassOf<UAbilityComponent> AbilityClass)
{
    if (!AbilityClass) return;

    UAbilityComponent* NewAbility = NewObject<UAbilityComponent>(this, AbilityClass);

    if (NewAbility)
    {
        NewAbility->RegisterComponentWithWorld(GetWorld());
    }
}
```

### ✅ 장점

- 매우 유연함: **조건에 따라 언제든지 붙였다 뗐다 가능**<br>
- **아이템 획득 / 캐릭터 변신 / 스킬 습득**처럼 실시간 변화에 적합<br>
- 에디터 설정에 의존하지 않음<br>

RegisterComponentWithWorld : 생성,초기화 후 World에 등록하여 생성<br>
(World에 안붙이면 GC가 수거해갈 수 있음...)<br>

게임 도중에 조건을 만족하면 컴포넌트를 부착할 수 있음<br>
상황에 따라 컴포넌트 (기능)을 붙였다가 떼는 방식이 장점<br>
(런타임에 가능하다는 점이 매우 큰 장점)<br>

### ❌ 단점

- **구조 추적이 어렵다** (코드 따라가야 함)<br>
- 생성 순서, 부착 순서, BeginPlay 호출 시점 등 조심해야 할 게 많음<br>
- 디버깅이 상대적으로 어려움<br>

구조가 눈에 잘 안보임<br>
(BP 확인하기 힘듦)<br>

순서 고려 문제도 존재함<br>
-> 디버깅이 힘들어짐<br>

## 2.3. Lyra의 동적 조립 시스템

## 전통적 방식

```cpp
ALyraCharacter::ALyraCharacter()
{
    // 모든 Component를 하드코딩
    HealthComponent = CreateDefaultSubobject<...>();
    CombatComponent = CreateDefaultSubobject<...>();
    // 변경 불가능한 구조
}
```

일반적인 방식<br>
(사실상 하드코딩?)<br>

구조 변경이 안됨<br>

## **Lyra 방식**

- 단 하나의 컴포넌트<br>

PawnExtComponent를 통하여<br>
점차 '확장'해 나감<br>
    
```cpp
ALyraCharacter::ALyraCharacter()
{
    // 단 하나의 Component만!
    PawnExtComponent = CreateDefaultSubobject<ULyraPawnExtensionComponent>("PawnExt");
}
```

- PawnData: Component 조립 설계도<br>
  (데이터 에셋)<br>

`어떠한 컴포넌트를 붙여주세요`라는 용도의 데이터 에셋<br>
이를 통해 캐릭터를 런타임 중에 완성함<br>

PrimaryData Asset은 기획자가 접근하기 좋은 편<br>

```cpp
UCLASS()
class ULyraPawnData : public UPrimaryDataAsset
{
    UPROPERTY(EditDefaultsOnly)
    TArray<TSubclassOf<UActorComponent>> Components;

    UPROPERTY(EditDefaultsOnly)
    TArray<TSubclassOf<ULyraAbilitySet>> AbilitySets;
};
```
    
- 런타임 조립 과정<br>

등록된 컴포넌트 등을 캐릭터에 부착<br>
(PawnExtComp가 처리)<br>

```cpp
void ULyraPawnExtensionComponent::SetupPawn(const ULyraPawnData* PawnData)
{
    for (TSubclassOf<UActorComponent> CompClass : PawnData->Components)
    {
        UActorComponent* NewComp = NewObject<UActorComponent>(GetOwner(), CompClass);
        NewComp->RegisterComponentWithWorld(GetWorld());
    }

    for (TSubclassOf<ULyraAbilitySet> AbilitySet : PawnData->AbilitySets)
    {
        GrantAbilitySet(AbilitySet);  // GAS 능력 세트 부여
    }
}
```
    
- 실전 예시 : 직업 변경 시스템<br>
    
```cpp
ULyraPawnData* WarriorData = LoadObject<ULyraPawnData>("Warrior");
ULyraPawnData* PaladinData = LoadObject<ULyraPawnData>("Paladin");

void TransformToPaladin(ALyraCharacter* Character)
{
    auto* PawnExt = Character->GetPawnExtension();

    PawnExt->RemoveAllComponents();
    PawnExt->SetupPawn(PaladinData);
}
```
    
데이터 중심의 설계 방식<br>
- Primary Data Asset에서 설정만 하면<br>
  런타임에서 캐릭터가 변함<br>

- 코드가 짧아짐<br>
- 유연성의 확보<br>

- 협업할 때, 컴포넌트를 나누어 작업 가능<br>

## Lyra 구조가 해결하는 문제

| 문제 | Lyra의 해결 방식 |
| --- | --- |
| 캐릭터마다 다른 기능 | 기능을 런타임에 붙임 (조립식) |
| 구조가 복잡하고 고정됨 | 데이터 기반 유연 구성 |
| 코드 확장 어려움 | 에셋만 바꿔도 구조 확장 |
| 디자이너 작업 어려움 | 에디터에서 캐릭터 구성 가능 |
| 협업 시 충돌 | 컴포넌트 단위 분리 → 충돌 최소화 |

# 3. 델리게이트와 이벤트 기반 통신 🎩

컴포넌트 기반이라면 '이벤트' 기반이 될 가능성이 높아짐<br>
- 다른 컴포넌트가 한 일을 나(컴포넌트)는 어떻게 알지?<br>

## 3.1. 델리게이트란?

> "이벤트가 발생했을 때, 미리 등록된 함수들을 호출하는 시스템"
> 

- **방송국 시스템**
    - HealthComponent = 방송국
    - HUD, Sound, GameMode = 청취자
    - 방송국은 누가 듣는지 모름
- ❌ 잘못된 방식: 직접 참조
    
```cpp
void TakeDamage(float Damage)
{
    Health -= Damage;

    // 직접 호출 = 지옥의 시작
    UIComp->UpdateHealthBar(Health);
    SoundComp->PlayHurtSound();
    CombatComp->InterruptAttack();
}

// 또는 ..
void Tick(float DeltaTime)
{
    if (OldHealth != CurrentHealth)
    {
        HUD->UpdateHealth(CurrentHealth);
        OldHealth = CurrentHealth;
    }
}
```

매 프레임마다 조건 체크를 하거나(Calling)<br>
연관시켜 함수를 호출<br>

디버깅이 까다로움<br>
(특히 Calling 방식은 디버깅이 매우 어려움)<br>
+ 멀티에서 특히 위험<br>
    
- ✅ 델리게이트 방식: 느슨한 결합
    - 선언 예시
        
```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float, NewHealth);

UPROPERTY(BlueprintAssignable)
FOnHealthChanged OnHealthChanged;
```

- 다른 시스템에서 구독 (예: HUD)<br>

Health 에서 만든 Delegate를 다른 쪽에서 '구독'<br>
        
```cpp
HealthComp->OnHealthChanged.AddDynamic(this, &UMyHUD::UpdateHealthUI);
```

- 이벤트가 발생하면 방송<br>

```cpp
void ULyraHealthComponent::SetHealth(float NewHealth)
{
    CurrentHealth = NewHealth;

    // 구독한 모든 함수들에게 "야 체력 바뀜!" 하고 방송
    OnHealthChanged.Broadcast(CurrentHealth);
}
```

## 3.2. 델리게이트 종류와 특징

C++의 원래 함수 포인터를 다소 쓰기 힘들기에 Unreal 기반의 Delegate 기능을 사용<br>

```cpp

void (*FuncPtr) = &Some;
FuncPtr();
```

void () 함수 포인터도 쓰기 힘듦<br>
- 가져온 특정 객체의 함수를 호출하려 하는데<br>
  해당 객체가 '사라져 있을 수 있음'<br>

그렇기에 Unreal 쪽에서 해당 부분을 체크해주는 Delegate를 사용<br> 

## 델리게이트 4종 비교 요약표

| 종류 | 선언 방식 | 리스너 수 | 블루프린트 연동 | 호출 방식 | 바인딩 방식 |
| --- | --- | --- | --- | --- | --- |
| **단일** | `DECLARE_DELEGATE` | 1개 | ❌ | `Execute()` | `BindUObject()` |
| **멀티** | `DECLARE_MULTICAST_DELEGATE` | 여러 개 | ❌ | `Broadcast()` | `AddUObject()`, `AddLambda()` |
| **블루프린트 멀티** | `DECLARE_DYNAMIC_MULTICAST_DELEGATE` | 여러 개 | ✅ | `Broadcast()` | `AddDynamic()` |
| **Event 보호형** | `DECLARE_EVENT` | 여러 개 | ❌ | `Broadcast()` | 외부는 `Add()`만 가능 |

---

## 1. `DECLARE_DELEGATE`

```cpp
DECLARE_DELEGATE(FOnActionDone);
FOnActionDone OnDone;

void Setup()
{
    OnDone.BindUObject(this, &UMyClass::Handler);  // 단 하나의 함수 연결
}

void Run()
{
    if (OnDone.IsBound())
        OnDone.Execute();  // 단 하나의 함수 호출
}
```

> 단일 대상, 가장 빠름. 호출 비용이 거의 없음.
> 

- **언제 사용?**<br>
    - Tick 안에서 매프레임 호출해도 부담 없는 경우<br>
    - 단 하나의 시스템만 반응해야 할 때<br>
- **주의**: 블루프린트랑은 절대 안 엮임<br>

매우 빠름<br>
(단 1개만 기억)<br>
(Tick 등에서도 고려는 가능)<br>

- 런타임 성능이 훌륭하지만<br>
  여러 함수를 등록 못하고<br>
  BP와 연동할 수 없음<br>

- 람다 방식을 사용할 수 없음<br>

- 애니메이션 끝났을때 등, 특정 상황에 호출하는 방식 고려 가능<br>
  (A->B 호출이 아니라, A에서 Delegate 만들고 B가 등록하는 방식이긴 함)<br>

## 2. `DECLARE_MULTICAST_DELEGATE`

```cpp
DECLARE_MULTICAST_DELEGATE_OneParam(FOnDamageTaken, float);

FOnDamageTaken OnDamage;

void Setup()
{
    OnDamage.AddUObject(this, &UMyClass::ReactToDamage);  // 객체 함수 연결
    OnDamage.AddLambda([](float Damage) {
        UE_LOG(LogTemp, Warning, TEXT("Damage: %.1f"), Damage);
    });  // 즉석 람다 등록도 가능
}

void TakeDamage(float Amount)
{
    OnDamage.Broadcast(Amount);  // 모든 리스너 호출
}
```

> 여러 구독자. 블루프린트는 안 됨. 빠르고 유연함.
> 

- **언제 사용?**<br>
    - C++에서 여러 시스템이 동시에 반응해야 할 때<br>
- **주의**: 블루프린트에서 연결 불가능. 순수 C++ 전용.<br>

다양한 함수를 등록 가능<br>

- 여전히 BP는 안됨<br>

- 바인딩 해제는 수동 관리<br>
  (삭제된 함수 호출하는 경우가 발생 가능함)<br>

## 3. `DECLARE_DYNAMIC_MULTICAST_DELEGATE`

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnDeath, AActor*, Killer);

UCLASS()
class UMyHealthComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintAssignable)
    FOnDeath OnDeath;  // 블루프린트에서도 연결 가능
};
```

> 블루프린트에서 바인딩 가능. 호출은 느리지만, 확장성 최강.
> 

- **언제 사용?**
    - 블루프린트에서도 이벤트를 연결하고 싶을 때
- **주의**: 런타임 성능은 다른 델리게이트보다 떨어짐 (리플렉션 기반 호출이기 때문에)


BP도 되고, 안정성 있는 호출도 가능함<br>
- 그만큼 성능을 희생함<br>

# 4. `DECLARE_EVENT`

```cpp
DECLARE_EVENT(UGameStateSubsystem, FOnMatchStarted)

class UMyGameState : public UGameStateSubsystem
{
    FOnMatchStarted OnMatchStarted;

public:
    FOnMatchStarted& GetMatchStartedEvent() { return OnMatchStarted; }

    void StartMatch()
    {
        OnMatchStarted.Broadcast();  // 내부에서만 가능
    }
};
```

> 특정 클래스에서만 Broadcast 가능하게 만들고 싶을 때
> 

- **언제 사용?**
    - **보안성**이 중요할 때
    - "내부 로직 외엔 누구도 이걸 호출하지 마라" 하고 싶을 때
- **주의**: 문법이 다소 특이하고 덜 쓰이지만, 고급 구조에서 유용

클래스를 제한하는 방식<br>
- 권한 제어용도<br>
- 외부에서 Event용 함수를 통하여 구독<br>
- BroadCast는 다른 클래스에서 호출 불가<br>
  (다만 복잡한 구조 + BP 연동이 안됨)<br>

# 4. HealthComponent 구현 📁

## 4.1. 헤더 파일 구조

```cpp
UCLASS(ClassGroup=(Lyra), meta=(BlueprintSpawnableComponent))
class ULyraHealthComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    ULyraHealthComponent();

    // 체력이 변경되었을 때 C++에서 사용할 델리게이트
    FLyraHealth_AttributeChanged OnHealthChanged;

    // 체력이 0이 되었을 때 블루프린트에서도 사용할 수 있는 델리게이트
    UPROPERTY(BlueprintAssignable)
    FLyraHealth_DeathEvent OnDeath;

protected:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Lyra|Health")
    float MaxHealth = 100.0f;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Lyra|Health")
    float CurrentHealth;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Lyra|Health")
    bool bIsDead = false;

public:
    virtual void BeginPlay() override;

    UFUNCTION()
    void HandleDamage(AActor* DamagedActor, float Damage,
                      const UDamageType* DamageType, AController* InstigatedBy,
                      AActor* DamageCauser);

    void SetHealth(float NewHealth);

    UFUNCTION(BlueprintCallable, Category = "Lyra|Health")
    void Heal(float Amount) { SetHealth(CurrentHealth + Amount); }

    UFUNCTION(BlueprintPure)
    float GetHealthNormalized() const { return MaxHealth > 0 ? CurrentHealth / MaxHealth : 0; }

private:
    void HandleDeath(AActor* Instigator);
};
```

## 4.2. 핵심 구현부

```cpp
void ULyraHealthComponent::BeginPlay()
{
    Super::BeginPlay();

    AActor* Owner = GetOwner();
    check(Owner);  // 필수 Actor가 없으면 강제 종료

    CurrentHealth = MaxHealth;

    // 언리얼 내장 데미지 이벤트에 바인딩
    Owner->OnTakeAnyDamage.AddDynamic(this, &ULyraHealthComponent::HandleDamage);
}

void ULyraHealthComponent::SetHealth(float NewHealth)
{
    if (FMath::IsNearlyEqual(CurrentHealth, NewHealth))
        return;

    float OldHealth = CurrentHealth;
    CurrentHealth = FMath::Clamp(NewHealth, 0.0f, MaxHealth);

    // 체력 변경 알림 (델리게이트 호출)
    OnHealthChanged.Broadcast(this, OldHealth, CurrentHealth);

    // 사망 조건 체크
    if (CurrentHealth <= 0.0f && !bIsDead)
    {
        HandleDeath(nullptr);
    }
}

void ULyraHealthComponent::HandleDeath(AActor* Instigator)
{
    if (bIsDead)
        return;

    bIsDead = true;

    AActor* Owner = GetOwner();

    UE_LOG(LogLyraHealth, Warning, TEXT("[%s] Died. Killer: %s"),
           *Owner->GetName(),
           Instigator ? *Instigator->GetName() : TEXT("Unknown"));

    // 사망 이벤트 발생 (블루프린트도 수신 가능)
    OnDeath.Broadcast(Owner, Instigator);

    // 상태 태그 등록
    Owner->Tags.Add(FName("Dead"));
}
```

# 5. Lyra의 설계 철학 📌

## 5.1. 4대 원칙

| 원칙 | 의미 | 구현 방법 |
| --- | --- | --- |
| **단일 책임** | 한 Component = 한 역할 | 체력만 관리, 나머진 위임 |
| **이벤트 기반** | 직접 호출 금지 | 델리게이트 Broadcast |
| **데이터 중심** | 설정은 코드가 아닌 에셋 | DataAsset 활용 |
| **확장 가능** | 수정 없이 기능 추가 | 구독만 추가하면 OK |

## 5.2. 왜 이 구조를 배워야 하는가?

### 💼 취업 시장에서

- "Component 패턴의 장점은?"
- "Lyra 구조 분석해보셨나요?"
- 구조적 사고력 증명

### 📁 포트폴리오에서

- 단순 기능 구현 < 구조적 설계
- 실무 수준의 아키텍처 이해도 증명

### 🏭 실무에서

- **AAA**: 100명+ 협업 시 필요
- **인디**: 확장 가능한 구조의 시작점