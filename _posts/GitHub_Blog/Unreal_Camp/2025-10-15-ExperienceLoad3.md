---
title: "Experience Load 3"
date : "2025-10-15 20:00:00 +0900"
last_modified_at: "2025-10-15T20:00:00"
categories:
  - Unreal
  - Lyra
  - 언리얼 5
tags:
  - Lyra
  - Experience
  - GameMode
---

# Experience Load 3

[Experience Load 2 관련 블로깅](https://hnjog.github.io/unreal/lyra/%EC%96%B8%EB%A6%AC%EC%96%BC%205/ExperienceLoad2/){:target="_blank"}<br>


## 캐릭터를 막는데 까지는 진행했으니, 실제 Experience Load를 구현해보자

[![Image](https://github.com/user-attachments/assets/e6d4e4aa-a5ef-4434-beca-a833485527ff)](https://github.com/user-attachments/assets/e6d4e4aa-a5ef-4434-beca-a833485527ff){: .image-popup}<br>

### 계속 내버려두었던 ASampleGameMode::HandleMatchAssignmentIfNotExpectingOne()에 내용 추가

```cpp
void ASampleGameMode::HandleMatchAssignmentIfNotExpectingOne()
{
	// 해당 함수에서 우리가 로딩할 Experience에 대해 PrimaryAssetId를 생성하여
	// OnMatchAssignmentGiven으로 넘겨준다

	// PrimaryAsset : '중요한 자산'이란 뜻으로
	// Unreal의 자산 시스템의 일부
	// PrimaryAsset은 로드 및 관리하는 시스템이 통합되어 있기에
	// 핵심 에셋들을 효율적으로 로드 & 언로드 할 수 있음
	// (일반 Asset 은 UObject)
	// 
	// FPrimaryAssetId : PrimaryAsset을 구별하기 위한 식별자이며
	// 내부에 AssetType과 AssetName을 가진다
	// (일종의 FName 변수이며, 각각 구별을 위한 태그로 사용자가 임의로 지정 가능)
	//
	FPrimaryAssetId ExperienceId;

	// precedence order(highest wins) -> 우선순위
	// - matchmaking assignment(if present)
	// - default experience
	UWorld* World = GetWorld();

	// fall back to the default experience
	// 기본 옵션으로 default하게 B_SampleDefaultExperience로 설정해놓기
	if (!ExperienceId.IsValid())
	{
		ExperienceId = FPrimaryAssetId(FPrimaryAssetType("SampleExperienceDefinition"), FName("B_SampleDefaultExperience"));
	}

	OnMatchAssignmentGiven(ExperienceId);
}
```

[언리얼의 AssetManager, PrimaryAsset 관련 블로깅](https://hnjog.github.io/unreal/AssetManager/){:target="_blank"}<br>

- 현 시점에서는 ExperienceId가 아무것도 없기에<br>
  default로 만든 PrimaryAssetId를 넣는다<br>
  - 차후 수정 예정<br>

- FPrimaryAssetId<br>
  : 언리얼이 에셋을 `고유`하게 식별하기 위해 만든 식별자 구조체<br>

```cpp
struct FPrimaryAssetId
{
    FPrimaryAssetType PrimaryAssetType;
    FName PrimaryAssetName;
};
```

- FPrimaryAssetType("SampleExperienceDefinition")<br>
  : 해당 텍스트를 가진 Primary Asset Type을 의미<br>
    (말 그대로 '에셋의 타입'을 나타냄)<br>
	- 단순히 FName 데이터이지만, 내부적으로 UAssetManager가 이를 통해<br>
	  관리할 수 있는 Primary Asset Class를 연결<br>
	- 그렇기에 우리의 `USampleExperienceDefinition` 타입을 가져올 것<br>

- PrimaryAssetName<br>
  : 해당하는 자산의 '이름'<br>
    `에셋 타입` 중에서 `B_SampleDefaultExperience`라는 이름을 가진 에셋을 구분하는 키<br>

한 틱 이후 이 함수를 호출하였기에<br>
본격적인 Experience Load의 시작 예정...!<br>

### OnMatchAssignmentGiven()에선 뭘해줄까?

```cpp
// 선택한 ExperienceId를 서버로 넘겨준다
void ASampleGameMode::OnMatchAssignmentGiven(FPrimaryAssetId ExperienceId)
{
	// ExperienceManagerComponent를 활용하여 Experience를 로딩하기 위해
	// ExperienceManagerComponent의 ServerSetCurrentExperience 를 호출
	check(ExperienceId.IsValid());

	check(GameState);
	USampleExperienceManagerComponent* ExperienceManagerComponent = GameState->FindComponentByClass<USampleExperienceManagerComponent>();
	check(ExperienceManagerComponent);

	// 네트워크 상에선 게임 모드가 Client가 아니라 Server에 존재
	ExperienceManagerComponent->ServerSetCurrentExperience(ExperienceId);
}
```

- GameState에서 ExperienceManagerComponent를 가져오고<br>
  ServerSetCurrentExperience 라는 함수로 넘겨준다<br>

- Lyra가 Dedicate Server 기반이기에<br>
  네트워크에 '전달'해주는 함수<br>
  (물론 우리는 현재 구조 공부중인 Client 기반이기에<br>
  내부에선 다른 처리를 할 예정)<br>

- 사실상 중간 다리 역할을 하는 함수 느낌이긴 하다<br>

## USampleExperienceManagerComponent 에서 본격적인 로딩을 처리

### ServerSetCurrentExperience()

```cpp
USampleExperienceManagerComponent.h

public:
	void ServerSetCurrentExperience(FPrimaryAssetId ExperienceId);
---

cpp

void USampleExperienceManagerComponent::ServerSetCurrentExperience(FPrimaryAssetId ExperienceId)
{
	USampleAssetManager& AssetManager = USampleAssetManager::Get();

	TSubclassOf<USampleExperienceDefinition> AssetClass;
	{
		FSoftObjectPath AssetPath = AssetManager.GetPrimaryAssetPath(ExperienceId);
		// B_ 블루프린트가 load 될 것
		AssetClass = Cast<UClass>(AssetPath.TryLoad()); // 동기식 load
	}

	// CDO : Class Default Object
	// 특정한 클래스의 기본 인스턴스
	// -> 해당 클래스의 기본 속성값을 가진 객체
	// (CDO의 값을 변경하는 경우, 차후 생성되는 인스턴스의 기본값이 변경됨)
	// 
	// '기본값의 모음'이자 객체화된 인스턴스
	// 
	// CDO 를 가져오는 이유?
	// 직렬화 하는 과정에서 Delta Serialization 을 지원하기 위함
	// -> 직렬화는 데이터를 저장하는 방식
	// -> Delta 직렬화는 데이터의 '변경 사항'만을 기록하여 데이터를 저장
	// 
	// CDO를 통해 기본값과 비교하여 바뀐 값만 저장하는 방식을 통해
	// 메모리 절약 및 네트워크 서버 성능 개선 가능
	// (바뀐 부분만 저장 및 전송)
	// 
	// CDO를 마음대로 수정하는것은 위험하지 않나?
	//
	const USampleExperienceDefinition* Experience = GetDefault<USampleExperienceDefinition>(AssetClass);
	check(Experience != nullptr);
	check(CurrentExperience == nullptr);
	{
		// CDO로 CurrentExperience를 설정
		CurrentExperience = Experience;
	}

	StartExperienceLoad();
}
```
- AssetManager를 통해<br>
  에셋 경로를 얻고, 그걸 통해<br>
  `USampleExperienceDefinition` 를 상속받은 **Asset Class**를 가져온다<br>
  - 아마 우리가 만든 BP 형식의 USampleExperienceDefinition일 것<br>
    (데이터는 BP 쪽에서 관리하고 등록만 C++로)<br>

- CDO (Class Default Object)를 가져와<br>
  CurrentExperience로 등록<br>
  (const 로 받기에 등록한 순간 수정 x - CDO 오염 x)<br>
  - CDO를 가져오는 이유?<br>
    : 네트워크 / 동기화 에 유리함<br>
	(서버 / 클라 모두 같은 BP 데이터를 사용할 것이 명확하므로)<br>
	(Delta Serialization은 '데이터'의 변경 사항만을 기록하여<br>
	데이터를 전송하기에 메모리 절약 + 서버 성능 개선)<br>
	- 최적화 방식을 위한 기법 중 하나라 추측<br>
	- CDO는 GC 대상이 아니므로, 괜한 메모리 누수 문제 걱정 x<br>

### 본격적인 로딩 함수 구현

```cpp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core",
    "CoreUObject",
    "Engine",
	"InputCore"
    // GAS
    "GameplayTags",
    // Game Features
    "ModularGameplay",
    "GameFeatures",
}
```

일단 Build.cs에서<br>
GameFeature와 관련된 플러그인을 추가한다!<br>
(GameFeatureSubsystem 관련 내용을 사용할 것)<br>

```cpp
ExperienceManagerComponent.h

public:
void StartExperienceLoad();
void OnExperienceLoadComplete();
void OnExperienceFullLoadCompleted();

```

세 함수를 만들고 먼저 `StartExperienceLoad`를 보자<br>

```cpp
void USampleExperienceManagerComponent::StartExperienceLoad()
{
	check(CurrentExperience);
	check(LoadState == ESampleExperienceLoadState::Unloaded);

	// Load 시작
	LoadState = ESampleExperienceLoadState::Loading;
	USampleAssetManager& AssetManager = USampleAssetManager::Get();

	// ServerSetCurrentExperience에선 ExperienceId를 넘겨주었는데, 여기선 CDO를 활용하여
	// GetPrimaryAssetId를 로딩할 대상으로 넣는다

	TSet<FPrimaryAssetId> BundleAssetList;
	BundleAssetList.Add(CurrentExperience->GetPrimaryAssetId());

	// load Assets associated with the experience
	// GameFeature 사용하여, Experience에 바인딩된 GameFeature Plugin을 로딩할 Bundle 이름을 추가
	// - Bundle : 우리가 로딩할 에셋의 카테고리 이름이라 생각하면 됨
	//
	TArray<FName> BundlesToLoad;
	{
		// OwnerNetMode가 NM_Standalone이면, Client/Server 둘 다 로딩에 추가
		const ENetMode OwnerNetMode = GetOwner()->GetNetMode();
		bool bLoadClient = GIsEditor || (OwnerNetMode != NM_DedicatedServer); // 서버가 아니면 클라
		bool bLoadServer = GIsEditor || (OwnerNetMode != NM_Client);	// 클라가 아니면 서버

		// Lyra는 멀티 서버 기반의 프로젝트
		// 클라에서만 필요한 것이,
		// 서버에서만 필요한 것이 존재하기에
		// BundledsToLoad에 각각 상황에 맞게 태그를 넣어
		// 적절한 PrimaryAsset을 Load시킨다
		if (bLoadClient)
		{
			BundlesToLoad.Add(UGameFeaturesSubsystemSettings::LoadStateClient); // TEXT("Client") 랑 동일
		}
		if (bLoadServer)
		{
			BundlesToLoad.Add(UGameFeaturesSubsystemSettings::LoadStateServer); // TEXT("Server") 랑 동일
		}
	}

	// Delegate 생성
	// load 완료시 호출할 녀석 달아줌
	FStreamableDelegate OnAssetsLoadedDelegate = FStreamableDelegate::CreateUObject(this, &ThisClass::OnExperienceLoadComplete);

	// 지금은 B_SampleDefaultExperience를 로딩하는 용도
	// 비동기 로딩 진행
	// 아깐 TryLoad()를 사용했지만
	// 여긴 CDO와 ChangeBundleStateForPrimaryAssets 등을 사용하여 비동기 로드를 진행?
	// -> 원래 lyra 프로젝트는 대규모 에셋을 관리하는 시스템을 이용
	// 
	// BundlesToLoad에는 현재 Server와 Client 이름이 들어가 있음
	// 
	// USampleExperienceDefinition에 매우 많은 데이터가 들어가 있다고 생각했을때,
	// (ex : 직업 세팅이나 아이템 세팅 등등)
	// -> 전부 로딩하고 싶지 않고, 만약 특정한 존재만 로딩을 하고 싶다면?
	// (ex : 직업 세팅 중 '전사'만 로딩하고 싶음)
	// ChangeBundleStateForPrimaryAssets
	// -> 특정한 BundleAssetList.Array를 통하여 Load할 데이터만
	//	  BundleToLoad에 집어넣어준다
	//
	TSharedPtr<FStreamableHandle> Handle = AssetManager.ChangeBundleStateForPrimaryAssets(
		BundleAssetList.Array(),
		BundlesToLoad,
		{}, false, FStreamableDelegate(), FStreamableManager::AsyncLoadHighPriority);

	if (!Handle.IsValid() || Handle->HasLoadCompleted())
	{
		// 로딩이 완료되었으면, ExecuteDelegate를 통해 OnAssetsLoadedDelegate를 호출하자:
		// - 아래의 함수를 확인해보자:
		FStreamableHandle::ExecuteDelegate(OnAssetsLoadedDelegate);
	}
	else
	{
		// 완료되지 않은 경우는 Delegate에 등록시킨다
		Handle->BindCompleteDelegate(OnAssetsLoadedDelegate);
		// 문제가 되는 경우(예외처리)
		Handle->BindCancelDelegate(FStreamableDelegate::CreateLambda([OnAssetsLoadedDelegate]()
			{
				// ExecuteIfBound : delegate가 무사한 경우, 실행한다
				OnAssetsLoadedDelegate.ExecuteIfBound();
			}));
	}

	// FrameNumber를 주목해서 보자
	static int32 StartExperienceLoad_FrameNumber = GFrameNumber;
}
```

- BundleAssetList.Add(CurrentExperience->GetPrimaryAssetId());<br>
  ID를 가져와 로딩할 대상에 넣는다<br>
  (이미 CurrentExperience는 아까 Server 쪽에서 한번 클래스네임을 가져오며<br>
  로드가 되었지만 관련 번들들이 모두 Load 된것은 아님!)<br>
	- 그렇기에 CDO로 객체 정의를 읽고<br>
  	  `실제 로딩할 에셋`을 비동기 로드하는 방식임<br>

- 이후 로딩 완료시 호출받을 Delegate 생성<br>
  (OnAssetsLoadedDelegate)<br>

- 비동기 로딩을 진행<br>
  Handle을 통하여 비동기 로딩을 진행한 후<br>
  로딩 완료시 OnExperienceLoadComplete 를 호출하도록<br>
  (물론 정말 바로되는 경우는 바로 Execute)<br>
  (문제가 생기는 경우에도 최소한 ExecuteIfBound를 호출하여<br>
  바인딩된 함수를 호출함)<br>

- 클라 / 서버 환경을 체크하고<br>
  활성화할 `번들`을 바꾼다<br>
  - ChangeBundleStateForPrimaryAssets 를 통해 필요한 데이터만 BundlesToLoad에 넣어줌<br>
    (번들 상태를 바꿀 Primary Asset 들의 목록을 BundleAssetList에 넣는 것!)<br>
  - 특정한 PrimaryAssetId 들에 대하여 Load해주는 역할<br>
    (Server 번들이라면 굳이 애니메이션 같이 '화면에 보여주는' 용도의 에셋은 필요 없으므로!)<br>
	(클라이언트도 지금은 Client 만 로드하지만 필요에 따라 번들을 더 추가할 수 있음)<br>
  - 번들(Bundle)?<br>
    : Primary Asset 내부에서 '함께' 로드/언로드 하도록 설정된 리소스 묶음<br>

그럼 `OnExperienceLoadComplete`는 뭘 할까?<br>

```cpp
void USampleExperienceManagerComponent::OnExperienceLoadComplete()
{
	OnExperienceFullLoadCompleted();
}
```

- OnExperienceLoadComplete는 당장 해줄만한 내용은 없음...<br>
  (이런거 만들면 혼난다...)<br>

- 그러나 지금은 초기이기에 그런것이며<br>
  원본에서는 Experience Load 이후<br>
  다른 내용들을 Load해주는 과정<br>

마지막으로 `OnExperienceFullLoadCompleted`를 보자!<br>

```cpp
void USampleExperienceManagerComponent::OnExperienceFullLoadCompleted()
{
	check(LoadState != ESampleExperienceLoadState::Loaded);

	LoadState = ESampleExperienceLoadState::Loaded;
	OnExperienceLoaded.Broadcast(CurrentExperience);
	OnExperienceLoaded.Clear();
}
```

- 로드를 완료했으므로<br>
  구독하고 간 함수들에게 BroadCast를 해준다!<br>

- GameMode와 PlayerState 쪽에서<br>
  이제 로딩이 완료되었을 때 등록한 함수들에게<br>
  호출이 가게 됨!<br>


캐릭터 생성과 같은 일들에 대한 처리는 다음 TIL에서...<br>