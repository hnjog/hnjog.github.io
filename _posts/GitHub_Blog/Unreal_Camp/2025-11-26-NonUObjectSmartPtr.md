---
title: "김하연 튜터님 강의 - 'Non-UObject 스마트 포인터와 소유권'"
date : "2025-11-24 12:00:00 +0900"
last_modified_at: "2025-11-24T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Smart Pointer
  - Non-UObject
---

# Non-UObject 스마트 포인터와 소유권에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

작은 데이터 하나 쓰려하는데 UObject를 쓰는것은 생각보다 무거움<br>
그런데 Unreal GC에 포함되지 않기에<br>
우리가 잘 관리해주어야 함<br>
- 프로그래머의 실수를 예방하려면?<br>
  -> 스마트 포인터!<br>

# 1. 기존 메모리 관리의 문제점을 복습해보자 💰

### Raw Pointer의 위험성

```cpp
// 메모리 누수 예시
InventoryItem* sword = new InventoryItem("철검");
// delete를 잊으면 메모리 누수 발생!

// 이중 삭제
delete sword;
delete sword; // 💥 크래시!

// 삭제된 메모리 접근
delete sword;
cout << sword->getName(); // 💥 크래시!
```

- Heap 메모리 어딘가에 만들어주기<br>
- 프로그래머의 실수 가능성<br>
    - 메모리 누수하면 몇시간 뒤에 터질수도 있음<br>
    - 2중 삭제<br>
    - 댕글리 포인터 문제<br>

- 안전한 타이밍을 고려하지 못한 메모리 실수는<br>
  생각보다 많이 일어남<br>

### 스마트 포인터?

```cpp
// 스마트 포인터 사용
TSharedPtr<InventoryItem> sword = MakeShared<InventoryItem>("철검");

// 사용
cout << sword->getName(); // 안전하게 접근 가능

// sword 변수가 스코프에서 사라지면
// 메모리는 자동으로 해제
```

- 메모리 자동으로 돌려주기!<br>
- 프로그래머의 실수를 예방 -> 인적 효율 상승<br>

# 2. TSharedPtr - 공유해서 쓰는 스마트 포인터 🧸

## 2-1. 공유 소유권 이해하기

```cpp
// 철검을 만들었음. 현재 사용자: 1명
TSharedPtr<InventoryItem> sword1 = MakeShared<InventoryItem>("철검");

// 두 번째 플레이어도 같은 검을 참조. 현재 사용자: 2명
TSharedPtr<InventoryItem> sword2 = sword1;

// 첫 번째 플레이어가 검을 내려놓음. 현재 사용자: 1명
sword1 = nullptr;

// 두 번째 플레이어도 검을 내려놓음. 현재 사용자: 0명
sword2 = nullptr;

// 이 때! 철검 객체가 자동으로 메모리에서 삭제됩니다.
```

- 참조 카운팅 방식을 통해<br>
  메모리 해제 타이밍을 잡음<br>
- 0이 된 순간, 메모리를 해제<br>

### 간단한 사용법

```cpp
// 1. 생성 - MakeShared 사용
TSharedPtr<InventoryItem> sword = MakeShared<InventoryItem>("철검");

// 2. 복사 - 같은 객체를 공유
TSharedPtr<InventoryItem> anotherSword = sword;

// 3. 사용 - 화살표 연산자로 접근
sword->Use();
FString name = sword->GetName();

// 4. null 체크 - IsValid() 사용
if (sword.IsValid())
{
    sword->Attack();
}

// 5. 해제 - nullptr 대입하면 참조 카운트 감소
sword = nullptr;
anotherSword = nullptr; // 이 순간 객체 자동 삭제
```

- Heap 생성 이후, 참조 카운트 1<br>
  (강한 참조)<br>

## 2-2. 참조 카운팅 시스템의 내부 구조

```cpp
template<typename T>
class TSharedPtr 
{
private:
    T* ObjectPtr;                     // 실제 객체
    FReferenceController* RefController; // 참조 카운트와 제어 정보
};
```

