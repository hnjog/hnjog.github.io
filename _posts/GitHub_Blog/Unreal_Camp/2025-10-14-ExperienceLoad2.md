---
title: "Experience Load 2"
date : "2025-10-14 16:00:00 +0900"
last_modified_at: "2025-10-14T16:00:00"
categories:
  - Unreal
  - Lyra
  - 언리얼 5
tags:
  - Lyra
  - Experience
  - GameMode
---

# Experience Load 2

[Experience Load 1 관련 블로깅](https://hnjog.github.io/unreal/lyra/%EC%96%B8%EB%A6%AC%EC%96%BC%205/ExperienceLoad1/){:target="_blank"}<br>

이전 블로깅 작성 후 시간이 꽤 지났지만<br>
다시 복습하면서 나아가보자<br>

## GameModeBase로 돌아와 InitGameState를 생성해주기

[![Image](https://github.com/user-attachments/assets/b7a3bbec-44a6-45ee-bc2a-0d40af469ed1)](https://github.com/user-attachments/assets/b7a3bbec-44a6-45ee-bc2a-0d40af469ed1){: .image-popup}<br>

이제 USampleExperienceManagerComponent 에서 만든<br>
CallOrRegister_OnExperienceLoaded() 을 사용하기 위하여<br>

게임 모드 베이스 쪽 클래스에서 InitGameState을 생성해준다<br>

```cpp
UCLASS()
class SAMPLES_API ASampleGameMode : public AGameModeBase
{
	GENERATED_BODY()

public:
	ASampleGameMode();

	virtual void InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage) override;
	virtual void InitGameState() final;

	void OnExperienceLoaded(const USampleExperienceDefinition* currentExperience);
	void HandleMatchAssignmentIfNotExpectingOne();
}

---

...

void ASampleGameMode::InitGameState()
{
	Super::InitGameState();

	// GameState 내부에 해당 클래스 존재시 반환
	// Experience 비동기 로딩을 위한 Delegate를 준비
	USampleExperienceManagerComponent* ExperienceManagerComponent = GameState->FindComponentByClass<USampleExperienceManagerComponent>();
	// InitGameState 가 호출된 시점에는 ExperienceManagerComponent 가 생성되어있음
	// (GameState 에서 생성하면서 해당 클래스를 생성해놓았으므로)
	check(ExperienceManagerComponent);

	// OnExperienceLoaded를 등록
	// 로딩이 되어있다면 바로 호출하고, 아니면 기다렸다 호출된다
	// 현재 시점에선 로드가 안되어있으므로 기다렸다 호출하기 위함
	ExperienceManagerComponent->CallOrRegister_OnExperienceLoaded(FOnSampleExperienceLoaded::FDelegate::CreateUObject(this, &ThisClass::OnExperienceLoaded));
}

void ASampleGameMode::OnExperienceLoaded(const USampleExperienceDefinition* currentExperience)
{
	// 아직 대기!
}
...

```

- InitGameState를 통해 GameState를 만든 후<br>
  `Experience 비동기 로딩`을 위하여<br>
  로딩 완료 후, `OnExperienceLoaded`를 호출하라고 **등록**함<br>
  (Experience들이 완전히 로딩되면 그재서야 GameMode가 Player를 생성시켜줌)<br>
  (Experience 로딩 여부는 ManagerComponent가 하겠지~)<br>

- OnExperienceLoaded<br>
  : 아직은 구현하지 않았지만 Player를 다시 생성시켜줄 예정<br>
  (모든 비동기 로딩이 끝났으니 Player를 안전하게 생성)<br>

- FindComponentByClass?<br>
  : 해당 개체 내부에서 해당 클래스가 있다면 반환 없으면 null<br>
  (해당 템플릿 클래스를 '상속'받아도 Ok!)<br>
  (다만 Find'Component'이기에 컴포넌트 타입만 가능하다)<br>
  - check 는 '초기화' 과정에서 사용할 법한 일종의 assert<br>
    해당 조건이 false라면 crash 시키고, 디버거를 통해 어디서<br>
	실패했는지를 알려준다<br>
	(디버깅용 장치로서, Init 시점 or 개발 과정에서 사용할 법 하다)<br>

- final을 붙인 이유??<br>
  : 게임 모드의 InitGameState를 상속한 후<br>
    더 이상 하위 게임 모드에선 수정하지 말 것을 당부<br>
	-> '게임 모드'에서 GameState 와 관련된 생성 로직을 건들지 말것!<br>

## 그래서 우리가 어디쯤 온거지?

[![Image](https://github.com/user-attachments/assets/6367d8e4-c6f4-4fc8-a11a-ab6ddd846bd6)](https://github.com/user-attachments/assets/6367d8e4-c6f4-4fc8-a11a-ab6ddd846bd6){: .image-popup}<br>

현재 1번 단계를 완료한 상황!<br>

- 2번 단계부터는 **엔진이 진행**해준다<br>
  - 해당 과정을 요약하자면, <br>
    World 내에서 PlayerStart를 찾아 시작 위치를 정함<br>
	(혹시 없다면 생성 가능한 랜덤한 위치를 정함)<br>
	이후, 우리가 설정한 기본 Pawn을 세팅<br>
	그리고 PostLogin 시점에서 Player Spawn 과정을 시작하게 됨<br>

그런데 여기서 한가지 문제가 발생한다<br>

- 고작 **1 프레임**으로 `ExperienceLoad`가 완료될리가 없다는 것!<br>

우리가 원하는 것은<br>

- 로딩 완료 후, 캐릭터를 소환하는 것<br>
  (그 Experience 정보들을 이용하여 내부 세팅을 하고 싶은 것이므로!)<br>

- 그렇기에 이러한 과정을 잠시 막아두어야 함<br>
  언제까지? 로딩이 완료될때까지!<br>
  (OnExperienceLoaded 에 Player Spawn 관련 코드가 들어가게 되는 이유!)<br>
  Restart 라는 함수를 통하여 생성해줄 예정<br>


[![Image](https://github.com/user-attachments/assets/48a1d413-67a8-412d-b456-938749eee9ee)](https://github.com/user-attachments/assets/48a1d413-67a8-412d-b456-938749eee9ee){: .image-popup}<br>

Restart 호출을 통하여 Experience Load 가 완료된 후<br>
플레이어를 Spawn 시켜주는 것이 우리의 목표!<br>

## 먼저 PlayerState로 가자

PawnData를 참조하고 캐싱해 놓는다고 이야기한 PlayerState 쪽으로 가<br>
PawnData와 관련된 부분을 작성하기<br>

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerState.h"
#include "SamplePlayerState.generated.h"

class USampleExperienceDefinition;
class USamplePawnData;

UCLASS()
class SAMPLES_API ASamplePlayerState : public APlayerState
{
	GENERATED_BODY()

	// AActor's interface
public:
	virtual void PostInitializeComponents() final;

	// member method
public:
	// 폰 데이터 캐싱을 위하여 Experience 로딩 완료때 가져오기 위함
	void OnExperienceLoaded(const USampleExperienceDefinition* CurrentExperience);

public:
	// Cacheing 하는 이유? (ExperienceManagerComponent에도 있지 않나?)
	// 나중에 PlayerState에 GAS 컴포넌트를 붙일 예정
	// -> GAS가 폰데이터를 참조할 것이기에 미리 캐싱 코드를 만들어 놓음
	UPROPERTY()
	TObjectPtr<const USamplePawnData> PawnData;

};

---
cpp

void ASamplePlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	const AGameStateBase* GameState = GetWorld()->GetGameState();
	check(GameState);

	USampleExperienceManagerComponent* ExperienceManagerComponent = GameState->FindComponentByClass<USampleExperienceManagerComponent>();
	check(ExperienceManagerComponent);

	ExperienceManagerComponent->CallOrRegister_OnExperienceLoaded(FOnSampleExperienceLoaded::FDelegate::CreateUObject(this, &ThisClass::OnExperienceLoaded));
}

void ASamplePlayerState::OnExperienceLoaded(const USampleExperienceDefinition* CurrentExperience)
{
	// 아직 대기!
}

```

- PawnData를 미리 캐싱해두는 이유?<br>
  GAS 쪽을 사용하기 위해 미리 캐싱을 해둠!<br>

- `PostInitializeComponents`?<br>
  : Beginplay 이전에 호출되는 가장 마지막 지점<br>
    (모든 컴포넌트 들이 준비된 상태에서의 마지막 세팅)<br>
	해당 상황에서 GameState쪽에 OnExperienceLoaded 를 Delegate 시켜 놓는다<br>

- PlayerState가 생성되는 시점에서는 GameState가 존재하기에!<br>
  GameState를 호출 가능!<br>
  그렇기에 GameState 내부의 ExpManager에게<br>
  Delegate를 걸 수 있음<br>

- OnExperienceLoaded 에서는 PawnData를 캐싱할 예정<br>

### 다음은 PawnData에서 PawnClass를 세팅해준다

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/DataAsset.h"
#include "SamplePawnData.generated.h"

UCLASS()
class SAMPLES_API USamplePawnData : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:
	USamplePawnData(const FObjectInitializer& ObjectInitalizer = FObjectInitializer::Get());
	
	// Pawn의 class
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Sample|Pawn")
	TSubclassOf<APawn> PawnClass;
};

```

'만들어질 Pawn'을 이쪽에서 TSubclassOf로 받을 예정<br>
(Bp)<br>

- 이전에 말하였듯 `PrimaryDataAsset` 이기에<br>
  BP 쪽에서 설정한 Pawn이 생성과정을 거쳐 생성되게 됨<br>
  
- 이걸 BP에서 만들었다면 이걸 `Experiecne Definition`에 넣어주자!<br>

```less
Experiecne Definition
|
 - Level
 - USamplePawnData
 |- PawnClass
```

다만 여전히 시작시 Character 자체가 '생성'됨!<br>

## 생성을 막아야 한다!

### GameMode로 가자

```cpp

GameMode.h

// HandleStartingNewPlayer
virtual void HandleStartingNewPlayer_Implementation(APlayerController* NewPlayer) final;

// SpawnDefaultPawnAtTransform
virtual APawn* SpawnDefaultPawnAtTransform_Implementation(AController* NewPlayer, const FTransform& SpawnTransform) final;

bool isExperienceLoaded() const;

--- 
cpp

bool ASampleGameMode::isExperienceLoaded() const
{
	check(GameState);
	USampleExperienceManagerComponent* ExperienceManagerComponent = GameState->FindComponentByClass<USampleExperienceManagerComponent>();
	check(ExperienceManagerComponent);

	return ExperienceManagerComponent->IsExperienceLoaded();
}

void ASampleGameMode::HandleStartingNewPlayer_Implementation(APlayerController* NewPlayer)
{
	if (isExperienceLoaded())
	{
		// experience가 load 된 이후에 pawn을 생성할 목적
		// 그리고 load 완료후 다시 이 함수를 호출
		Super::HandleStartingNewPlayer_Implementation(NewPlayer);
	}
}

APawn* ASampleGameMode::SpawnDefaultPawnAtTransform_Implementation(AController* NewPlayer, const FTransform& SpawnTransform)
{
	// 당장은 그냥 부모로 넘겨준다
	return Super::SpawnDefaultPawnAtTransform_Implementation(NewPlayer,SpawnTransform);
}
```

- `HandleStartingNewPlayer_Implementation`?<br>
  : 새로운 플레이어가 들어올때 호출<br>
   보통 PostLogin() 직후 호출되어<br>
   그 플레이어의 Pawn을 Spawn시키고 Possess 시키는 절차를 Trigger 함<br>
   (일반적으로 이걸 커스텀 Override 하는 경우는 RestartPlayer 같은 함수로<br>
   다시 호출시키는 것을 감안함 -> 첫 접속시 몇가지 로직을 끼워넣고 싶은 경우)<br>

- `SpawnDefaultPawnAtTransform_Implementation`?<br>
  : 실제 Pawn 생성 시에 호출<br>
    ChoosePlayerStart 같은 걸로 '위치'를 구한 후<br>
	Pawn을 스폰시키는 함수<br>
    (보통 Pawn Class, 스폰 방식 변경이 필요할때 커스텀 고려)<br>

---
호출 흐름에 대한 간략한 타임라인<br>

```
플레이어 접속 확정
   ↓
PostLogin(NewPlayer)
   ↓
HandleStartingNewPlayer_Implementation(NewPlayer)   // 새로 들어온 플레이어를 ‘게임에 투입’시키는 입구
   ↓
RestartPlayer(NewPlayer)                            // 표준 스폰 루틴
   ↓
ChoosePlayerStart(NewPlayer) → StartSpot 선정
   ↓
SpawnDefaultPawnFor(...) 또는 SpawnDefaultPawnAtTransform_Implementation(NewPlayer, StartSpot->GetTransform())
   ↓
NewPlayer->Possess(SpawnedPawn)
   ↓
SetPlayerDefaults(Pawn) / Pawn::Restart() / OnPossess / PossessedBy 등 초기화 콜백
```
---


[![Image](https://github.com/user-attachments/assets/48a1d413-67a8-412d-b456-938749eee9ee)](https://github.com/user-attachments/assets/48a1d413-67a8-412d-b456-938749eee9ee){: .image-popup}<br>

위에서 본 이미지를 다시 한번 보면<br>
저 1번과 3번에 각각 포함되는 함수들이다!<br>

따라서<br>
- HandleStartingNewPlayer_Implementation에서<br>
  Experience Load가 완료되지 않았다면<br>
  Spawn 시키지 않도록 설정해버린다!<br>
  (그렇기에 우리가 Experience가 완료되면 별도로 캐릭터를 Spawn 시켜주어야 함!)<br>

- 해당 함수 구현시 더이상 DefaultPawn 이 생성되지 않아<br>
  Experience Load가 완료될때까지 생성을 막을 수 있다!<br>


실제 Experience Load는 다음 TIL에...<br>