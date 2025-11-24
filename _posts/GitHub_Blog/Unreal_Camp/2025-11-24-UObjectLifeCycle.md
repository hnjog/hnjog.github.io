---
title: "김하연 튜터님 강의 - 'UObject 생명주기 - GC와 메모리 관리'"
date : "2025-11-24 12:00:00 +0900"
last_modified_at: "2025-11-24T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - UObject
  - Unreal Life Cycle
  - Unreal GC
---

# 가비지 컬렉터 시스템과 UObject의 생명주기에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. 큰 그림: 언리얼 메모리 모델 잡기 👾

## 1-1. C++ vs 언리얼 GC: 왜 완전히 다를까?

### 일반 C++의 문제점

```cpp
MyClass* ptr = new MyClass();  // 생성
// ... 사용 ...
delete ptr;                    // 삭제 (필수!)
```

```cpp
void ProblematicFunction()
{
    MyClass* obj = new MyClass();

    if (SomeCondition())
    {
        return;  // 앗! obj를 delete하지 못했다!
    }

    delete obj;  // 이 줄에 영원히 도달하지 못함
}
```

```cpp
MyClass* obj = new MyClass();
delete obj;
delete obj;  // 💥 같은 메모리를 두 번 지움 = 크래시!

// 또는
obj->DoSomething();  // 💥 이미 지워진 메모리에 접근 = 크래시!
```

- 메모리 컨트롤 관련 실수 발생<br>

### 언리얼 GC의 해결책

```cpp
UObject* obj = NewObject<UObject>();  // 생성
// ... 사용 ...
// delete 안 씀! 언리얼이 알아서 처리
```

```cpp
UObject* player = NewObject<UObject>();    // 플레이어 생성
UObject* weapon = NewObject<UObject>();    // 무기 생성

player = nullptr;  // 플레이어를 더 이상 참조하지 않음

// 잠시 후 GC가 동작하면:
// - weapon: 여전히 weapon 변수가 참조하고 있음 → 살아남음 ✅
// - player가 가리키던 객체: 아무도 참조 안 함 → 자동 삭제 ✅
```

### **언리얼 GC 규칙**

1. **UObject 상속 클래스만 관리**
2. UPROPERTY로 참조 표시 필수

- 추적하기 위한 정보를 제공받아야 함<br>
  즉, GC에게 '이 정보는 중요해'라고 알려주는 것<br>

```cpp
class UMyClass : public UObject
{
    UPROPERTY()  // ✅ 이 참조는 중요하다고 GC에게 알림
    UObject* SafeRef;

    UObject* UnsafeRef;  // ❌ GC가 이 참조를 모름
};
```

```cpp
class UInventory : public UObject
{
    GENERATED_BODY()
    
    UPROPERTY()
    UObject* ImportantItem;  // GC가 추적함
    
    UObject* ForgottenItem;  // GC가 모름!
};

void Example()
{
    UInventory* Inv = NewObject<UInventory>();
    Inv->ImportantItem = NewObject<UObject>();  // 안전
    Inv->ForgottenItem = NewObject<UObject>();  // 위험
    
    // 나중에...
    Inv->ForgottenItem->DoSomething();  // 💥 이미 삭제된 객체 접근!
}
```

- 이러면 GC가 돌고 나서 터질 수 있음<br>
  위험한 방식<br>

3. 절대 delete 하면 안 됨

```cpp
UObject* obj = NewObject<UObject>();
delete obj;  // ❌ 절대 금지! 언리얼 내부가 망가짐

// 올바른 방법:
obj = nullptr;  // 참조만 끊으면 GC가 알아서 처리
```

- 임의로 지워버리면 GC의 존재 이유가...<br>

## 1-2. UObject란? 언리얼 엔진의 핵심 상자

- Unreal 엔진의 핵심 클래스<br>
  (엔진에서 까보면, 매우 엄청난 양의 코드를 볼 수 있음)<br>

### 일반 C++ 클래스의 한계

```cpp
class MyCppCharacter
{
private:
    int Health = 100;
    std::string Name = "Hero";

public:
    void TakeDamage(int Damage) { Health -= Damage; }
    int GetHealth() const { return Health; }
};
```

❌ 블루프린트 지원 안됨, 에디터 통합 안됨, 자동 저장 안됨<br>

### UObject의 마법

```cpp
UCLASS(BlueprintType)  // 블루프린트에서 사용 가능하게!
class MYGAME_API UMyCharacter : public UObject
{
    GENERATED_BODY()

private:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, SaveGame, meta = (AllowPrivateAccess = "true"))
    int32 Health = 100;  // 에디터 수정 + 블루프린트 접근 + 자동 저장!

public:
    UFUNCTION(BlueprintCallable)  // 블루프린트에서 호출 가능!
    void TakeDamage(int32 Damage);
};
```

✅ 블루프린트 지원, 에디터 통합, 자동 저장, 메모리 자동 관리<br>

### UObject가 제공하는 핵심 기능들

1. 블루프린트 통합<br>
2. 에디터 통합<br>
3. 자동 저장/불러오기<br>

```cpp
UPROPERTY(SaveGame)
int32 PlayerLevel;  // 자동으로 세이브 파일에 포함!

UPROPERTY(SaveGame)
TArray<FString> CompletedQuests;  // 퀘스트 목록도 자동 저장!
```
    
4. 네트워크 리플리케이션<br>