- `FReferenceController` 안에 들어 있는 값<br>
    - `SharedRefCount` : 강한 참조 (TSharedPtr)가 몇 개인지<br>
    - `WeakRefCount` : 약한 참조 (TWeakPtr)가 몇 개인지<br>

- 일반 포인터보다 비용이 조금 더 나가긴 하지만<br>
  그렇게까지 신경쓸 정도는 아닌 편<br>

```cpp
// 객체 생성
TSharedPtr<InventoryItem> sword1 = MakeShared<InventoryItem>("철검");
TSharedPtr<InventoryItem> sword2 = sword1;
TSharedPtr<InventoryItem> sword3 = sword1;

sword1 = nullptr;  // SharedRefCount = 2
sword2 = nullptr;  // SharedRefCount = 1
sword3 = nullptr;  // SharedRefCount = 0 → 객체 삭제!
```

## 2-3. 실제 게임에서 어떻게 쓰일까?

```cpp
class GameManager 
{
private:
    TArray<TSharedPtr<Quest>> ActiveQuests;

public:
    void StartQuest(FString QuestName)
    {
        // 하나의 퀘스트 객체만 생성
        TSharedPtr<Quest> NewQuest = MakeShared<Quest>(QuestName);

        // 게임 매니저가 관리
        ActiveQuests.Add(NewQuest);

        // 여러 시스템이 동일한 객체를 공유
        UIManager->ShowQuest(NewQuest);
        AudioManager->PlayQuestSound(NewQuest);
        SaveManager->RegisterQuest(NewQuest);
    }
};
```

- SharedPtr을 제작하고<br>
  여러 Manager 등에게 공유<br>

- 만든 시점 이후, 참조 카운트가 하나 사라져도(범위 벗어남)<br>
  GameManager가 들고 있기에 다른 Manager 들이 사용 가능<br>

## 2-4. MakeShared vs. MakeShareable

Unreal에서의 사용 방식 2개<br>

### **MakeShared - 권장하는 방법**

```cpp
TSharedPtr<Quest> quest = MakeShared<Quest>("드래곤 토벌");
```

- 참조 카운트 블록이 하나로 묶여있음<br>
  (같이 묶여 있음)<br>

### **MakeShareable - 예전 방법**

```cpp
Quest* rawQuest = new Quest("드래곤 토벌");
TSharedPtr<Quest> quest = MakeShareable(rawQuest);
```

- 참조 카운트 블록을 따로 생성<br>
  (잠재적인 파편화 유발 가능성이 존재)<br>
- 이 방식이 남아 있는 경우?<br>
  - Raw Pointer를 Shared 로 감싸고 싶음<br>
    (ex : 외부 라이브러리)<br>
  - 그런 경우를 위해 남겨둔 레거시 방식<br>

```cpp
// 커스텀 삭제 함수 예시
void CustomDeleter(MyObject* Obj)
{
    // 특별한 정리 로직
    Obj->Cleanup();
    delete Obj;
}

TSharedPtr<MyObject> Ptr(new MyObject(), [](MyObject* Obj){ CustomDeleter(Obj); });
```

- 불가치하게 먼저 Pointer를 사용하는 경우 등에 대비하여 MakeShareable<br>
- 그 외에는 MakeShared 사용 권장<br>

## 2-5. TSharedRef vs TSharedPtr

```cpp
// TSharedPtr - 비어있을 수도 있다
TSharedPtr<Quest> maybeQuest = nullptr;
if (maybeQuest.IsValid()) // null 체크 필요
{
    maybeQuest->StartQuest();
}

// TSharedRef - 절대 비어있을 수 없다
TSharedRef<Quest> definiteQuest = MakeShared<Quest>("드래곤 토벌");
definiteQuest->StartQuest(); // 바로 사용 가능, null 체크 불필요
```

- Ptr : Null 가능<br>
  - IsValid 체크 필요<br>

