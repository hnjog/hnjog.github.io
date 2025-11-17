---
title: "문승현 튜터님 강의 - '메모리 구조와 언리얼 객체 시스템'"
date : "2025-11-17 13:00:00 +0900"
last_modified_at: "2025-11-17T13:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Memory Struct
  - Unreal GC
---

## 🖥️ "게임.exe"가 실행될 때의 모습

여러분이 빌드한 "MyGame.exe"를 실행하면, 운영체제(OS, Windows 등)는 이 프로그램을 위해 "프로세스"라는 독립된 실행 환경을 구축<br>

OS는 이 프로세스에게 가상 메모리(Virtual Memory)라는 거대한 '개인 영토'를 할당<br>
- 이 영토는 실제 물리 RAM과는 분리된, 프로그램만이 볼 수 있는 0번지부터 시작하는 거대한 주소 공간<br>
- 이 영토는 목적에 따라 크게 4개의 핵심 구역으로 나뉨<br>
- 이러한 영역 분류를 'segment' 라고도 함<br>

16기가 DRAM을 가진다고 20GB 게임을 실행하지 못하는게 아님<br>
- `가상 메모리`는 '프로그램/프로세스'이(가) 온전히 자신이 OS를 독점한다고 여기는 착각<br>
- 실제 DRAM 에서는 여러 개의 프로세스들과 함께 쪼개져 있음<br>

### 1. Code (Text) 영역

- **비유:** 게임의 모든 규칙과 행동이 적힌 **"설계도 원본"**<br>
- **내용:** 우리가 작성한 C++ 코드가 컴파일되어 CPU가 직접 읽을 수 있는 기계어 명령어(Machine Code)로 번역된 데이터가 저장<br>
- **수명:** 프로그램 시작 시 로드되어, 프로그램이 종료될 때까지 유지<br>

### 🎯 핵심 특징

1. **읽기 전용 (Read-Only) / 실행 전용 (Execute-Only):**<br>
    - 가장 중요한 특징입니다. CPU는 이곳의 명령어를 '읽고 실행'할 수만 있습니다.<br>
    - 만약 실행 중에 프로그램이 이 Code 영역의 내용을 수정하려 시도하면 (e.g., `0x00401010 = 5;`),<br>
      OS는 이를 심각한 보안 위협(e.g., 버퍼 오버플로우를 통한 코드 주입)으로 간주하고 프로세스를 즉시 강제 종료시킵니다.<br>

2. **고정 크기 (Fixed Size):**<br>
    - 프로그램이 실행되는 도중에 이 영역의 크기나 내용이 변하지 않습니다.<br>

여러분이 C++로 구현하는 `AMyCharacter::Jump()` 함수, <br>
`UGameplayStatics::ApplyDamage()` 같은 모든 로직의 '실체'가 바로 이곳에 있습니다.<br>

이해를 위해 다음의 코드를 봐봅시다.<br>

```cpp
// 1. 우리가 작성한 C++ 코드 (MyGame.cpp)
int Add(int a, int b)
{
    int result = a + b; // <- 이 로직이...
    return result;
}

void Start()
{
    int Value = Add(10, 20); // <- 함수 "호출"
}
```

해당 코드는 정확하지는 않지만<br>
아래와 같은 모습으로 Code 영역에 저장됩니다.<br>

```cpp
// 2. 컴파일된 후 'Code 영역'에 저장되는 모습 (어셈블리어)
 
0x00401010 (Add함수 시작): PUSH EBP      ; 스택 프레임 설정
0x00401011:               MOV EBP, ESP
0x00401013:               MOV EAX, [EBP+8]  ; 'a' (10)를 EAX 레지스터로
0x00401016:               ADD EAX, [EBP+12] ; 'b' (20)를 EAX에 더함
0x00401019:               MOV [EBP-4], EAX  ; 'result' 변수 위치에 EAX (30) 저장
0x0040101C:               POP EBP           ; 스택 프레임 복구
0x0040101D:               RET               ; 호출 지점(Start)으로 복귀

// 'Start' 함수가 Add(10, 20)을 호출하면, CPU의 실행 포인터(EIP/RIP)가 '0x00401010' 주소로 점프하여 이 명령어들을 순차적으로 실행합니다.
```

### 2. Static (Data / BSS) 영역