```cpp
UPROPERTY(Replicated)  // 이것만으로 네트워크 동기화!
int32 PlayerScore;

// 함수도 네트워크를 통해 호출 가능
UFUNCTION(Server, Reliable)
void ServerFireWeapon();
```

5. 리플렉션<br>
6. 메모리 자동 관리<br>

```cpp
// 전통적인 C++
Character* player = new Character();
Weapon* sword = new Weapon();
// ... 나중에 반드시 delete player; delete sword;

// UObject 방식
UCharacter* player = NewObject<UCharacter>();
UWeapon* sword = NewObject<UWeapon>();
// delete 없어도 GC가 알아서 정리!
```
    

### 언리얼 클래스 계층

```
UObject (모든 것의 시작)
├── AActor (3D 월드에 존재하는 것들)
│   ├── APawn (조종 가능한 객체)
│   │   └── ACharacter (걸어다니는 캐릭터)
│   ├── AController (두뇌 역할)
│   └── AGameMode (게임 규칙 관리)
├── UActorComponent (액터에 붙이는 부품들)
│   ├── UMeshComponent (모델 표시)
│   └── UMovementComponent (이동 처리)
└── UObject (순수 데이터/로직)
    ├── UGameInstance (게임 전체 관리)
    └── UUserWidget (UI 요소)
```

1. **통일성** - 모든 객체가 같은 방식으로 동작<br>
2. **확장성** - 새 기능 추가 시 모든 객체가 자동으로 혜택<br>
3. 상호 운용성 - 서로 다른 시스템끼리 쉽게 연동<br>

## 1-3. Outer 체인: 누가 누구의 주인인가?

### Outer = 소유 관계

```
🏢 아파트 건물 (World - 게임 세계)
  └── 🏠 101호 (Level - 현재 맵)
       └── 👤 하연 튜터 (Actor - 플레이어 캐릭터)
            ├── 📱 스마트폰 (Component - 인벤토리)
            │    └── 💬 카카오톡 (SubObject - 아이템)
            └── 👛 지갑 (Component - 장비)
                 ├── 💳 신용카드 (SubObject)
                 └── 💵 현금 (SubObject)
```

**핵심:** 주인이 사라지면 소유물도 자동 삭제<br>

- Outer와 Owner는 이름이 비슷하지만 다른 개념임<br>
  - Outer<br>
   : UObject 계열 객체에서 '특정 상위 객체'에 속하는지를 판단<br>
     Unreal에서 계층적 생성 구조<br>
     (보통 UObject는 Outer를 주로 사용)<br>
  - Owner<br>
   : 네트워크 권한 / 컨트롤 소유자 <br>
     생성 당시의 소유자<br>
    (Actor는 Owner를 통해 RPC 권한 등을 부여)<br>

- 이 둘을 다르게 설정한다면<br>
  부모와 관리자를 따로 두고 싶은 경우에 주로 사용<br>
  - 런타임에서 Outer는 바꿀 수 없으나 Owner는 변경이 가능함<br>
  - 다만 다르게 설정하는 경우, '코드 가독성'이 떨어질 수 있음<br>
    (명확한 의도를 표현하려면 주석을 사용햅로 것)<br>
    (ex : GC 적으로 ~~ 해서 Outer와 Owner를 다르게 설정했습니다)<br>

### 전통적인 방식 vs Outer 방식

1. 자동 메모리 정리의 핵심<br>

```cpp
// 전통적인 방식 (복잡함)
void PlayerDied()
{
    delete player->inventory;
    delete player->weapon;
    delete player->armor;
    delete player->skills[0];
    delete player->skills[1];
    // ... 수십 개의 객체를 일일이 정리
    delete player;

    // 하나라도 빼먹으면 메모리 누수!
}
```

```cpp
// Outer 방식 (간단함)
void PlayerDied()
{
    player->Destroy();  // 이 한 줄로 끝!
    // 플레이어가 소유한 모든 것들이 자동으로 연쇄 정리됨
}
```
    
2. GC가 Outer 체인을 따라 작동<br>
3. 논리적 구조를 코드로 표현<br>

- 연쇄적인 파괴를 통한 정리<br>

### 자동 정리 과정

```cpp
// 레벨 바뀔 때
GetWorld()->LoadLevel(TEXT("NewLevel"));

// 실제로는 이런 일들이 자동으로
// 1. 현재 Level의 모든 Actor들에게 "곧 사라진다" 알림
// 2. 각 Actor의 모든 Component들 정리
// 3. 각 Component의 모든 SubObject들 정리
// 4. Level 자체 삭제
// → 맵 전체가 깔끔하게 정리됨!
```

- World -> Level -> 각 Actor들에 대한 삭제 처리<br>

```cpp
void APlayerCharacter::Die()
{
    Destroy();  // 플레이어가 죽을 때

    // 자동으로 정리되는 것들
    // - 인벤토리의 모든 아이템
    // - 장착한 모든 장비
    // - 활성화된 모든 스킬
    // - 캐릭터의 모든 컴포넌트
}
```

### NewObject에서 Outer 지정

```cpp
// NewObject의 기본 형태
UObject* NewObj = NewObject<UMyClass>(
    this,                    // ← 이 부분이 Outer (주인)
    UMyClass::StaticClass(), // 클래스 타입
    TEXT("ObjectName")       // 이름 (선택)
);

// 다양한 Outer 선택 예시
UObject* TempObj = NewObject<UObject>(GetTransientPackage());     // 임시 객체
UObject* WorldObj = NewObject<UObject>(GetWorld());              // 월드 레벨
UObject* GameObj = NewObject<UObject>(GetGameInstance());        // 게임 전체
UActorComponent* Comp = NewObject<UActorComponent>(MyActor);     // 액터 소유
```