- Ref : Null 불가, 객체 있어야 함<br>
  - 뭔가 들어있는게 보장됨<br>
  - 생성시, 반드시 뭔가 넣어야 하며, 빈 깡통으로 쓸 수 없음<br>

```cpp
// TSharedRef → TSharedPtr: 항상 가능 (암시적 변환)
TSharedRef<Quest> QuestRef = MakeShared<Quest>("드래곤 토벌");
TSharedPtr<Quest> QuestPtr = QuestRef; // ✅ 자동 변환

// TSharedPtr → TSharedRef: 조건부로 가능 (명시적 변환 필요)
TSharedPtr<Quest> MaybeQuest = MakeShared<Quest>("보물 찾기");
if (MaybeQuest.IsValid())
{
    TSharedRef<Quest> QuestRef = MaybeQuest.ToSharedRef(); // ✅ 유효할 때만
}

// 주의: null인 TSharedPtr을 TSharedRef로 변환하면 크래시!
TSharedPtr<Quest> NullQuest = nullptr;
// TSharedRef<Quest> BadRef = NullQuest.ToSharedRef(); // ❌ 런타임 assert!
```

- 서로 변환 가능?<br>
  - Ref -> Ptr : 항상 가능<br>
  - Ptr -> Ref : 명시적 변환해야 함<br>
    (이 때 ptr이 null이라면 크래시)<br>

- Null일 수 없는 경우, Ref 를 사용하면 불필요한 체크를 피할 수 있어 편리<br>

# 3. TWeakPtr - 순환 참조 문제를 해결하는 열쇠 🔐

## 3-1. 순환 참조 문제가 뭔지 복습

```cpp
class Parent
{
public:
    TSharedPtr<Child> MyChild; // 부모 → 자식 (강한 참조)
};

class Child
{
public:
    TSharedPtr<Parent> MyParent; // 자식 → 부모 (강한 참조)
};

void CreateCircularReference()
{
    TSharedPtr<Parent> parent = MakeShared<Parent>();
    TSharedPtr<Child> child = MakeShared<Child>();

    parent->MyChild = child;   // 자식 참조 카운트 +1
    child->MyParent = parent;  // 부모 참조 카운트 +1
}
```

- Shared_Ptr 끼리 서로를 가리키면<br>
  양 측의 강한 참조가 0이 안되기에<br>
  메모리에 무한히 남아버림<br>

## 3-2. TWeakPtr이 어떻게 이 문제를 해결할까?

```cpp
class Parent
{
public:
    TSharedPtr<Child> MyChild; // 강한 참조
};

class Child
{
public:
    TWeakPtr<Parent> MyParent; // 약한 참조로 변경!
};
```

```cpp
void FixedCircularReference()
{
    TSharedPtr<Parent> parent = MakeShared<Parent>();
    TSharedPtr<Child> child = MakeShared<Child>();

    parent->MyChild = child;   // 강한 참조 → 자식 보호
    child->MyParent = parent;  // 약한 참조 → 부모 삭제 방해하지 않음
}
```

- 연관성이 비교적 적은 편 or 개념 상 하위 객체에<br>
  WeakPtr을 사용하여 순환 참조 해결<br>

## 3-3. Pin() 메서드 사용

```cpp
if (TSharedPtr<Parent> p = MyParent.Pin())
{
    // Pin 성공: 부모 객체가 아직 살아있음
    // p가 있는 동안은 RefCount가 올라가므로 안전하게 접근 가능
    p->DoSomething();
}
else
{
     // Pin 실패: 부모 객체가 이미 삭제됨
    UE_LOG(LogTemp, Warning, TEXT("부모는 이미 삭제됨!"));
}
```

- SharedPtr을 통해 가져오기에 RefCount 추가하여<br>
  당장은 사라지지 않아 안전하게 사용 가능<br>

### **Pin() 메서드 (단순화된 내부 구현)**

