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

예시적인 Build.cs<br>
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