### 자주 하는 실수들

1. 잘못된 Outer 선택<br>

```cpp
// ❌ 문제: UI를 액터가 소유하게 함
UUserWidget* Widget = NewObject<UUserWidget>(SomeActor);
// → 액터가 죽으면 UI도 사라짐!

// ✅ 해결: UI는 플레이어 컨트롤러가 소유
UUserWidget* Widget = NewObject<UUserWidget>(GetPlayerController());
// → 액터가 죽어도 UI는 유지됨
```

2. Outer 체인 끊기<br>

```cpp
// ❌ 위험: 중간 객체를 다른 곳으로 이동
UObject* Parent = NewObject<UObject>(this);
UObject* Child = NewObject<UObject>(Parent);

Parent->Rename(nullptr, SomeOtherObject);  // Parent를 다른 곳으로 이동
// → Child가 고아가 될 수 있음!
```

this -> parent -> child<br>

- Rename<br>
  - 첫 요소로 이름 재 지정<br>
  - 두번째 요소로 Outer 지정<br>

- 현재 객체에서 this로 지정하였으나<br>
  parent가 SomeOtherObject 로 지정하면서<br>
  GC에 혼란을 가질 수 있음<br>
  (잠재적인 미묘한 버그가 생길 가능성이 있음)<br>
  - 디버깅이 어려운...<br>

- Outer 체인은 처음 만든 후 건들지 만들 것을 추천<br>
  - 정말 옮겨야 한다면 자식들도 같이 옮길 것<br>

# 2. 생성, 수명, GC 핵심

## 2-1. 생성 패턴: `CreateDefaultSubobject` vs `NewObject`

- 타이밍에 따라 미리 사용하기<br>

### CreateDefaultSubobject - 미리 준비

```cpp
class AMyActor : public AActor
{
public:
    AMyActor()  // 생성자: 이 액터가 태어날 때 딱 한 번 실행
    {
        // "이 액터의 필수품들을 미리 준비해둬"
        MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComponent"));
        CollisionComponent = CreateDefaultSubobject<USphereComponent>(TEXT("Collision"));
    }

private:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UStaticMeshComponent* MeshComponent;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    USphereComponent* CollisionComponent;
};
```

- 생성자에서 호출<br>
- 에디터와의 연동을 목적으로 함<br>
  (에디터 쪽에서 확인 가능하며 수정 가능)<br>
- 게임 시작 전에 이미 생성되어 있음<br>
  - 안정적 / 예측 가능<br>
  - 자동 등록<br>
    (Actor와 같이 삭제 등)<br>

- 생성자에서만 쓸 수 있는 이유?<br>
  - Unreal의 객체 생성 과정이 간단하지 않음<br>
    객체 생성 -> 여러 설정 값 세팅(에디터 설정 등) -> 컴포넌트 설정 -> 이후 BeginPlay 등을 호출<br>

  - CDO를 기반으로 생성하기에<br>
    런타임에 부착되는 경우, 해당 객체는 CDO가 아닌 별도의 인스턴스여야 함<br>


- 반드시 필요한 기본 구성에 사용<br>

### NewObject - 필요할 때

```cpp
void AMyActor::OnLevelUp()  // 레벨업했을 때 호출되는 함수
{
    if (PlayerLevel >= 10)  // 10레벨 이상이면
    {
        // "지금 당장 특수능력을 만들어줘!"
        USpecialAbility* NewAbility = NewObject<USpecialAbility>(
            this,                           // 누가 주인? → 이 액터
            USpecialAbility::StaticClass(), // 어떤 타입?
            TEXT("FireballAbility")         // 이름은 뭐로 할까?
        );

        Abilities.Add(NewAbility);  // 능력 목록에 추가
    }
}
```

- 그때 그때 호출(조건 등)<br>
- 수동 관리가 많이 필요함<br>
  - 데이터,UPROPERTY 세팅 등<br>

### 언제 무엇을 쓸까?

| 구분 | CreateDefaultSubobject | NewObject |
| --- | --- | --- |
| **사용 시점** | 생성자에서만 | 언제든지 |
| **목적** | 기본 구성품 (필수) | 동적 생성 (선택) |
| **에디터 지원** | ✅ 완전 통합 | ❌ 런타임 전용 |
| **예측 가능성** | ✅ 항상 같음 | ❌ 상황마다 다름 |
| **사용 예시** | 메시, 카메라, 기본 컴포넌트 | 아이템, 능력, 임시 객체 |

## 2-2. UObject 라이프사이클

```cpp
UCLASS()
class UMyObject : public UObject
{
    GENERATED_BODY()

public:
    // 1단계: "방금 태어났어요!"
    virtual void PostInitProperties() override
    {
        Super::PostInitProperties();  // 부모의 초기화부터 먼저!

        // 기본값 검증 및 보정
        if (SomeImportantValue <= 0)
        {
            SomeImportantValue = 100;  // 잘못된 값 보정
        }
    }

    // 2단계: "저장된 데이터를 불러왔어요!"
    virtual void PostLoad() override
    {
        Super::PostLoad();

        // 구버전 데이터를 신버전으로 변환
        if (DataVersion < CURRENT_VERSION)
        {
            UpgradeDataFormat();
        }
    }

    // 3단계: "곧 죽을 예정이에요..."
    virtual void BeginDestroy() override
    {
        // 다른 객체들에게 작별 인사
        NotifyOthersAboutMyDeath();

        Super::BeginDestroy();  // 부모의 정리 작업
    }

    // 4단계: "완전히 사라져요"
    virtual void FinishDestroy() override
    {
        // 마지막 정리 작업
        Super::FinishDestroy();
    }
};
```