```cpp
// TWeakPtr::Pin() 내부 동작 (단순화)
TSharedPtr<T> TWeakPtr<T>::Pin() const
{
    if (IsValid()) // 원본 객체가 아직 살아있다면
    {
        // RefController를 통해 새로운 SharedPtr 생성
        return TSharedPtr<T>(ObjectPtr, RefController);
    }
    return TSharedPtr<T>(); // 이미 삭제된 경우 → 빈 SharedPtr 반환
}
```

### 안전한 사용 패턴

```cpp
class Child
{
private:
    TWeakPtr<Parent> MyParent; // 부모에 대한 약한 참조
    
public:
    void TalkToParent()
    {
        // 1. Pin()으로 약한 참조 → 강한 참조 승격
        TSharedPtr<Parent> ParentPtr = MyParent.Pin();

        // 2. 유효성 검사
        if (ParentPtr.IsValid()) 
        {
            // 3. 안전한 접근 (Pin으로 보호된 범위 내)
            ParentPtr->ListenToChild();
            ParentPtr->RespondToChild("잘했구나!");

            // 복잡한 작업도 문제 없음 (스코프 끝날 때까지 삭제 안 됨)
            for (int i = 0; i < 100; ++i)
            {
                ParentPtr->ProcessChildRequest(i);
            }
        }
        else
        {
            // 부모 객체가 이미 삭제됨
            UE_LOG(LogTemp, Warning, TEXT("부모가 이미 삭제되어 통신 불가"));
            HandleOrphanState(); // 고아 상태 처리
        }

        // 4. ParentPtr이 스코프를 벗어나면 자동으로 RefCount 감소
    }
    
    // ❌ 잘못된 예시 (하지 말 것)
    void DangerousDirectAccess()
    {
        // Pin 없이 바로 접근 → 컴파일 에러
        // MyParent->ListenToChild();

        // 매번 Pin() 호출하는 것도 비효율적
        if (MyParent.Pin())
        {
            MyParent.Pin()->ListenToChild(); // Pin()을 두 번 호출 → 불필요한 RefCount 증감
        }
    }
};
```

- Pin 성능?<br>
  : Shared_ptr을 직접 복사하는 것과 비슷하거나 약간 적을 수 있음<br>
  (성능 상의 이슈를 걱정할 필요는 현재로선 딱히?)<br>

### 실전 예제: 이벤트 리스너

```cpp
class EventManager
{
private:
    TArray<TWeakPtr<IEventListener>> Listeners; // 약한 참조로 리스너 관리
    
public:
    void RegisterListener(TSharedPtr<IEventListener> Listener)
    {
        // TSharedPtr를 TWeakPtr로 저장 (자동 변환)
        Listeners.Add(Listener);
    }
    
    void BroadcastEvent(const FGameEvent& Event)
    {
        // 역순 순회로 안전하게 삭제 처리
        for (int32 i = Listeners.Num() - 1; i >= 0; --i)
        {
            if (TSharedPtr<IEventListener> Listener = Listeners[i].Pin())
            {
                // ✅ 살아있는 리스너에게 이벤트 전달
                Listener->OnEvent(Event);
            }
            else
            {
                // ❌ 이미 삭제된 리스너 → 목록에서 제거
                Listeners.RemoveAt(i);
                UE_LOG(LogTemp, Log, TEXT("삭제된 리스너 제거됨"));
            }
        }
    }
};
```

- 이전에 보았던 EnemyManager 처럼<br>
  하나의 적이 피격받았을때, 다른 Enemy들도 반응하고 싶을 때 사용가능한 패턴<br>

## 3-4. 실제 게임에서 TWeakPtr 활용하기 - UI 시스템

### 문제 상황: 모두 강한 참조일 때

```cpp
class UIWidget
{
private:
    TSharedPtr<UIWidget> ParentWidget;     // 부모 (강한 참조)
    TArray<TSharedPtr<UIWidget>> Children; // 자식들 (강한 참조)

public:
    void AddChild(TSharedPtr<UIWidget> Child)
    {
        Children.Add(Child);
        Child->ParentWidget = AsShared(); // 자식이 부모를 강하게 참조
        // 🚨 순환 참조 발생! 부모와 자식이 서로를 놓지 않음 → 메모리 누수
    }
};
```

