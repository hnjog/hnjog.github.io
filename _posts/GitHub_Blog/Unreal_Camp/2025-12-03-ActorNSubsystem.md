---
title: "김하연 튜터님 강의 - '액터와 서브시스템 아키텍쳐'"
date : "2025-12-03 12:00:00 +0900"
last_modified_at: "2025-12-03T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Actor
  - Subsystem
---

# 액터와 서브시스템 아키텍쳐에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. Actor 생성/등록/초기화의 전체 흐름 ❤️‍🔥

## 1.1 Actor는 어떻게 태어나는가? - 세 가지 탄생 시나리오

| 시나리오 | 설명 | 특징 |
| --- | --- | --- |
| **C++ SpawnActor** | 코드에서 직접 생성 | 런타임 동적 생성 |
| **Blueprint SpawnActor** | 블루프린트에서 생성 | 내부적으로 C++ SpawnActor 호출 |
| **Pre-placed Actor** | 레벨에 미리 배치 | .umap에서 역직렬화로 생성 |

- 에디터 드래그 시<br>
  SpawnActor가 아닌, Level이 Load될 때 '활성화' 되는 것!<br>

```cpp
// SpawnActor 기본 사용법
AMyActor* NewActor = GetWorld()->SpawnActor<AMyActor>(
    AMyActor::StaticClass(),
    SpawnLocation,
    SpawnRotation
);
```

> ⚠️ Pre-placed Actor는 레벨 로딩 중 생성되므로, 생성자에서 다른 Actor 참조 시 아직 로드 안 됐을 수 있음
> 

- 에디터에 '이미 배치된' 것을 기반으로 로직을 짜는 방식은<br>
  실제 패키징 된 게임에서 문제를 발생시킬 수 있음<br>
  (이런 버그는 재현도 어려움)<br>

## 1.2 SpawnActor의 내부 여행 - 한 걸음씩 따라가기

### Step 1: StaticClass 준비

```cpp
AMyActor::StaticClass()
```

- `리플렉션 결과물`인 **UClass** 객체를 가져옴<br>
- 메모리 크기, 생성자 정보 등 `메타데이터` 포함<br>

### Step 2: AllocateObject - 메모리 할당

```cpp
UObject* NewObject = StaticAllocateObject(InClass, InOuter, InName, InFlags, ...);
```

- **InOuter**: 나를 `소유`하는 **상위 객체** (Actor는 보통 World나 Level)<br>
- Outer 체인이 '가비지 컬렉션'에서 중요한 역할<br>

### Step 3: Constructor 호출

```cpp
AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;

    // CreateDefaultSubobject는 생성자에서만 호출 가능!
    RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("Root"));
    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    MeshComp->SetupAttachment(RootComponent);
}
```

> ⚠️ CreateDefaultSubobject는 **CDO에 컴포넌트를 등록**하므로 생성자에서만 호출 가능
> 
> 
> 이 시점의 컴포넌트는 아직 월드에 등록되지 않은 상태 ("태어나기 전의 태아")
> 

- CDO 를 복사해서 인스턴스가 만들어진 것은 지난 시간에 배운 내용<br>
  (UObject 기반의 복사 루틴을 따름)<br>

- 해당 내용은 CDO에 Component라는 새로운 요소를 추가<br>
  즉, 설계도에 내용을 더해주는 내용임<br>

- 코드 에만 존재하고, 메모리에 올라온 것은 아님!<br>
  (생성자에선 기본적인 설정만 해야하는것을 권장하는 이유)<br>

### Step 4: PostInitProperties - 프로퍼티 초기화 완료

```cpp
void AMyActor::PostInitProperties()
{
    Super::PostInitProperties();
    // 모든 UPROPERTY가 기본값으로 초기화된 상태
    // 아직 Actor는 월드에 없음 (출생신고 전)
}
```

- 메모리에 올라옴<br>
- 월드 등과 연관은 아직 없음<br>

### Step 5: PreRegisterAllComponents - 등록 준비

- 컴포넌트 목록 정리 단계<br>

### Step 6: RegisterComponent - 진짜 중요한 순간!

```cpp
void UActorComponent::RegisterComponent()
{
    // 월드에 등록
    // Physics Scene에 등록 (충돌 컴포넌트)
    // Render Scene에 등록 (렌더링 컴포넌트)
    // Tick 함수 등록
}
```

**RegisterComponent가 하는 일:**<br>

| 등록 대상 | 효과 |
| --- | --- |
| 물리 엔진 | 충돌 검사 가능 |
| 렌더링 시스템 | 화면에 표시 |
| Tick 시스템 | 매 프레임 업데이트 |

