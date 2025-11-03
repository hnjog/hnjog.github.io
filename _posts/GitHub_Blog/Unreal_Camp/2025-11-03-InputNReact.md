---
title: "김하연 튜터님 강의 - '입력과 상호작용의 구조화'"
date : "2025-11-03 12:00:00 +0900"
last_modified_at: "2025-11-03T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Unreal Input System
  - Enhanced Input
---

# 입력과 상호작용의 구조화에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

## 1. 전통 Input System의 문제와 패러다임 전환 🫤

입력 : 의외로 파고들면 생각할 것이 많은 부분<br>

- 같은 입력이라도 상황에 따라 다르게 처리함<br>
  (핸드폰의 버튼부터, 모바일 게임의 터치 시스템 등등)<br>

### 기존 입력 시스템의 문제점 (UE4)

```cpp
void ACharacter::SetupPlayerInputComponent(UInputComponent* Input)
{
    Input->BindAction("Interact", IE_Pressed, this, &ACharacter::OnInteract);
}

void ACharacter::OnInteract()
{
    if (bIsInCleaningMode)
    {
        if (bNearCarpet)
            CleanCarpet();
        else if (bNearWindow)
            CleanWindow();
    }
    else if (bIsInCombatMode)
    {
        if (bHasWeapon)
            AttackWithWeapon();
        else
            Punch();
    }
    else if (bNearVehicle)
    {
        if (bInVehicle)
            ExitVehicle();
        else
            EnterVehicle();
    }
    // ... 계속 이어짐
}
```

**문제점**

- 상황에 따른 다른 동작 불가능<br>
- 확장성 부족<br>
- 코드 재사용성 낮음<br>

키와 행동이 1:1로 강하게 결합됨<br>
-> 상황에 따라 행동의 정의를 다르게 바꿔주어야 함<br>

- E키에 문열기를 달았다면<br>
  : E키를 또 사용하려면 if로 분기를 사용<br>

### Enhanced Input System의 해결책

```cpp
void ACharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    UEnhancedInputComponent* Input = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    
    // "상호작용하고 싶다"는 의도만 바인딩
    Input->BindAction(InteractAction, ETriggerEvent::Triggered, this, &ACharacter::OnInteract);
}

void ACharacter::OnInteract()
{
    // if문이 없음ㅇㅋ
    // 현재 상황(Context)이 자동으로 의미를 결정
}
```

### 구조의 차이

```cpp
전통 방식
[E키] → [OnInteract 함수] → [내부에서 if문으로 분기] → [행동]

Enhanced 방식
[E키] → [InteractAction (의도)] → [Context가 해석] → [행동]
```

- Context를 통하여 분기문을 지움<br>
- 입력과 행동을 분리 시켜준다<br>

## 2. Enhanced Input 아키텍처 📐

## 2-1. InputAction의 정체 - 데이터와 타입

### 🔹 **InputAction은 단순한 데이터 에셋**

- 클래스: `UInputAction` → `UDataAsset` 상속<br>
- 역할: “입력의 타입과 의미”를 정의하는 설정 파일<br>

그냥 데이터 에셋!<br>
게임에서 사용할 입력 설정 정보를 담아놓은 것<br>

```cpp
enum class EInputActionValueType : uint8
{
    Boolean,   // 참/거짓
    Axis1D,    // 1차원 축
    Axis2D,    // 2차원 벡터
    Axis3D     // 3차원 벡터
};
```

| 타입 | 데이터 예시 | 사용 예 |
| --- | --- | --- |
| Boolean | `true / false` | 점프, 상호작용, 공격 |
| Axis1D | `-1.0 ~ +1.0` | 앞뒤 이동(W/S), 좌우 회전 |
| Axis2D | `(X, Y)` | 평면 이동 (WASD) |
| Axis3D | `(X, Y, Z)` | 3D 회전, 비행, 자유 이동 |

- 엔진이 입력에 대한 수치 값을 제공<br>

### 🔹 Axis2D 예시 (Move Action)

```cpp
void AMyCharacter::Move(const FInputActionValue& Value)
{
    FVector2D Movement = Value.Get<FVector2D>();
    AddMovementInput(GetActorForwardVector(), Movement.Y);
    AddMovementInput(GetActorRightVector(), Movement.X);
}
```

- W, D 동시 입력 → `(0,1)+(1,0)=(1,1)` → 대각선 이동
- 길이 √2 → 정규화 필요 (엔진 옵션으로 자동 처리 가능)

### 🔹 Axis1D로 분리하는 이유

