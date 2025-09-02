---
title: "Experience"
date : "2025-09-02 15:00:00 +0900"
last_modified_at: "2025-09-02T15:00:00"
categories:
  - Unreal
  - Lyra
tags:
  - Lyra
  - Experience
  - Game Feature Plugin
  - Action Sets
---

## Experience
GameMode + GameState 구성을 '비동기'로 로드/전환 하는<br>
'게임 타입' 패키지<br>
(유저가 경험할 rule과 게임 정보 객체)<br>

- UPrimaryDataAsset을 이용한 구현<br>
  - AssetManager에서 관리하는 특별한 DataAsset<br>
  - C++ 로 정의하는 경우 '그 클래스 명'이 곧 AssetType이 됨<br>
  - Blueprint로 선언하게 되면 '전부 같은' PrimaryDataAsset으로 해석될 수 있음<br>
    -> 이 부분은 GetPrimaryAssetId()에 대한 Override로 식별 방식을 바꿔줘야<br>
	   AssetManager가 올바르게 스캔/로드/언로드 해줄 수 있음<br>
  - 그렇기에 Bp 사용시 2가지 중 하나를 골라 작업 방식을 잡는다<br>
    - GetPrimaryAssetId()를 오버라이드해서 '명확한 ID' 구분<br>
	- Class Default Object(CDO)를 기준으로 동작<br>

- CDO 와 Primary Asset ID의 관계?<br>
  : UPrimaryDataAsset::GetPrimaryAssetId() 는 CDO를 기준으로 호출됨<br>
    (해당 CDO를 통해 AssetType,Name을 확인)<br>
	(안정적으로 클래스별 AssetType과 Name을 구분)<br>


### UExperienceDefinition

- 경험(게임 타입)을 구성하는 정의용 데이터(PrimaryDataAsset)
  (게임 시작 시, 어떤 GameFeature를 사용할지)<br>
  (어떤 Action Set(능력/입력/컴포넌트 부여 등)을 적용할지)<br>
  (기본 Pawn/Controller 세팅)<br>

- 데이터만 바꿔도 빌드 없이 바꿀 수 있는 데이터 주도 방식<br>
  필요한 플러그인/Asset 만 로드하여 초기 로딩을 줄임<br>

```
UCLASS()
class SAMPLES_API USampleExperienceDefinition : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:
	USampleExperienceDefinition(const FObjectInitializer& ObjectInitalizer = FObjectInitializer::Get());

#if WITH_EDITORONLY_DATA
	virtual void UpdateAssetBundleData() override;
#endif
public:
	UPROPERTY(EditDefaultsOnly, Category = Gameplay)
	TObjectPtr<USamplePawnData> DefaultPawnData;

	// 마킹과 기억용
	// 게임 모드에 따른 GameFeature plugin 로딩, 이에 대한 연결고리
	// ShooterCore와 관련된 작업 이후 진행
	UPROPERTY(EditDefaultsOnly, Category = Gameplay)
	TArray<FString> GameFeaturesToEnable;

	// Gameplay 용도에 맞게 분류의 목적으로 사용
	UPROPERTY(EditDefaultsOnly, Category = Gameplay)
	TArray<TObjectPtr<USampleExperienceActionSet>> ActionSets;

	// 일반적인 GameFeatureAction으로 구성
	UPROPERTY(EditDefaultsOnly, Category = "Actions")
	TArray<TObjectPtr<UGameFeatureAction>> Actions;
};
```

### UUserFacingExperience

- 프런트엔드에서 Experience 목록을 빠르게 보여주기 위한 메타데이터 용<br>

- 이후 User의 '선택'에 따라 실제 데이터를 로드<br>
  
- '특정 모드'를 선택하는 용도로 사용<br>
  (사용자에게 '어떠어떠한 게임모드(경험)'이 있다고 알려줌)<br>

```
class UCommonSession_HostSessionRequest;

UCLASS()
class SAMPLES_API USampleUserFacingExperience : public UPrimaryDataAsset
{
	GENERATED_BODY()

public:
	// Map 로딩 및 Experience 전환을 위해, MapID와 ExperienceID를 활용하여 HostSessionRequest를 생성
	// const 가 붙을 시, BlueprintPure가 자동으로 true가 되며
	// 실행단위로 표현되지 않기에 주의
	UFUNCTION(BlueprintCallable, BlueprintPure = false)
	UCommonSession_HostSessionRequest* CreateHostingRequest() const;

public:
	// map에 대한 정보
	// meta 를 이용하여 관련 정보만 저장하기 위함
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Experience, meta = (AllowedTypes = "Map"))
	FPrimaryAssetId MapID;

	// 폰 데이터
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Experience, meta = (AllowedTypes = "SampleExperienceDefinition"))
	FPrimaryAssetId ExperienceID;

};
```