> ⚠️ RegisterComponent 없이는 컴포넌트가 유령 상태 - 존재하지만 아무 일도 안 함
> 

- World 에 일부가 되는 순간임<br>
  - 5.0의 카오스 물리 엔진에 등록<br>
  - 렌더링에 등록하여 '그릴 수 있도록 함'<br>
  - Tick 함수가 있다면 등록하여 매 프레임 호출<br>

```cpp
// 잘못된 예 - RegisterComponent 누락
UStaticMeshComponent* NewMesh = NewObject<UStaticMeshComponent>(this);
NewMesh->SetStaticMesh(SomeMesh);
NewMesh->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);
// 메시가 안 보임!

// 올바른 예
UStaticMeshComponent* NewMesh = NewObject<UStaticMeshComponent>(this);
NewMesh->SetStaticMesh(SomeMesh);
NewMesh->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);
NewMesh->RegisterComponent();  // 필수!
```

### Step 7: PostActorCreated - Actor 생성 완료 알림

```cpp
void AMyActor::PostActorCreated()
{
    Super::PostActorCreated();
    // Actor 완전히 생성, 컴포넌트도 등록됨
    // 하지만 아직 BeginPlay 전!
}
```

- begin play 이전이란 점 유의<br>

### Step 8: OnConstruction - 컨스트럭션 스크립트

```cpp
void AMyActor::OnConstruction(const FTransform& Transform)
{
    Super::OnConstruction(Transform);
    // 절차적 메시 생성 등
}
```

**Constructor vs OnConstruction:**<br>

| 구분 | Constructor | OnConstruction |
| --- | --- | --- |
| 호출 시점 | 메모리 할당 직후 | 컴포넌트 등록 후 |
| 호출 횟수 | 객체당 1번 | 에디터 수정 시마다 |
| Transform 접근 | NO | YES |
| 주 용도 | 기본 컴포넌트 생성 | 절차적 생성, 에디터 미리보기 |

- 에디터 수정에 유용하므로 알아 두기<br>

## 1.3 생명주기 흐름 정리

```
1. StaticClass() - 클래스 메타정보 준비
       ↓
2. AllocateObject - 메모리 할당
       ↓
3. Constructor - 생성자 호출 (CreateDefaultSubobject)
       ↓
4. PostInitProperties - 프로퍼티 초기화 완료
       ↓
5. PreRegisterAllComponents - 등록 준비
       ↓
6. RegisterComponent (각 컴포넌트마다) - 월드에 등록!
       ↓
7. PostActorCreated - 생성 완료 알림
       ↓
8. OnConstruction - 컨스트럭션 스크립트
       ↓
   (아직 BeginPlay 아님!)
```

- 에디터에서 레벨을 열었을 때 배치된 Actor들은 여기까지만 실행된 상태<br>

# 2. BeginPlay의 정확한 호출 조건 🍷

## 2.1 BeginPlay는 생성 직후에 안 온다!

BeginPlay 호출 조건 **두 가지**:

1. Actor가 생성되고 등록 완료<br>
2. **World가 Play 상태**<br>

```cpp
// 엔진 내부 동작 (개념)
void UWorld::BeginPlay()
{
    for (AActor* Actor : AllActors)
    {
        if (!Actor->HasBegunPlay())
            Actor->BeginPlay();
    }
}
```

> 레벨에 미리 배치된 Actor 100개 → 게임 시작 시 100개 모두 BeginPlay 호출 (거의 동시에)
> 

- 에디터에 배치하여도 '시작'하지 않으면 호출 안됨<br>
- Beginplay가 매우 많이 (동시에) 호출되는 상황이기에<br>
  Beginplay에 각종 초기화를 '너무' 몰아두면, 생각치 못한 버그 가 나올 수 있으므로 주의<br>
  - UX 경험 저하 부터, 간헐적 크래시 등<br>
  - Delegate, Lazy Load 등의 대체 방식 고려 가능<br>

## 2.2 동적 스폰 Actor의 BeginPlay

```cpp
// 게임 플레이 중 실행하면
AMyActor* NewActor = GetWorld()->SpawnActor<AMyActor>(...);
// SpawnActor 내부에서 BeginPlay까지 호출되고 리턴
// 이 시점에서 NewActor->HasBegunPlay() == true
```