- 카메라 회전은 X/Y 축 민감도를 다르게 조정해야 함
- 따라서 LookX, LookY를 각각 `Axis1D`로 정의

```cpp
IA_LookX → Axis1D  (마우스 X축)
IA_LookY → Axis1D  (마우스 Y축)
```

## 2-2. 입력 처리 파이프라인 개요

입력은 매 프레임 아래 과정을 거침.

```
1단계: [키 입력] 
   물리적으로 E키를 누름
   Raw Input Value = 1.0

2단계: [Modifier 적용]
   입력값을 변환
   예: Scale ×2.0 → 2.0

3단계: [Modified Value]
   변환된 최종 입력값 = 2.0

4단계: [Trigger 평가]
   이 입력이 조건을 만족하는가?
   예: 0.5초 이상 눌렀나?

5단계: [조건 만족 시]
   → InputAction 이벤트 발생
   → 바인딩된 함수 호출
```

- 초당 60프레임이면 60번 처리됨<br>

- 등록한 조건을 만족해야 inputAction에 바인딩한 이벤트 호출!<br>

## 2-3. Modifier - 입력값 변환

| 이름 | 기능 | 예시 |
| --- | --- | --- |
| Negate | 부호 반전 | W(+1), S(+1) → Negate → -1 |
| Scale | 민감도 조절 | 1.0 × 0.5 = 0.5 |
| Dead Zone | 잡음 제거 | 스틱의 미세한 흔들림 무시 |
| Swizzle | 축 재배열 | (X,Y,Z) → (Y,X,Z) |

### 🔹 예시 – Scale Modifier

```cpp
// 마우스 민감도 조절
// UserSetting.Sensitivity = 0.5
Mapping.Modifiers.Add(NewObject<UInputModifier_Scale>());
Modifier->Scale = FVector(0.5f);
```

## 2-4. Trigger - 조건 평가

입력 이벤트가 “언제” 발동되는지를 정의<br>

| Trigger | 조건 | 예시 |
| --- | --- | --- |
| Down | 키가 눌렸는가 | 기본 클릭 |
| Released | 키를 뗐는가 | 활쏘기 |
| Hold | N초 유지 | 차지 공격 |
| HoldAndRelease | N초 유지 후 뗐는가 | 완충 후 발사 |
| Tap | N초 이내 눌렀다 뗐는가 | 더블탭 회피 |
| Pulse | 주기적으로 트리거 | 자동 연사 |

### 🔹 예시 – Hold Trigger (차지 공격)

```cpp
BindAction(AttackAction, ETriggerEvent::Started, this, &AChar::LightAttack);
BindAction(AttackAction, ETriggerEvent::Triggered, this, &AChar::HeavyAttack);
```

- `Started`: 키 눌린 즉시 (약공격)
- `Triggered`: 0.5초 이상 눌렀을 때 (강공격)

## 2-5. InputAction의 생명 주기 - 엔진 내부 처리

```cpp
// UEnhancedPlayerInput.cpp (단순화 버전)
void UEnhancedPlayerInput::ProcessInputStack(float DeltaTime)
{
    // 1️⃣ 활성 IMC 수집
    for (auto& IMC : ActiveContexts)
    {
        for (auto& Mapping : IMC->Mappings)
        {
            if (IsKeyPressed(Mapping.Key))
            {
                // 2️⃣ Modifier 적용
                FInputActionValue Value = Mapping.Key.GetRawValue();
                for (auto& Mod : Mapping.Modifiers)
                    Value = Mod->ModifyRaw(Value);

                // 3️⃣ 같은 Action 값 누적
                ActionValues[Mapping.Action].Accumulate(Value);
            }
        }
    }

    // 4️⃣ Trigger 평가 및 5️⃣ 이벤트 호출
    for (auto& Pair : ActionValues)
    {
        auto Action = Pair.Key;
        auto Value  = Pair.Value;
        auto Event  = EvaluateTriggers(Action, Value, DeltaTime);

        if (Event != ETriggerEvent::None)
            BroadcastInputAction(Action, Event, Value);
    }
}
```

- ActiveContexts는 우선순위에 따라 정렬이 됨<br>
  (UI Context 와 일반 GameContext의 우선순위가 다르면 높은 쪽이 처리)<br>

- 입력값을 누적시킨 후<br>
  마지막에 조건에 만족하는 Input Action들에 대하여 BroadCast<br>

### 🔹 단계 요약

| 단계 | 역할 |
| --- | --- |
| 1 | ActiveContexts(IMC) 순회, 매핑 수집 |
| 2 | Modifier 적용 (값 변환) |
| 3 | 같은 Action 값 누적 (합산) |
| 4 | Trigger 조건 평가 |
| 5 | 이벤트 브로드캐스트 (바인딩 함수 호출) |