- **비유:** 프로그램 전체가 공유하는 **"중앙 공용 창고"**<br>
- **내용:** 전역 변수(Global Variable)와 정적 변수(Static Variable)가 저장됩니다.<br>
- **수명:** 프로그램 시작부터 (main 함수 실행 전) 종료까지, **프로세스의 수명과 동일**합니다.<br>

### 🎯 핵심 특징

이 영역은 다시 두 부분으로 나뉩니다.<br>

1. **Data (Initialized) 영역:**<br>
    - `초기값이 0이 아닌 전역/정적 변수`가 저장됩니다.<br>
    - `.exe` 파일 자체에 이 초기값(e.g., `100`, `"MyGame"`)이 저장되어 있다가, 프로그램 로드 시 메모리에 그대로 복사됩니다.<br>
2. **BSS (Uninitialized) 영역:**<br>
    - **초기값이 없거나 `0`으로 초기화된 전역/정적 변수**가 저장됩니다.<br>
    - `.exe` 파일에는 "이만큼의 공간이 필요하다"는 정보만 기록됩니다. (파일 크기 절약)<br>
    - 프로그램 로드 시 OS가 이 영역 전체를 **0으로 자동 초기화**해줍니다.<br>
    

전역 변수는 모든 곳에서 접근 가능해 편리하지만, <br>
데이터가 언제 어디서 변경되었는지 추적하기가 <br>
극도로 어려워져(스파게티 코드) 디버깅을 불가능하게 만듭니다.<br>

언리얼에서는 원시 C++ 전역 변수 대신 <br>
`UGameInstance`나 `UGameSingleton` (또는 각종 Subsystem)을<br>
사용하도록 강력히 권장합니다. <br>

이 객체들 자체는 '힙(Heap)'에 생성되지만, <br>
게임의 수명 내내 존재하며 전역적인 데이터 저장소 역할을 합니다.<br>

```cpp
// 1. Data 영역 (초기값이 0이 아님)
int G_TotalScore = 100;
const char* G_GameName = "MyUnrealGame"; // (문자열 리터럴도 특수 영역에 저장됨)

// 2. BSS 영역 (초기값 없거나 0 -> OS가 0으로 채워줌)
bool G_IsGameRunning; // = 0 (false)
float G_DamageModifiers[100]; // 모든 요소가 0.0f로 초기화됨

void MyFunction()
{
    // 3. Data 영역 (함수 내 'static' 변수)
    // 이 변수는 스택이 아닌 Static 영역에 저장됩니다.
    static int FunctionCallCount = 0; 
    
    // 이 코드는 프로그램 실행 중 *최초 1회만* 실행됩니다.
    FunctionCallCount++; 
    
    // 함수가 종료되어도 이 값은 유지됩니다. (공용 창고에 보관됨)
    // (첫 호출: 1, 두 번째 호출: 2, ...)
    UE_LOG(LogTemp, Warning, TEXT("Function Call Count: %d"), FunctionCallCount);
}

// 위험성:
void AnotherFunction()
{
    // G_TotalScore가 100일 것으로 예상했지만...
    if (G_TotalScore > 50)
    {
        // ...
    }
    // 이 함수와 전혀 상관없는 다른 모듈, 다른 스레드에서
    // G_TotalScore = -999; 라고 바꿨을 수도 있습니다.
    // 이것이 C++ 전역 변수의 가장 큰 문제입니다.
}
```

### 3. Stack (스택) 영역

- **비유:** 함수(작업자)를 위한 **"임시 개인 작업대"**<br>
- **내용:** 함수 호출 시 생성되는 **지역 변수(Local Variable)**, 함수에 전달되는 **매개 변수(Parameter)**, 함수가 끝나고 돌아갈 **복귀 주소(Return Address)**.<br>
- **수명:** 함수(또는 `{}` 스코프)가 시작될 때 생성되고, 함수가 `return` 할 때 **즉시 사라집니다.**<br>

### 🎯 핵심 특징

1. **LIFO (Last-In, First-Out):**<br>
    - '마지막에 들어온 것이 가장 먼저 나간다'는 자료구조입니다.<br>
    - `main()`이 `A()`를 호출하고, `A()`가 `B()`를 호출하면, 스택에는 `main -> A -> B` 순서로 "스택 프레임"이 쌓입니다.<br>
    - 종료는 역순 (`B -> A -> main`)으로 이루어집니다.<br>