- 가벼운 관계도<br>

```
UserFacing
- Map // 맵 정보
- Experience // 해당 맵에서 사용할 Experience
  - PawnData
     - Pawn
     - Input
	 - Ability
```

### UExperienceManagerComponent

- GameState 측에서 생성하여 Experience를 관리하는 용도의 컴포넌트<br>
  (클라 / 서버 양측 모두 사용)<br>

- 해당 Map의 Experience 들의 전체 수명주기를 관리<br>
  
#### Game 객체들의 개념과 생명주기

| 개념               | 범위                 | 생성/소멸 시점                         | 주요 역할                                           |
| ---------------- | ------------------ | -------------------------------- | ----------------------------------------------- |
| **GameInstance** | 앱 실행 전체(프로세스 단위)   | 게임 실행 시 1회 생성 → 앱 종료 시 소멸        | 게임 전역 데이터 보관 (플레이어 프로필, 세션 정보, 로딩 간 유지해야 할 데이터) |
| **World**        | 맵 단위               | 맵 로드 시 생성 → 언로드 시 소멸             | 맵(레벨) + 액터들의 집합. 게임플레이가 실제로 일어나는 공간             |
| **Level**        | World 내부의 하위 단위    | Persistent Level + SubLevel 로드 시 | 지형, 빛, 액터 등을 배치한 공간 데이터                         |
| **GameMode**     | 서버 전용, World 단위    | World 시작 시 생성 → World 종료 시 소멸    | 게임 규칙·흐름 제어 (승리 조건, 팀 설정, 플레이어 스폰 규칙 등)         |
| **GameState**    | 서버·클라 모두, World 단위 | World 시작 시 생성 → World 종료 시 소멸    | GameMode의 상태를 네트워크로 공유 (점수, 남은 시간, 팀 정보 등)      |

- GameInstance <br>
  : 앱 단위로 한번만 생성되며, 레벨 전환시에도 사라지지 않음<br>
  (전역적 데이터 관리)<br>

- World<br>
  : 하나의 '게임 세상', 맵이 로드되면 World가 만들어짐<br>
    (각 클라이언트는 별도의 WOrld Instance를 가진다)<br>

- Level<br>
  : World의 한 부분<br>
    (World 내부에서 여러 Level을 로드/언로드)<br>

- GameMode<br>
  : 서버에만 존재<br>
    현재 World를 어떤 규칙으로 플레이할지 결정<br>
	(플레이어 시작 위치, 점수 계산, 종료 조건 등의 로직 포함)<br>
	(맵이 바뀌면 새로운 GameMode 생성)<br>

- GameState<br>
  : 서버/클라 모두 존재<br>
    GameMode가 가진 현재 상태(게임 상태)를 '네트워크'로 동기화<br>
	클라에서 GameMode는 없지만 GameState로 현재 게임 상황을 확인<br>

- Map?<br>
  : 에디터에서 저장되는 '파일의 단위'(.umap)<br>
    (에디터 기준으로 하나의 '주요' 'Level')<br>


생성 흐름<br>

```
[게임 실행 시작]
       │
       ▼
 ┌─────────────┐
 │ GameInstance│   ← 앱 단위, 단 하나만 생성
 └─────────────┘
       │
       ▼
 ┌─────────────┐
 │   World     │   ← 맵(Persistent Level) 로드 시 생성
 └─────────────┘
       │
   ┌───┴─────────┐
   ▼             ▼
┌───────┐    ┌─────────┐
│Level  │    │ GameMode │ ← 서버 전용
│(맵 조각)   └─────────┘
└───────┘          │
   ▲                ▼
   │          ┌─────────┐
   │          │GameState│ ← 서버·클라 공통
   │          └─────────┘
   │
 (필요 시 SubLevel Streaming)

```

### Game Feature Plugins / Action Sets

- Game Feature Plugins<br>
  : '필요한 때만' 활성화/비활성화 할 수 있는 '특정한 기능 묶음'<br>
    (Plugin 시스템의 확장)<br>
    Experience 로딩 중, 이를 활성화하여<br>
	Ability, Input, UI, Component 등을 주입<br>
	(ex : Shoot 모드 전용 무기,UI,Ability)<br>


- Action Set<br>
  : '행동 묶음'에 관한 데이터 자산<br>
  특정한 '객체'(플레이어/월드 등)에<br>
  어떠한 능력/입력/컴포넌트 등을 붙일지를 기술함<br>
  (ex : 특정 Player에게 GameAbility, Enhanced Input Mapping Context, Pawn Data 지정)<br>
  코드를 직접 수정하지 않아도 Experience 전환으로<br>
  다양한 '변경'을 유연하게 적용 가능해짐<br>

