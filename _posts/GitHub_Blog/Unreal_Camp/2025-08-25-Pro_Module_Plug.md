---
title: "언리얼의 구성 계층"
date : "2025-08-25 13:30:00 +0900"
last_modified_at: "2025-08-25T13:30:00"
categories:
  - Unreal
tags:
  - Module
  - Plugin
  - Project
---

## Unreal의 구성 요소

<img width="1225" height="535" alt="Image" src="https://github.com/user-attachments/assets/abc5d941-4211-4bf1-b465-15f1465e30ab" /><br>

언리얼은 '모듈', '플러그인', '프로젝트'라는<br>
계층적인 구성 요소를 가지며<br>
이들은 각각 다른 목적을 지니고 있다<br>

```
Project (MyRPG)
 ├─ Module (CoreGameplay)
 ├─ Module (UI)
 ├─ Plugin (MyDungeonGen)
 │    └─ Module (DungeonRuntime)
 │    └─ Module (DungeonEditor)
 └─ Plugin (Niagara)
```

### Module
UE4/5 의 빌드 단위이다<br>
(.CPP와 .H를 포함하는 단위)<br>

*.Bulid.cs 파일로 정의되며<br>
특정한 '기능적 집합'으로 묶어둔 것<br>
(렌더링, AI, Network 등)<br>

보통 게임을 만들게 되면 하나가 생긴다<br>

```
// Copyright Epic Games, Inc. All Rights Reserved.

#include "Samples.h"

#include"SampleLogChannels.h"
#include "Modules/ModuleManager.h"

class SamplesModule : public FDefaultGameModuleImpl
{
	// 모듈 : h + cpp 로 이루어진 최소한의 기능
	// 플러그인 : 다양한 실험적, 외부 기능 (모듈을 여러개 포함할 수 있음)
	// 프로젝트 : 모듈 + 플러그인 으로 이루어지는 전체 구조
public:
	virtual void StartupModule() override;
	virtual void ShutdownModule() override;
};

void SamplesModule::StartupModule()
{
	FDefaultGameModuleImpl::StartupModule();

	UE_LOG(LogSample, Warning, TEXT("StartUpModule!"));
}

void SamplesModule::ShutdownModule()
{
	FDefaultGameModuleImpl::ShutdownModule();
}

// IMPLEMENT_PRIMARY_GAME_MODULE 은 해당 엔진 내에서 하나만 존재해야 함 -> 핵심이 되는 모듈에게만 사용할것
// 모듈마다 이러한 IMPLEMENT 시리즈를 하나씩은 설정해 줘야 함
IMPLEMENT_PRIMARY_GAME_MODULE(SamplesModule, Samples, "Samples" );
```

- IMPLEMENT_PRIMARY_GAME_MODULE<br>
  : 해당 게임의 메인 모듈을 등록하는 매크로<br>
  (프로그램 진입점을 정하듯, 해당 게임 프로젝트의 메인이 될 모듈 설정)<br>
  (메인 엔트리 지점을 설정 - C++ 의 int main()과 비슷하다)<br>

```
언리얼의 간략한 실행 흐름

[OS] 
   ↓
 WinMain()   ← (플랫폼별 진입점, 유저가 안 건드려도 됨)
   ↓
 GuardedMain()
   ↓
 LaunchEngineLoop("Samples")
   ↓
 Load Module: "Samples"
   ↓
 ┌─────────────────────────────┐
 │ SamplesModule (Primary)     │
 │   ├─ StartupModule()        │
 │   └─ ShutdownModule()       │
 └─────────────────────────────┘
   ↓
 GameInstance / World / GameMode 초기화
   ↓
 [게임 루프 시작!]

```

- 보통 하나의 프로젝트(.uproject)에서 메인 모듈(Primary)은 하나<br>
  (즉 위 매크로를 2번 설정하면 빌드 중에 에러가 발생한다)<br>
  (컴파일 에러가 아니므로 찾기 힘들어진다...)<br>
  (반드시 하나는 설정해야 함)<br>

- 엔진이 '게임 루프'를 돌리기 전에<br>
  '내 프로젝트'가 무엇인지를 알아야 하기에 설정해야 한다<br>
  (위의 모듈 클래스가 딱히 하는것은 없어보이지만... 꼭 있어야 함)<br>
  (기본적으로 프로젝트 만들때, 알아서 설정해줌)<br>
  ('커스텀'이 가능한 정도만 알고 있자)<br>

- 그 외의 모듈 등록은<br>
  IMPLEMENT_MODULE 을 사용하여 일반 모듈을 등록<br>

Build.cs<br>
(해당 프로젝트에 사용할 모듈 이름들을 추가)<br>

```
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
            "GameplayTasks",
            "GameplayAbilities",
            // Game Features
            "ModularGameplay",
            "GameFeatures",
            "ModularGameplayActors",
			//Input
            "InputCore",
            "EnhancedInput",
			// CommonUser
			"CommonUser",
			// CommonGame
			"CommonGame",
			// CommonUI
			"CommonUI",
			// UMG
			"UMG",
			// UIExtension
			"UIExtension",
			// Slate
			"Slate",
			"SlateCore",
        });

		PrivateDependencyModuleNames.AddRange(new string[] {  });

		// Uncomment if you are using Slate UI
		// PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
		
		// Uncomment if you are using online features
		// PrivateDependencyModuleNames.Add("OnlineSubsystem");

		// To include OnlineSubsystemSteam, add it to the plugins section in your uproject file with the Enabled attribute set to true
	}
}
```

- 프로젝트나 플러그인은 '여러 모듈'을 둘 수 있음<br>
- 각각의 모듈마다 public, private 폴더를 통해<br>
  외부 접근 코드와 내부 코드를 분류하는 방식<br>
- 독립적인 빌드/링킹의 단위이기에 '불필요'한 모듈은<br>
  빌드하지 않음<br>
  (Build.cs는 '해당 프로젝트'에서 필요한 '모듈'만 빌드하면<br>
  된다)<br>

### Plugin
하나 이상의 모듈을 담는 '기능적 패키지'<br>
프로젝트 외부에서도 쉽게 재사용이 가능하다<br>
(ex : 마켓 플레이스의 플러그인 기능들)<br>

- *.uplugin 파일의 존재(플러그인 메타 데이터)<br>
- Plugins 폴더에 넣으면 프로젝트 단위에서 활성화/비활성화 가능<br>
  (플러그인을 만드는 경우, 해당 프로젝트의 Plugins 폴더 내에 생성된다<br>
  그리고 에디터에서 활성화/비활성화가 가능해짐)<br>

- Niagara, Paper2D 등의 플러그인이 대표적<br>

### Project
게임 전체를 포함하는 최상위의 단위<br>
*.uproject 파일이 메타데이터의 역할<br>

컨텐츠, 설정, 소스 코드와 플러그인을 모두 포함<br>

- 실행 가능한 게임 = 프로젝트 라 인식<br>
- 여러 모듈과 플러그인을 포함<br>
- 최종적인 '빌드 타겟'도 프로젝트 단위로 결정됨<br>


정리용 계층 표
```
언리얼 엔진 플랫폼
└── Project (최상위 단위, 게임/앱)
     ├─ Module (코드 묶음)
     ├─ Plugin (재사용 가능한 확장 기능)
     │    └─ Module (여러 개 가능)
     └─ Content/Config (에셋, 설정 등)
```