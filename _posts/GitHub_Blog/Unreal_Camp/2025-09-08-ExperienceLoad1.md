---
title: "Experience Load 1"
date : "2025-09-08 16:00:00 +0900"
last_modified_at: "2025-09-08T16:00:00"
categories:
  - Unreal
  - Lyra
  - 언리얼 5
tags:
  - Lyra
  - Experience
  - GameMode
---

## Experience에 대해서는...

[Experience 관련 블로깅](https://hnjog.github.io/unreal/lyra/Experience/){:target="_blank"}<br>

[![Image](https://github.com/user-attachments/assets/68bb1508-ab02-46f6-878b-1ac3959443b6)](https://github.com/user-attachments/assets/68bb1508-ab02-46f6-878b-1ac3959443b6){: .image-popup}<br>

이전에 배운 내용을 요약하는 블로깅 주소와<br>
이미지 파일<br>

이번에는 이러한 Experience를 불러오는(Loading)<br>
방식에 대하여 알아보려 한다<br>

핵심적인 요소는<br>
- GameMode(AGameModeBase)<br>
- ExperienceManagerComponent(UGameStateComponent)<br>

## GameMode와 Experience 사전 개념 요약

[![Image](https://github.com/user-attachments/assets/d8d73dcb-6a13-4328-b25a-69d864b711d9)](https://github.com/user-attachments/assets/d8d73dcb-6a13-4328-b25a-69d864b711d9){: .image-popup}<br>

GameMode와 Experience의 관계도<br>

```
[GameMode] 
 ├─ GameStateClass → [GameState]
 │   └─ ExperienceManagerComponent → [ExperienceDefinition]
 │
 ├─ PlayerStateClass → [PlayerState]
 │   └─ PawnData (PawnClass, InputConfig, CameraMode)
 │
 ├─ PlayerControllerClass → [PlayerController]
 │   └─ PlayerCameraManager
 │
 └─ DefaultPawnClass → [Character]
     ├─ PawnExtensionComponent
     └─ CameraComponent
```

- GameMode <br>
  : 서버 전용의 권위(Authority) 보유 클래스<br>
   사용할 GameState,PlayerState, PlayerController, Pawn 을 결정<br>
   Experience 로딩 트리거를 '시작'<br>
   (InitGameState를 통해 ExperienceManager에게<br>
   로딩 트리거 전달)<br>

- GameState<br>
  : 클라/서버 공통 존재(동기화 요구)<br>
  ExperienceManager를 소유<br>
  (일반적으로 GameMode와 1:1 대응)<br>
  (Lyra는 하나의 게임 모드 안에 여러 게임이 존재)<br>
  (이 경우, ExperienceManager가 Experience 전환을 대응)<br>

- ExperienceManager<br>
  : 현재 적용 게임에 적용할 ExperienceDefinition을 보관<br>
    Experience 로딩을 담당<br>
	(PrimaryAsset 로드, GameFeature 활성화, ActionSet 적용 등)<br>
	완료시 OnExperienceLoaded 델리게이트를 통해 broadcast<br>

- ExperienceDefinition<br>
  : 데이터 에셋<br>
  사용할 Pawn, CameraMode, AbilitySet, GameFeature 등을 정의<br>

- PlayerState<br>
  : 플레이어 상태 정보 저장(네트워크 동기화 필요)<br>
    PawnData를 참조하고 캐싱해놓는다<br>
	(차후 gas 관련 용도)<br>

- PawnData<br>
  : 플레이어 Pawn에 대한 데이터 설정<br>
    (PawnClass, Input, CameraMode 등을 포함)<br>

- PlayerController<br>
  : 입력 처리, 카메라 제어, UI 상호작용 담당<br>
	PlayerCameraManager를 소유해 카메라 로직 제어<br>

- PlayerCameraManager<br>
  : 카메라 모드를 관리<br>
  (시야/시점 조정)<br>

- Character(DefaultPawn)<br>
  : 실제 플레이어 캐릭터의 Pawn<br>
  PawnExtensionComponent를 통해 확장<br>

- PawnExtensionComponent<br>
  : Pawn의 기능확장을 위한 컴포넌트<br>
    (다만, Pawn과 컴포넌트가 종속적인 관계가 되지 않도록<br>
	관리하는 중간 다리 역할을 한다)<br>

- CameraComponent<br>
  : CameraManager가 다루게 될 실제 카메라 속성/기능<br>


(여담으로 Modular Gameplay와<br>
Game Features, Gameplay Abilities 플러그인을 미리 켜두는 것을 추천)<br>

이후<br>
Bulid.cs에 추가해준다<br>

```cs
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class Samples : ModuleRules
{
	public Samples(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
	
		PublicDependencyModuleNames.AddRange(new string[] {
            "Core",
            "CoreUObject",
            "Engine",
            // GAS
            "GameplayTags",
            // Game Features
            "ModularGameplay",
			//Input
            "InputCore",
        });

		PrivateDependencyModuleNames.AddRange(new string[] {  });
	}
}
```

## GameMode 세팅

프로젝트 세팅에서<br>
만든 GameMode를 설정하고<br>
해당 생성자에서 DefaultClass 들을 설정하면<br>
기본적으로 해당 클래스들이 선택된다<br>

```cpp
ASampleGameMode::ASampleGameMode()
{
	// GameState를 설정해주었기에 World가 생성되는 시점에 생성자로 ExperienceManager를 생성
	GameStateClass = ASampleGameState::StaticClass();
	PlayerControllerClass = ASamplePlayerController::StaticClass();
	PlayerStateClass = ASamplePlayerState::StaticClass();
	DefaultPawnClass = ASampleCharacter::StaticClass();
	HUDClass = ASampleHUD::StaticClass();
}
```

InitGame에서<br>
한 프레임 뒤에 HandleMatchAssignmentIfNotExpectingOne()를<br>
호출하도록 설정<br>
(InitGameState가 호출된 후, 호출됨)<br>

- 사실 InitGame()이 호출되는 시점은<br>
  World가 생성되는 시점이기에<br>
  GameState가 생성되지 않았음...<br>

[![Image](https://github.com/user-attachments/assets/72c565d1-c45e-4d35-ac63-57e53a1cd14b)](https://github.com/user-attachments/assets/72c565d1-c45e-4d35-ac63-57e53a1cd14b){: .image-popup}<br>

- 이런식으로 프레임이나, 일부 엔진 로딩 방식을 따르지 않고<br>
  별도로 로딩 시스템을 만드는 이유?<br>
  - 엔진 업데이트를 통한 내부 수정에 영향을 받지 않기 위함<br>

```cpp
void ASampleGameMode::InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage)
{
	Super::InitGame(MapName, Options, ErrorMessage);

	// 호출 시점엔 아직 GameInstance를 통해 초기화 작업이 진행중이므로
	// 해당 프레임에선 Lyra의 Comcept인 Experience 처리를 진행할 수 없음
	// - 이를 위해 한 프레임 뒤에 이벤트를 받아 처리를 이어서 진행
	// InitGameState가 호출된 후, HandleMatchAssignmentIfNotExpectingOne 가 호출된다
	// InitGame(이 떄 한 프레임 뒤에 HandleMatchAssignmentIfNotExpectingOne 호출) 
	// -> GameState 생성자 호출(ExperienceManager 생성) -> InitGameState 호출로 인하여
	// OnExperienceLoaded 등록(이 때, Pawn들을 Restart 시킴)
	// 
	// HandleMatchAssignmentIfNotExpectingOne 를 통해 한 프레임 뒤,
	// Manager에게 Experience를 Load하도록 시키며,
	// 완료 될시 OnExperienceLoaded 가 호출
	//
	GetWorld()->GetTimerManager().SetTimerForNextTick(this, &ThisClass::HandleMatchAssignmentIfNotExpectingOne);
}

```

HandleMatchAssignmentIfNotExpectingOne?<br>
- 실제로는 dedicate 서버 기반 프로젝트이기에 이런 함수명<br>

## ExperienceManagerComponent 생성
GameStateComponent를 상속받기에<br>
"ModularGameplay" 플러그인을 설정하고 build.cs에 추가 필요<br>

역할<br>
- GameState를 owner로 가지며, Experience의 상태 정보를 가지는 Component<br>
- 추가로 Manager의 역할로서 Experience 로딩 상태 업데이트 및 이벤트를 관리<br>

UGameStateComponent?<br>
- 결국 UActorComponent를 상속받는 클래스<br>
	-> 단순히 Actor에만 붙이는 '컴포넌트'를 GameState 등에 붙일 수 있도록 하는 클래스이다<br>
	(GameState에 붙이는 추가적인 기능 같은 것)<br>
	(-> Unity에서 다양한 GameObject에 컴포넌트를 붙일 수 있듯)<br>
	GameFeature의 시스템 기능으로 unity처럼 부품을 붙였다 뗄수 있음<br>
	(확장성이 좋아짐, 다만 디버깅이 힘들어지고, 부품이 많아질 순 있음)<br>

### GameState에 가서 먼저 컴포넌트 생성해주기

```cpp
.h

UCLASS()
class SAMPLES_API ASampleGameState : public AGameStateBase
{
	GENERATED_BODY()
public:
	ASampleGameState();

public:
	UPROPERTY()
	TObjectPtr<USampleExperienceManagerComponent> ExperienceManagerComponent;

};

---
.cpp

#include "SampleGameState.h"
#include "SampleExperienceManagerComponent.h"

ASampleGameState::ASampleGameState()
{
	ExperienceManagerComponent = CreateDefaultSubobject<USampleExperienceManagerComponent>(TEXT("ExperienceManagerComponent"));
}
```

### 다시 돌아와서...

멤버 변수로 뭘 넣을 것인지?<br>

```cpp
public:
	// 가리키는 객체를 수정할 수 없도록 const 를 걸어준다
	// 그래도 ptr은 새로운 Experience를 가르킬 수 있음
	// c++ 의 type* const 방식
	//
	// 로딩 요청을 받아야 로딩을 한다
	UPROPERTY()
	TObjectPtr<const USampleExperienceDefinition> CurrentExperience;
```

- CurrentExperience : 로딩을 할 대상<br>


#### 로딩 완료 체크 시점?

[![Image](https://github.com/user-attachments/assets/289a2427-73bb-4ccf-b32f-ecbdcafc2ae7)](https://github.com/user-attachments/assets/289a2427-73bb-4ccf-b32f-ecbdcafc2ae7){: .image-popup}<br>

InitGameState!<br>
- 이 시점에서 GameMode가 매니저에게 '완료 다 되면 알려달라고'<br>
  구독하고 간다<br>

그렇기에 2가지가 필요한데<br>
- '로딩 상태'에 대한 정의<br>
  (Enum class)	

```cpp
enum class ESampleExperienceLoadState
{
	Unloaded,
	Loading,
	LoadingGameFeatures,
	ExecutingActions,
	Loaded,
	Deactivating,
};
```

- '로딩 완료'에 따른 알림<br>
  (DELEGATE가 필요한 시점이다)<br>

```cpp
// Multicast : 하나의 이벤트에 여러 함수 연결 가능
DECLARE_MULTICAST_DELEGATE_OneParam(FOnSampleExperienceLoaded, const USampleExperienceDefinition*);
```

이후 로딩을 완료한 ExperienceDefinition를 통해<br>
해당 게임 모드에서 사용할 데이터를 넘겨준다<br>

해당하는 두 요소와 넘겨줄 Experience를<br>
매니저의 멤버 변수로 추가<br>

```cpp
// 가리키는 객체를 수정할 수 없도록 const 를 걸어준다
// 그래도 ptr은 새로운 Experience를 가르킬 수 있음
// c++ 의 type* const 방식
//
// 로딩 요청을 받아야 로딩을 한다
UPROPERTY()
TObjectPtr<const USampleExperienceDefinition> CurrentExperience;

// Experience의 로딩 상태를 모니터링
ESampleExperienceLoadState LoadState = ESampleExperienceLoadState::Unloaded;

// Experience 로딩이 완료된 이후, BroadCasting Delegate
FOnSampleExperienceLoaded OnExperienceLoaded;
```

### IsExperienceLoaded()

```
// 로드 완료 + Experience 존재하는지 체크
bool IsExperienceLoaded() { return (LoadState == ESampleExperienceLoadState::Loaded) && (CurrentExperience != nullptr); }
```

- 외부에서 이 함수를 통해 로딩 완료를 확인하는 용도<br>

### CallOrRegister_OnExperienceLoaded()?

```cpp
Header

// FOnSampleExperienceLoaded::FDelegate - 해당 델리게이트에서 요구하는 함수 타입을 인자로 받는다는 뜻
// &&(RValue Reference) 
// - '이동'하여 객체를 복사하는 대신 해당 자원을 그대로 사용함
// - 리터럴, 람다, 함수반환값 등과 같이 임시 생성된 객체를 포함받을 수 있음
// &(LValue Reference)
// - 이 방식은 메모리 상에 존재하는 객체만 받을 수 있음
// 
// 아마 RValue인 람다 or std::function 등만 인자로 받을 수 있어 보임
//
// OnExperienceLoaded 에 바인딩 하거나, Experience 로딩이 완료되었다면 호출
void CallOrRegister_OnExperienceLoaded(FOnSampleExperienceLoaded::FDelegate&& Delegate);

---

cpp

void USampleExperienceManagerComponent::CallOrRegister_OnExperienceLoaded(FOnSampleExperienceLoaded::FDelegate&& Delegate)
{
	if (IsExperienceLoaded())
	{
		// 요청하는 함수에게 이미 로딩이 되었으므로
		// 현재 Experience를 인자로 건네주며 실행시킨다(callback)
		Delegate.Execute(CurrentExperience);
	}
	else
	{
		// movetemp 를 통해 rvalue Reference 위치를 이동 시킴
		// Delegate 리스트에 해당 함수를 등록
		//
		// Delegate 객체는 내부적으로 필요한 변수를 메모리 할당해놓음
		// (함수 인자 등, 여러 상태를 저장하는 변수
		// 
		// ex)
		// TArray<int> a = {1,2,3,4}
		// delegate_type delegate = [a](){return a.num;}
		// -> a는 delegate_type 내부에 new로 할당이 된다
		//  -> 복사 비용을 낮추기 위하여 Move를 통하여 전달
		OnExperienceLoaded.Add(MoveTemp(Delegate));
	}
}
```

- 이미 로드된 Experience라면<br>
  바로 Delegate를 호출시켜 준다<br>
  (그대로 현재 Experience를 넘겨주어 실행)<br>
  (사실상 바로 callback)<br>

- 아니라면 Delegate에 등록시켜서<br>
  나중에 로딩 완료되었을때<br>
  한번에 broadcast 하는 용도<br>

- rvalue로 넘긴 이유는<br>
  '임시 Delegate'를 move를 통하여<br> 
  현재 delegate에 등록<br>
  (Add 할때, move를 통해 소유권을 양도 받고 있음)<br>

- ::FDelegate?<br>
  : 해당 델리게이트 타입에 '바인딩'할 '객체'<br>
  (보통 함수 포인터를 전달하는 방식)<br>
  더 쉽게 말하자면<br>
  Delegate에 들어간 '원소 타입'이라고 봐도 좋음<br>
  따라서 우리는 const USampleExperienceDefinition*를 인자로 받는<br>
  void 함수를 전달 받은 것과 같음<br>