- AsShared()?<br>
  : 자기 자신을 가리키는 Shared_ptr를 만들어 반환<br>
   (TSharedFromThis 를 상속받아야 가능함)

- 서로 Shared Ptr로 묶는 것은 주의할 것<br>

### 해결책: 자식 → 부모를 TWeakPtr로 변경

```cpp
// AsShared()를 사용하려면 TSharedFromThis를 상속받아야 함
class UIWidget : public TSharedFromThis<UIWidget>
{
private:
    TWeakPtr<UIWidget> ParentWidget;       // 자식 → 부모 : 약한 참조 ✅
    TArray<TSharedPtr<UIWidget>> Children; // 부모 → 자식 : 강한 참조 유지

public:
    void NotifyParent(FString Message)
    {
        // 1. 부모가 아직 살아있는지 확인
        TSharedPtr<UIWidget> Parent = ParentWidget.Pin();

        if (Parent.IsValid())
        {
            // 2. Pin 성공 → 부모 안전하게 접근
            Parent->ReceiveMessage(Message);
            UE_LOG(LogTemp, Log, TEXT("부모에게 메시지 전달: %s"), *Message);
        }
        else
        {
            // 3. 부모가 이미 삭제됨
            UE_LOG(LogTemp, Warning, TEXT("부모가 없어 메시지 전달 실패"));
        }
    }

    void AddChild(TSharedPtr<UIWidget> Child)
    {
        Children.Add(Child);                  // 부모 → 자식 : 강한 참조
        Child->ParentWidget = AsShared();     // 자식 → 부모 : 약한 참조 (TWeakPtr에 대입하면 자동 변환)
        // ✅ 순환 참조 없음! 부모 삭제 시 자식도 자연스럽게 정리
    }
};
```

- AsShared 가 TWeakPtr에 대입하면 자동 변환되기에 안전<br>

# 4. TUniquePtr - 독점 소유권의 힘 🏛️

## 4-1. 독점 소유권이 뭔지 이해해보자

```cpp
// 생성
TUniquePtr<Weapon> myWeapon = MakeUnique<Weapon>("레이저 소드");

// 복사 불가능
// TUniquePtr<Weapon> anotherWeapon = myWeapon; // ❌ 컴파일 에러!

// 이동은 가능
TUniquePtr<Weapon> newOwner = MoveTemp(myWeapon); // ✅ 소유권 이전
// 이제 myWeapon은 nullptr
```

### 이게 왜 좋을까?

- 메모리 효율성이 뛰어남<br>
- 소유권이 명확<br>
- 자동 정리가 확실<br>

## 4-2. RAII 패턴이란?

```cpp
void SaveGameFunction()
{
    // 파일 자동 관리
    TUniquePtr<FileHandle> saveFile = MakeUnique<FileHandle>("save.dat");

    saveFile->WriteData("플레이어 레벨: 50");
    saveFile->WriteData("골드: 10000");
    saveFile->WriteData("현재 위치: 던전 입구");

    // 함수 종료 시 파일 자동으로 닫힘
}
```

- Resource Acquisition Is Initialization<br>
 : 메모리/리소스의 생명주기를 '객체'의 생명주기에 묶어 자동으로 관리<br>

- 객체 생성 시, 자원 획득<br>
  파괴 시, 자원 해제<br>
  이 개념에 가장 적합한 스마트 포인터임<br>
  - 스마트 포인터는 생성자 및 소멸자를 자동으로 호출하기에<br>

## 4-3. Move 의미론을 이해해보자