- SpawnActor 시, Beginplay 보통 호출됨<br>
  - 다만 SpawnActor를 사용하는 Beginplay 내부에서<br>
    다시 SpawnActor를 사용하면, 순서가 엉망이 될 수 있음<br>
    (현재 SpawnActor -> BeginPlay -> Spawn Actor 시,<br>
     새로 생성되는 녀석의 Beginplay 부분이 끼어들 가능성이 있다?)<br>

## 2.3 BeginPlay 호출의 내부 조건

```cpp
bool CanBeginPlay()
{
    if (bHasBegunPlay) return false;         // 중복 호출 방지
    if (IsPendingKill()) return false;       // Actor가 valid해야 함
    if (!GetWorld()->HasBegunPlay()) return false;  // World가 Play 상태
    if (!bActorInitialized) return false;    // 완전히 초기화됨
    return true;
}
```

> 💡 BeginPlay는 '딱 한 번'만 호출됨 (bHasBegunPlay 플래그)
> 

## 2.4 왜 BeginPlay에서 초기화해야 하나?

Constructor 대신 BeginPlay에서 초기화해야 하는 이유가 세 가지

### 이유 1: 다른 Actor 참조

Constructor 시점에는 다른 Actor들이 아직 생성되지 않았을 수 있음.

```cpp
// ❌ Constructor에서 다른 Actor 찾기
AMyActor::AMyActor()
{
    // TargetActor가 아직 로드 안 됐을 수 있음
    TargetActor = UGameplayStatics::GetActorOfClass(GetWorld(), ATargetActor::StaticClass());
}

// ✅ BeginPlay에서 다른 Actor 찾기
void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    // 이 시점에는 레벨의 모든 Actor가 생성 완료됨
    TargetActor = UGameplayStatics::GetActorOfClass(GetWorld(), ATargetActor::StaticClass());
}
```

레벨에 배치된 Actor들의 로딩 순서는 보장되지 않음. <br>

에디터에서는 정상 동작하다가 패키지 빌드에서 크래시가 나는 원인<br>


- 생성자에선 다른 Actor가 메모리에서 로딩 되지 않을 수 있음<br>
  (에디터에선 메모리에 잘 올라와 있으니 될수도 있으나, 패키징 시 위험)<br>

### 이유 2: 델리게이트 바인딩

```cpp
// ❌ Constructor에서 델리게이트 바인딩
AMyActor::AMyActor()
{
    // GameMode가 아직 없을 수 있음
    if (AGameMode* GM = UGameplayStatics::GetGameMode(this))
    {
        GM->OnGameStart.AddDynamic(this, &AMyActor::HandleGameStart);
    }
}

// ✅ BeginPlay/EndPlay 짝으로 바인딩
void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    if (AGameMode* GM = Cast<AGameMode>(UGameplayStatics::GetGameMode(this)))
    {
        GM->OnGameStart.AddDynamic(this, &AMyActor::HandleGameStart);
    }
}

void AMyActor::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    if (GameMode)
    {
        GameMode->OnGameStart.RemoveDynamic(this, &AMyActor::HandleGameStart);
    }
    Super::EndPlay(EndPlayReason);
}

```

BeginPlay/EndPlay 짝으로 바인딩하면 메모리 누수와 댕글링 포인터 문제를 예방할 수 있음.<br>

### 이유 3: 레벨 재시작 문제

레벨 재시작 시 CDO에서 값을 복사해오는 방식으로 리셋되면<br>
**Constructor가 다시 호출되지 않을 수 있음.** 반면 BeginPlay는 레벨 시작 시 항상 호출!<br>

```cpp
// Constructor에서 점수 초기화 - 레벨 재시작 시 리셋 안 될 수 있음
AMyPlayerState::AMyPlayerState()
{
    Score = 0;
}

// BeginPlay에서 점수 초기화 - 레벨 시작마다 확실히 리셋
void AMyPlayerState::BeginPlay()
{
    Super::BeginPlay();
    Score = 0;
}

```

### 정리

| 초기화 위치 | 용도 |
| --- | --- |
| Constructor | CreateDefaultSubobject, Tick 설정, 기본값 |
| BeginPlay | 다른 Actor 참조, 델리게이트 바인딩, 게임 시작 초기화 |

## 3. Tick과 RegisterComponent의 관계 ⚡

## 3.1 Tick이 돌아가려면 뭐가 필요한가?

### 조건 1: bCanEverTick = true

```cpp
AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;  // 기본값 false!
}
```

- 성능 때문에 보통 꺼두는 편<br>

### 조건 2: RegisterComponent 완료

- World에 등록해야 함<br>

### 조건 3: 활성화 상태