- 최초 탄생 -> 데이터 로드 -> ... 활동 중 -> 죽을 예정 -> 완전 사망<br>

- PostInitProperties : 초기 기본 데이터 세팅<br>
- PostLoad : 데이터 로딩 후, 검증 등<br>
- BeginDestroy : Destory 호출 당하여, 파괴될 예정이므로, 안전한 Destory를 위해 연결고리 해제 등<br>
- FinishDestory : BeginDestory에서 미처 정리하지 못한 것 들을 세팅 or 출력<br>

### Super:: 호출의 중요성

```cpp
// ❌ 이렇게 하면 큰일 납니다!
virtual void PostInitProperties() override
{
    // Super::PostInitProperties(); 생략함!
    MyCustomInitialization();
    // → 언리얼 엔진의 기본 초기화가 안 됨!
}

// ✅ 올바른 방법
virtual void PostInitProperties() override
{
    Super::PostInitProperties();  // 부모의 초기화 먼저
    MyCustomInitialization();    // 그 다음에 내 초기화
}
```

- Super 깜빡하면 언리얼 엔진이 지원한 초기화를 생략하므로<br>
  잠재적인 버그 위험 가능성이 존재<br>

### GetWorld() 사용법

```cpp
void UMyObject::SomeFunction()
{
    UWorld* World = GetWorld();  // 내가 속한 월드 가져오기

    if (World)  // 월드가 있다면
    {
        // 월드 관련 작업들 가능
        AActor* NewActor = World->SpawnActor<AActor>();  // 액터 스폰
        // 타이머 설정, 다른 액터 찾기 등
    }
    else  // 월드가 없다면
    {
        // 순수 데이터 처리만 가능
        ProcessPureData();
        CalculateNumbers();
    }
}
```

- UObject는 World와 관련이 없을수도 있기에<br>
  전제하여 사용하지 말 것<br>

## 2-3. GC 동작 원리: Mark & Sweep

### GC 작업 과정

```
게임 상황: 플레이어가 무기를 버렸다

시작 상태:
[World] → [PlayerController] → [PlayerCharacter] → [InventoryComponent]
                                                  ↘
                                                   [WeaponItem] → [WeaponMesh]

무기를 버린 후:
[World] → [PlayerController] → [PlayerCharacter] → [InventoryComponent]

                              (연결 끊어짐)

                                                   [WeaponItem] → [WeaponMesh]

GC 실행:
1. Mark 단계:
   - World부터 시작 ✓
   - PlayerController 표시 ✓
   - PlayerCharacter 표시 ✓
   - InventoryComponent 표시 ✓
   - WeaponItem은 아무도 참조 안 함 → 표시 없음 ✗
   - WeaponMesh도 표시 없음 ✗

2. Sweep 단계:
   - WeaponItem 삭제
   - WeaponMesh 삭제
```

- Mark & Sweep 방식으로 동작함<br>
  - RootSet 부터 연결된 UObject들을 Marking!<br>
    (RootSet : 안 지워질 최중요 객체들)<br>
  - Sweep할때, Marking 된 애들은 마킹만 지우고<br>
    안된 애들은 제거 처리<br>

### 수동 GC 실행

```cpp
// 특별한 상황에서 강제 실행
GEngine->ForceGarbageCollection(true);

// ⚠️ 주의: 남용하면 게임이 끊겨요!
// GC 실행 중에는 게임이 잠깐 멈춤
```

- true 하면 완벽한 GC 실행<br>

- 레벨 전환 등 아주 특별한 상황에서 호출 권장<br>
  (모든 객체를 검사해야 함)<br>
  (+ 메모리 사용량이 늘어남)<br>

- GC는 자동으로 일정 주기마다 호출되지만<br>
  임의의 타이밍에 직접 호출할 수 있는 정도를 알아 둘 것<br>

## 2-4. UPROPERTY의 진실: GC가 보는 것과 못 보는 것

### 왜 UPROPERTY 볼까?

1. 성능 때문

```cpp
class UMyComplexObject : public UObject
{
    // UObject 포인터들 (GC가 관심있어 함)
    UObject* Obj1;
    UObject* Obj2;

    // 일반 C++ 타입들 (GC가 관심없음)
    int32 SomeNumber;
    float SomeFloat;
    FString SomeString;
    TArray<int32> NumberArray;

    // 외부 라이브러리 타입들 (GC가 알 수 없음)
    std::vector<int> CppVector;
    std::shared_ptr<SomeClass> SharedPtr;

    // 그 외 수십, 수백 개의 변수들...
};
```

- UObject 관련만 볼 수 있음<br>

2. 타입 안전성 때문

```cpp
class UConfusingObject : public UObject
{
    void* VoidPointer;          // 이게 뭘 가리키는지 알 수 없음
    int32* IntPointer;          // int를 가리키는 포인터
    UObject* ObjectPointer;     // UObject를 가리키는 포인터

    // GC는 이 중에서 뭘 체크해야 할까요?
    // → ObjectPointer만 체크하면 됨!
    // → 하지만 그걸 어떻게 구분할까?
    // → UPROPERTY로 표시하면 확실!
};
```