## 3. Context 시스템의 실체 - Subsystem과 Stack 🕵🏻

- 활성화된 IMC의 관리자는 ULocalPlayerSubsystem<br>

## 3-1. Subsystem이란?

### 🔹 **언리얼의 Subsystem 계층**

Subsystem은 “특정 범위의 생명주기를 가지는 관리 객체”.<br>

| Subsystem 종류 | 생명 범위 | 예시 |
| --- | --- | --- |
| `UGameInstanceSubsystem` | 게임 전체 | 세이브 데이터 관리 |
| `UWorldSubsystem` | 월드 단위 | AI 네비게이션, 날씨 |
| **`ULocalPlayerSubsystem`** | **플레이어 단위** | ✅ **Enhanced Input 관리** |

- Player 생성 시, 생성<br>
  파괴 시 파괴<br>

- 레벨 변화에 따라서도 유지<br>

### 🔹 **Enhanced Input Subsystem 구조**

```
UGameInstance
 └─ ULocalPlayer (플레이어 1)
      └─ UEnhancedInputLocalPlayerSubsystem
           └─ UEnhancedPlayerInput
 └─ ULocalPlayer (플레이어 2)
      └─ UEnhancedInputLocalPlayerSubsystem
           └─ UEnhancedPlayerInput

```

- 플레이어마다 **독립적인 입력 관리 시스템**<br>
- LocalPlayer 기준이라, Possess가 바뀌어도 유지됨<br>
  → (Spectator로 전환해도 입력 컨텍스트 유지 가능)<br>


### 🔹 **Subsystem 접근 경로**

```cpp
void UContextManagerComponent::InitializeSubsystem()
{
    AActor* Owner = GetOwner();  // 보통 캐릭터
    APlayerController* PC = Cast<APlayerController>(Owner->GetInstigatorController());
    ULocalPlayer* LocalPlayer = PC->GetLocalPlayer();

    CachedSubsystem =
        ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(LocalPlayer);
}

```

경로 요약:<br>

```
Actor → PlayerController → LocalPlayer → Subsystem
```

> 게임 시작 직후나 AI 캐릭터는 PlayerController가 `nullptr`일 수 있음.
> 
> 
> 항상 **null 체크** 필수.
> 


## 3-2. Priority System - 우선순위의 실제

### 🔹 **내부 구조 (단순화)**

```cpp
class UEnhancedInputLocalPlayerSubsystem : public ULocalPlayerSubsystem
{
    TArray<FInputMappingContextAndPriority> AppliedInputContexts;

    void AddMappingContext(UInputMappingContext* IMC, int32 Priority);
    void RemoveMappingContext(UInputMappingContext* IMC);
};

struct FInputMappingContextAndPriority
{
    UInputMappingContext* MappingContext;
    int32 Priority;

    bool operator<(const FInputMappingContextAndPriority& Other) const
    {
        return Priority > Other.Priority; // 높은 Priority가 먼저
    }
};
```

- AppliedInputContexts<br>
  : 활성화된 IMC 관리<br>

- IMC를 관리하는 구조체를 통하여<br>
  우선순위에 따라 IMC를 정렬<br>
  (TArray 이지만, 값이 추가되거나 할때, 재정렬 하는 방식)<br>

### 🔹 **AddMappingContext 동작 흐름**

```cpp
void AddMappingContext(UInputMappingContext* IMC, int32 Priority)
{
    // 이미 등록되어 있으면 Priority 업데이트
    for (auto& Ctx : AppliedInputContexts)
        if (Ctx.MappingContext == IMC)
        {
            Ctx.Priority = Priority;
            AppliedInputContexts.Sort();
            return;
        }

    // 새로 추가
    AppliedInputContexts.Add({IMC, Priority});
    AppliedInputContexts.Sort();
}

```

- IMC들은 항상 Priority 순서로 정렬됨.<br>
- 높은 Priority일수록 먼저 처리.<br>

### 🔹 **처리 순서 예시**

| 순서 | IMC 이름 | Priority | ConsumeInput | 결과 |
| --- | --- | --- | --- | --- |
| ① | IMC_UI | 100 | ✅ | UI 입력만 처리, 게임 입력 차단 |
| ② | IMC_Combat | 50 | ❌ | UI에서 처리 안한 입력만 통과 |
| ③ | IMC_Default | 0 | ❌ | 기본 입력 처리 |

```
→ 위에서 아래로 순회
   → ConsumeInput = true면 중단
```

### 🔹 **ConsumeInput 예시**