```cpp
// Actor 레벨
SetActorTickEnabled(true);
SetActorTickEnabled(false);

// Component 레벨
MyComponent->SetComponentTickEnabled(true);
MyComponent->SetComponentTickEnabled(false);
```

## 3.2 Tick 등록의 내부 구조

```cpp
// Tick Manager 개념
class FTickTaskManager
{
    TArray<FTickFunction*> AllTickFunctions;

    void RunTick(float DeltaTime)
    {
        for (FTickFunction* TickFunc : AllTickFunctions)
        {
            if (TickFunc->IsTickEnabled())
                TickFunc->ExecuteTick(DeltaTime);
        }
    }
};
```

> RegisterComponent → Tick Manager에 등록 → Tick 호출됨
> 

- Tick 매니저를 통해, 등록된 녀석을 싹다 돌려줌<br>

## 3.3 동적 컴포넌트 추가 패턴

```cpp
void AMyActor::AddDynamicMeshComponent()
{
    // 1. NewObject로 생성 (CreateDefaultSubobject 아님!)
    UStaticMeshComponent* NewMesh = NewObject<UStaticMeshComponent>(this, TEXT("DynamicMesh"));

    // 2. 설정
    NewMesh->SetStaticMesh(SomeMesh);
    NewMesh->SetRelativeLocation(FVector(100, 0, 0));

    // 3. Attach (Register 전에!)
    NewMesh->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);

    // 4. Register 필수!
    NewMesh->RegisterComponent();
}
```

> ⚠️ AttachToComponent를 RegisterComponent 전에 해야 함 (Register 시점에 Transform 확정)
> 

- 런타임에 컴포넌트 추가 시<br>
  - Register를 '마지막'에 '반드시' 호출하기<br>
  - Register시 World 좌표가 확정됨<br>
    (부모가 없는 상태로 등록되어서, Relative가 World 기준으로 작동하게 됨)<br>

## 3.4 컴포넌트 제거 패턴

```cpp
void AMyActor::RemoveDynamicMeshComponent()
{
    if (DynamicMesh)
    {
        DynamicMesh->UnregisterComponent();  // 1. 월드에서 제거
        DynamicMesh->DestroyComponent();     // 2. 메모리 해제
        DynamicMesh = nullptr;               // 3. 포인터 정리
    }
}
```

- 순서 주의하기!<br>
  - World 등록 해제 -> 메모리 해제 -> 포인터 정리<br>

## 3.5 Tick 순서 제어

```cpp
AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;

    // Tick 그룹 설정
    PrimaryActorTick.TickGroup = TG_PrePhysics;

    // 특정 Actor 다음에 Tick (의존성 지정하기)
    PrimaryActorTick.AddPrerequisite(OtherActor, OtherActor->PrimaryActorTick);
}
```

| TickGroup | 시점 | 용도 |
| --- | --- | --- |
| `TG_PrePhysics` | 물리 시뮬레이션 전 | 캐릭터 이동 |
| `TG_DuringPhysics` | 물리 시뮬레이션 중 | - |
| `TG_PostPhysics` | 물리 시뮬레이션 후 | 물리 결과 반영 |

- AddPrerequisite 가 좀 까다로움<br>
  - 해당 액터가 유효해야 함<br>
  - 순환 의존성 주의하기<br>
  - 어쩔수 없을때만 고려<br>

## 4. 네트워크 환경에서의 Actor Lifecycle 🧸

## 4.1 네트워크의 기본: 역할(Role)의 이해

```cpp
ENetRole Role = GetLocalRole();
```

| Role | 설명 | 예시 |
| --- | --- | --- |
| `ROLE_Authority` | 진짜 주인 | 서버의 모든 Actor |
| `ROLE_AutonomousProxy` | 로컬 플레이어가 조종 | 내 클라이언트의 내 캐릭터 |
| `ROLE_SimulatedProxy` | 남이 조종 | 내 클라이언트의 다른 플레이어 캐릭터 |
| `ROLE_None` | 복제 안 됨 | - |

**예시: 플레이어 A, B가 있는 게임**<br>

| 위치 | A의 캐릭터 | B의 캐릭터 |
| --- | --- | --- |
| 서버 | Authority | Authority |
| A 클라이언트 | AutonomousProxy | SimulatedProxy |
| B 클라이언트 | SimulatedProxy | AutonomousProxy |

## 4.2 역할에 따라 BeginPlay가 다르게 동작한다