- UObject 인 것만 파악하고 싶음<br>

3. 의도 파악 때문

```cpp
class UIntentionExample : public UObject
{
    UObject* ImportantReference;    // 이건 중요한 참조일까? 임시일까?
    UObject* TemporaryPointer;      // 잠깔 쓰고 버릴 포인터일까?
    UObject* CachePointer;          // 성능을 위한 캐시일까?

    // 개발자만이 각 포인터의 의도를 알고 있음
    // UPROPERTY = "이건 중요하니까 GC가 신경써줘"
    // UPROPERTY 없음 = "이건 임시니까 신경 안 써도 돼"
};
```

- 임시용인지 아닌지 코드를 보며 알 수 있음<br>

### UPROPERTY 있음 vs. 없음

```cpp
UCLASS()
class UGCTestObject : public UObject
{
    GENERATED_BODY()

public:
    // ✅ UPROPERTY 있음 - GC가 보호함
    UPROPERTY()
    UObject* SafeReference;

    UPROPERTY()
    TArray<UObject*> SafeArray;

    // ❌ UPROPERTY 없음 - GC가 모름
    UObject* UnsafeReference;
    TArray<UObject*> UnsafeArray;

    void TestTheReferences()
    {
        // 테스트용 객체들 생성
        UObject* TestObj1 = NewObject<UObject>(this);
        UObject* TestObj2 = NewObject<UObject>(this);

        // 참조 설정
        SafeReference = TestObj1;     // ✅ GC가 TestObj1을 보호
        UnsafeReference = TestObj2;   // ❌ GC가 TestObj2를 모름!

        // 잠시 후 GC 실행되면...
        // TestObj1 → 살아남음 ✅
        // TestObj2 → 삭제됨! 💥
    }
};
```

### 다양한 UPROPERTY 적용 사례

1. 기본 참조들

```cpp
class UBasicReferences : public UObject
{
    UPROPERTY()
    UObject* SingleObject;              // 단일 객체 참조

    UPROPERTY()
    TArray<UObject*> ObjectArray;       // 배열 참조 (배열 안의 모든 객체 보호됨)

    UPROPERTY()
    TMap<FString, UObject*> ObjectMap;  // 맵 참조 (Value 객체들만 보호됨)
};
```
    
2. 구조체 안의 참조도 보호됨
    
```cpp
USTRUCT()
struct FMyStruct
{
    GENERATED_BODY()

    UPROPERTY()
    UObject* NestedObject;  // 구조체 안의 참조도 GC가 봄!
};

UCLASS()
class UStructContainer : public UObject
{
    UPROPERTY()
    FMyStruct MyStruct;  // 구조체 전체를 UPROPERTY로 하면
                            // 구조체 안의 UObject 참조들도 모두 보호됨!
};
```

- 구조체도 사용 가능함<br>
- 또 구조체 내부의 다양한 UObject들도 체크<br>

### UPROPERTY 없는 참조 안전하게 다루기

```cpp
class UAdvancedUsage : public UObject
{
    // 이런 것들은 UPROPERTY가 안 됨
    TSharedPtr<FMyData> SharedData;           // 스마트 포인터
    std::unique_ptr<SomeClass> UniquePtr;     // C++ 스마트 포인터

public:
    // 방법 1: 수명을 명확히 관리
    void SafeUsagePattern()
    {
        auto Data = MakeShared<FMyData>();
        SharedData = Data;

        ProcessData(SharedData.Get());  // 이 함수 안에서만 사용

        SharedData.Reset();  // 명시적 해제
    }

    // 방법 2: BeginDestroy에서 정리
    virtual void BeginDestroy() override
    {
        SharedData.Reset();     // 죽기 전에 모든 포인터 정리
        UniquePtr.reset();

        Super::BeginDestroy();
    }
};
```

- 스마트 포인터는 UPROPERTY가 먹히지 않음<br>
- 안전한 사용방식?<br>
  - 명확한 사용 방식을 통해 안전한 제거<br>
  - BeginDestory 타이밍에 호출하여 메모리 누수를 확실히 막음<br>


## 2-5. AddToRoot: 강력하지만 위험한 최후의 수단

- AddToRoot : 해당 객체를 RootSet에 추가해주는 함수<br>
- GC의 핵심 부분에 올려놓기에<br>
  게임 종료,레벨 전환 등 끝날때까지 삭제되지 않는 녀석에 넣어야 함<br>
  
- RootSet은 GC가 반드시 건드리지 않기에<br>
  종료 시점에서 반드시 스스로 메모리 해제 등을 해야 함<br>
  (Shutdown 을 직접 호출할 것)<br>

### 올바른 사용법 1: 싱글톤

```cpp
// 게임 전체에서 하나만 존재해야 하는 매니저
class USoundManager : public UObject
{
    static USoundManager* Instance;

public:
    static USoundManager* GetInstance()
    {
        if (!Instance)
        {
            Instance = NewObject<USoundManager>(GetTransientPackage());
            Instance->AddToRoot();  // Root에 추가 → 게임이 끝날 때까지 보존
        }
        return Instance;
    }

    static void Shutdown()
    {
        if (Instance)
        {
            Instance->RemoveFromRoot();  // Root에서 제거 (중요!)
            Instance = nullptr;
        }
    }
};

// 게임 종료 시 - 반드시 호출해야 함!
void UMyGameInstance::Shutdown()
{
    USoundManager::Shutdown();  // 수동으로 정리
    Super::Shutdown();
}
```

