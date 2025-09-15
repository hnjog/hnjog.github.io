---
title: "Unreal TIL (0915)"
date : "2025-09-15 20:00:00 +0900"
last_modified_at: "2025-09-15T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Actor Life Cycle
---

## 생성자에서 ConstructorHelpers::FObjectFinder 를 사용하였는데 에디터에서 변함이 없다?

```cpp
#include "Item.h"

AItem::AItem()
{
		// Scene Component를 생성하고 루트로 설정
		SceneRoot = CreateDefaultSubobject<USceneComponent>(TEXT("SceneRoot"));
		SetRootComponent(SceneRoot);

		// Static Mesh Component를 생성하고 Scene Component에 Attach
		StaticMeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("StaticMesh"));
		StaticMeshComp->SetupAttachment(SceneRoot);
		
		// Static Mesh를 코드에서 설정
		static ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT("/Resources/Props/SM_Chair"));
		if (MeshAsset.Succeeded())
		{
			StaticMeshComp->SetStaticMesh(MeshAsset.Object);
		}

		// Material을 코드에서 설정
		static ConstructorHelpers::FObjectFinder<UMaterial> MaterialAsset(TEXT("/Resources/Materials/M_Metal_Gold"));
		if (MaterialAsset.Succeeded())
		{
			StaticMeshComp->SetMaterial(0, MaterialAsset.Object);
		}
}
```

강의 예시코드이나, 나는 내가 생각한대로 임의로 TEXT 내부의 부분을 수정하여 작성하였다<br>

그런데 지속적으로 Fail 로그가 발생하였기에 무엇인가 잘못되었다고 느꼈고<br>

이전에 수정한 부분이 문제라는 것을 깨달았다<br>

올바른 코드<br>

```cpp
static ConstructorHelpers::FObjectFinder<UStaticMesh> MeshAsset(TEXT("/Game/Resources/Props/SM_Chair.SM_Chair"));
static ConstructorHelpers::FObjectFinder<UMaterial> MaterialAsset(TEXT("/Game/Resources/Materials/M_Metal_Gold.M_Metal_Gold"));
```

#### Unreal의 경로 지정 규칙

- .**uasset** 확장자는 필요 없음<br>
  Unreal은 콘텐츠 브라우저에서의 '패키지 경로'를 사용<br>
  '물리 파일 확장자'는 해당 경로에 필요 없음<br>

- **/Game** 루트를 사용할것<br>
  프로젝트의 Content 폴더가 런타임에선 `/Game` 으로 매핑됨<br>

- 패키지 이름 + 오브젝트 이름 을 사용할 것<br>
  패키지 이름 : `/Game/Resources/Props/SM_Chair` <br>
  오브젝트 이름 : `.SM_Chair` (일반적으론 파일명과 동일함)<br>

- 기본적으로 에셋의 '우클릭 - Copy Reference'에서<br>
  '/Game/...' 부분부터 활용이 가능<br>

이렇게 잘 '지정' 해주니 실패 로그는 사라졌으나<br>
여전히 에디터에서 바뀌지 않는다...?<br>

#### 인스턴스들의 '생성자' 변경은 바로 반영되지 않는 경우가 많음

- `ConstructorHelpers` 로 경로를 바꾼다던가, '생성자'에서 설정했다던가 등등<br>

- 생성자에서 설정한 값은 **CDO(클래스 기본값)**에 들어가고<br>
  '이미 배치된' 인스턴스는 자기 복사본이 존재하기에<br>
  새로운 CDO가 자동으로 덮어 쓰진 않음<br>
  (OnConstruction 함수를 사용하면 개선될 수 있긴 함)<br>
  (-> FTransform 변경시 호출)<br>

- 다만 이것이 직접적인 원인은 아니다<br>
  우리는 'static'을 통한 '지역 변수'를 만들었고<br>
  해당 정적 지역 변수는 프로세스에서 한번만 호출되기에<br>
  에디터를 다시 껐다 켜야 정상 작동함<br>
  (아니면 그냥 static을 지우는 방식도 존재함)<br>

### AssetManager와 ConstructorHelpers

- 작은 프로토타입이라면 ConstructorHelpers 사용이 간단<br>

- 중대형 프로젝트로 갈 수록 UAssetManager 고려<br>
  (+ Soft Refs/Streamablemanager)<br>

#### 비교 표