```cpp
void AMyCharacter::BeginPlay()
{
    Super::BeginPlay();

    // 서버에서만 (Authority)
    if (HasAuthority())
    {
        InitializeServerLogic();
    }

    // 로컬 플레이어만 (AutonomousProxy)
    if (IsLocallyControlled())
    {
        SetupCamera();
        BindInput();
        CreateHUD();
    }

    // 모든 곳에서
    InitializeVisuals();
}
```

- 필요에 따라<br>
  초기화를 다르게<br>

## 4.3 SpawnActor는 서버에서만!

```cpp
void AMyGameMode::SpawnMonster()
{
    // GameMode는 서버에만 존재
    AMonster* Monster = GetWorld()->SpawnActor<AMonster>(MonsterClass, SpawnLocation);
    // 서버에서 스폰 → 자동으로 모든 클라이언트에 복제
}
```

> ⚠️ 클라이언트에서 SpawnActor하면 그 클라이언트에만 존재하는 "가짜" Actor
> 

- 클라에서 Spawn하면 사실상 '클라'에서만 보이고<br>
  서버에선 '없는 녀석'<br>

## 4.4 Replication Graph의 생명주기 관리

기존 복제 시스템:<br>

- 매 프레임 모든 Actor 순회 → 비효율적 (1000 Actor × 100 플레이어 = 10만 번 판단)<br>

**Replication Graph**: Actor를 노드로 그룹화<br>

```
ReplicationGraph
├── GridSpatialization2D (위치 기반)
│   ├── Cell[0,0]: Actor1, Actor2, Actor3
│   ├── Cell[0,1]: Actor4, Actor5
│   └── ...
├── AlwaysRelevant (항상 복제)
│   └── GameState, PlayerStates...
└── PlayerStateFrequencyLimiter
    └── ...
```

### 멱등성(Idempotency)

- 같은 Actor는 항상 같은 방식으로 처리<br>
- 같은 위치면 항상 같은 Node<br>

## 4.5 Relevancy: 누구한테 뭘 보내줄까?

```cpp
bool AActor::IsNetRelevantFor(const AActor* RealViewer, const AActor* ViewTarget,
                              const FVector& SrcLocation) const
{
    // 기본: 거리 기반
    float DistSq = (GetActorLocation() - SrcLocation).SizeSquared();
    return DistSq < NetCullDistanceSquared;
}

// 커스터마이즈 예시
bool AMyActor::IsNetRelevantFor(...) const
{
    // 팀원에게는 거리 상관없이 항상 복제
    if (IsTeammate(RealViewer))
        return true;
    return Super::IsNetRelevantFor(RealViewer, ViewTarget, SrcLocation);
}
```

- 연관성<br>
  - 거리에 따라 Replicate 빈도를 줄이는 등의 처리가 가능<br>
  - 대역폭을 아끼면서 UX 경험을 개선 가능<br>

## 4.6 Dormancy: 잠자는 Actor

변하지 않는 Actor는 휴면 상태로:<br>

```cpp
AMyTree::AMyTree()
{
    NetDormancy = DORM_Initial;  // 바로 휴면
}

void AMyTree::OnDamaged()
{
    FlushNetDormancy();  // 깨우기
    Health -= 10;        // 이제 변경사항 복제됨
}
```

## 4.7 TearOff: 빠잉

서버와의 연결을 끊음 (래그돌 등):<br>

```cpp
void AMyCharacter::Die()
{
    if (HasAuthority())
    {
        TearOff();  // 더 이상 복제 안 됨
    }
    GetMesh()->SetSimulatePhysics(true);  // 각 클라이언트에서 독립적으로 시뮬
}
```

## 5. Subsystem Architecture 😱

## 5.1 Subsystem이 뭔가요?

문제 상황<br>

- SaveManager를 Actor로? → 레벨 바뀌면 파괴됨<br>
- GameInstance에 다 넣으면? → 코드가 뚱뚱해짐<br>

**Subsystem**: 특정 "범위(Scope)"에 바인딩된 `싱글톤` 객체<br>

- 언리얼이 관리해주는 '싱글톤'에 가까움<br>
  - 특정 타이밍과 연관되기에 생명주기만 판단하면 잘 이용 가능<br>

## 5.2 Subsystem의 종류

### 1. UEngineSubsystem

```cpp
class UMyEngineSubsystem : public UEngineSubsystem
{
    // 엔진 시작 ~ 엔진 종료
    // 에디터에서도 존재
};
```

- 그렇게 쓸일이 많진 않음<br>
  (엔진 급의 전역)<br>

### 2. UEditorSubsystem

```cpp
class UMyEditorSubsystem : public UEditorSubsystem
{
    // 에디터 시작 ~ 에디터 종료
    // 패키지 게임에는 없음
};
```