2. **매우 빠름 (가장 중요):**<br>
    - 메모리를 "할당(Allocation)"하고 "해제(Deallocation)"하는 과정이,<br>
      단순히 **스택 포인터(SP) 레지스터**의 주소값을 더하고 빼는 **단일 CPU 명령어**로 이루어집니다.<br>

    - 힙(Heap)처럼 '어디에 빈 공간이 있나?'를 검색할 필요가 전혀 없습니다.<br>

3. **제한된 크기와 위험 (Stack Overflow):**<br>
    - 스택 영역은 크기가 상대적으로 작습니다 (OS/설정마다 다르지만 보통 몇 MB).<br>

    - 재귀 호출이 너무 깊어지거나 스택에 너무 큰 배열(e.g., `int BigArray[5000000]`)을 선언하면,<br>
      할당된 스택 영역을 초과하여 `스택 오버플로우(Stack Overflow)`가 발생해 프로그램이 즉시 다운됩니다.<br>
      - 스택이 **다른 영역**을 침범하거나 그 중간의 **Guard page**를 침범하려하면<br>
        OS가 Kill하게 됨(`Segmentation Fault`)<br>

스택은 빠르기 때문에 가능한 한 스택을 활용하는 것이 좋습니다.<br>
하지만 "복사 비용"을 항상 생각해야 합니다.<br>

`FTransform`, `FHitResult`, `TArray` 같은 **큰 구조체(Struct)나 컨테이너**를<br>
함수에 '값으로 전달(Pass-by-Value)'하면,<br>
이 모든 데이터가 스택에 **통째로 복사**됩니다.<br>
이는 스택 공간 낭비이자 심각한 성능 저하입니다.<br>

- *나 &를 사용하는 이유!<br>

> [나쁜 예] void ProcessHitResult(FHitResult Hit)호출 시마다 수백 바이트의 FHitResult가 스택에 복사됩니다.
> 
> 
> **[좋은 예]** `void ProcessHitResult(const FHitResult& Hit)`*'const 참조(&)'를 사용하면, 원본의 '주소'(포인터, 8바이트)만 스택에 복사됩니다. (읽기 전용)*
> 

```cpp
// (주소는 높은 곳 -> 낮은 곳으로 자라남)
// 3. CalculateDamage 호출됨
int CalculateDamage(int BaseDamage, const FVector& HitLocation) 
{
    // 4. 스택 프레임에 'Bonus', 'DamageScale' 생성
    int Bonus = 10;                // (예: 스택 주소 0x7FFF...E00C)
    float DamageScale = 1.5f;      // (예: 스택 주소 0x7FFF...E008)
    
    // 'HitLocation'은 const&이므로, 
    // FVector(12바이트) 자체가 복사되지 않고 '주소'(8바이트)만 전달됩니다.
    if (HitLocation.Z > 100.0f)
    {
        // 데미지 스케일을 3배(헤드샷)로 적용
        DamageScale = 3.0f; 
        UE_LOG(LogTemp, Warning, TEXT("Headshot!"));
    }
    
    return (BaseDamage + Bonus) * DamageScale;
    
} // 5. CalculateDamage 리턴. 
  //    스택 포인터가 원래 위치로 '즉시' 돌아갑니다.
  //    0x7FFF...E00C, E008 영역은 "버려짐" (무효화)

void AMyPlayer::TakeDamage(int DamageAmount) // 1. TakeDamage 함수 시작
{
    // 2. 스택 프레임에 'PlayerHealth', 'HitPos' 생성
    int PlayerHealth = 100;      // (예: 스택 주소 0x7FFF...E01C)
    FVector HitPos = FVector(0,0,100); // (예: 스택 주소 0x7FFF...E010)

    int FinalDamage = CalculateDamage(DamageAmount, HitPos); // (50, HitPos 주소)
    // (예: 스택 주소 0x7FFF...E004에 리턴값(90) 저장)

    PlayerHealth -= FinalDamage;
    
} // 6. TakeDamage 리턴. 스택 프레임 전체(E01C ~ E004)가 "버려짐"
```

### 4. Heap (힙) 영역