| ✨ **항목**           | 🎨 **`ConstructorHelpers`**<br>(FObjectFinder / FClassFinder) | 🚀 **UAssetManager**<br>(+ `TSoftObjectPtr`, `StreamableManager`) |
| ------------------ | ------------------------------------------------------------- | ----------------------------------------------------------------- |
| 🧭 **철학**          | “생성자에서 **즉시·동기 로드** 필요 → 경로로 바로 찾기”                           | “참조만 들고 있다가 **필요 시 비동기/선행 로드 + 자산군 관리**”                          |
| 📍 **사용 위치**       | **생성자 한정** (Actor/Component 내부)                               | 어디서든 가능 (게임 전 구간), 주로 **PrimaryAsset(데이터에셋)** 중심                  |
| ⚡ **로드 방식**        | 항상 **동기 로드** → 시작 시 hitch 발생 가능                               | **비동기/우선순위/프리패치/캐시/취소** 등 유연한 처리                                  |
| 📦 **의존성·묶음**      | 개별 경로 **하드코딩**, 묶음/번들 개념 없음                                   | **PrimaryAssetType/Bundle**로 논리적 그룹 관리 (`Gameplay`, `Cosmetic` 등) |
| 📈 **확장성(규모)**     | 소규모 프로젝트·툴·샘플에 적합                                             | **중·대규모 제작 표준** → 수백\~수천 리소스에도 안정적                                |
| 🔗 **경로 변경 내성**    | 하드 경로 변경에 취약 → 리다이렉터 실패 시 크래시 위험                              | **Soft Reference** 기반 → 리팩터/리타겟 친화적                               |
| 💾 **메모리·스타트업**    | 시작 시 메모리 사용량 증가, 맵/PIE 부팅 지연 가능                               | **필요 시점 로드**로 메모리 최적화, 맵 전환/로딩 화면 연계 용이                           |
| 🔧 **요구 설정·초기 투자** | 거의 없음 (바로 사용 가능)                                              | **AssetManager 설정**(INI 규칙, PrimaryAsset 등록) 및 데이터자산 설계 필요        |
| 🎨 **디자이너 워크플로**   | 코드에 경로 고정 → 디자이너 자유도 낮음                                       | **DataAsset/SoftRef** 노출 → **BP/데이터 주도** 제작 용이                    |
| 📦 **DLC·패치·쿡 규칙** | 세밀 제어 어려움                                                     | **PrimaryAssetLabel/쿡 규칙**으로 빌드 크기·의존성·품질 제어                      |
| 🏆 **대표 쓰임새**      | 한두 개 **절대 필요한** 기본 메쉬/머티리얼/위젯                                 | 맵/무기/적/스킨/사운드팩 등 **게임 콘텐츠 전반**                                    |


## Actor Life Cycle 함수들

1. **생성자 (Constructor)**<br>
    - 호출 시점: C++ 클래스 객체가 **메모리에 생성**될 때 딱 한 번 호출<br>
    - 아직 액터가 월드 (World)에 완전히 등록되지 않은 상태이므로, **다른 액터나 월드 관련 기능**을 안전하게 호출하기 어려움<br>
    - 주로 `CreateDefaultSubobject` 등을 사용해 **컴포넌트 생성** 및 **초기 변수 세팅**에 활용<br>
2. **PostInitializeComponents()**<br>
    - 호출 시점: 액터의 모든 **컴포넌트**가 생성·초기화된 뒤 '한 번' 호출<br>
    - 각 컴포넌트가 이미 준비된 상태, **컴포넌트 간 상호작용**(예: 서로 다른 컴포넌트 참조 설정)이 필요한 코드를 배치 가능<br>
    - 생성자에선 ‘생성/할당’만, PostInitializeComponents()에서 컴포넌트들 사이의 의존 관계를 설정<br>
3. **BeginPlay()**<br>
    - 호출 시점: **Play In Editor** (PIE)나 **런타임**에서 게임이 시작될 때, 혹은 이미 실행 중인 게임에 `SpawnActor` 등으로 새 액터가 생성될 때 **한 번** 호출<br>
    - 이 시점에는 이미 **월드와 다른 액터**들이 준비된 상태이므로, 자유롭게 상호작용이 가능<br>
    - AI, 게임 모드, 플레이어 컨트롤러 등 **다른 시스템과 연동**도 주로 BeginPlay에서 초기화<br>
    - 타이머, Delegate(Event) 바인딩 등을 시작하기에도 적합<br>
4. **Tick(float DeltaTime)**<br>
    - 호출 시점: 매 프레임마다 호출 (다만, 액터의 `PrimaryActorTick.bCanEverTick = true;` 설정 필요)<br>
    - **실시간 업데이트**가 필요한 로직 (캐릭터 이동, 물리 연산, 카메라 추적 등)을 처리<br>
    - 이벤트 (Event) 기반으로 전환할 수 있는 부분은 `Tick`을 사용하지 않는 편이 **성능**에 유리<br>