- 게임 로직에선 쓰지 말것!<br>

### 3. UGameInstanceSubsystem

```cpp
class UMyGameInstanceSubsystem : public UGameInstanceSubsystem
{
    // 게임 시작 ~ 게임 종료
    // 레벨 바뀌어도 살아있음!
};
```

- 게임 Instance와 수명이 같기에 범용적으로 사용 가능<br>
  - Save Data, Inventory 등등<br>

### 4. UWorldSubsystem

```cpp
class UMyWorldSubsystem : public UWorldSubsystem
{
    // World 생성 ~ World 파괴
    // 레벨별로 다른 인스턴스
};
```

- World 와 수명이 같음<br>
  - 보통 GameManager 같은 한 레벨에서<br>
    보조하는 역할의 로직 매니저 등에 고려 가능함<br>
    (ex : WaveManager)<br>

- 생성 자체는 서버/ 클라 양쪽에서 생성되지만<br>
  별도로 Replicate 같은 동기화는 되지 않음<br>
  (자동 동기화 x)<br>
  - 정말 클라와의 동기화가 필요한 경우는 RPC 함수 등을 고려할 것<br>
  - 게임 로직은 기본적으로 서버에서 동작하므로, 그 부분을 잘 염두해두자<br>

### 5. ULocalPlayerSubsystem

```cpp
class UMyLocalPlayerSubsystem : public ULocalPlayerSubsystem
{
    // LocalPlayer 생성 ~ LocalPlayer 제거
    // 분할화면이면 플레이어마다 별도 인스턴스
};
```

- Local 에서 사용 가능<br>
  UI용 매니저 or 입력, 설정 등에 활용 가능<br>
  - 확장성을 고려하지 않는다면 GameInstance 쪽에 넣는 것도... 가능은 함<br>

## 5.3 생명주기 비교표

| Subsystem | 생성 | 파괴 | 용도 |
| --- | --- | --- | --- |
| EngineSubsystem | 엔진 시작 | 엔진 종료 | 전역 서비스, Config |
| EditorSubsystem | 에디터 시작 | 에디터 종료 | 에디터 도구 |
| GameInstanceSubsystem | 게임 시작 | 게임 종료 | 세이브, 업적, 매치 데이터 |
| WorldSubsystem | World 생성 | World 파괴 | 웨이브, AI 매니저 |
| LocalPlayerSubsystem | 플레이어 추가 | 플레이어 제거 | UI, 입력, 설정 |

- GameInstance 와 World 서브 시스템을 가장 많이 사용<br>

## 5.4 Subsystem 만들기

```cpp
// MyGameInstanceSubsystem.h
UCLASS()
class UMyGameInstanceSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    void SaveGame();
    void LoadGame();

private:
    UPROPERTY()
    USaveGame* CurrentSaveGame;
};

// MyGameInstanceSubsystem.cpp
void UMyGameInstanceSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    UE_LOG(LogTemp, Log, TEXT("Save System Initialized!"));
    CurrentSaveGame = nullptr;
}

void UMyGameInstanceSubsystem::Deinitialize()
{
    UE_LOG(LogTemp, Log, TEXT("Save System Shutting Down!"));
    Super::Deinitialize();
}
```

- Initialize/Deinitialize 가 Beginplay / Endplay 를 대체하는 편<br>
  : 초기화/할당 해제 를 이 타이밍에 가능함<br>

## 5.5 Subsystem 사용하기

```cpp
// GameInstanceSubsystem
UMySaveSubsystem* SaveSys = GetGameInstance()->GetSubsystem<UMySaveSubsystem>();
SaveSys->SaveGame();

// WorldSubsystem
UMyWaveSubsystem* WaveSys = GetWorld()->GetSubsystem<UMyWaveSubsystem>();
WaveSys->StartNextWave();

// LocalPlayerSubsystem
UMyHUDSubsystem* HUDSys = GetLocalPlayer()->GetSubsystem<UMyHUDSubsystem>();
HUDSys->ShowPauseMenu();
```

- 전역적으로 가져다 사용이 가능!<br>

## 5.6 Subsystem 오용 사례

### 오용 1: Actor가 해야 할 일을 Subsystem에

```cpp
// 나쁜 예
class UCharacterMovementSubsystem : public UWorldSubsystem
{
    void MoveCharacter(ACharacter* Char, FVector Direction);
};

// 좋은 예 - Character가 직접 처리
class AMyCharacter : public ACharacter
{
    void MoveCharacter(FVector Direction);
};
```