| 상황 | ConsumeInput | 결과 |
| --- | --- | --- |
| UI 메뉴 | ✅ | UI만 반응, 게임 무시 |
| 인게임 HUD | ❌ | HUD도 반응, 게임도 반응 |

```cpp
// 실제 동작
if (Mapping.bConsumeInput)
    break; // 아래 IMC로 전파 중단
```

### 🔹 **실전 예시 – UI 열기/닫기**

```cpp
void OpenMenu()
{
    Subsystem->AddMappingContext(IMC_Menu, 100);
}

void CloseMenu()
{
    Subsystem->RemoveMappingContext(IMC_Menu);
}
```

| 상태 | Subsystem 내 IMC |
| --- | --- |
| 기본 | IMC_Default(0) |
| 메뉴 열림 | IMC_Menu(100), IMC_Default(0) |
| 메뉴 닫힘 | IMC_Default(0) |

E키 → “문 열기” → “UI 클릭” 으로 동적 전환.<br>

## 3-3. ContextManager의 역할 - 의미론적 레이어

### 🔹 **Subsystem의 한계**

| 한계 | 설명 |
| --- | --- |
| 상황 개념 없음 | IMC 객체만 알고 “청소 모드” 의미를 모름 |
| Stack 구조 없음 | Priority 기반, “최근 추가” 개념 없음 |
| 중복 방지 없음 | 같은 IMC 여러 번 추가 가능 |
| 타입 안정성 부족 | 문자열/객체 기반, enum 미지원 |
| 상황 추적 불가 | “이전 컨텍스트” 정보 없음 |

- SubSystem 자체는 그냥 Priority 기준으로 정렬하여 관리만 함<br>
  (IMC에 뭐가 들었는지 관심 없음)<br>

- 별도의 매니저를 통하여 위의 단점을 보조함<br>

### 🔹 **우리가 추가하는 구조**

```cpp
USTRUCT()
struct FActiveContext
{
    EGameplayContext Context;              // enum 기반 이름
    UInputMappingContext* MappingContext;  // 실제 IMC
    float ActivationTime;                  // 활성화 시점
};

UCLASS()
class UContextManagerComponent : public UActorComponent
{
    TArray<FActiveContext> ContextStack;
    UEnhancedInputLocalPlayerSubsystem* CachedSubsystem;
    TMap<EGameplayContext, UInputMappingContext*> ContextMappings;
};

```

- 자체적으로 ContextManageComponent를 구현?<br>

### 🔹 **PushContext() 구현**

```cpp
void UContextManagerComponent::PushContext(EGameplayContext NewContext)
{
    if (!CachedSubsystem) return;

    // 중복 방지
    for (auto& Ctx : ContextStack)
        if (Ctx.Context == NewContext) return;

    UInputMappingContext* IMC = *ContextMappings.Find(NewContext);
    int32 Priority = ContextStack.Num(); // Stack 크기 = Priority
    CachedSubsystem->AddMappingContext(IMC, Priority);

    ContextStack.Add({NewContext, IMC, GetWorld()->GetTimeSeconds()});
}

```

### 🔹 **PopContext() 구현**

```cpp
void UContextManagerComponent::PopContext()
{
    if (ContextStack.Num() == 0) return;

    const FActiveContext& Top = ContextStack.Last();
    CachedSubsystem->RemoveMappingContext(Top.MappingContext);
    ContextStack.Pop();
}
```

> LIFO 구조 – 마지막으로 들어간 Context가 먼저 제거됨.
> 

### 🔹 **GetCurrentContext()**

```cpp
EGameplayContext UContextManagerComponent::GetCurrentContext() const
{
    return ContextStack.Num() == 0 ?
        EGameplayContext::Default :
        ContextStack.Last().Context;
}
```

### 🧠 **의미론적 레이어의 가치**

```
[우리 레이어]
  EGameplayContext::WindowCleaning
  EGameplayContext::CombatMode
    ↓ (ContextMappings)
[엔진 레이어]
  IMC_WindowCleaning
  IMC_Combat
    ↓ (Subsystem)
[입력 처리]
  E키 → InteractAction → "유리 닦기"
```

- 레이어를 자체적으로 하나 두어<br>
  엔진과 보조하여 사용함으로서<br>
  입력 처리를 더 효율적으로 구현 가능<br>

- 상황에 따른 다른 동작을 더 세부적으로 분류 가능<br>
  (State Machine까지 구현하지 않을 수 있음)<br>

## 3-4. Priority 전략 - 우리의 선택

### 🔹 **문제: Priority 0 고정의 위험**