```cpp
// 무기를 만들어요
TUniquePtr<Weapon> sword = MakeUnique<Weapon>("엑스칼리버");

// 복사 시도 - 불가능
// TUniquePtr<Weapon> copiedSword = sword; // ❌ 컴파일 에러

// 이동 - 가능
TUniquePtr<Weapon> movedSword = MoveTemp(sword); // ✅ ㅇㅋ

if (sword.IsValid())
{
    // 이 블록은 실행되지 않음
    UE_LOG(LogTemp, Log, TEXT("sword는 여전히 유효"));
}
else
{
    // 이 블록이 실행됨
    UE_LOG(LogTemp, Log, TEXT("sword는 이제 nullptr"));
}

// movedSword가 엑스칼리버를 소유
movedSword->Attack(); // ✅ 정상 작동
```

- Move : RValue와 연관된 개념이며<br>
  데이터의 소유권을 넘긴다는 느낌<br>
  (넘긴 순간부터, 넘긴 포인터는 이제 사용하면 안됨)<br>
  - 복사는 성능적으로 무거운 편<br>
  - 이동은 포인터하나 바꾸면 됨!<br>

- 소유권을 넘길때, 기존의 객체가 '더 이상 필요 없어야 함'<br>
  (이러한 값 형태를 표현하기에 RValue를 사용)<br>
  - 그렇기에 C++에서 RValue 캐스팅을 해야 하는 이유<br>

### 실제 활용 패턴

```cpp
// 팩토리 패턴에서의 활용
TUniquePtr<Enemy> CreateEnemy(EnemyType Type)
{
    switch(Type)
    {
        case EnemyType::Orc:
            return MakeUnique<OrcEnemy>(); // 자동으로 Move 발생
        case EnemyType::Dragon:
            return MakeUnique<DragonEnemy>(); // 자동으로 Move 발생
        default:
            return nullptr;
    }
}

// 사용하는 쪽
void SpawnEnemies()
{
    TUniquePtr<Enemy> Orc = CreateEnemy(EnemyType::Orc);
    
    // 컨테이너에 저장할 때도 Move 사용
    TArray<TUniquePtr<Enemy>> Enemies;
    Enemies.Add(MoveTemp(Orc)); // Move로 효율적으로 저장
    
    // 이제 Orc는 nullptr
    // Enemies 배열이 오크의 유일한 소유자
}
```

- 팩토리는 Unique_Ptr로만 생성하여 넘겨줌<br>
- 사용하는 쪽에서 반환받을때 Unique_Ptr을 TArray로 관리<br>
  (Move를 통해 사용)<br>

## 4-4. 실제 게임에서 TUniquePtr 활용하기

```cpp
class TextureManager
{
private:
    // 각 텍스처는 매니저가 독점적으로 소유
    TMap<FString, TUniquePtr<Texture>> LoadedTextures;
    
public:
    void LoadTexture(FString TextureName)
    {
        // 새 텍스처를 로드
        TUniquePtr<Texture> NewTexture = MakeUnique<Texture>(TextureName);
        
        UE_LOG(LogTemp, Log, TEXT("텍스처 로딩 시작: %s"), *TextureName);
        
        // 맵에 이동 (소유권 이전)
        LoadedTextures.Add(TextureName, MoveTemp(NewTexture));
        
        UE_LOG(LogTemp, Log, TEXT("텍스처 로드 완료: %s"), *TextureName);
        // 이 시점에서 NewTexture는 nullptr
    }
    
    void UnloadTexture(FString TextureName)
    {
        // 맵에서 제거하면 TUniquePtr이 자동으로 메모리를 해제
        if (LoadedTextures.Remove(TextureName) > 0)
        {
            UE_LOG(LogTemp, Log, TEXT("텍스처 언로드 완료: %s"), *TextureName);
        }
        else
        {
            UE_LOG(LogTemp, Warning, TEXT("언로드할 텍스처가 없어요: %s"), *TextureName);
        }
    }
    
    Texture* GetTexture(FString TextureName)
    {
        // 텍스처를 찾아서 원시 포인터로 반환
        // (소유권은 넘기지 않고, 접근만 허용)
        if (TUniquePtr<Texture>* Found = LoadedTextures.Find(TextureName))
        {
            return Found->Get(); // 원시 포인터 반환
        }
        return nullptr; // 못 찾았음
    }
    
    // 매니저가 소멸될 때 모든 텍스처가 자동으로 정리
    ~TextureManager()
    {
        UE_LOG(LogTemp, Log, TEXT("텍스처 매니저 종료 - 모든 텍스처 정리"));
        // LoadedTextures 맵이 정리되면서 모든 TUniquePtr도 자동으로 정리
    }
};
```