- 객체가 자체적으로 할 수 있는 일이라면<br>
  전역 매니저가 일을 대신할 필요는 없음<br>
  - 전역 매니저를 만들지 않는 가능성도 열어둘 것<br>

### 오용 2: GameMode에 모든 걸 넣기

```cpp
// 나쁜 예 - 3000줄짜리 괴물
class AMyGameMode : public AGameModeBase
{
    void SaveGame();
    void LoadGame();
    void UnlockAchievement();
    void FindMatch();
    // ... 100개 더
};

// 좋은 예 - 분리
class AMyGameMode : public AGameModeBase
{
    // GameMode 본연의 역할만
    virtual void PostLogin(APlayerController* NewPlayer) override;
};

class USaveGameSubsystem : public UGameInstanceSubsystem { ... };
class UAchievementSubsystem : public UGameInstanceSubsystem { ... };
class UMatchmakingSubsystem : public UGameInstanceSubsystem { ... };
```

- GameMode에 모든 걸 넣으면 GodClass..<br>
- 다른 서브 시스템과 같이 사용하여<br>
  책임을 나누기<br>

### 오용 3: Subsystem에서 Tick 남용

```cpp
// 나쁜 예
class UMyWorldSubsystem : public UWorldSubsystem, public FTickableGameObject
{
    virtual void Tick(float DeltaTime) override
    {
        for (AActor* Actor : AllActors)
            ProcessActor(Actor);  // 매 프레임 무거운 로직
    }
};
```

> 💡 Subsystem은 "요청이 왔을 때" 일하는 게 맞음. 매 프레임 로직은 Actor에.
> 

- 매 프레임 로직이 있다면<br>
  Subsystem이 아니라 다른 방식을 고려해보자<br>
  - ex) 매프레임마다 서버로 정보 보내기 등의 로직이라면 가능할수도 있긴 함<br>

## 6. 실제 게임 구조 설계 예시 🥴

## 6.1 각 시스템의 역할 분담

### Actor의 역할

```cpp
AMyCharacter : 이동, 공격, 데미지 처리, 애니메이션
AWeapon : 발사, 재장전, 데미지 계산
AMonster : AI 행동, 어그로, 스킬 사용
APickupItem : 상호작용, 획득 효과
ADoor : 열기/닫기, 잠금 상태
```

### GameMode의 역할

```cpp
AMyGameMode
├── 플레이어 스폰 위치 결정
├── 게임 시작/종료 조건
├── 승패 판정
└── 팀 배정
```

- 매칭과 승패 규칙 등 본래의 역할만 해주기!<br>
  - 여러가지 넣기 좋기에 크기가 커지기 쉬우니 주의하자<br>

### GameState의 역할

```cpp
AMyGameState
├── 현재 라운드
├── 남은 시간
├── 각 팀의 점수
└── 매치 단계 (대기중/진행중/종료)
```

- 데이터 위주로 넣어야 함!<br>
  - 전광판 느낌을 잊지 말자<br>

### GameInstanceSubsystem의 역할

```cpp
USaveGameSubsystem
├── 세이브 파일 관리
├── 자동 저장
└── 로드

UPlayerProgressSubsystem
├── 언락된 아이템
├── 플레이어 레벨
└── 업적 상태

UMatchmakingSubsystem
├── 서버 검색
├── 로비 관리
└── 매치 참가/생성
```

### WorldSubsystem의 역할

```cpp
UWaveSpawnSubsystem
├── 현재 웨이브 번호
├── 스폰 스케줄
└── 몬스터 풀 관리

UEnvironmentSubsystem
├── 날씨 시스템
├── 낮/밤 주기
└── 환경 이벤트

UAIManagerSubsystem
├── AI 풀링
├── 네비게이션 쿼리 캐싱
└── 전역 AI 상태
```

### LocalPlayerSubsystem의 역할

```cpp
UHUDSubsystem
├── UI 위젯 관리
├── 알림 시스템
└── 미니맵

UInputSubsystem
├── 키 바인딩
├── 입력 모드 전환
└── 게임패드/키보드 전환

USettingsSubsystem
├── 그래픽 설정
├── 오디오 설정
└── 조작 설정
```

## 6.2 실제 흐름 예시: 몬스터 웨이브 시스템