```cpp
CachedSubsystem->AddMappingContext(IMC, 0);
```

- 모든 IMC가 같은 우선순위면 순서 예측 불가<br>
- 엔진 버전에 따라 동작이 달라질 수 있음<br>

### 🔹 **해결: Stack 크기를 Priority로 사용**

```cpp
int32 Priority = ContextStack.Num();
CachedSubsystem->AddMappingContext(IMC, Priority);
```

- Priority를 Stack값으로 지정하여<br>
  우선순위를 확실히 나누는 방식<br>

| Context | Priority | 의미 |
| --- | --- | --- |
| Default | 0 | 기본 |
| FloorCleaning | 1 | 위에 쌓임 |
| WindowCleaning | 2 | 최우선 |

> 숫자가 클수록 최근에 추가된 Context.
> 
> 
> 항상 “가장 최신 상황이 최우선”.
> 

### 🔹 **Pop 후 자동 복귀**

```cpp
PopContext(); // 최상위 제거 → 아래 Context가 우선순위 승계
```

### ⚡ **실제 동작 예시**

| 단계 | Stack | Subsystem 상태 | 결과 |
| --- | --- | --- | --- |
| 1 | [Default(0)] | IMC_Default(0) | 문 열기 |
| 2 | [Default, Floor(1)] | IMC_Floor(1), IMC_Default(0) | 카펫 청소 |
| 3 | [Default, Window(1)] | IMC_Window(1), IMC_Default(0) | 유리 닦기 |

## 3-5. 전체 구조 요약

```
[플레이어 입력]
     ↓
[InputAction]  ← 값/타입
     ↓
[ContextManager]  ← 의미/상황 (우리 시스템)
     ↓
[Subsystem]  ← IMC 관리 (엔진 시스템)
     ↓
[Input Mapping Context]  ← 실제 키 해석
     ↓
[함수/행동 실행]
```

| 항목 | 역할 |
| --- | --- |
| Subsystem | IMC를 Priority 순서로 관리 |
| ConsumeInput | 입력 전파 제어 |
| ContextManager | enum 기반 의미 추가 |
| Stack 구조 | LIFO, 동적 전환 지원 |
| Priority 전략 | Stack 크기 = Priority |
| GetCurrentContext | 현재 상황 조회 API |

## 4. State Machine - 행동 허가증 🛸

- '상태 확인'에 대한 처리<br>

## 4-1. Context vs. State - 외적 상황과 내적 상태

| 구분 | Context | State |
| --- | --- | --- |
| **의미** | 외적 상황 (환경/모드) | 내적 상태 (현재 동작) |
| **질문** | “어떤 환경인가?” | “지금 무엇을 하고 있는가?” |
| **변경 주체** | 플레이어의 선택 | 시스템의 전환 |
| **예시** | 전투 모드, 청소 모드, 차량 탑승 | Idle, Cleaning, Attacking, Dead |
| **지속성** | 명시적 전환까지 유지 | 조건 충족 시 자동 변경 |

> 같은 장소(Context)라도, **상태(State)**에 따라 가능한 행동이 다르다.
> 

Context : 플레이어를 둘러싼 '환경'<br>
    (플레이어의 선택에 따라 환경의 변화)<br>

State : 플레이어의 '상태'<br>
    (시스템을 따르는 '게임 캐릭터'의 현재 상황)<br>
    (실제 있는 데이터의 상황으로 판단하는 상태)<br>

### 🔹 **코드 비교**

### ❌ Context만 있을 때 (문제 발생)

```cpp
void OnInteract()
{
    if (CurrentContext == FloorCleaning)
        CleanFloor();  // 계속 호출됨 → 애니메이션 중복
}
```

### ✅ State까지 고려한 경우

```cpp
void OnInteract()
{
    if (CurrentContext == FloorCleaning && CurrentState == Idle)
    {
        CleanFloor();
        CurrentState = Cleaning;
    }
}
```

> State가 “허가증” 역할을 함.
> 
> 
> Idle → Cleaning 전환은 허용, Cleaning 중이면 거부.
> 

## 4-2. Finite State Machine (FSM) 패턴

### 🔹**구성요소**

1. **State** – 가능한 상태 집합<br>
2. **Transition** – 상태 간 이동 규칙<br>
3. **Guard** – 전환 허용 조건<br>

### 🔹 **1️⃣ State 정의**

```cpp
enum class ECleaningState : uint8
{
    Idle,
    Cleaning,
};
```

> Enum 사용으로 의미 명확, 타입 안정성 확보.
> 

### 🔹 **2️⃣ Transition 규칙 (다이어그램)**