- Map에서 제거하자마자 메모리 해제<br>

- 소멸자 시점에서 Unique_ptr이 정리되기에<br>
  메모리 누수에 안전<br>


# 5. 언제 뭘 써야 할까? - 선택 가이드 🙄

- 가이드이며 정답은 아님<br>
  (케바케)<br>

## **5-1. 스마트 포인터 선택하는 방법**

### **1단계: 이게 UObject인가?**

```cpp
// UObject 계열 - 엔진이 관리
UPROPERTY()
class AMyActor* GameActor;

// 일반 C++ 클래스 - 스마트 포인터 필요
class GameLogic {};  // UObject 상속 안 함
TSharedPtr<GameLogic> Logic = MakeShared<GameLogic>();
```

- Unreal GC에게 맡기자!<br>
- 정말 작은 경우에만 스마트 포인터 고민<br>

### **2단계: 혼자서만 사용하나? (소유권 패턴 파악)**

```cpp
// 독점 소유가 명확한 경우 - TUniquePtr 적합
class AudioManager 
{
private:
    TUniquePtr<SoundEngine> Engine; // 오디오 매니저만 엔진을 소유
};

// 여러 곳에서 접근 - 다음 단계 판단 필요
class GameData 
{
    // UI, 게임로직, 세이브시스템 등 여러 곳에서 필요
};
```

- 혼자서 사용한다면 Unique_Ptr이 제일 효율적<br>

### **3단계: 여러 곳에서 공유해야 하나? (접근 패턴 분석)**

```cpp
// ❌ 비권장 - 오버헤드 발생
void ProcessQuest(TSharedPtr<Quest> InQuest)
{
    InQuest->Update();
}

// ✅ 권장 - 더 효율적
void ProcessQuest(const Quest& InQuest)
{
    InQuest.Update();
}

// 또는 포인터로
void ProcessQuest(Quest* InQuest)
{
    if (InQuest)
    {
        InQuest->Update();
    }
}
```

- Shared_ptr을 매개변수로 주었을때도, RefCount가 늘어나는 점은 유의할 것<br>
  (참조 or 포인터 권장)<br>

### **4단계: null이 될 수 있나? (생명주기 안정성)**

```cpp
// null 가능 - TSharedPtr
TSharedPtr<NetworkConnection> Connection; 

// 항상 유효 - TSharedRef
TSharedRef<InputManager> Input = MakeShared<InputManager>(); 
```

- Reference 는 'Null' 대입이 안된다...<br>

### **5단계: 순환 참조 위험이 있나? (관계 구조 분석)**

```cpp
// 순환 참조 위험 - TWeakPtr 필요
class UIPanel 
{
    TArray<TSharedPtr<UIWidget>> Children; // 강한 참조
};

class UIWidget 
{
    TWeakPtr<UIPanel> Parent; // 약한 참조로 순환 방지
};
```

- Weak 고려!<br>

## **5-2. 실제 선택하기 어려운 상황들과 해결법**

### 상황 1: “지금은 임시인데, 나중에 요구가 바뀔 수 있는” 경우

해결 접근법: 처음에는 안전하게, 소유권이 명확해지면 최적화<br>

- 스마트 포인터에서 가장 중요한 것은<br>
  '`소유권 설계`'임!<br>

- 처음부터 설계하면 구조가 너무 딱딱해질 수 있음<br>