- **비유:** 필요할 때마다 빌려 쓰는 **"동적 임대 창고"**<br>
- **내용:** 프로그래머가 코드 실행 중에 동적으로 할당(new)하는 모든 데이터.<br>
- **수명:** 프로그래머가 `new`로 요청(할당)할 때 생성되며, `delete`로 명시적으로 해제하거나 GC가 수거할 때까지 **절대 자동으로 사라지지 않습니다.** (함수가 끝나도 유지됨)<br>

### 🎯 핵심 특징

1. **유연성 (Flexibility):**<br>
    - 프로그램 실행 중에 필요한 만큼 (OS가 허락하는 한) 큰 메모리를 할당받을 수 있습니다. 스택과 달리 크기 범위가 훨씬 큽니다.<br>
      - 메인 메모리를 초월한 데이터도 할당을 받을 수는 있으나<br>
        (다만 OS 설정에 따라 다름)<br>
        실제 접근 시에 터지는 경우가 발생은 함<br>

2. **느린 속도 (Slow):**<br>
    - 스택과 달리, `new`를 호출하면 메모리 관리자(OS/Allocator)가 힙 영역을 탐색해<br>
      요청한 크기(e.g., 500바이트)를 할당할 수 있는 빈 공간(Free Block)을 "검색"해야 합니다.<br>

    - 해제(`delete`) 시에도 이 공간을 '빈 공간 목록'에 다시 추가하는 작업이 필요합니다.<br>
      이 과정은 스택 포인터 이동보다 수천 배 느릴 수 있습니다.<br>

3. **단편화 (Fragmentation):**<br>
    - 할당(new)과 해제(delete)를 반복하면, 힙 영역이 잘게 쪼개진 "빈 공간"들로 가득 차게 됩니다.<br>
    - 결과: "총 여유 공간은 100MB인데, 연속된 50MB짜리 빈 공간이 없어서" 할당에 실패할 수 있습니다.<br>
      - 오브젝트 풀링은 단편화를 예방하기 위한 좋은 선택지가 될 수도 있음<br>
        (첫 할당 이후, 게임을 종료할때까지 재활용)<br>
      - 그 외에도 한번에 선형적으로 생성 / 리셋 -> 큰 공간 위주의 할당 전략이 가능<br>
        

4. **메모리 누수 (Memory Leak) - 가장 큰 위험:**<br>
    - 힙에 `new`로 할당한 객체를 사용한 뒤, `delete` 하는 것을 잊어버리면 발생합니다.<br>
    - 위의 단편화와 연계되면 악순환이 발생 가능하므로 주의할 것<br>
    

**💥 C++ 힙 관리의 2가지 치명적인 문제**

힙 영역을 수동으로 관리하는 C++의 `new`/`delete` 방식은<br>
두 가지 심각한 문제를 안고 있습니다.<br>

- TMI) new / delete 와 malloc / free의 차이점<br>
  new : 생성자 호출, 하나의 연산자임 (연산자 오버로딩 가능)<br>
  malloc : 메모리 할당만 함<br>

**[문제 1] 메모리 누수 (Memory Leak)**

`new`로 빌린 메모리를 `delete`로 반납하는 것을 잊는 경우입니다.<br>

```cpp
// ----------------------------------------
// 순수 C++의 '메모리 누수' 예제
// ----------------------------------------
class MyData { public: int Value = 100; };

void CreateData_Leak()
{
    // 1. 'Ptr' 변수 자체는 *스택*에 생성
    // 2. 'Ptr'이 가리키는 *실제 데이터*는 *힙*에 생성 (주소: 0x1000A000)
    MyData* Ptr = new MyData(); 

    // 3. 'delete Ptr;'을 잊어버렸습니다!
} 
// 4. 함수 리턴
  
// 5. *스택*에 있던 'Ptr' 변수는 "사라집니다".
  
// 6. 결과:
//    아무도 'MyData' 객체(0x1000A000)의 주소를 모르므로,
//    이 메모리는 프로그램 종료 시까지 영원히 "누수"됩니다.
```

**[문제 2] 댕글링 / Stale 포인터 (Dangling Pointer)**

메모리 누수보다 더 심각한 문제입니다. `delete`는 했지만, <br>
다른 포인터가 여전히 그 '삭제된' 주소를 가리키고 있다가 접근할 때 발생합니다.<br>