```
   ┌──────────┐
   │   Idle   │◄──────────────┐
   └────┬─────┘               │
        │                     │
        │ StartCleaning()     │ InterruptCleaning()
        │ (E키 입력)          │ (이동/점프/시간 종료)
        ↓                     │
   ┌──────────┐               │
   │ Cleaning │───────────────┘
   └──────────┘
```

### 🔹 **3️⃣ Guard (조건 함수)**

```cpp
bool CanStartCleaning() const { return CurrentState == Idle; }
bool CanMove() const          { return CurrentState != Cleaning; }
bool CanJump() const          { return CurrentState != Cleaning; }
```

## State Machine 클래스 구조

```cpp
class UCleaningStateMachine : public UActorComponent
{
    GENERATED_BODY()

private:
    ECleaningState CurrentState = ECleaningState::Idle;
    float StateStartTime = 0.f;
    float AnimationDuration = 0.f;

    UPROPERTY()
    class AEnhancedChallengeCharacter* OwnerCharacter;

public:
    // 상태 전환
    void StartCleaning(float Duration);
    void InterruptCleaning();

    // 조건 검사
    bool CanStartCleaning() const;
    bool CanMove() const;
    bool CanJump() const;

    // 조회
    ECleaningState GetCurrentState() const { return CurrentState; }

protected:
    virtual void BeginPlay() override;
    virtual void TickComponent(float DeltaTime, ...) override;

private:
    void SetState(ECleaningState NewState);
    void StopCurrentMontage();
};
```

```cpp
void UCleaningStateMachine::BeginPlay()
{
    Super::BeginPlay();
    
    // 캐릭터 참조 캐싱
    OwnerCharacter = Cast<AEnhancedChallengeCharacter>(GetOwner());
    
    if (!OwnerCharacter)
    {
        UE_LOG(LogTemp, Error, TEXT("StateMachine: Owner is not EnhancedChallengeCharacter!"));
    }
}
```

```cpp
void UCleaningStateMachine::StartCleaning(float Duration)
{
    // 1단계: Guard 체크
    if (!CanStartCleaning())
    {
        UE_LOG(LogTemp, Warning, TEXT("Cannot start cleaning: wrong state (%d)"), 
            (int32)CurrentState);
        return;
    }
    
    // 2단계: Exit Current State
    // Idle 상태에서는 특별히 할 게 없음
    // 다른 상태였다면 여기서 정리 작업
    
    // 3단계: Transition (상태 변경)
    ECleaningState OldState = CurrentState;
    CurrentState = ECleaningState::Cleaning;
    
    // 4단계: Enter New State (새 상태 초기화)
    StateStartTime = GetWorld()->GetTimeSeconds();
    AnimationDuration = Duration;
    
    // 5단계: Event (상태 변경 알림)
    UE_LOG(LogTemp, Log, TEXT("State changed: %d -> %d"), 
        (int32)OldState, (int32)CurrentState);
    
    // 델리게이트로 외부에 알림 (나중에 설명)
    OnStateChanged.Broadcast(OldState, CurrentState);
}
```

```cpp
void UCleaningStateMachine::TickComponent(
    float DeltaTime, 
    ELevelTick TickType, 
    FActorComponentTickFunction* ThisTickFunction)
{
    Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
    
    // Update State: Cleaning 상태일 때만 체크
    if (CurrentState == ECleaningState::Cleaning)
    {
        // 경과 시간 계산
        float CurrentTime = GetWorld()->GetTimeSeconds();
        float ElapsedTime = CurrentTime - StateStartTime;
        
        // 애니메이션 시간이 다 됐나?
        if (ElapsedTime >= AnimationDuration)
        {
            // 자동 전환: Cleaning → Idle
            UE_LOG(LogTemp, Log, TEXT("Cleaning finished (%.2fs), returning to Idle"), 
                ElapsedTime);
            
            SetState(ECleaningState::Idle);
        }
    }
}
```

```cpp
void UCleaningStateMachine::InterruptCleaning()
{
    // 현재 Cleaning 상태일 때만
    if (CurrentState == ECleaningState::Cleaning)
    {
        UE_LOG(LogTemp, Log, TEXT("Cleaning interrupted!"));
        
        // 애니메이션 정지
        StopCurrentMontage();
        
        // 상태 전환
        SetState(ECleaningState::Idle);
    }
}
```

