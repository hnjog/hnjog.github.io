---
title: "Experience Load 4"
date : "2025-10-17 14:00:00 +0900"
last_modified_at: "2025-10-17T14:00:00"
categories:
  - Unreal
  - Lyra
  - 언리얼 5
tags:
  - Lyra
  - Experience
---

# Experience Load 4

[Experience Load 3 관련 블로깅](https://hnjog.github.io/unreal/lyra/%EC%96%B8%EB%A6%AC%EC%96%BC%205/ExperienceLoad3/){:target="_blank"}<br>


## 로딩을 완료하였으니 ExperiendeLoad 완료에 따른 처리를 해보자

[![Image](https://github.com/user-attachments/assets/fe11264e-85bc-41ba-a076-83542a183b5e)](https://github.com/user-attachments/assets/fe11264e-85bc-41ba-a076-83542a183b5e){: .image-popup}<br>

### GameMode 에서 GetPawnDataForController() 구현

```cpp
h

const USamplePawnData* GetPawnDataForController(const AController* InController) const;

---
cpp

const USamplePawnData* ASampleGameMode::GetPawnDataForController(const AController* InController) const
{
	// 게임 도중에 PawnData가 오버라이드 되었을 경우,
	// PawnData는 PlayerState에서 가져오게 됨
	if (InController)
	{
		if (const ASamplePlayerState* SamplePS = InController->GetPlayerState<ASamplePlayerState>())
		{
			// 현재 캐싱이 되어있는 PawnData가 이미 있다면
			// 그대로 반환한다 (이 경우는 playerState에서 바로 사용하는 것과 똑같긴 하다)
			if (const USamplePawnData* PawnData = SamplePS->GetPawnData<USamplePawnData>())
			{
				return PawnData;
			}
		}
	}

	// fall back to the default for the current experience
	// 아직 PlayerState에 PawnData가 설정되어 있지 않은 경우,
	// ExperienceManagerComponent의 CurrentExperience로부터 가져와서 설정한다
	check(GameState);
	USampleExperienceManagerComponent* ExperienceManagerComponent = GameState->FindComponentByClass<USampleExperienceManagerComponent>();
	check(ExperienceManagerComponent);

	// load가 완료 되었다면 현재 Experience 쪽으로 가서 가져옴
	if (ExperienceManagerComponent->IsExperienceLoaded())
	{
		const USampleExperienceDefinition* Experience = ExperienceManagerComponent->GetCurrentExperienceChecked();
		if (Experience->DefaultPawnData)
		{
			return Experience->DefaultPawnData;
		}
	}

	// load가 된것도 아닌데 여기 들어왔네?
	return nullptr;
}
```

- PlayerState에서 PawnData를 세팅하기 위하여 사용할 함수<br>

- PlayerState도 Experience 로딩이 완료된 시점에서 호출되나<br>
  GameState 쪽에서도 제대로 로드가 되었는지를 확인하기 위한것<br>
	- 이미 PawnData가 세팅되어 있다면 Pass<br>
	- 아니라면 ExperienceManager를 통해 Experience를 가져온 후,<br>
	  내부의 PawnData를 반환한다<br>

#### ExperienceManagerComponent 의 GetCurrentExperienceChecked() 는?

```cpp
const USampleExperienceDefinition* USampleExperienceManagerComponent::GetCurrentExperienceChecked() const
{
	check(LoadState == ESampleExperienceLoadState::Loaded);
	check(CurrentExperience != nullptr);
	return CurrentExperience;
}
```

- 로딩 완료 여부 및 현재 Experience를 '체크'<br>
  (애초에 저 조건에 맞지 않으면 Init 로직이 이상한 것이므로)<br>

- 현재 Experience 로딩<br>

- Check는 이전에도 말하였듯 assert 처럼 게임을 아예 크래쉬 내지만<br>
  Init 이나 Load 같은 상황에서는 꽤나 유용한 함수이다<br>
  - 애초에 정상적인 환경이 아니면 초기에 실패를 시키고 원인을 빨리 찾을 수 있게함<br>
  - 그러나 그 외의 부분에 사용하는 것은 고려할 것<br>
    (절대 들어올리 없는 거라 생각해서 assert 사용했는데 진짜 게임이 터지는 경우 존재)<br>
	(이 경우는 Log를 사용하는 로깅 방식이 더 효율적일 수 있음)<br>

## PlayerState 의 OnExperienceLoaded 를 구현

```cpp
h
UCLASS()
class SAMPLES_API ASamplePlayerState : public APlayerState
{
	...

	template<class T>
	const T* GetPawnData() const { return Cast<T>(PawnData); }
	// 폰 데이터 캐싱을 위하여 Experience 로딩 완료때 가져오기 위함
	void OnExperienceLoaded(const USampleExperienceDefinition* CurrentExperience);
	void SetPawnData(const USamplePawnData* InPawnData);

	...
}
---
cpp

void ASamplePlayerState::OnExperienceLoaded(const USampleExperienceDefinition* CurrentExperience)
{
	if (ASampleGameMode* GameMode = GetWorld()->GetAuthGameMode<ASampleGameMode>())
	{
		// CurrentExperience 내부에 DefaultPawnData가 존재하는데
		// 굳이 GameMode를 경유해서?
		// 
		// 1. 안정성 검사
		// - Experience 로딩이 완료된 시점이긴 하나
		//   혹시 모르니 GameState를 통해 Manager에 CurrentExperience가 제대로 로드 되었는지를 확인하기 위함
		//   (처음 들어온 시점에선 자신의 PawnData는 비어있을 테니,
		//	  GameState의 GetPawnDataForController 내부에서
		//	  ExperienceManager 의 Loaded 함수를 통해 로딩 여부를 다시 체크한다)
		// 
		// 2. PawnData를 가져오는 함수를 통일시킴
		//  - 현 시점에선 PlayerState가 사용하지만
		//    GameMode, Experience, PlayerState가 모두 PawnData를 각각 사용한다면 
		//    각자의 호출이 다르기에 유지보수 비용이 더 들어가게 됨
		//

		const USamplePawnData* NewPawnData = GameMode->GetPawnDataForController(GetOwningController());
		check(NewPawnData);

		SetPawnData(NewPawnData);
	}
}

void ASamplePlayerState::SetPawnData(const USamplePawnData* InPawnData)
{
	check(InPawnData);
	check(!PawnData); // 2번 설정되지 않게 하기 위해
	PawnData = InPawnData;
}
```

- GameMode 쪽에서 GetPawnDataForController()를 가져와 PawnData로 사용<br>
  (이쪽 로딩이 이미 완료되었다면 바로 return 해준다)<br>
  - 더블체크의 의미<br>
  - 또한 PawnData 가져오는 함수를 GameMode가 일임하게 하여<br>
    PawnData는 GameMode 쪽에서 구하도록 함<br>
	- 차후 다른 클래스에서 PawnData가 필요하더라도<br>
	  현재 GameMode 쪽에서 바로 호출할 수 있으므로<br>
	  `호출 체이닝`이 없음<br>

- PawnData 세팅을 위한 `Set` 과 Template 캐스팅용의 `Get` 함수를 각각 만들어 준다<br>
  - 이전 TIL에 적었듯 GAS 구현 시점에서 사용하기 위하여<br>
    미리 PawnData 캐싱을 해두는 용도<br>

## GameModeBase의 OnExperienceLoaded() 를 통해 Player를 Restart!

```cpp
void ASampleGameMode::OnExperienceLoaded(const USampleExperienceDefinition* currentExperience)
{
	// PlayerController 순회
	for (FConstPlayerControllerIterator Iterator = GetWorld()->GetPlayerControllerIterator(); Iterator; Iterator++)
	{
		APlayerController* PC = Cast<APlayerController>(*Iterator);
		// Load가 지금 완료되었지만,
		// 먼저 Spawn되어 아무것도 하지 못한 Case가 있을 수 있음
		// 그렇기에 전부 순회하면서 한번씩 체크

		// PC가 Pawn을 Possess하지 않았다면, RestartPlayer을 통해 Pawn을 다시 Spawn
		// - Onpossess를 참고
		if (PC && PC->GetPawn() == nullptr)
		{
			if (PlayerCanRestart(PC))
			{
				// 재시작?
				// 스타트 지점을 찾고,
				// 내부적으로 Pawn을 가져오고 Spawn 해준다
				// 이후 SetPawn, 그리고 Possess를 진행
				// 사실상 초기에 Player 생성되는 로직을 다시 진행하는 것과 같다
				// 
				// 우리는 Experience가 로드될때까지 Pawn 생성을 막아뒀기에
				// 초반에 해당 로직이 실행되지 못하는데
				// 그것을 Restart를 통해 다시 진행시켜줌
				RestartPlayer(PC);
			}
		}
	}
}
```

- Restart를 통해 '이미' 생성을 막았던 Player들을<br>
  다시 배치해준다<br>

- RestartPlayer<br>
  : Player 시작 지점을 찾고 Spawn 진행<br>
   - 초기에 Player 생성 로직과 동일한 진행을 하기에<br>
      우리가 GameMode에서 override한<br>
	  `HandleStartingNewPlayer_Implementation`<br>
	  `SpawnDefaultPawnAtTransform_Implementation`<br>
	  들을 호출시킨다<br>
   - 마무리로 Possess까지 시켜주는 '우리가 원하는' 함수<br>

## 뭔가 빠졌나?

- 캐릭터 자체는 잘 소환이 되나<br>
  우리가 만든 `SimplePawnData` 내부의 Pawn Class 가 아닌<br>
  기본 지정 Pawn Class가 생성된다<br>
  (우린 SampleCharacter를 설정해놓음)<br>

- 그래도 Restart 와 Experience Load 자체는 성공적으로 된다<br>

- PawnData를 제대로 설정이 안되고 있는것!<br>

### GameMode에서 PawnClass를 얻어오는 부분을 수정하자 (Override)

[![Image](https://github.com/user-attachments/assets/9369e65b-6471-48c3-b3bf-a96de0f20032)](https://github.com/user-attachments/assets/9369e65b-6471-48c3-b3bf-a96de0f20032){: .image-popup}<br>

- 그렇기에 ExperienceLoad 완료 후, SetPawnData가 된 녀석을 가져와<br>
  그 녀석을 생성할 PawnClass로 잡아준다<br>

```cpp
GameMode.h

virtual UClass* GetDefaultPawnClassForController_Implementation(AController* InController) final;

---
cpp

UClass* ASampleGameMode::GetDefaultPawnClassForController_Implementation(AController* InController)
{
	// GetDefaultPawnClassForController 를 활용해
	// PawnData로부터 PawnClass를 가져옴
	if (const USamplePawnData* PawnData = GetPawnDataForController(InController))
	{
		if (PawnData->PawnClass)
		{
			return PawnData->PawnClass;
		}
	}

	// 원래는 여기서 생성자에서 등록된 ASampleCharacter를 통해 SampleCharacter를 세팅하지만
	// Experience가 load 되었다면 그걸 가져온다
	return Super::GetDefaultPawnClassForController_Implementation(InController);
}
```

- 위에서 만들어준 `GetPawnDataForController`()를 통해<br>
  PawnData를 가져온다<br>
  (캐싱해둔 것이 있다면 Experience 쪽에 등록된 PawnData를 가져와 사용)<br>

- 그러나 Experience 로딩이 제대로 되어 있지 않다면<br>
  Experience Manager를 통해 가져오며<br>
  그것도 안되어있는 상황일때 Defalut Pawn Class를 이용<br>

-> lyra는 Override를 통하여 엔진의 흐름을 잘 이용하는 구조를 보여줌<br>

### Experience Load의 마무리!

여기까지 해주면 성공적으로 PawnData가 생성됨!<br>

- 설정한 Experience<br>

[![Image](https://github.com/user-attachments/assets/5df63af9-6695-4955-8dd0-a857afd6394b)](https://github.com/user-attachments/assets/5df63af9-6695-4955-8dd0-a857afd6394b){: .image-popup}<br>

- PawnData에 넣어준 PawnClass<br>

[![Image](https://github.com/user-attachments/assets/53fbec28-c33f-402b-8d71-e92afafd3cc3)](https://github.com/user-attachments/assets/53fbec28-c33f-402b-8d71-e92afafd3cc3){: .image-popup}<br>

- 생성된 Pawn<br>

[![Image](https://github.com/user-attachments/assets/43c7ee93-7409-4be9-9dd3-c63690fc0cd4)](https://github.com/user-attachments/assets/43c7ee93-7409-4be9-9dd3-c63690fc0cd4){: .image-popup}<br>