- 자기 자신이 Static으로 선언하여도<br>
  GC가 UPROPERTY를 파악할 수 없음<br>

- Tmi : 싱글톤 구현을 Subsystem 등으로 선언하는 것이 더 괜찮을 수 있음<br>

### 올바른 사용법 2: 비동기 작업 보호

```cpp
// 비동기 로딩 중에는 삭제되면 안 되는 객체
class UAssetLoader : public UObject
{
public:
    void LoadAssetAsync(const FSoftObjectPath& AssetPath)
    {
        AddToRoot();    // 로딩 시작 전에 자신을 보호
        bIsLoading = true;

        // 비동기 로딩 시작
        UAssetManager& AssetManager = UAssetManager::Get();
        StreamableHandle = AssetManager.LoadAssetAsync(AssetPath,
            FStreamableDelegate::CreateUObject(this, &UAssetLoader::OnLoadComplete)
        );
    }

private:
    void OnLoadComplete()
    {
        bIsLoading = false;
        RemoveFromRoot();  // 로딩 완료 → 보호 해제

        // 이제 이 객체는 필요없으면 GC가 정리할 수 있음
    }
};
```

- 특정 시점에만 RootSet에 두고<br>
  끝나면 자기 자신을 보호 해제<br>

### 올바른 사용법 3: 임시 보호

```cpp
// 위험한 작업 중 임시 보호
void RiskyOperation(UObject* ImportantObject)
{
    ImportantObject->AddToRoot();       // 위험한 작업 전에 임시 보호

    LoadSomeHeavyAssets();              // GC 유발 가능한 작업들
    CreateThousandsOfObjects();
    CallComplexBlueprintFunction();

    ImportantObject->RemoveFromRoot();  // 작업 완료 즉시 보호 해제
}

```

- GC를 유발할 작업들을 파악하고<br>
  임시적으로 보호<br>

### 위험한 사용 패턴들

1. RemoveFromRoot 없는 무한 보존<br>
    
```cpp
// ❌ 이렇게 하면 메모리 누수!
class UBadManager : public UObject
{
    GENERATED_BODY()
    
    TArray<UObject*> ImportantData;
    
public:
    void CreateImportantData()
    {
        UObject* Data = NewObject<UObject>(this);
        Data->AddToRoot();  // Root에 추가
        
        ImportantData.Add(Data);
        
        // 문제: 언제 RemoveFromRoot()를 할지 정하지 않음!
        // 결과: 이 객체는 게임이 끝날 때까지 메모리에 영원히 남음
    }
};
```

- 생성할수록 RootSet에 계속 추가되어 안지워지는 객체 생성<br>
  (메모리 누수)<br>

2. 불필요한 Root 추가
    
```cpp
// ❌ 이미 보호받는 객체에 또 Root 추가
class AMyActor : public AActor
{
public:
    void BeginPlay() override
    {
        Super::BeginPlay();

        AddToRoot();  // ❌ 잘못된 생각: "혹시 몰라서 Root에 추가하자"

        // 문제: Actor는 이미 World가 Root 역할을 함!
        // 중복 보호 = 불필요한 성능 낭비 + 복잡성 증가
    }
};
```

- 애초에 안쓰는게 좋은 함수임<br>

### 더 좋은 대안들

1. UPROPERTY 사용 (95% 상황에서 최선)
    
```cpp
// 대안 1: UPROPERTY 사용
class UGoodManager : public UObject
{
    UPROPERTY()  // AddToRoot 없이도 안전하게 관리
    TArray<UObject*> ManagedObjects;

public:
    void AddManagedObject(UObject* NewObj)
    {
        ManagedObjects.Add(NewObj);  // UPROPERTY로 충분히 보호됨
    }
};
```

- UPROPERTY만 잘 활용하여도 어느정도 안전<br>

2. Outer 관계 활용

```cpp
// 대안 2: Outer 관계 활용
class UDataContainer : public UObject
{
    GENERATED_BODY()
    
public:
    UObject* CreateSubData(TSubclassOf<UObject> DataClass)
    {
        // this가 Outer → Container가 사라지면 SubData도 자동 정리
        return NewObject<UObject>(this, DataClass);
    }
};
```

- Outer를 통해 제거 순서를 정하기<br>
- 부모가 살아있는 동안은 자식은 꽤나 안전해짐<br>

3. 게임 인스턴스나 월드에 등록

```cpp
// 대안 3: 게임 인스턴스나 월드에 등록
class UGlobalDataManager : public UObject
{
    GENERATED_BODY()
    
public:
    static UGlobalDataManager* CreateInstance(UGameInstance* GameInstance)
    {
        // GameInstance가 Outer → 게임이 끝날 때까지 보존
        return NewObject<UGlobalDataManager>(GameInstance);
        // AddToRoot() 불필요! GameInstance가 이미 Root이므로 자동 보호
    }
};
```

- World나 GameInstance를 Outer로 잡으면<br>
  사실상 RootSet에 등록된 것과 비슷한 효과를 얻을 수 있음<br>

# 3. 참조 관리 기초 🛹

## 3-1. Hard vs Weak: 선택 규칙과 안전 패턴

### Hard Reference (강한 참조)