```cpp
void UCleaningStateMachine::SetState(ECleaningState NewState)
{
    // 같은 상태로의 중복 전환 방지
    if (CurrentState == NewState)
    {
        UE_LOG(LogTemp, Warning, TEXT("Already in state: %d"), (int32)NewState);
        return;
    }
    
    // 이전 상태 저장
    ECleaningState OldState = CurrentState;
    
    // Exit Old State
    switch (OldState)
    {
    case ECleaningState::Cleaning:
        // 청소 상태를 빠져나갈 때 정리
        StopCurrentMontage();
        break;
        
    case ECleaningState::Idle:
        // Idle은 정리할 게 없음
        break;
    }
    
    // Transition
    CurrentState = NewState;
    
    // Enter New State
    switch (NewState)
    {
    case ECleaningState::Idle:
        // Idle은 초기화할 게 없음
        break;
        
    case ECleaningState::Cleaning:
        // Cleaning은 StartCleaning에서 초기화함
        break;
    }
    
    // Event
    UE_LOG(LogTemp, Log, TEXT("State changed: %d -> %d"), 
        (int32)OldState, (int32)NewState);
    
    // 델리게이트 브로드캐스트
    OnStateChanged.Broadcast(OldState, NewState);
}
```

## 4-3. Animation과 State의 분리

- Animation의 재생은 믿기 힘든 상태<br>

- State의 제어하에 존재해야<br>
  논리적인 버그가 생기지 않음<br>

### ⚠️ **잘못된 방식**

```cpp
PlayAnimMontage(CleaningMontage); // 애니메이션만 실행
```

- 중단되거나 스킵 가능 → 신뢰도 낮음
- 논리적 상태 불일치 발생

### ✅ **올바른 방식**

```cpp
void OnInteract()
{
    // 1. State 확인
    if (!StateMachine->CanStartCleaning())
    {
        return;  // 불가능하면 아무것도 안 함
    }
    
    // 2. 애니메이션 재생
    float Duration = PlayAnimMontage(CleaningMontage);
    
    // 3. State Machine에 알림 (여기가 핵심!)
    StateMachine->StartCleaning(Duration);
}
```

> State가 논리의 중심,
> 
> 
> Animation은 표현만 담당.
> 

### **StopCurrentMontage()**

```cpp
void UCleaningStateMachine::StopCurrentMontage()
{
    // 캐릭터가 유효한지 확인
    if (!OwnerCharacter)
    {
        UE_LOG(LogTemp, Warning, TEXT("OwnerCharacter is null!"));
        return;
    }
    
    // AnimInstance 가져오기
    USkeletalMeshComponent* Mesh = OwnerCharacter->GetMesh();
    if (!Mesh)
    {
        return;
    }
    
    UAnimInstance* AnimInstance = Mesh->GetAnimInstance();
    if (!AnimInstance)
    {
        return;
    }
    
    // 몽타주가 재생 중인지 확인
    if (AnimInstance->IsAnyMontagePlaying())
    {
        // 0.25초 블렌드 타임으로 부드럽게 정지
        AnimInstance->Montage_Stop(0.25f);
        
        UE_LOG(LogTemp, Log, TEXT("Stopped montage with 0.25s blend"));
    }
}
```

> 0.25초 블렌드 타임으로 부드럽게 전환.
> 

## 4-4. Single Source of Truth

모든 시스템은 **StateMachine만 참조**해야 한다.<br>

- 조건 확인을 StateMachine만 하면 되기에 일관성이 생김<br>

```cpp
// UI 시스템
void UpdateHUD()
{
    switch (StateMachine->GetCurrentState())
    {
    case Idle:
        HUD->Hide();
        break;
    case Cleaning:
        HUD->ShowMessage("Cleaning...");
        break;
    }
}

// 전투 시스템
void TakeDamage(float Damage)
{
    // 청소 중엔 피해 안 받음
    if (StateMachine->GetCurrentState() == Cleaning)
    {
        return;
    }
    
    Health -= Damage;
}

// 다른 입력
void OnAttack()
{
    // 청소 중엔 공격 불가
    if (StateMachine->GetCurrentState() != Idle)
    {
        return;
    }
    
    Attack();
}
```

> StateMachine이 논리적 진실의 단일 원천(Single Source of Truth).
> 

## 5. Enhanced Input System이 추구한 설계 원칙 🧠

## 5-**1. 단일 책임 원칙 (SRP)**

> Single Responsibility Principle
> 
> 
> → 하나의 클래스는 하나의 책임만 가져야 한다.
> 

### 🔹 **우리 시스템의 구조**

| 클래스 | 책임 |
| --- | --- |
| `UContextManagerComponent` | 입력 의미 해석 (IMC 관리) |
| `UCleaningStateMachine` | 상태 전이와 행동 허가 |
| `AEnhancedChallengeCharacter` | 입력 위임 및 통합 |