```
[게임 시작]
    │
    ▼
UWaveSpawnSubsystem::Initialize()
    │ - 웨이브 데이터 로드
    │ - 스폰 포인트 캐싱
    │
    ▼
[웨이브 시작 트리거]
    │
    ▼
UWaveSpawnSubsystem::StartWave(WaveIndex)
    │
    ├─▶ 스폰 스케줄 생성
    │
    └─▶ 타이머로 SpawnMonster() 호출
            │
            ▼
        GetWorld()->SpawnActor<AMonster>(...)
            │
            ├─▶ [서버에서]
            │       AMonster::Constructor
            │           ↓
            │       RegisterComponent
            │           ↓
            │       AMonster::BeginPlay
            │           ↓
            │       [Replication] → 클라이언트로 복제
            │
            └─▶ [각 클라이언트에서]
                    AMonster 생성 (복제)
                        ↓
                    AMonster::BeginPlay
```

## 6.3 흔한 설계 실수와 해결책

### 실수 1: 순환 의존성

```cpp
// 나쁜 예: A가 B를 알고, B도 A를 알음
class AMonster
{
    UWaveSubsystem* WaveSystem;
};

class UWaveSubsystem
{
    TArray<AMonster*> Monsters;
};

// 좋은 예: 델리게이트로 느슨한 결합
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnMonsterDied, AMonster*, Monster);

class AMonster
{
public:
    FOnMonsterDied OnDied;
    void Die() { OnDied.Broadcast(this); }
};

class UWaveSubsystem
{
    void OnMonsterSpawned(AMonster* Monster)
    {
        Monster->OnDied.AddDynamic(this, &UWaveSubsystem::HandleMonsterDied);
    }

    void HandleMonsterDied(AMonster* Monster)
    {
        AliveCount--;
        CheckWaveComplete();
    }
};
```

- 하나 건들면 다 건드려야 하는 방식은...<br>
  유지보수가 나쁨<br>

- 서로를 알아야 하는지 반드시 확인하고<br>
  아니라면 Delegate 등의 사용 방식을 고려하자<br>

### 실수 2: 잘못된 생명주기 선택

```cpp
// 잘못된 예: 세이브 시스템을 WorldSubsystem으로
class USaveSubsystem : public UWorldSubsystem
{
    // 레벨 바뀌면 사라짐! 세이브 데이터도 손실!
};

// 좋은 예: GameInstanceSubsystem 사용
class USaveSubsystem : public UGameInstanceSubsystem
{
    // 게임 내내 유지됨
};
```

## 6.4 WorldSubsystem으로 Actor 스포너 만들기

### Step 1: Subsystem 클래스 생성

```cpp
// SpawnManagerSubsystem.h
UCLASS()
class USpawnManagerSubsystem : public UWorldSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    void SpawnTestActor();

private:
    UPROPERTY()
    TSubclassOf<AActor> TestActorClass;
};

// SpawnManagerSubsystem.cpp
void USpawnManagerSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    UE_LOG(LogTemp, Warning, TEXT("=== SpawnManagerSubsystem Initialize ==="));
}

void USpawnManagerSubsystem::Deinitialize()
{
    UE_LOG(LogTemp, Warning, TEXT("=== SpawnManagerSubsystem Deinitialize ==="));
    Super::Deinitialize();
}

void USpawnManagerSubsystem::SpawnTestActor()
{
    UWorld* World = GetWorld();
    if (World)
    {
        FVector Location(0, 0, 100);
        FRotator Rotation(0, 0, 0);

        AActor* SpawnedActor = World->SpawnActor<AStaticMeshActor>(Location, Rotation);

        UE_LOG(LogTemp, Warning, TEXT("Actor Spawned! HasBegunPlay: %s"),
            SpawnedActor->HasActorBegunPlay() ? TEXT("True") : TEXT("False"));
    }
}
```

### Step 2: 테스트 Actor에 로그 추가

```cpp
ATestActor::ATestActor()
{
    UE_LOG(LogTemp, Warning, TEXT("[TestActor] Constructor"));

    RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("Root"));
    MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    MeshComponent->SetupAttachment(RootComponent);
}

void ATestActor::PostInitProperties()
{
    Super::PostInitProperties();
    UE_LOG(LogTemp, Warning, TEXT("[TestActor] PostInitProperties"));
}

void ATestActor::PostActorCreated()
{
    Super::PostActorCreated();
    UE_LOG(LogTemp, Warning, TEXT("[TestActor] PostActorCreated"));
}

void ATestActor::BeginPlay()
{
    Super::BeginPlay();
    UE_LOG(LogTemp, Warning, TEXT("[TestActor] BeginPlay"));
}
```

### Step 3: 결과 확인

```
[TestActor] Constructor
[TestActor] PostInitProperties
[TestActor] PostActorCreated
[TestActor] BeginPlay
Actor Spawned! HasBegunPlay: True
```