```cpp
MyData* Ptr_A = new MyData(); // A가 객체 생성 (0x1000A000)
MyData* Ptr_B = Ptr_A;      // B가 A와 동일한 객체를 가리킴

delete Ptr_A; // A가 객체를 삭제함
Ptr_A = nullptr; 

// Ptr_B는 Ptr_A가 객체를 삭제한 사실을 모릅니다.
// Ptr_B는 여전히 '삭제된' 주소 0x1000A000을 가리키고 있습니다. (Stale Pointer)

Ptr_B->Value = 300; // 💥 CRASH! (존재하지 않는 메모리에 접근)
```

- Weak_Ptr 이나 Shared_Ptr 같은 스마트 포인터가 권장되는 이유 중 하나<br>

**💡 언리얼의 해결책: GC와 `UPROPERTY`**

언리얼 엔진의 모든 `UObject` (Actor, Component, Widget 등)는 이 힙(Heap) 영역에 생성됩니다.<br>

언리얼은 위에서 언급된 [문제 1: 메모리 누수]와 [문제 2: 댕글링 포인터]를 <br>
가비지 컬렉션(GC)과 `UPROPERTY`라는 하나의 시스템으로 동시에 해결합니다.<br>

- 추가적으로 GC는 잠재적으로 Heap의 문제인 '단편화'와 '메모리 누수' 문제에 도움을 줌<br>
  - GC가 정기적으로 참조되지 않는 UObject 제거 -> 메모리 누수를 제거<br>
  - 

```cpp
// (헤더 파일: AMyPlayer.h)
UCLASS()
class AMyPlayer : public APawn
{
    GENERATED_BODY()

    // 1. "UPROPERTY()" 매크로
    //    이것이 GC에게 정보를 등록하는 마법의 키워드입니다.
    UPROPERTY() 
    TObjectPtr<AMyActor> MyActorPtr; // (TObjectPtr은 UE5의 안전한 포인터)
};

// (소스 파일: AMyPlayer.cpp)
void AMyPlayer::SpawnMyActor()
{
    // 2. 힙에 'AMyActor' 객체 생성 (주소: 0x2000B000)
    AMyActor* MySpawnedActor = GetWorld()->SpawnActor<AMyActor>();

    // 3. 'UPROPERTY' 멤버 변수에 주소(0x2000B000) 저장
    this->MyActorPtr = MySpawnedActor; 
}
```

`UPROPERTY()`가 두 가지 문제를 각각 어떻게 해결하는지 보겠습니다.<br>

**1. [메모리 누수 문제] 해결: 강한 참조 (Strong Reference)**

`UPROPERTY()`는 GC에게 "이 객체(AMyActor)는 AMyPlayer가 여전히 사용 중이니,<br>
 '쓰레기'가 아니다"라고 알려주는 **'강한 참조(Strong Reference)'** 신호입니다.<br>

- **결과:** `AMyPlayer` 객체가 파괴되지 않는 한, GC는 `MyActorPtr`가 참조하는 `AMyActor`를 절대 '쓰레기'로 취급하지 않고 삭제하지 않습니다.<br>

- 개발자는 `new`만 하고 `delete`를 잊는 **[문제 1: 메모리 누수]** 걱정을 할 필요가 없습니다.<br>
  나중에 `AMyPlayer`가 파괴되거나 `MyActorPtr = nullptr;`가 되어 이 '참조'가 끊기면, GC가 알아서 `AMyActor`를 수거해갑니다.<br>

**2. [댕글링 포인터 문제] 해결: 자동 Null-Setting**

`UPROPERTY()`의 진짜 핵심입니다. C++의 [문제 2]처럼, <br>
`AMyActor`가 어떤 이유로든 (e.g., `AMyActor->Destroy()` 호출) **정당하게 삭제되었을 때**를 가정해 봅시다.<br>

- **`UPROPERTY`가 없었다면?**`AMyPlayer`는 `AMyActor`가 삭제된 사실을 모릅니다. <br>
  `MyActorPtr` 변수에는 여전히 `0x2000B000`이라는 '삭제된' 주소(Stale Pointer)가 남아있고,<br>
  다음 `Tick`에서 `MyActorPtr->...`를 호출하는 순간 **즉시 크래시**가 발생합니다.<br>

- **`UPROPERTY`가 있다면? (GC의 '사망 통지서')**`UPROPERTY()`는 GC에게<br>
  "만약 이 객체(AMyActor)가 삭제되면, 나(MyActorPtr)에게 즉시 알려줘!"라고 등록하는 '사망 통지' 예약과 같습니다.<br>
    