```cpp
UCLASS()
class UInventory : public UObject
{
    GENERATED_BODY()

public:
    // 이게 Hard Reference (강한 참조)
    UPROPERTY()
    TArray<UObject*> Items;  // UObject* = 일반 포인터 = Hard
};
```

```cpp
UInventory* Inventory = NewObject<UInventory>();
UItem* Sword = NewObject<UItem>();

Inventory->Items.Add(Sword);  // Inventory가 Sword를 "소유"
// 이제 Sword는 Inventory가 살아있는 한 절대 삭제되지 않음
```

- 일반적인 사용 방식<br>
- 참조 중엔 절대 사라지지 않음<br>
  GC의 보호를 받음<br>

- 소유와 책임<br>

### Weak Reference (약한 참조)

```cpp
UCLASS()
class UItemWidget : public UObject
{
    GENERATED_BODY()

public:
    // 이게 Weak Reference (약한 참조)
    UPROPERTY()
    TWeakObjectPtr<AActor> ObservedActor;  // TWeakObjectPtr = Weak

    void UpdateDisplay()
    {
        // Weak 참조는 항상 유효성 체크 필수!
        if (ObservedActor.IsValid())  // "아직 살아있나?"
        {
            AActor* Actor = ObservedActor.Get();  // 실제 포인터 가져오기
            FString Name = Actor->GetName();      // 이제 안전하게 사용
        }
        else
        {
            // 대상이 이미 삭제됨 → UI를 비우거나 기본값 표시
        }
    }
};
```

- Weak를 통해 파악<br>
- RefCount를 늘리지 않음<br>

### 사용법 비교

```cpp
// Hard Reference 체크
UPROPERTY()
AActor* MyActor;

void CheckHardRef()
{
    if (IsValid(MyActor))  // 전역 함수 IsValid()
    {
        MyActor->DoSomething();  // 바로 사용 가능
    }
}

// Weak Reference 체크
UPROPERTY()
TWeakObjectPtr<AActor> MyActorWeak;

void CheckWeakRef()
{
    if (MyActorWeak.IsValid())  // 멤버 함수 IsValid()
    {
        AActor* Actor = MyActorWeak.Get();  // Get()으로 포인터 획득
        Actor->DoSomething();               // 이제 안전하게 사용
    }
}
```

- Hard : IsValid로 체크 가능<br>
  - != nullptr 보다 더 안전한 검증 가능(메모리 검증도 해주므로)<br>

- Weak의 경우는 자체적인 멤버 함수를 통해 살아있는지 여부를 확인<br>

### 실제 게임 상황에서의 선택 기준

- 전형적인 `소유` 관계 (Hard)<br>
  - 직접적인 연관성이 있고, '관리' 등이 필요함<br>
  - 하나가 사라지면, 나머지도 사라짐<br>

```cpp
// 인벤토리 시스템
class UInventory : public UObject
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TArray<UItem*> Items;  // Hard: 인벤토리가 아이템들을 소유
};

// 스킬 시스템  
class USkillManager : public UObject
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TArray<USkill*> LearnedSkills;  // Hard: 매니저가 스킬들을 관리
};
```
    
- 전형적인 `관찰` 관계 (Weak)<br>
  - 필요는 하지만, 같이 사라질 정도는 아님<br>

```cpp
// UI 시스템
class UHealthBarWidget : public UUserWidget
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TWeakObjectPtr<ACharacter> WatchedCharacter;  // Weak: UI는 관찰만 함
};

// 미니맵 시스템
class UMinimapWidget : public UUserWidget  
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TWeakObjectPtr<APawn> TrackedPlayer;  // Weak: 추적만 함
};
```
    

### 자주 하는 실수들

1. 실수 1: UPROPERTY 없는 생 포인터<br>
    
```cpp
// ❌ 절대 금지! - GC가 모르는 포인터
class UBadExample : public UObject
{
    UObject* DangerousPtr;  // UPROPERTY 없음 = 크래시 위험!
};
```

2. 실수 2: Weak 참조를 체크 없이 사용<br>

```cpp
// ❌ 위험한 코드 - 삭제된 객체에 접근할 수 있음
void BadWeakUsage()
{
    AActor* Actor = WeakPtr.Get();  // IsValid() 체크 없이 바로 Get()
    Actor->DoSomething();  // 💥 크래시 가능성!
}

// ✅ 올바른 코드 - 항상 체크 후 사용
void GoodWeakUsage()
{
    if (WeakPtr.IsValid())  // 먼저 살아있는지 확인
    {
        AActor* Actor = WeakPtr.Get();
        Actor->DoSomething();  // 안전
    }
}
```
    
3. 실수 3: 모든 걸 Hard로 하기<br>

## 3-2. 순환 참조 차단: 메모리 누수의 주범을 막자

### 문제: 순환 참조

```cpp
// ❌ 순환 참조 - 재앙의 시작
class UParent : public UObject
{
    UPROPERTY()
    UChild* Child;  // Parent가 Child를 Hard 참조
};

class UChild : public UObject
{
    UPROPERTY()
    UParent* Parent;  // Child도 Parent를 Hard 참조 (문제!)
};
```

```
Parent ─(강한 참조)→ Child
  ↑                    ↓
  └──(강한 참조)───────┘

"나는 저 애가 있어야 살 수 있어!"
"나도 저 애가 있어야 살 수 있어!"
결과: 둘 다 영원히 메모리에서 안 사라짐
```

- 순환 참조는 발견하기 어려움<br>