```cpp
// 초기: 요구가 불명확 → 안전한 공유 모델
class InventoryManager
{
private:
    TSharedPtr<PlayerInventory> Inventory;
public:
    TSharedPtr<PlayerInventory> GetInventory() { return Inventory; }
};

// 최적화: 독점 소유가 명확해졌을 때만 전환
class InventoryManager
{
private:
    TUniquePtr<PlayerInventory> Inventory;  // 독점 소유
public:
    PlayerInventory* GetInventory() { return Inventory.Get(); } // 접근만 허용
};
```

### 상황 2: “성능이 중요한지, 안전성이 중요한지” 애매한 경우

- 성능 기준만으로 스마트 포인터를 선택하는 것은 잘못된 접근<br>
    - Shared_Ptr이 무겁다고 Unique_ptr로 바꾸는 것은 잘못된 선택일 수 있다!<br>

```cpp
// 독점 소유 → TUniquePtr
class Renderer
{
private:
    TUniquePtr<RenderQueue> Queue;  
public:
    void Render() { Queue->Process(); }
};

// 여러 시스템에서 참조 → TSharedPtr 또는 TWeakPtr
class EventSystem
{
private:
    TArray<TWeakPtr<EventListener>> Listeners; 
};
```

### 상황 3: “팀원들의 숙련도가 제각각인” 경우

팀에 C++ 초보자가 있다면, 기술적으로 최적이 아니더라도 **이해하기 쉬운 선택**이 더 나을 수 있음.<br>

- 혼자 작업하는게 아님<br>
- 최적화는 일단 **'안정성'이 보장**되어야 하는 것임을 잊지 말자<br>

```cpp
// 고급 팀(숙련자 위주): 효율적 구조 선택 가능
class ResourceManager
{
private:
    TMap<FString, TUniquePtr<Resource>> Resources;
public:
    Resource* Get(const FString& Name); // 소유권 전달 없음
};

// 다양한 숙련도 팀: 안전한 공유 모델이 유지보수에 유리
class ResourceManager
{
private:
    TMap<FString, TSharedPtr<Resource>> Resources;
public:
    TSharedPtr<Resource> Get(const FString& Name); 
};
```

### **상황 4: "레거시 코드와 섞어야 하는" 경우**

기존 프로젝트에 스마트 포인터를 도입할 때는 점진적으로 적용<br>

- 예전 코드를 뜯어고치는 것은 다시한번 생각해보기<br>
    - 간단한 부분부터 천천히 고쳐보기<br>

```cpp
// 1단계: 새 코드부터 스마트 포인터 적용
class NewFeatureManager
{
private:
    TUniquePtr<NewFeature> Feature;
public:
    NewFeature* GetFeature() { return Feature.Get(); }
};

// 2단계: 레거시 코드에는 raw 포인터로 전달
class LegacySystem
{
public:
    void UseFeature(NewFeature* F) { /* 기존 방식 유지 */ }
};

// 3단계: 레거시 영역을 점진적으로 스마트 포인터화
```

### 정리

- 기본적으로 UObject라면 GC 관리에<br>
- UObject가 아니라면 Smart 포인터를 통해<br>
  메모리 누수 상황을 방지하기<br>

- Smart 포인터의 설정은 '소유권'에 있다는 점을 인식<br>
  - 혼자만 사용하는게 '정말 명확'하다면 Unique_Ptr<br>
  - 기본적으로 Shared_Ptr이 안전<br>
    - 다만 상호 Shared_ptr 연결이라면 한쪽을 Weak로<br>
  - Weak_ptr은 '확인'만 하기 위함<br>
    (명확한 소유 주체가 따로 있는 경우, Weak_ptr로 확인)<br>
    (Shared_Ptr만 체크할 수 있음!)<br>

### TMI - TObjectPtr

UObject을 추적하는 포인터<br>
- 스마트 포인터는 아님<br>
- GC 최적화용으로 감싸는 레핑용 포인터<br>
- 기본 포인터보단 사용이 권장되긴 함(Unreal 엔진)<br>
  - UObject를 멤버 변수로 '보관'하는 경우<br>
  - 매개변수,지역변수 등에선 RawPointer가 더 가벼우며, 불필요한 최적화 대상을 늘리지 않음<br>