- GC가 `AMyActor`(0x2000B000)를 삭제하기로 결정하는 순간,<br>
  **엔진 GC 코드가** `MyActorPtr` 변수의 메모리에 **직접 접근**하여 그 값을 `0x2000B000`에서 **`nullptr`로 강제 덮어써버립니다.**<br>
    
- **결과 (크래시 방지):** 개발자는 `if (MyActorPtr)` 또는 `if (IsValid(MyActorPtr))`라는 간단한 검사만으로, '삭제된' 객체에 접근하려는 시도를 100% 막을 수 있습니다.<br>

```cpp
// AMyPlayer::Tick(float DeltaTime)
// ...
// [!!!] 어딘가에서 AMyActor->Destroy()가 호출되고 GC가 실행됨
//
// <--- GC가 MyActorPtr 변수의 값을 0x2000B000 -> nullptr로 덮어썼음
//
// ... 다시 AMyPlayer::Tick 실행

if (MyActorPtr) // false 입니다! (GC가 nullptr로 바꿈)
{
    // 이 블록은 실행되지 않습니다. (크래시가 완벽하게 방지됨!)
    MyActorPtr->SomeFunction(); 
}
else
{
    // 안전하게 여기로 진입합니다.
    UE_LOG(LogTemp, Warning, TEXT("내 Actor가 GC에 의해 사라졌습니다."));
}

```

**[참고] 자동 `nullptr`의 작동 원리 (엔진 내부)**<br>

이 '자동 `nullptr` 덮어쓰기'는 마법이 아니라, 언리얼 헤더 툴(UHT)과 **GC**의 합작품입니다.<br>

1. **컴파일 시 (UHT):** `UPROPERTY()` 매크로를 발견하면, UHT가 "AMyPlayer 클래스는 OOO 바이트 위치에 AMyActor를 가리키는 포인터(MyActorPtr)가 있다"는 '명단'을 생성하여 엔진에 등록합니다.<br>
2. **런타임 시 (GC 실행):** GC가 `AMyActor`(0x2000B000)를 '쓰레기'로 판단하고 삭제하기로 결정합니다.<br>
3. **Null-Setting (핵심):** GC는 객체를 *즉시 삭제하지 않고*, 1번에서 만든 '명단'을 전부 스캔합니다.<br>
    - GC: "현재 삭제될 `0x2000B000` 주소를 가리키는 `UPROPERTY`가 누구누구지?"<br>
    - GC: "아, AMyPlayer 객체의 `MyActorPtr` 변수가 가리키고 있네."<br>
    - GC가 `MyActorPtr` 변수의 메모리에 직접 접근하여, 그 값을 `nullptr`로 강제 덮어써버립니다.<br>
    - 이제 `AMyActor`를 안전하게 `delete` 합니다.<br>

### 언리얼 GC 심층 탐구 (Root Set과 Mark and Sweep)

우리는 `UPROPERTY`가 C++의 메모리 누수와 댕글링 포인터 문제를 해결하는 '마법'이라고 배웠습니다.<br>

이 모듈에서는 그 '마법'을 부리는 가비지 컬렉터(GC)가 "누가 쓰레기이고, 누가 아닌지"를 정확히 어떻게 판단하는지 그 내부 원리를 파헤쳐 봅니다.<br>

**1. GC의 기본 전제: UObject**

GC의 관리를 받는 모든 객체는 `UObject`를 상속받아야 합니다.<br>

- `new` 키워드가 아닌, `NewObject<T>()` (UObject용) 또는 `GetWorld()->SpawnActor<T>()` (AActor용)를 사용해야 합니다.<br>
- 이 함수들은 객체를 '힙(Heap)'에 생성함과 동시에, "GC야, 내가 방금 이 객체를 만들었으니 너의 관리 목록에 추가해!"라고 '출생 신고'를 하는 것과 같습니다.<br>
- 이제 GC는 이 객체의 '생사'를 추적하기 시작합니다.<br>

**2. GC의 핵심 원리: 도달 가능성 (Reachability)**

GC는 "이 객체가 필요한가?"라는 어려운 질문에 대답하지 않습니다. 대신 "이 객체에 '도달'할 수 있는가?"라는 간단한 질문에만 대답합니다.<br>

