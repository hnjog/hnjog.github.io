---
title: "UObject Ptr"
last_modified_at: "2025-08-01T16:00:00"
categories:
  - Unreal
tags:
  - TObjectPtr
  - Unreal GC
---

## UObject 기반 클래스에 대한 포인터
언리얼 강의를 듣던 중<br>
TOjbectPtr라는 독특한 '포인터'를 발견하였다<br>
(T* 방식의 일반 '생' 포인터와는 다른)<br>

Unreal은 왜 이러한 포인터를 사용하고 권장할까?<br>

### TObjectPtr
언리얼의 'UObject' 기반 클래스를 감싸는<br>
'스마트 포인터' 방식의 래퍼 클래스<br>

```
TObjectPtr<AActor> MyActor;  // AActor* MyActor; 대신 사용 권장
```

해당 클래스 내부에서 ->나 * 같은 연산자를 지원하기에<br>
C++의 포인터 처럼 사용이 가능<br>

그렇다면 기존 포인터 대신 사용할만한 메리트는?<br>

| 항목                              | 이유 / 설명                                                   |
| ------------------------------- | --------------------------------------------------------- |
| 🔧 **핸들 기반 구조 (FObjectHandle)** | 단순 주소값이 아닌 인덱스/시리얼 기반으로 객체를 관리 → PIE나 리로딩, 리플렉션에서도 안정     |
| 🧠 **무효화 감지 가능**                | 참조 대상 객체가 파괴될 경우 자동으로 `nullptr` 처리되어, dangling pointer 방지 |
| 📦 **시리얼라이즈 / 복제 지원 강화**        | raw pointer보다 명시적이고 안전하게 저장/복원 가능                         |
| 🧵 **멀티스레딩에서 안정성**              | 객체 인덱스를 통한 핸들 기반 처리 → GC와의 충돌 방지                          |
| 🔎 **리플렉션 & GC 통합 최적화**         | `UPROPERTY()`와 함께 사용 시, Unreal의 모든 시스템에서 완전하게 통합 처리 가능    |

- TObjectPtr은 내부적으로 FObjectHandle이나 FObjectPtr을 기반으로 한다<br>
  (아래쪽 GC 부분에서 추가 설명)<br>

- (== nullptr) 체크 시 내부적으로 유효 검사 실행하여<br>
  Dangling Pointer 상황 방지 가능<br>
  (일반 포인터는 GC가 삭제하더라도 <br>
  그 값이 '유효'한지 체크할 수 없기에 추천 x)<br>

- TObjectPtr는 FArchive 기반 직렬화 시에 객체 경로 정보 + 핸들 정보가 함께 저장<br>
  (저장/불러오기, 레벨 스트리밍, Editor의 되돌리기 등에서 객체 참조 복원을 정확히)<br>

- Unreal의 리플렉션 시스템 (UProperty, FArchive, FReferenceCollector)은<br>
  TObjectPtr 타입을 기반으로 설계되었기에<br>
  유용한 기능을 제공 가능<br>
  (ex : Editor의 Detail 패널에서 정보 확인 가능)<br>

---

### 일반 포인터(T*)도 여전히 사용한다

| 상황                        | 이유                                              |
| ------------------------- | ----------------------------------------------- |
| 🔹 **함수 파라미터 / 로컬 변수**    | GC에 등록할 필요 없음. 임시 참조로 충분                        |
| 🔹 **성능 최적화가 필요한 저수준 코드** | GC 트래커/리플렉션에서 제외됨 → 오버헤드 없음                     |
| 🔹 **비-UObject를 참조할 때**   | GC 대상이 아님 (ex. `FVector*`, `int*`, `MyStruct*`) |
| 🔹 **자체적으로 생명주기를 관리할 때**  | ex. 게임플레이 코드에서 일시적으로만 쓰는 레퍼런스                   |

상황에 따라 적절하게 사용이 가능<br>


### TMI : Unreal GC

#### 언리얼의 GC가 객체를 추적하는 기준은 UPROPERTY()

객체는 T* 이든 TObjectPtr이든 UPROPERTY를 통해 추적됨<br>

```
UPROPERTY()
AActor* MyActor; // RefCount에 추가
```

#### 언리얼 GC는 ObjectHandle 기반

byte를 읽는 방식이 아닌<br>
객체의 인덱스와 시리얼 번호를 통해 참조<br>

-> PIE나 레벨 재로딩 등에서 '참조'를 유지<br>
   (객체 파괴 시, 시리얼 번호가 달라져 '무효화'를 감지)<br>