5. **Destroyed()**
    - 호출 시점: `Destroy()` 함수를 **직접 호출**하여 액터를 제거할 때 **직전에** 호출, 다만 **레벨 전환**이나 **게임 종료** 에 따라 스킵될 수 있음에 주의<br>
    - `Destroyed()`가 불린 뒤에는 최종적으로 `EndPlay()`가 호출<br>
    - **수동으로 액터를 제거**할 때, 마지막 정리 코드를 넣을 수 있는 곳<br>
        - 다만, 게임 종료나 레벨 언로드 시에는 호출되지 않을 수 있으므로, **모든 중요한 정리**를 `Destroyed()`에만 의존하진 말 것<br>
    - 정리할 자원?
        - **수동 할당한 메모리**: `new` 또는 동적 할당한 오브젝트가 있다면 이 타이밍에 `delete`하거나 해제<br>
        - **스폰된 자식 액터**: 이 액터가 생성한 다른 액터나 컴포넌트 중, **자동으로 해제되지 않는** 것이 있다면 제거<br>
        - **Delegate / Event 바인딩**: 게임 전역적 또는 외부 클래스에 바인딩해둔 델리게이트가 있다면 해제<br>
        - **사운드/파티클 등**: 필요 시 이 액터가 재생 중인 사운드나 파티클을 수동으로 정리<br>
6. **EndPlay(const EEndPlayReason::Type EndPlayReason)**
    - 호출 시점: 액터가 더 이상 월드에서 활동하지 않게 될 때 호출 (파괴, 레벨 전환, 게임 종료 등)<br>
    - `EEndPlayReason::Type`으로 어떤 이유로 EndPlay가 호출되었는지(파괴, 레벨 언로드, 게임 종료 등)를 구분 가능<br>
    - **게임 종료나 레벨 언로드** 같은 상황에서도 `EndPlay()`는 상대적으로 호출 보장이 높음<br>
        - **중요한 정리 로직 (자원 해제, Timer 해제, 상태 저장 등)은 `EndPlay()`에 넣는 것**이 보다 안전<br>
    - 정리할 자원?<br>
        - **타이머**: `GetWorldTimerManager().ClearTimer(…)` 와 같이 타이머를 정리<br>
        - **동적 할당 리소스**: 여전히 해제되지 않은 동적 메모리가 남아 있다면 정리<br>
        - **데이터 저장**: 게임 진행 상황 (점수, 인벤토리 등)을 파일/DB에 저장하거나, 상위 시스템에 콜백을 보내는 로직도 EndPlay에서 처리 가능<br>


### 정리 표

| 🌟 **함수**                         | ⏰ **호출 시점**                                   | 🔎 **특징 / 주의사항**                                                                             | 🛠 **주요 작업 예시**                                                              |
| --------------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| ✨ **Constructor**                 | 🟢 객체가 **메모리에 생성될 때** (단 한 번)                 | ▫ 월드/다른 액터와 **안전한 상호작용 불가**<br>▫ `CreateDefaultSubobject`로 **컴포넌트 생성 & 변수 초기화**              | ▫ 컴포넌트 생성 (`CreateDefaultSubobject`)<br>▫ 멤버 변수 기본값 설정                       |
| 🔧 **PostInitializeComponents()** | 🟡 모든 **컴포넌트 생성·초기화 완료 후** (한 번)              | ▫ **컴포넌트 간 의존 관계** 설정 가능<br>▫ Constructor에서 처리하기 어려운 초기화 보완                                  | ▫ 서로 다른 컴포넌트 참조 연결<br>▫ 복잡한 초기화 로직 배치                                        |
| ▶ **BeginPlay()**                 | 🔵 **게임 시작 시** 또는 **SpawnActor**로 생성될 때 (한 번) | ▫ 월드·다른 액터 **완전 준비 상태**<br>▫ AI, GameMode 등 외부 시스템과 자유로운 상호작용                                | ▫ 타이머/델리게이트 바인딩<br>▫ AI 초기화, 시스템 연동<br>▫ UI·사운드 등 실행                         |
| ⏳ **Tick(float DeltaTime)**       | 🟣 매 프레임마다                                    | ▫ `PrimaryActorTick.bCanEverTick = true` 필요<br>▫ 실시간 업데이트 처리<br>▫ **성능**상 이벤트 기반 대체 고려       | ▫ 이동·물리 연산<br>▫ 카메라 추적, 애니메이션 보간                                             |
| 💥 **Destroyed()**                | 🟥 `Destroy()` 호출 시 **액터 제거 직전**              | ▫ 레벨 전환·게임 종료 시 **생략될 수 있음**<br>▫ EndPlay도 뒤이어 호출됨                                           | ▫ 수동 할당 메모리 해제<br>▫ 자식 액터/컴포넌트 제거<br>▫ 델리게이트/이벤트 해제<br>▫ 사운드·파티클 정리          |
| 🛑 **EndPlay(EndPlayReason)**     | ⚫ 액터가 **월드 활동 종료** 시 (파괴·레벨 언로드·게임 종료 등)      | ▫ **EndPlayReason**으로 종료 원인 확인 가능<br>▫ Destroyed보다 **호출 보장도 높음**<br>▫ **중요 정리 로직**은 여기 배치 권장 | ▫ 타이머 해제 (`ClearTimer`)<br>▫ 동적 리소스/메모리 해제<br>▫ 게임 데이터 저장<br>▫ 상위 시스템에 상태 콜백 |