- **살아있는 객체:** 어떻게든 '도달'할 수 있는 객체.<br>
- **쓰레기 객체:** 절대 '도달'할 수 없는 객체.<br>

그럼 "어디로부터" 도달하는 것이 기준일까요? 바로 루트 셋(Root Set)입니다.<br>

**3. GC의 출발점: 루트 셋 (The Root Set)**

루트 셋은 **"절대로 GC의 대상이 되어서는 안 되는, 항상 살아있다고 보장된"** 객체들의 '최상위' 목록입니다. GC는 항상 이 '루트 셋'에서부터 탐색을 시작합니다.<br>

- **루트 셋의 예:**
    - `UGameInstance` (게임이 켜져 있는 내내 존재)<br>
    - `UWorld` (현재 로드된 레벨)<br>
    - `AddToRoot()` 함수로 명시적으로 '루트'에 추가된 객체<br>
    - (그 외 엔진이 관리하는 핵심 싱글톤 객체들)<br>
      (아마 SubSystem 계열?)<br>

**4. GC의 핵심 알고리즘: Mark and Sweep (표시 및 쓸기)**

GC가 실행되면 (보통 일정 시간 간격으로), 다음 두 단계를 거칩니다.<br>
(일정 주기 + 특정 조건에 GC 호출)<br>

- 보통 다음과 같은 조건에 따라 추가적으로 GC가 호출됨<br>
  - 메모리 사용량 증가로 인한 호출<br>
  - 맵(World) 전환 시 자동 호출<br>
  - Editor 에서 특정 에디터 이벤트 호출에 따라 호출<br>
  - 개발자가 명시적으로 호출 가능<br>
    (`CollectGarbage(GARBAGE_COLLECTION_KEEPFLAGS);`)<br>

**1단계: Mark (표시)**

"살아있는" 객체를 모두 '표시'하는 단계입니다.<br>

1. GC가 루트 셋(Root Set)에 있는 모든 객체를 "살아있음(Reachable)"이라고 표시(Mark)합니다.<br>
2. 이 객체들이 참조하는 **모든 `UPROPERTY()` 포인터**를 따라갑니다.<br>
3. `UPROPERTY()` '다리'를 건너 만나는 모든 `UObject`를 "살아있음"이라고 표시(Mark)합니다.<br>
4. 이 과정을 재귀적으로 반복하여, 루트 셋에서 `UPROPERTY()` 다리를 통해 도달할 수 있는 **모든 객체를 "살아있음"으로 표시**합니다.<br>
- **`UPROPERTY()`가 없는 포인터 (`AMyActor* Ptr`):** GC는 이 포인터를 '다리'로 인식하지 않습니다.<br>
  이 포인터가 가리키는 객체는 루트 셋에서 도달할 수 없으므로 '표시'되지 않습니다.<br>
  (앞서 댕글링에서 'Stale Pointer' 문제가 발생하는 원인입니다.)<br>

**2단계: Sweep (쓸기)**

"쓰레기"를 '쓸어'버리는 단계입니다.<br>

1. GC가 '출생 신고'된 **모든 `UObject`의 목록**을 쭉 훑어봅니다.<br>
2. 각 객체에게 질문합니다. "너, 1단계(Mark)에서 **'살아있음' 표시**를 받았니?"<br>
3. **[답변 1] "네 (표시됨)":**<br>
    - "알았어, 넌 살아있어. 다음 GC를 위해 표시만 지울게." (Unmark)<br>
4. **[답변 2] "아니요 (표시 안 됨)":**<br>
    - "넌 루트 셋에서 도달할 수 없는 쓰레기(Garbage)구나."<br>
    - 이 객체(`0x2000B000`)를 가리키는 **모든 `UPROPERTY` 포인터**를 찾아서, 그 값을 **`nullptr`로 덮어씁니다.** (Null-Setting)<br>
    - 이 객체의 소멸자(`BeginDestroy()`, `FinishDestroy()`)를 호출합니다.<br>
    - 이 객체가 사용하던 힙 메모리를 해제(delete)합니다.<br>

- Mark단계에서<br>
  RootSet -> UObject 들을 체크하여 마킹<br>

- Sweep 단계에서는<br>
  모든 등록된 UObject를 살펴보며 마킹되었는지 체크<br>