---

## 언리얼의 주요한 포인터 래퍼들

### TWeakObjectPtr (약한 참조)
참조 대상이 GC로 사라져도 '유효'하지 않음<br>
'소유'하지 않기에 RefCount에 직접 영향주지 않음<br>
IsValid(), Get() 등으로 안전하게 접근해야 한다<br>

```
TWeakObjectPtr<AActor> CachedActor;

if (CachedActor.IsValid())
    CachedActor->DoSomething();
```

(캐싱, 옵저버 패턴, Timer 콜백 등으로 사용)<br>

### TSoftObjectPtr (지연 로딩 가능한 소프트 참조)
객체를 경로(Path)로 저장함 (ex : "/Game/Some/Asset")<br>
메모리에 없으면 동기 or 비동기 로드<br>
GC에 영향 X, 하지만 UPROPERTY() 사용 시 에디터에 저장 가능<br>
리소스를 메모리에 나중에 올려 메모리를 절약 <br>


```
TSoftObjectPtr<UTexture2D> SoftTex;

// 동기 로드 (게임 일시 정지 가능성 있음)
UTexture2D* Loaded = SoftTex.LoadSynchronous();  

// 또는 비동기 로드:
StreamableManager.RequestAsyncLoad(SoftTex.ToSoftObjectPath(), Callback);
```

(에디터 쪽에서 리소스를 참조할 때 사용)<br>

### TLazyObjectPtr (느슨한 참조)
실제 참조하는 객체를 메모리를 유지하지 않음<br>
객체의 ObjectID만 기억하고 있다가<br>
필요할 때 Get() 함수를 통해 찾아옴<br>
(Soft처럼 Load하지 않기에 주의)<br>

```
TLazyObjectPtr<AActor> LazyActor;

AActor* RealActor = LazyActor.Get();  // 없으면 nullptr 반환
```

(SaveGame 이나 레벨 간 참조 등에 사용 가능)<br>

### TScriptInterface (Interface 참조 포인터)
UObject 기반 인터페이스를 저장<br>
내부적으로는 (UObject*, Interface*) 를 쌍으로 들고 있음<br>
Execute_XXX() 호출 대신, 그냥 ptr->MyFunction() 가능<br>

```
TScriptInterface<IInteractable> InteractTarget;

InteractTarget->Interact(); // 호출 가능
```

(BP, C++ 인터페이스의 호환이 가능)<br>

### FObjectPtr / FObjectHandle (저레벨 핸들 시스템 (UE5))
UObject의 포인터 대신 인덱스 + 포인터 캐시 구조<br>
네트워크, 복제, 패키지 로딩 등 내부 시스템 최적화를 위함<br>
일반적으로 잘 다루지 않음 (TObjectPtr 내부 구현에 쓰임)<br>


### 정리

| 타입                             | 용도 요약                         | GC 추적 | 지연 로딩 | 강한 참조 | 특징 요약                                   |
| ------------------------------ | ----------------------------- | ----- | ----- | ----- | --------------------------------------- |
| `TObjectPtr<T>`                | ✅ 기본 권장 스마트 포인터               | ✅     | ❌     | ✅     | `T*`처럼 쓰되 GC/Editor/리플렉션에 안전, UE5 표준화   |
| `TWeakObjectPtr<T>`            | ❗ 약한 참조 (GC 감시, 생명주기 보장 X)    | ✅     | ❌     | ❌     | 객체가 살아있을 때만 유효, 캐시/UI/임시 참조에 적합         |
| `TSoftObjectPtr<T>`            | 🔄 지연 로딩 가능한 에셋 참조            | ❌     | ✅     | ❌     | 경로 기반 참조. 메모리 절약 & 비동기 로딩 가능            |
| `TLazyObjectPtr<T>`            | 💤 느슨한 참조 (ObjectID 기반)       | ❌     | ❌     | ❌     | 실제 객체 대신 ID만 저장. SaveGame 등에서 존재 여부 추적용 |
| `TScriptInterface<I>`          | 🧩 Interface + UObject 포인터    | ✅     | ❌     | ✅     | 인터페이스 함수 호출용 포인터. UObject 기반 인터페이스만 지원  |
| `FObjectPtr` / `FObjectHandle` | 🔧 내부용 핸들 기반 참조 구조 (UE5 Core) | ✅     | ❌     | ✅     | GC 및 리플렉션 최적화를 위한 low-level 포인터 구조      |