### 해결법 1: 한쪽을 Weak으로

```cpp
// ✅ 올바른 코드 - 순환 참조 해결!
UCLASS()
class UParent : public UObject
{
    GENERATED_BODY()
public:
    UPROPERTY()
    UChild* Child;  // Parent → Child는 Hard (소유)

    void CreateChild()
    {
        Child = NewObject<UChild>(this);
        Child->SetParent(this);  // 자식에게 부모 알려주기
    }
};

UCLASS()
class UChild : public UObject
{
    GENERATED_BODY()
public:
    UPROPERTY()
    TWeakObjectPtr<UParent> Parent;  // Child → Parent는 Weak (역참조)

    void SetParent(UParent* NewParent)
    {
        Parent = NewParent;
    }

    void DoSomethingWithParent()
    {
        if (Parent.IsValid())  // 항상 체크!
        {
            UParent* P = Parent.Get();
            // 안전하게 부모 사용
        }
    }
};
```

```
Parent ─(강한 참조)→ Child
  ↑                    ↓
  └──(약한 참조)───────┘

Parent: "나는 Child를 소유하고 책임진다"
Child: "Parent를 지켜보긴 하지만 붙잡고 있지는 않아"
결과: Parent가 필요없어지면 Child도 함께 정리됨
```

- 둘 중 하나를 Weak로 바꾸어 처리<br>

### 실제 게임 예시들

```cpp
// 인벤토리 시스템
class UInventory : public UObject
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TArray<UItem*> Items;  // Hard: 인벤토리가 아이템 소유
};

class UItem : public UObject  
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TWeakObjectPtr<UInventory> OwnerInventory;  // Weak: 아이템이 소유자 참조
    
    void UseItem()
    {
        if (OwnerInventory.IsValid())
        {
            UInventory* Inv = OwnerInventory.Get();
            // 인벤토리에서 자신을 제거하거나 다른 작업 수행
        }
    }
};
```

```cpp
// UI 시스템
class UPlayerHUD : public UUserWidget
{
    GENERATED_BODY()
    
public:
    UPROPERTY()
    TArray<UWidget*> ChildWidgets;  // Hard: HUD가 자식 위젯들 소유
};

class UHealthBar : public UUserWidget
{
    GENERATED_BODY()
    
public:
    UPROPERTY()  
    TWeakObjectPtr<UPlayerHUD> ParentHUD;  // Weak: 자식이 부모 참조
};
```

### 해결법 2: 인터페이스로 의존성 줄이기

```cpp
// 1. 인터페이스 정의 - "체력을 가진 것"
UINTERFACE()
class UHealthOwner : public UInterface
{
    GENERATED_BODY()
};

class IHealthOwner
{
    GENERATED_BODY()
public:
    virtual float GetHealth() const = 0;
    virtual void TakeDamage(float Damage) = 0;
};

// 2. 캐릭터가 인터페이스 구현
UCLASS()
class AMyCharacter : public AActor, public IHealthOwner
{
    GENERATED_BODY()

    float Health = 100.0f;

public:
    virtual float GetHealth() const override { return Health; }
    virtual void TakeDamage(float Damage) override { Health -= Damage; }
};

// 3. UI가 인터페이스로 소통
UCLASS()
class UHealthBar : public UUserWidget
{
    GENERATED_BODY()

    UPROPERTY()
    TWeakObjectPtr<UObject> Target;  // 구체적인 타입을 모름!

public:
    void UpdateHealthBar()
    {
        if (!Target.IsValid()) return;

        UObject* Obj = Target.Get();

        // 인터페이스를 구현했는지 확인
        if (Obj->GetClass()->ImplementsInterface(UHealthOwner::StaticClass()))
        {
            // 인터페이스를 통해 소통
            IHealthOwner* HealthOwner = Cast<IHealthOwner>(Obj);
            float CurrentHealth = HealthOwner->GetHealth();
            // UI 업데이트...
        }
    }
};
```

- 연관성을 줄이는 방식<br>

### 해결법 3: 델리게이트로 이벤트 기반 소통

```cpp
class UEventSender : public UObject
{
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float, NewHealth);

    UPROPERTY(BlueprintAssignable)
    FOnHealthChanged OnHealthChanged;

    void ChangeHealth(float NewHealth)
    {
        OnHealthChanged.Broadcast(NewHealth);  // 이벤트 발생
    }
};

class UEventReceiver : public UObject
{
public:
    void StartListening(UEventSender* Sender)
    {
        if (Sender)
        {
            // 이벤트 바인딩
            Sender->OnHealthChanged.AddDynamic(this, &UEventReceiver::OnHealthChanged);
        }
    }

    virtual void BeginDestroy() override
    {
        // ⚠️ 중요: 죽기 전에 반드시 언바인딩!
        if (EventSender && IsValid(EventSender))
        {
            EventSender->OnHealthChanged.RemoveDynamic(this, &UEventReceiver::OnHealthChanged);
        }

        Super::BeginDestroy();
    }

    UFUNCTION()
    void OnHealthChanged(float NewHealth)
    {
        // 이벤트 처리
    }

private:
    UPROPERTY()
    TWeakObjectPtr<UEventSender> EventSender;  // Weak 참조로 저장
};
```

- Delegate를 사용할 수 있다면<br>
  연결을 느슨하게 할 수 있음<br>

- 특정한 타이밍에 호출시킴에 따라<br>
  '굳이' 부모/자식 을 알 필요가 없음<br>