```cpp
ContextManagerComponent  → 입력 해석만 담당
CleaningStateMachine     → 상태 관리만 담당
EnhancedChallengeCharacter → 위임만 담당
```

### ❌ **나쁜 예 – 책임 혼합**

```cpp
void ACharacter::OnInteract()
{
    // 입력 처리
    if (CurrentMode == CleaningMode) { ... }

    // 상태 확인
    if (bIsBusy) { ... }

    // 애니메이션 / UI / 사운드 전부 여기서
    PlayMontage(...);
    UpdateHUD(...);
    PlaySound(...);
}

```

→ 유지보수 지옥. 테스트 불가능. 의존 관계 꼬임.

### ✅ **좋은 구조 – 분리된 책임**

- `ContextManager`: 현재 모드(외적 상황)만 판단<br>
- `StateMachine`: 지금 가능한가(내적 상태)만 판단<br>
- `Character`: 호출만 담당<br>

> 각 컴포넌트를 독립적으로 테스트 가능
> 
> 
> 수정해도 다른 부분 영향 없음
> 
> 재사용성 극대화
> 

## **5-2. 개방-폐쇄 원칙 (OCP)**

> Open–Closed Principle
> 
> 
> → “확장에는 열려 있고, 수정에는 닫혀 있어야 한다.”
> 

### 🔹 **새 Context 추가 예시**

```cpp
enum class EGameplayContext : uint8
{
    Default,
    FloorCleaning,
    WindowCleaning,
    Fishing,      // ← 새 기능 추가
};

```

1️⃣ `IMC_Fishing` 에셋 생성

2️⃣ `ContextMappings`에 등록

3️⃣ 끝. 기존 코드는 그대로.

> 확장만 했지, 기존 코드는 수정하지 않았다.
> 

### 🔹 **새 State 추가 예시**

```cpp
enum class ECleaningState : uint8
{
    Idle,
    Cleaning,
    Resting,     // ← 추가
    Exhausted    // ← 추가
};

void StartResting()
{
    if (CurrentState == Idle || CurrentState == Exhausted)
        SetState(Resting);
}

```

- Idle ↔ Cleaning 로직은 그대로
- 새로운 전이(Transition)만 추가

## **5-3. 의존성 역전 원칙 (DIP)**

> Dependency Inversion Principle
> 
> 
> → 구체적 입력(E키, 게임패드 버튼)에 의존하지 않고,
> 
> **추상적 의도(InputAction)** 에 의존해야 한다.
> 

### ❌ **나쁜 예**

```cpp
if (IsKeyPressed(EKeys::E))  // 키보드에 고정 의존
    CleanFloor();
```

- 키보드에서만 작동
- 게임패드나 VR 컨트롤러 추가 시 코드 수정 필요

### ✅ **좋은 예**

```cpp
void OnInteract(const FInputActionValue& Value)
{
    // 어떤 키인지 모름
    // 어떤 장치인지도 모름
    ExecuteInteraction();  // 추상화된 의도만 실행
}
```

> InputAction = “상호작용하고 싶다”라는 추상 신호
> 
> 
> → 장치(IMC)만 바꾸면 그대로 동작
> 

## **5-4. 확장 시나리오**

### **시나리오 1: 멀티 도구 시스템**

```cpp
enum class EGameplayContext : uint8
{
    // 청소
    FloorCleaning,
    WindowCleaning,
    CarCleaning,

    // 전투
    Sword,
    Bow,
    Magic,

    // 이동 수단
    OnFoot,
    InCar,
    InBoat,
};

```

- 각각 IMC를 따로 등록<br>
- `PushContext()`로 모드 전환 시 완전히 다른 조작 체계<br>

### **시나리오 2: 복합 State Machine**

```cpp
class ACharacter
{
    UCleaningStateMachine* CleaningState;
    UCombatStateMachine* CombatState;
    UMovementStateMachine* MovementState;
};

void OnAttack()
{
    if (CombatState->CanAttack() &&
        CleaningState->IsIdle() &&
        MovementState->CanAct())
    {
        Attack();
    }
}
```

> 여러 FSM을 병렬로 운용해도 간섭 없이 동작.
> 

### **시나리오 3: 데이터 주도 설계**

```cpp
USTRUCT()
struct FContextDefinition
{
    EGameplayContext ContextType;
    UInputMappingContext* IMC;
    TArray<UAnimMontage*> AvailableActions;
    float StaminaCost;
};

```

- Context 정의를 **Data Asset**으로 관리
- 디자이너가 밸런스 조정 가능
- 코드 수정 없이 행동 세부 조정 가능