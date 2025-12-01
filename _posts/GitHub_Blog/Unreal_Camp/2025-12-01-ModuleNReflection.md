---
title: "김하연 튜터님 강의 - '모듈 시스템과 리플렉션 아키텍처'"
date : "2025-12-01 12:00:00 +0900"
last_modified_at: "2025-12-01T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Module
  - Reflection
  - UBT
---

# 모듈 시스템과 리플렉션 아키텍처에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. Unreal 모듈 시스템의 전체 구조 💨

## 1.1. 엔진 자체가 어떻게 모듈 단위로 나뉘는가

### 언리얼 엔진 소스 코드(Engine/Source)

**① Runtime 폴더**<br>
`게임 실행` 시 필요한 코드가 위치<br>

| 모듈 | 설명 |
| --- | --- |
| Core | 문자열 처리, 컨테이너(TArray, TMap), 로깅, 수학 함수 |
| CoreUObject | UObject 시스템, 리플렉션, 가비지 컬렉션, 직렬화 |
| Engine | 액터, 컴포넌트, 월드, 레벨 등 게임 프레임워크 핵심 |
| Renderer | 렌더링 시스템 (Nanite, Lumen 포함) |
| PhysicsCore | 물리 엔진 인터페이스 |
| AudioMixer | 오디오 시스템 |
| Networking | 네트워크 복제, RPC |

**② Editor 폴더**<br>
에디터 전용 코드가 위치하며, 패키징 시 제외됨<br>
- 게임에는 필요 없음!<br>

| 모듈 | 설명 |
| --- | --- |
| UnrealEd | 에디터 핵심 프레임워크 |
| LevelEditor | 레벨 편집 기능 |
| MaterialEditor | 머티리얼 편집 |
| BlueprintGraph | 블루프린트 노드 그래프 시스템 |
| Kismet | 블루프린트 VM과 컴파일러 |
| PropertyEditor | 디테일 패널 |

**③ Developer 폴더**<br>
개발 중에만 필요한 코드(`프로파일링` 도구, `자동화 테스트` 등). Shipping 빌드에서 제외<br>

**④ ThirdParty 폴더**<br>
외부 라이브러리(zlib, libPNG, FreeType, OpenSSL, Vulkan, PhysX 등)<br>

**⑤ Programs 폴더**<br>
`빌드 및 실행 도구` 프로그램이 위치<br>

| 도구 | 설명 |
| --- | --- |
| UnrealBuildTool (UBT) | 빌드 시스템 |
| UnrealHeaderTool (UHT) | 리플렉션 코드 생성기 |
| UnrealLightmass | 라이트맵 베이킹 |
| UnrealPak | PAK 파일 생성 |
| ShaderCompileWorker | 셰이더 컴파일 워커 |

### Module vs Plugin vs Project의 차이

| 구분 | 정의 | 특징 |
| --- | --- | --- |
| **Module** | 코드의 기본 빌딩 블록 | `Build.cs` 파일 1개, Public/Private 폴더, **하나의 DLL**로 컴파일 |
| **Plugin** | 하나 이상의 모듈을 포함하는 컨테이너 | `.uplugin` 파일로 정의, 활성화/비활성화 가능 |
| **Project** | 게임 자체 | `.uproject` 파일로 정의, Primary Game Module 포함 |

정리?<br>
- 모듈 : 라이브러리 1개<br>
- 플러그인 : 패키지 (여러 lib + 설정 / 문서)<br>
- 프로젝트 : 실제 사용하는 프로젝트<br>

## 1.2. 모듈이 왜 필요할까?

### 1. 컴파일 시간 단축

사실 가장 큰 이유<br>
매우 거대한 프로젝트를 처음부터 끝까지 컴파일 하는 것은<br>
몇 시간씩 걸림<br>

- 프로젝트가 클 수록 효과적인 장점<br>

### 2. 기능 단위의 캡슐화

모듈의 폴더 구조를 통한 접근 제어<br>

| 폴더 | 접근 범위 |
| --- | --- |
| Public | 외부 모듈에서 include 가능 |
| Private | 해당 모듈 내부에서만 include 가능 |

```
MyAIModule/
├── Public/
│   ├── MyAIController.h      // 외부에서 사용할 클래스
│   └── AITypes.h             // 공개 타입 정의
└── Private/
    ├── BehaviorTree/         // 내부 구현
    │   ├── BTNode_Attack.cpp
    │   └── BTNode_Attack.h
    ├── Utility/              // 내부 유틸리티
    │   └── AIMathUtils.h
    └── MyAIController.cpp
```

**장점**<br>

- 내부 구현 변경 시 외부 코드 수정 불필요<br>
  (복잡한 구현은 private 쪽에 숨기기)<br>
  (모듈용 코드만 노출되었기에 내부 구현을 마음대로 바꾸어도 됨)<br>

- 컴파일 시간 감소 (Private 헤더 수정의 파급효과 최소화)<br>

- 실수로 내부 함수 사용 방지<br>
  (include가 안되므로, 다른 팀원이 임의로 가져다 쓰는 상황을 예방)<br>

### 3. 블루프린트와 리플렉션의 연결

UnrealHeaderTool(UHT)은 모듈 단위로 동작<br>

Build.cs에 의존성이 설정되지 않으면 UHT가 해당 헤더를 인식하지 못함.<br>

> "UCLASS가 인식되지 않습니다" 에러의 흔한 원인: `모듈 의존성 설정` 누락
> 

```cpp
UCLASS(BlueprintType)
class MYGAME_API AMyCharacter : public ACharacter
{
    GENERATED_BODY()
    
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Health;
    
    UFUNCTION(BlueprintCallable)
    void TakeDamage(float Amount);
};
```

## 1.3 모듈 타입

### Runtime Module과 Editor Module

모듈 타입을 설정하여<br>
실제 게임에 사용할 녀석들만 Module 화 할 수 있음<br>

| 타입 | 용도 | 패키징 포함 |
| --- | --- | --- |
| **Runtime** | 게임 실행 코드 | ✅ |
| **Editor** | 에디터 전용 코드 | ❌ |
| **Developer** | 개발 빌드 전용 (프로파일링 등) | Development만 |

```json
// Runtime 모듈 선언 예시
{
  "Name": "MyGame",
  "Type": "Runtime",
  "LoadingPhase": "Default"
}
```

**규칙**: Editor → Runtime 참조 가능, Runtime → Editor 참조 불가<br>

Runtime에서 에디터 기능 사용 시<br>

```cpp
#if WITH_EDITOR
// 에디터 전용 코드
#endif
```

### Loading Phase의 이해

모듈 로딩 시점을 제어하는 옵션<br>

| Phase | 용도 |
| --- | --- |
| EarliestPossible | 가장 빠른 시점 (저수준 모듈) |
| PostConfigInit | 설정 시스템 초기화 직후 |
| PreDefault | Default보다 먼저 (커스텀 애니메이션 노드 등) |
| **Default** | 일반적인 게임플레이 모듈 (기본값) |
| PostEngineInit | 엔진 완전 초기화 후 |
| None | 자동 로드 안 함 (수동 로드 필요) |

- 로딩 시점을 통해 설정하기<br>
  다만, 기본적으론 Default 사용하는 편<br>
  (엔진 내부의 초기화 설정 등이 아니라면)<br>

### Monolithic vs Modular 빌드

| 방식 | 특징 | 사용 시점 |
| --- | --- | --- |
| **Modular** | 모듈별 DLL 생성, Hot Reload 가능 | 에디터 개발 (기본값) |
| **Monolithic** | 단일 실행 파일 | Shipping/콘솔 빌드 |

**MODULE_API 매크로**<br>

다른 모듈에서 접근하는 클래스에 반드시 필요<br>
(안 붙여도 Editor에서 잘 돌아가지만, 패키징 시 링킹 에러 발생)<br>
(해당 모듈안에서만 사용하는 것이 아니라면 붙이기!)<br>

```cpp
UCLASS()
class MYGAME_API AMyCharacter : public ACharacter
{
    // ...
};
```

- Modular 빌드: `__declspec(dllexport/dllimport)` 역할<br>
- Monolithic 빌드: 빈 매크로<br>

## 1.4 Plugin 구조 심화

### .uplugin 파일 구조

```json
{
  "FileVersion": 3,
  "Version": 1,
  "VersionName": "1.0.0",
  "FriendlyName": "My Awesome Plugin",
  "Description": "플러그인 설명",
  "Category": "Gameplay",
  "CanContainContent": true,
  "Modules": [
    {
      "Name": "MyPlugin",
      "Type": "Runtime",
      "LoadingPhase": "Default"
    },
    {
      "Name": "MyPluginEditor",
      "Type": "Editor",
      "LoadingPhase": "Default"
    }
  ],
  "Plugins": [
    {
      "Name": "EnhancedInput",
      "Enabled": true
    }
  ]
}
```

**주요 필드**

| 필드 | 설명 |
| --- | --- |
| CanContainContent | true면 Content 폴더(에셋) 포함 가능 |
| Modules | 포함된 모듈 목록 |
| Plugins | 의존하는 다른 플러그인 (로딩 순서 보장) |

### "모듈당 Build.cs 파일 1개" 규칙

**폴더 구조 예시**

```
MyPlugin/
├── MyPlugin.uplugin
├── Content/
└── Source/
    ├── MyPlugin/                 // Runtime 모듈
    │   ├── MyPlugin.Build.cs
    │   ├── Private/
    │   └── Public/
    └── MyPluginEditor/           // Editor 모듈
        ├── MyPluginEditor.Build.cs
        ├── Private/
        └── Public/
```

- 모듈마다 Build.cs 설정<br>

**필수 규칙**

1. Build.cs 파일명 = 모듈명 (MyPlugin → MyPlugin.Build.cs)<br>
2. Build.cs 없으면 UBT가 모듈로 인식 안 함<br>
3. 최소 1개의 .cpp 파일 필요<br>
4. IMPLEMENT_MODULE 매크로로 모듈 등록 필수<br>

```cpp
// MyPluginModule.cpp
IMPLEMENT_MODULE(FMyPluginModule, MyPlugin)
```

### Plugin 위치와 연결 방식

| 위치 | 범위 |
| --- | --- |
| Engine/Plugins | `모든 프로젝트`에서 사용 가능 (주로 엔진 내장) |
| 프로젝트/Plugins | `해당 프로젝트` 전용 |
| Engine/Plugins/Marketplace | 에픽 런처로 설치한 플러그인 |

플러그인 활성화 시 .uproject 파일의 Plugins 배열에 기록된다.<br>

## 핵심 정리

1. **모듈**은 코드의 기본 단위이며, `하나의 Build.cs와 Public/Private` 폴더를 갖는다<br>
2. **Runtime/Editor/Developer** 타입으로 패키징 포함 여부가 결정된다<br>
3. **LoadingPhase**로 모듈 로딩 시점을 제어한다<br>
4. **MODULE_API 매크로**는 외부 공개 클래스에 필수다<br>
5. **플러그인**은 여러 모듈의 컨테이너이며, `.uplugin`으로 정의한다<br>

# 2. Build.cs와 빌드 파이프라인 (UBT) 이해하기 🤪

## 2.1. Build.cs의 실제 의미

### ModuleRules 클래스 해부

Build.cs는 `C# 코드`로 작성되며,<br>
ModuleRules 클래스를 상속받아 모듈의 빌드 설정을 정의<br>
(UBT가 C#으로 구현되어있음)<br>

```csharp
using UnrealBuildTool;
using System.IO;

public class MyGame : ModuleRules
{
    public MyGame(ReadOnlyTargetRules Target) : base(Target)
    {
        // PCH 사용 방식
        // 자주 쓰는 Header의 미리 컴파일 하는 용도
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

        // Include What You Use 강제
        // true 시, 불필요한 include 사용시 경고 설정
        bEnforceIWYU = true;

        // Public 의존성
        PublicDependencyModuleNames.AddRange(new string[]
        {
            "Core",
            "CoreUObject",
            "Engine"
        });

        // Private 의존성
        PrivateDependencyModuleNames.AddRange(new string[]
        {
            "Slate",
            "SlateCore"
        });

        // 에디터 전용 의존성
        if (Target.bBuildEditor)
        {
            PrivateDependencyModuleNames.Add("UnrealEd");
        }
    }
}
```

**주요 설정 옵션**

| 설정 | 설명 |
| --- | --- |
| `PCHUsage` | Precompiled Header 사용 방식. `UseExplicitOrSharedPCHs` 권장 |
| `bEnforceIWYU` | "Include What You Use" 원칙 강제. 불필요한 include 경고 |
| `CppStandard` | C++ 표준 버전 지정 |

### Public vs Private Dependency의 진짜 차이

- `의존성`에 관한 차이<br>
  - 다른 모듈이 해당 모듈을 사용할 때<br>
    해당 모듈의 public 의존성 역시 필요로 하게 됨<br>
    - 즉, 필요로 하는 기능이라면 public에 넣어<br>
      다른 모듈들도 사용하도록 함<br>
  - 반대로 해당 모듈 자체적으로만 사용한다면<br>
    private에 넣어 굳이 '사용하지' 않는 모듈을<br>
    넣지 않도록 함<br>

**PublicDependencyModuleNames**

이 모듈의 Public 헤더(.h)에서 사용하는 모듈을 등록<br>

**핵심 특성**: Public 의존성은 전파(propagate)됨.<br>

```
ModuleA (PublicDependency: Engine, Core)
    ↑
ModuleB (PublicDependency: ModuleA)
```

→ ModuleB는 자동으로 Engine, Core 의존성도 받음<br>

**PrivateDependencyModuleNames**

이 모듈의 Private 구현(.cpp)에서만 사용하는 모듈을 등록<br>

**핵심 특성**: Private 의존성은 **전파되지 않음.**<br>

```
ModuleA (PrivateDependency: Slate)
    ↑
ModuleB (PublicDependency: ModuleA)
```

→ ModuleB는 Slate에 접근 불가. 직접 의존성 추가 필요<br>

**실제 적용 예시**<br>

```cpp
// MyCharacter.h (Public 폴더)
#include "GameFramework/Character.h"  // Engine 모듈
#include "AbilitySystemInterface.h"    // GameplayAbilities 모듈
```

```cpp
// MyCharacter.cpp (Private 폴더)
#include "Components/WidgetComponent.h" // UMG 모듈
#include "NavigationSystem.h"           // NavigationSystem 모듈
```

```csharp
// Build.cs
PublicDependencyModuleNames.AddRange(new string[]
{
    "Core", "CoreUObject", "Engine",
    "GameplayAbilities"  // 헤더에서 사용
});

PrivateDependencyModuleNames.AddRange(new string[]
{
    "UMG",               // cpp에서만 사용
    "NavigationSystem"
});
```

- public/ private 처럼<br>
  폴더에서 사용하는 걸로 구분할 수 있음<br>

---

### Include Path 설정

기본적으로 Public/Private 폴더는 자동으로 include path에 추가된다.<br>
하위 폴더는 필요 시 직접 추가한다.<br>

```csharp
PublicIncludePaths.AddRange(new string[]
{
    Path.Combine(ModuleDirectory, "Public/Interfaces")
});

PrivateIncludePaths.AddRange(new string[]
{
    Path.Combine(ModuleDirectory, "Private/Utils")
});
```

> 5.5 이상에서 bAddDefaultIncludePaths가 true(기본값)이면 Public, Private, Classes 폴더가 자동 추가됨ㅇㅇ
> 

### UObject 사용 시 필수 모듈들

**UCLASS 사용 시 필수**<br>

| 모듈 | 포함 내용 |
| --- | --- |
| `Core` | FString, TArray, TMap, FMath, UE_LOG |
| `CoreUObject` | UObject, UCLASS, UPROPERTY, UFUNCTION, GC, 직렬화 |

**AActor/UActorComponent 사용 시 추가**<br>

| 모듈 | 포함 내용 |
| --- | --- |
| `Engine` | AActor, UActorComponent, UWorld, ACharacter |

**자주 사용하는 모듈 목록**

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "Core",
    "CoreUObject",
    "Engine",
    "InputCore",           // 기본 입력
    "EnhancedInput",       // 향상된 입력 (5.1+)
    "GameplayAbilities",   // GAS
    "GameplayTags",        // 게임플레이 태그
    "AIModule",            // AI 시스템
    "NavigationSystem",    // 내비게이션
    "UMG",                 // UI
    "Slate", "SlateCore"   // 저수준 UI
});
```

## 2.2 UBT가 Build.cs를 해석하는 방식

### 모듈 그래프 생성

UBT는 빌드 시작 시 다음 과정을 수행<br>

1. 모든 .uproject, .uplugin 파일 스캔 → 모듈 목록 파악<br>
2. 각 모듈의 Build.cs 파일 읽기 → 의존성 파악<br>
3. `방향 그래프`(Directed Graph) 구성<br>

```
MyGame → Engine → Core
MyGame → CoreUObject → Core
MyGame → GameplayAbilities → Engine → Core
```

**순환 의존성 체크**<br>

```
ModuleA → ModuleB → ModuleC → ModuleA  // 순환 발생!
```

→ "Circular dependency detected between modules" 에러 발생<br>

**해결 방법**

1. 공통 부분을 별도 모듈로 분리<br>
2. 인터페이스 모듈을 만들어 의존성 역전<br>
3. 구조 재설계<br>

### 빌드 순서 결정

`위상 정렬`(Topological Sort)을<br>
통해 빌드 순서를 결정한다.<br>

```
빌드 순서:
1. Core (의존성 없음)
2. CoreUObject (Core에 의존)
3. Engine (Core, CoreUObject에 의존)
4. GameplayAbilities (Engine에 의존)
5. MyGame (위의 모든 것에 의존)
```

### Include Path 확정

각 모듈의 최종 include path는 직접 의존성 + 간접 의존성의 Public include path를 모두 포함<br>

```
MyGame의 include path:
- MyGame/Public, MyGame/Private
- Engine/Public (직접 의존)
- Core/Public (Engine을 통한 간접 의존)
- CoreUObject/Public (직접 의존)
- GameplayAbilities/Public (직접 의존)
```

- 버전 업에 따라 패키징 오류가 발생하는 등의 상황은<br>
  '의존성'이 달라져서 그럴 가능성이 존재함<br>

### UHT 호출 대상 판단

UHT(UnrealHeaderTool) 처리가 필요한 조건<br>

- 헤더 파일에 `UCLASS`, `USTRUCT`, `UENUM` 등의 매크로 존재<br>
- 해당 헤더에 `#include "XXX.generated.h"` 존재<br>

> 💡 순수 C++ 유틸리티 모듈(UObject 미사용)은 UHT 처리 대상이 아니며, 빌드 시간이 빠름
> 

## 2.3 Build.cs와 리플렉션의 직접적인 관계

### 의존성이 없으면 UHT가 스캔하지 않는다

**핵심**: Build.cs에 의존성이 없으면 UHT가 해당 모듈의 타입 정보를 알 수 없음.<br>

```cpp
// MyAbility.h
#include "Abilities/GameplayAbility.h"  // GameplayAbilities 모듈

UCLASS()
class UMyAbility : public UGameplayAbility  // 부모 클래스 인식 불가!
{
    GENERATED_BODY()
};
```

- UHT가 GameAbility 인식이 불가능하기에 터짐!<br>
  (컴파일 에러)<br>
- 여러번 부딪혀본 그 문제...<br>

**해결책**<br>

- Build.cs에 의존성 추가<br>

```csharp
PublicDependencyModuleNames.Add("GameplayAbilities");
```

- .uproject에 플러그인 활성화<br>

```json
{
  "Plugins": [
    { "Name": "GameplayAbilities", "Enabled": true }
  ]
}
```

### "UCLASS가 인식되지 않습니다" 에러의 진짜 원인

| 원인 | 해결책 |
| --- | --- |
| Build.cs에 모듈 의존성 없음 (가장 흔함) | 필요한 모듈 의존성 추가 |
| `GENERATED_BODY()` 매크로 누락 | 클래스 본문에 매크로 추가 |
| `.generated.h` include 누락 | `#include "MyClass.generated.h"` 추가 |
| 헤더 include 순서 문제 | `.generated.h`를 **항상 마지막**에 배치 |

- Build.cs의 의존성 추가<br>
- Generate body 를 지우거나 없는 경우 등<br>
- generate.h 가 빠진다던가<br>
  그 뒤에 include 한 경우<br>

**올바른 헤더 구조**

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyClass.generated.h"  // 반드시 마지막!

UCLASS()
class MYGAME_API AMyClass : public AActor
{
    GENERATED_BODY()
};
```

### 모듈이 UHT 대상이 되려면

**세 가지 조건**<br>

1. 최소 하나의 헤더에 `UCLASS`/`USTRUCT`/`UENUM` 존재<br>
2. 해당 헤더에 `#include "XXX.generated.h"` 존재<br>
3. Build.cs에 필요한 모든 의존성 정의<br>

**순수 C++ 모듈의 장점**<br>

UObject 없이 일반 C++ 클래스만 사용하는 모듈:<br>

- generated.h 미생성<br>
- UHT 파싱 시간 없음<br>
- 빌드 시간 단축<br>
- 모듈 의존성 단순화<br>

→ `수학` 라이브러리, `알고리즘` 라이브러리, `서드파티 래퍼` 등에 적합<br>

## 핵심 정리

1. **Build.cs**는 C# 코드로 모듈의 빌드 설정을 정의한다<br>
2. **Public 의존성**은 전파되고, **Private 의존성**은 전파되지 않는다<br>
3. **UBT**는 `모듈 의존성 그래프`를 생성하고 `빌드 순서를 결정`한다<br>
4. **Build.cs에 의존성이 없으면 UHT가 해당 모듈의 타입을 인식하지 못한다**<br>
5. UCLASS 에러의 90%는 의존성 누락, GENERATED_BODY 누락, generated.h 문제 중 하나다<br>

# 3. UBT → UHT 전체 파이프라인 탐색 ⛏️

## 3.1. 빌드 흐름 전체

### 전체 파이프라인 개요

Visual Studio에서 빌드 버튼을 누르면 다음 단계가 순차적으로 실행<br>

```
─────────────────────────────────────────────────────────────
  1. UBT (UnrealBuildTool) 실행                              
     └─ C#으로 작성된 빌드 오케스트레이터                     
─────────────────────────────────────────────────────────────
  2. Build.cs / Target.cs 파싱                               
     └─ 모든 모듈의 빌드 설정 수집                            
─────────────────────────────────────────────────────────────
  3. 모듈 의존성 그래프 생성                                  
     └─ 빌드 순서 결정, 순환 의존성 체크                       
─────────────────────────────────────────────────────────────
  4. UHT (UnrealHeaderTool) 실행                             
     ├─ 모든 헤더 파일 스캔                                  
     ├─ UCLASS/UPROPERTY 등 매크로 파싱                       
     └─ .generated.h / .gen.cpp 파일 생성                    
─────────────────────────────────────────────────────────────
  5. 실제 C++ 컴파일                                         
     ├─ Windows: MSVC (cl.exe)                               
     ├─ Mac: Clang                                           
     └─ 5.5: UBA로 분산 컴파일 가능                           
─────────────────────────────────────────────────────────────
  6. 링킹                                                    
     ├─ Modular: 각 모듈별 DLL 생성                          
     └─ Monolithic: 하나의 실행 파일 생성                     
─────────────────────────────────────────────────────────────
```

**5.5 이상의 신규 기능**<br>

| 기능 | 설명 |
| --- | --- |
| **UBA** (Unreal Build Accelerator) | 빌드 분산 처리. UHT와 컴파일 단계에서 사용 가능 |
| **Horde** | CI/CD 시스템. Job Pool Coordinator 역할 |
| **Zen Loader** | 에셋 로딩 최적화. 에디터/맵/PIE 시작 시간 단축 |

### 각 단계의 실패 시점 이해하기

| 단계 | 에러 예시 | 원인 |
| --- | --- | --- |
| **1. UBT 시작** | `Could not find NetFxSDK install dir` | SDK 설치, 환경 변수 문제 |
| **2. Build.cs 파싱** | `error CS1002: ; expected` | C# 문법 에러 |
| **3. 모듈 그래프** | `Circular dependency detected` | 순환 의존성, 없는 모듈 참조 |
| **4. UHT** | `Unable to find class 'UGameplayAbility'` | 리플렉션 관련, 의존성 누락 |
| **5. C++ 컴파일** | `error C2065: undeclared identifier` | C++ 문법 오류, 타입 불일치 |
| **6. 링킹** | `error LNK2001: unresolved external symbol` | MODULE_API 누락, 구현 없음 |



**링크 에러의 흔한 원인**

1. MODULE_API 매크로 누락<br>
2. 함수 선언은 있는데 구현이 없음<br>
3. Build.cs에 라이브러리 의존성 누락<br>
4. 에디터 빌드와 패키징 빌드의 차이<br>

## 3.2. Generated.h / .get.cpp의 진짜 의미

### 이 파일들은 어디에 생기나

UHT가 생성하는 파일들은 **Intermediate 폴더**에 위치<br>

**에디터 빌드**

```
프로젝트/
└── Intermediate/
    └── Build/
        └── Win64/
            └── UnrealEditor/
                ├── Inc/
                │   └── MyGame/
                │       ├── UHT/
                │       │   ├── MyCharacter.generated.h
                │       │   └── MyGameMode.generated.h
                │       └── MyGame.init.gen.cpp
                └── Development/
                    └── MyGame/
```

**패키징(Shipping) 빌드**

```
프로젝트/
└── Intermediate/
    └── Build/
        └── Win64/
            └── MyGame/          // UnrealEditor가 아님
                └── Inc/
                    └── MyGame/
```

> 빌드 캐시 문제 발생 시` Intermediate 폴더 삭제 후 재빌드`하면 해결되는 경우가 많은 이유임.
> 

### .generated.h 파일의 내용

```cpp
// MyCharacter.generated.h

// 중복 include 방지 가드
#ifdef MYGAME_MyCharacter_generated_h
#error "MyCharacter.generated.h already included"
#endif
#define MYGAME_MyCharacter_generated_h

#define FID_MyGame_Source_MyGame_Public_MyCharacter_h_15_GENERATED_BODY \
PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
    // StaticClass() - 이 클래스의 UClass 반환
    MYGAME_API static UClass* GetPrivateStaticClass(); \
    // GetClass() 오버라이드
    MYGAME_API virtual UClass* GetClass() const override; \
private: \
    // 네이티브 함수 등록
    static void StaticRegisterNativesAMyCharacter(); \
    friend struct Z_Construct_UClass_AMyCharacter_Statics; \
public: \
    // 클래스 선언
    DECLARE_CLASS(AMyCharacter, ACharacter, COMPILED_IN_FLAGS(0), \
                  CASTCLASS_None, TEXT("/Script/MyGame"), MYGAME_API) \
    // 직렬화 지원
    DECLARE_SERIALIZER(AMyCharacter) \
    // 네트워크 복제 지원
    enum {WithNetSerializer = true}; \
PRAGMA_ENABLE_DEPRECATION_WARNINGS
```

**generated.h가 제공하는 기능**<br>

| 기능 | 설명 |
| --- | --- |
| `StaticClass()` | `AMyCharacter::StaticClass()`로 UClass 포인터 획득 |
| `GetClass()` | 인스턴스에서 클래스 정보 조회 |
| 네이티브 함수 등록 | UFUNCTION들을 리플렉션 시스템에 등록 |
| 직렬화 지원 | 에셋 저장/로드, 네트워크 복제 |
| 클래스 메타데이터 | 클래스 플래그, 이름, 경로 |

### .gen.cpp 파일의 역할

.gen.cpp에는 리플렉션 정보의 **실제 구현**이 포함<br>

```cpp
// MyGame.init.gen.cpp

#include "UObject/GeneratedCppIncludes.h"

// 링커가 이 파일을 포함하도록 하는 빈 함수
void EmptyLinkFunctionForGeneratedCodeMyGame() {}

// UClass 구축 함수
UClass* Z_Construct_UClass_AMyCharacter()
{
    static UClass* OuterClass = nullptr;
    if (!OuterClass)
    {
        // 부모 클래스 UClass 먼저 구축
        UClass* SuperClass = ACharacter::StaticClass();

        // 이 클래스의 UClass 생성
        OuterClass = AMyCharacter::StaticClass();

        // UPROPERTY로 선언된 프로퍼티 등록
        // UFUNCTION으로 선언된 함수 등록
        // Category, ToolTip 등 메타데이터 설정
    }
    return OuterClass;
}

// 네이티브 함수 등록
void AMyCharacter::StaticRegisterNativesAMyCharacter()
{
    UClass* Class = AMyCharacter::StaticClass();
    static const FNameNativePtrPair Funcs[] = {
        { "TakeDamage", &AMyCharacter::execTakeDamage },
    };
    FNativeFunctionRegistrar::RegisterFunctions(Class, Funcs, UE_ARRAY_COUNT(Funcs));
}

// Thunk 함수 - 블루프린트에서 호출 시 사용
DEFINE_FUNCTION(AMyCharacter::execTakeDamage)
{
    P_GET_PROPERTY(FFloatProperty, Z_Param_Amount);
    P_FINISH;
    P_NATIVE_BEGIN;
    P_THIS->TakeDamage(Z_Param_Amount);
    P_NATIVE_END;
}

```

**gen.cpp 덕분에 가능한 것들**<br>

1. 클래스 이름으로 클래스 찾기: `FindObject<UClass>(nullptr, TEXT("/Script/MyGame.MyCharacter"))`<br>
2. 프로퍼티 이름으로 값 설정: 디테일 패널, 블루프린트 핀<br>
3. 함수 이름으로 함수 호출: 블루프린트 노드, 콘솔 명령어<br>
4. 네트워크 복제: `어떤 프로퍼티를 복제`할지 파악<br>
5. 가비지 컬렉션: `어떤 UObject* 포인터를 추적`할지 파악<br>

## 3.3. 리플렉션 가능/불가능 파일 종류

### UHT가 처리하는 것들

**클래스/구조체/열거형**

```cpp
UCLASS()      // UObject 상속 클래스
USTRUCT()     // 값 타입 구조체
UENUM()       // 열거형
UINTERFACE()  // 인터페이스
```

**멤버**

```cpp
UPROPERTY()   // 프로퍼티 (변수)
UFUNCTION()   // 함수
UPARAM()      // 함수 파라미터 추가 정보
UMETA()       // UENUM 값에 메타데이터 추가
```

**델리게이트**

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE()
DECLARE_DYNAMIC_DELEGATE()
UDELEGATE()   // 델리게이트에 추가 정보
```

### UHT가 처리하지 않는 것들

| 종류 | 예시 | 비고 |
| --- | --- | --- |
| 일반 C++ 클래스 | `class FMyUtility { };` | UObject 상속 안 함 |
| 일반 함수/변수 | `void MyFunc();` `int32 Counter;` | 매크로 없음 |
| 템플릿 | `template<typename T> class TMyTemplate { };` | UHT 처리 불가 |
| constexpr | `constexpr int32 MaxHealth = 100;` | 리플렉션 대상 아님 |
| 조건부 컴파일 | `#if SOME_CONDITION UPROPERTY() ... #endif` | UHT 혼란 가능 |

> 예외: `WITH_EDITOR`, `WITH_EDITORONLY_DATA`는 UHT가 인식함
> 

> UHT는 **완전한 C++ 파서가 아니다**. 언리얼 `매크로 주변의 코드만 이해`하므로, 복잡한 전처리기 조건문은 파싱 실패를 유발할 수 있음.
> 

## 3.4. 왜 UE는 “정적 리플렉션”을 택했을까?

## 성능상의 이유

**C#/Java 런타임 리플렉션**<br>

```csharp
Type type = obj.GetType();
PropertyInfo prop = type.GetProperty("Health");
object value = prop.GetValue(obj);  // 런타임에 메타데이터 조회
```

- 메모리에 별도 `메타데이터 테이블` 유지 필요<br>
- 조회 시마다 문자열 비교 `오버헤드`<br>
- 컴파일 타임 `타입 안전성 검증 불가`<br>

**언리얼의 컴파일 타임 리플렉션**<br>

UHT가 빌드 시점에 메타데이터를<br>
C++ 코드로 생성하여 실행 파일에 내장한다.<br>

```cpp
// 생성된 코드 (개념적)
static FPropertyInfo HealthPropertyInfo = {
    TEXT("Health"),                      // 이름
    offsetof(AMyCharacter, Health),      // 오프셋 (컴파일 타임 결정)
    sizeof(float),                       // 크기
    EPropertyFlags::EditAnywhere | EPropertyFlags::BlueprintReadWrite
};
```

| 장점 | 설명 |
| --- | --- |
| 빠른 런타임 조회 | 해시 테이블 룩업 수준 |
| 외부 파일 불필요 | 메타데이터가 실행 파일에 내장 |
| 컴파일 타임 안전성 | 잘못된 리플렉션은 빌드 에러 |

### 에디터 연동

**디테일 패널 동작 원리**<br>

```cpp
UPROPERTY(EditAnywhere, Category="Stats", meta=(ClampMin=0, ClampMax=100))
float Health;
```

1. 선택된 액터의 UClass 획득<br>
2. 해당 UClass의 모든 FProperty 순회<br>
3. 각 프로퍼티의 플래그와 메타데이터 확인<br>
4. EditAnywhere → 표시, Category → 분류, ClampMin/Max → 슬라이더 범위<br>

**블루프린트 노드 동작 원리**<br>

```cpp
UFUNCTION(BlueprintCallable, Category="Combat",
          meta=(DisplayName="Apply Damage"))
void TakeDamage(float Amount);
```

1. UClass의 모든 UFunction 조회<br>
2. BlueprintCallable 플래그가 있는 함수만 노출<br>
3. 파라미터 타입 정보로 입력 핀 생성<br>
4. DisplayName으로 노드 이름 표시<br>

- 전부 리플렉션 데이터를 기반으로 이루어지는 것!<br>

### 가비지 컬렉션 통합

GC는 ***UPROPERTY로 마킹된 UObject* 포인터만** 추적<br>

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    UPROPERTY()
    UStaticMeshComponent* MeshComponent;  // ✅ GC가 추적

    UMaterial* CachedMaterial;            // ❌ GC가 모름 → dangling pointer 위험!
};
```

**규칙**: `UObject*`를 멤버로 가지면 **반드시 UPROPERTY()를 붙인다**.<br>

```cpp
// 올바른 방법들
UPROPERTY()
TObjectPtr<UMaterial> CachedMaterial;  // 5.0+ 권장

UPROPERTY(Transient)  // 직렬화 안 함, GC는 추적
UMaterial* TempMaterial;
```

### 블루프린트 시스템 통합

블루프린트는 리플렉션 시스템 위에서 동작<br>

- 블루프린트 클래스 = `UBlueprintGeneratedClass` (UClass 상속)<br>
- 블루프린트 변수 = `FProperty`<br>
- 블루프린트 함수 = `UFunction`<br>

```cpp
// C++에서 블루프린트 클래스 정보 조회
UClass* BPClass = LoadClass<AActor>(nullptr, TEXT("/Game/BP_MyActor.BP_MyActor_C"));
for (TFieldIterator<FProperty> It(BPClass); It; ++It)
{
    FProperty* Prop = *It;
    UE_LOG(LogTemp, Log, TEXT("Property: %s"), *Prop->GetName());
}
```

C++이든 블루프린트든 **동일한 리플렉션 API로 접근 가능**<br>

## 핵심 정리

1. **빌드 파이프라인**: UBT 실행 → Build.cs 파싱 → 의존성 그래프 → UHT → C++ 컴파일 → 링킹<br>
2. **generated.h**: StaticClass(), 네이티브 함수 등록, 직렬화 지원 코드 포함<br>
3. **gen.cpp**: 리플렉션 정보의 실제 구현. 프로퍼티/함수 등록, Thunk 함수 포함<br>
4. **UHT 대상**: UCLASS, USTRUCT, UENUM, UPROPERTY, UFUNCTION 등 언리얼 매크로<br>
5. **정적 리플렉션 장점**: 빠른 런타임 조회, 컴파일 타임 안전성, 에디터/GC/블루프린트 통합<br>
6. *UObject 멤버는 반드시 UPROPERTY()*붙여야 GC가 추적한다<br>

# 4. UE 리플렉션 시스템의 구조적 해부 🤣

## 4.1. UPROPERTY와 UFUNCTION의 진짜 동작 원리

## Property/FField 시스템

5.0에서 프로퍼티 시스템이 `UProperty`에서 `FProperty/FField` 시스템으로 변경<br>

**변경 이유**<br>

| 구분 | UProperty (구버전) | FProperty (현재) |
| --- | --- | --- |
| 상속 | UObject 상속 | 일반 C++ 객체 |
| GC | 추적 대상 | 추적 대상 아님 |
| 오버헤드 | 클래스당 프로퍼티 수만큼 GC 부담 | 가벼움 |

**FProperty 계층 구조**<br>

```
FField
└── FProperty
    ├── FNumericProperty
    │   ├── FByteProperty
    │   ├── FIntProperty
    │   ├── FFloatProperty
    │   ├── FDoubleProperty
    │   └── ...
    ├── FBoolProperty
    ├── FObjectPropertyBase
    │   ├── FObjectProperty        // UObject*
    │   ├── FWeakObjectProperty    // TWeakObjectPtr
    │   ├── FSoftObjectProperty    // TSoftObjectPtr
    │   └── FClassProperty         // TSubclassOf<>
    ├── FStructProperty            // USTRUCT
    ├── FStrProperty               // FString
    ├── FNameProperty              // FName
    ├── FTextProperty              // FText
    ├── FArrayProperty             // TArray
    ├── FMapProperty               // TMap
    ├── FSetProperty               // TSet
    ├── FDelegateProperty          // 델리게이트
    ├── FMulticastDelegateProperty // 멀티캐스트 델리게이트
    └── FInterfaceProperty         // TScriptInterface
```

### 메타데이터 저장 방식

`UPROPERTY/UFUNCTION`의 `지정자`(Specifiers)는 두 종류로 구분<br>

**1. 플래그(Flags) - 런타임에 필요한 정보**<br>

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Replicated)
float Health;
```

- 64비트 정수에 `비트`로 저장<br>
- 런타임에 접근 가능<br>

```cpp
FProperty* Prop = ...;
if (Prop->HasAnyPropertyFlags(CPF_BlueprintReadOnly))
{
    // BlueprintReadOnly 플래그 존재
}
```

**2. 메타데이터(Metadata) - 에디터 전용 정보**

```cpp
UPROPERTY(EditAnywhere, meta=(DisplayName="체력", ClampMin="0", ClampMax="100",
                              ToolTip="캐릭터의 현재 체력"))
float Health;
```

- `TMap<FName, FString>` 형태로 저장<br>
- **Shipping 빌드에서 제거됨**<br>

```cpp
// 에디터에서만 사용 가능
#if WITH_EDITOR
FString DisplayName;
if (Prop->HasMetaData(TEXT("DisplayName")))
{
    DisplayName = Prop->GetMetaData(TEXT("DisplayName"));
}
#endif
```

> **게임 로직에서 메타데이터를 읽으면 안 된다**. Shipping 빌드에서 존재하지 않음.
> 

### 주요 UPROPERTY Specifiers

**에디터 노출**<br>

| Specifier | 설명 |
| --- | --- |
| `EditAnywhere` | 블루프린트 CDO와 인스턴스 모두에서 편집 가능 |
| `EditDefaultsOnly` | 블루프린트 CDO에서만 편집 가능 |
| `EditInstanceOnly` | 레벨에 배치된 인스턴스에서만 편집 가능 |
| `VisibleAnywhere` | 보이지만 편집 불가 |
| `VisibleDefaultsOnly` | CDO에서만 보임, 편집 불가 |
| `VisibleInstanceOnly` | 인스턴스에서만 보임, 편집 불가 |

**블루프린트 노출**<br>

| Specifier | 설명 |
| --- | --- |
| `BlueprintReadOnly` | 블루프린트에서 읽기만 가능 |
| `BlueprintReadWrite` | 블루프린트에서 읽기/쓰기 가능 |
| `BlueprintGetter=FuncName` | 커스텀 getter 함수 지정 |
| `BlueprintSetter=FuncName` | 커스텀 setter 함수 지정 |

**네트워크 복제**<br>

| Specifier | 설명 |
| --- | --- |
| `Replicated` | 서버→클라이언트 복제 |
| `ReplicatedUsing=FuncName` | 복제 시 콜백 함수 호출 |
| `NotReplicated` | 명시적으로 복제 안 함 |

**직렬화**<br>

| Specifier | 설명 |
| --- | --- |
| `Transient` | 저장/로드 안 함 |
| `DuplicateTransient` | 복제 시 복사 안 함 |
| `SaveGame` | 세이브 게임에 포함 |
| `SkipSerialization` | 직렬화 건너뜀 |

**메모리**<br>

| Specifier | 설명 |
| --- | --- |
| `Instanced` | 컴포넌트처럼 소유 객체에 귀속 |
| `Export` | 복사 시 서브오브젝트로 내보냄 |
| `NoClear` | 에디터에서 None으로 설정 불가 |

## 4.2 리플렉션 시스템이 생성하는 Runtime Metadata

### 클래스 정보

모든 UCLASS는 런타임에 다음과 같은 메타데이터에 접근할 수 있음.<br>

```cpp
UClass* MyClass = AMyCharacter::StaticClass(); // 해당 클래스의 메타 데이터들을 가져올 수 있음

// 기본 정보
FString ClassName = MyClass->GetName();        // "MyCharacter"
FString PathName = MyClass->GetPathName();     // "/Script/MyGame.MyCharacter"
FString FullName = MyClass->GetFullName();     // "Class /Script/MyGame.MyCharacter"

// 상속 관계
UClass* SuperClass = MyClass->GetSuperClass(); // ACharacter의 UClass
bool bIsActor = MyClass->IsChildOf(AActor::StaticClass());         // true
bool bIsCharacter = MyClass->IsChildOf(ACharacter::StaticClass()); // true

// 인터페이스 확인
bool bHasInterface = MyClass->ImplementsInterface(UAbilitySystemInterface::StaticClass());

// CDO (Class Default Object) 접근
AMyCharacter* DefaultObj = GetMutableDefault<AMyCharacter>();
// 또는
AMyCharacter* DefaultObj = Cast<AMyCharacter>(MyClass->GetDefaultObject());

// 클래스 플래그
bool bIsAbstract = MyClass->HasAnyClassFlags(CLASS_Abstract);
bool bIsNative = MyClass->HasAnyClassFlags(CLASS_Native);
```

- 클래스 정보를 런타임 상에 이용 가능<br>

### 프로퍼티 순회

```cpp
// 이 클래스에서 선언된 프로퍼티만 (부모 제외)
for (TFieldIterator<FProperty> It(MyClass, EFieldIteratorFlags::ExcludeSuper); It; ++It)
{
    FProperty* Prop = *It;
    FString PropName = Prop->GetName();
    FString PropType = Prop->GetCPPType();

    UE_LOG(LogTemp, Log, TEXT("Property: %s (%s)"), *PropName, *PropType);
}

// 부모 클래스 프로퍼티 포함
for (TFieldIterator<FProperty> It(MyClass, EFieldIteratorFlags::IncludeSuper); It; ++It)
{
    // ...
}

// 특정 프로퍼티 찾기 및 값 읽기/쓰기
FProperty* HealthProp = MyClass->FindPropertyByName(TEXT("Health"));
if (HealthProp)
{
    // 값 읽기
    void* ValuePtr = HealthProp->ContainerPtrToValuePtr<void>(MyCharacter);
    float HealthValue;
    HealthProp->CopySingleValue(&HealthValue, ValuePtr);

    // 값 쓰기
    float NewHealth = 50.0f;
    HealthProp->CopySingleValue(ValuePtr, &NewHealth);
}
```

- 클래스 정보를 기반으로 프로퍼티 순회<br>

### 함수 정보와 동적 호출

```cpp
// 함수 찾기
UFunction* Func = MyClass->FindFunctionByName(TEXT("TakeDamage"));
if (Func)
{
    // 파라미터 정보 출력
    UE_LOG(LogTemp, Log, TEXT("Function: %s"), *Func->GetName());
    UE_LOG(LogTemp, Log, TEXT("  NumParams: %d"), Func->NumParms);

    for (TFieldIterator<FProperty> It(Func); It; ++It)
    {
        FProperty* Param = *It;
        UE_LOG(LogTemp, Log, TEXT("  Param: %s (%s)"),
               *Param->GetName(), *Param->GetCPPType());
    }

    // 동적 호출 (파라미터를 메모리 블록으로 전달)
    struct FTakeDamageParams
    {
        float Amount;
    };

    FTakeDamageParams Params;
    Params.Amount = 25.0f;

    MyCharacter->ProcessEvent(Func, &Params);
}
```

> 이 기능은 블루프린트, 콘솔 명령어, 네트워크 RPC 등에서 활용<br>
> 

## 4.3 RepNotify/NetSerialize는 어떻게 실행될까?

### RepNotify의 내부 동작

**선언**<br>

```cpp
UPROPERTY(ReplicatedUsing=OnRep_Health)
float Health;

UFUNCTION()
void OnRep_Health();
```

**UHT가 생성하는 리플렉션 정보 (개념적)**<br>

```cpp
static const FRepRecord RepRecords[] = {
    { TEXT("Health"), offsetof(AMyCharacter, Health), REPNOTIFY_OnRep_Health }
};
```

**복제 흐름**<br>

```
[서버]
1. Health 값 변경
2. NetDriver가 변경 감지
3. 해당 프로퍼티 직렬화하여 패킷 생성
4. 클라이언트로 전송

[클라이언트]
1. 패킷 수신
2. NetDriver가 리플렉션 테이블에서 프로퍼티 찾음
3. 값을 '역직렬화'하여 적용
4. ReplicatedUsing 지정 시 해당 함수 찾음
5. 함수 포인터로 OnRep_Health() 호출
```

### 함수 시그니처가 중요한 이유

리플렉션 시스템이 함수 시그니처를 검증.<br>
파라미터 타입까지 저장되어 런타임에 체크<br>

**올바른 시그니처**<br>

```cpp
UFUNCTION()
void OnRep_Health();  // 파라미터 없음

UFUNCTION()
void OnRep_Health(float OldHealth);  // 이전 값 받기 (5.0+)
```

**잘못된 시그니처**<br>

```cpp
void OnRep_Health(int32 WrongType);  // ❌ 타입 불일치
void OnRep_Health() const;           // ❌ const 함수 불가
virtual void OnRep_Health();         // ❌ UFUNCTION 없이 virtual만
```

**이전 값 받기 예시**

```cpp
void AMyCharacter::OnRep_Health(float OldHealth)
{
    float Delta = Health - OldHealth;
    if (Delta < 0)
    {
        // 데미지 받음, VFX 재생
        PlayDamageEffect(FMath::Abs(Delta));
    }
    else if (Delta > 0)
    {
        // 힐 받음, VFX 재생
        PlayHealEffect(Delta);
    }
}
```

## 4.4 블루프린트 노출의 원리

### 리플렉션 테이블 기반 핀 생성

블루프린트 에디터가 노드를 생성할 때 리플렉션 정보를 읽음.<br>

```cpp
UFUNCTION(BlueprintCallable, Category="Combat")
float ApplyDamage(float BaseDamage, AActor* DamageCauser, TSubclassOf<UDamageType> DamageType);
```

**블루프린트 시스템의 UFunction 분석 결과**<br>

| 항목 | 값 |
| --- | --- |
| 노드 이름 | "Apply Damage" (CamelCase 자동 분리) |
| 카테고리 | "Combat" |
| 입력 핀 | Base Damage(float), Damage Causer(Actor), Damage Type(Class) |
| 출력 핀 | float (반환값) |
| 실행 핀 | 위쪽(입력), 아래쪽(출력) |

**핀 색상 (타입별)**<br>

| 타입 | 색상 |
| --- | --- |
| float | 연두색 |
| int32 | 청록색 |
| bool | 빨간색 |
| Object | 파란색 |
| Struct | 진파란색 |
| String | 분홍색 |

### Meta 태그의 역할

Meta 태그로 블루프린트 노드를 세밀하게 커스터마이징할 수 있음.<br>

```cpp
UFUNCTION(BlueprintCallable, Category="Combat",
          meta=(DisplayName="데미지 적용",
                Keywords="hurt attack damage",
                ToolTip="대상에게 데미지를 적용합니다.\n데미지 타입에 따라 효과가 달라집니다.",
                AdvancedDisplay="DamageType",
                DefaultToSelf="Target"))
float ApplyDamage(
    UPARAM(DisplayName="데미지량") float BaseDamage,
    AActor* Target,
    AActor* DamageCauser,
    TSubclassOf<UDamageType> DamageType);
```

**주요 Meta 태그**<br>

| Meta 태그 | 설명 |
| --- | --- |
| `DisplayName` | 노드/핀에 표시될 이름 |
| `Keywords` | 검색 시 추가 키워드 |
| `ToolTip` | 마우스 오버 시 설명 |
| `AdvancedDisplay` | 기본적으로 숨겨지는 핀 (확장 시 표시) |
| `DefaultToSelf` | 해당 파라미터의 기본값을 Self로 설정 |
| `HidePin` | 특정 핀을 완전히 숨김 |
| `CompactNodeTitle` | 컴팩트 노드일 때의 짧은 제목 |
| `BlueprintInternalUseOnly` | C++에서만 호출 가능, BP에서 숨김 |
| `DevelopmentOnly` | Development 빌드에서만 동작 |

## 핵심 정리

1. **FProperty 시스템**: UObject 상속하지 않아 GC 오버헤드 없음<br>
2. **플래그 vs 메타데이터**: 플래그는 런타임 사용, 메타데이터는 에디터 전용(Shipping에서 제거)<br>
3. **런타임 메타데이터**: 클래스 정보, 프로퍼티 순회, 함수 동적 호출 모두 가능<br>
4. **RepNotify**: 리플렉션 테이블 기반으로 동작하며, 함수 시그니처가 정확해야 함<br>
5. **블루프린트 노드**: UFunction 분석 → 핀 생성 → Meta 태그로 커스터마이징<br>

# 5. 런타임 로딩 과정: 모듈 → 리플렉션 → 액터 생성

## 5.1 모듈 로딩 시퀀스

### 엔진 시작 시 로딩 순서

언리얼 엔진은 시작 시 정해진 순서로 모듈을 로드<br>

```
────────────────────────────────────────────────────────────────
  1. Core 계열 모듈                                             
     Core, CoreUObject                                         
     (가장 기본적인 타입, 리플렉션 시스템)                       
────────────────────────────────────────────────────────────────
  2. Engine 계열 모듈                                           
     Engine, Renderer, Physics 등                               
     (게임 프레임워크의 핵심)                                    
────────────────────────────────────────────────────────────────
  3. Plugin 모듈들                                              
     LoadingPhase에 따라 순차적으로                              
     EarliestPossible → PostConfigInit → PreDefault →           
     Default → PostDefault → PostEngineInit                     
────────────────────────────────────────────────────────────────
  4. Project 모듈                                               
     게임 모듈                                                   
────────────────────────────────────────────────────────────────
  5. Editor 모듈들 (에디터 실행 시에만)                          
     UnrealEd, LevelEditor 등                                   
────────────────────────────────────────────────────────────────
```

**모듈 인터페이스 구현**<br>

```cpp
// MyGameModule.h
#pragma once

#include "Modules/ModuleManager.h"

class FMyGameModule : public IModuleInterface
{
public:
    // 모듈 로드 시 호출
    virtual void StartupModule() override;

    // 모듈 언로드 시 호출
    virtual void ShutdownModule() override;

    // 모듈 접근 헬퍼
    static inline FMyGameModule& Get()
    {
        return FModuleManager::LoadModuleChecked<FMyGameModule>("MyGame");
    }

    static inline bool IsAvailable()
    {
        return FModuleManager::Get().IsModuleLoaded("MyGame");
    }
};
```

```cpp
// MyGameModule.cpp
#include "MyGameModule.h"

#define LOCTEXT_NAMESPACE "FMyGameModule"

void FMyGameModule::StartupModule()
{
    UE_LOG(LogTemp, Log, TEXT("MyGame 모듈 시작"));

    // 가능한 작업:
    // - 커스텀 콘솔 명령어 등록
    // - 에셋 타입 등록
    // - 슬레이트 스타일 등록
    // - 서브시스템 초기화
}

void FMyGameModule::ShutdownModule()
{
    UE_LOG(LogTemp, Log, TEXT("MyGame 모듈 종료"));
    // 할당한 리소스 정리
}

#undef LOCTEXT_NAMESPACE

// Primary Game Module 등록
IMPLEMENT_PRIMARY_GAME_MODULE(FMyGameModule, MyGame, "MyGame");

// 일반 모듈인 경우
// IMPLEMENT_MODULE(FMyGameModule, MyGame)
```

### Plugin 모듈 로딩 순서

플러그인은 .uplugin의 LoadingPhase와 플러그인 간 의존성에 따라 로드<br>

```json
// PluginA.uplugin
{
  "Modules": [
    {
      "Name": "PluginA",
      "LoadingPhase": "Default"
    }
  ],
  "Plugins": [
    {
      "Name": "PluginB",
      "Enabled": true
    }
  ]
}
```

- PluginA가 PluginB에 의존 → **PluginB가 먼저 로드**
- 순환 의존성 존재 시 → 로딩 실패

## 5.2 리플렉션 테이블이 메모리에 올라오는 시점

### StaticRegisterNatives

각 UCLASS는 `StaticRegisterNativesXXX` 함수를 가지며,<br>
이 함수가 네이티브 함수(UFUNCTION)를 리플렉션 시스템에 등록한다.<br>

```cpp
// 생성된 코드 (MyGame.gen.cpp)
void AMyCharacter::StaticRegisterNativesAMyCharacter()
{
    UClass* Class = AMyCharacter::StaticClass();
    static const FNameNativePtrPair Funcs[] = {
        { "TakeDamage", &AMyCharacter::execTakeDamage },
        { "Heal", &AMyCharacter::execHeal },
        // UFUNCTION으로 선언된 모든 함수들
    };
    FNativeFunctionRegistrar::RegisterFunctions(Class, Funcs, UE_ARRAY_COUNT(Funcs));
}
```

> 이 함수들은 엔진 초기화 과정에서 자동 호출된다 (모듈 로딩 시 static 초기화 단계).
> 

### StaticClass()의 동작

```cpp
UClass* AMyCharacter::StaticClass()
{
    static UClass* ReturnClass = nullptr;
    if (!ReturnClass)
    {
        ReturnClass = GetPrivateStaticClass();
        check(ReturnClass);
    }
    return ReturnClass;
}
```

**Lazy Initialization(지연 초기화)** 방식으로 동작한다.<br>

| 장점 | 설명 |
| --- | --- |
| 메모리 절약 | 사용되지 않는 클래스는 메모리에 올라가지 않음 |
| 시작 시간 단축 | 모든 클래스를 미리 초기화할 필요 없음 |

### Z_Construct_UClass 함수들

.gen.cpp의 `Z_Construct_UClass_XXX` 함수가 실제로 UClass를 구축<br>

```cpp
UClass* Z_Construct_UClass_AMyCharacter()
{
    // 이미 생성되었으면 재사용
    if (!Z_Registration_Info_UClass_AMyCharacter.OuterSingleton)
    {
        // 부모 클래스 먼저 구축
        UClass* SuperClass = Z_Construct_UClass_ACharacter();

        // 이 클래스 구축
        // 1. 기본 정보 설정 (이름, 플래그, 크기 등)
        // 2. 프로퍼티 등록 (UPROPERTY로 선언된 것들)
        // 3. 함수 등록 (UFUNCTION으로 선언된 것들)
        // 4. 메타데이터 설정 (Category, ToolTip 등)
        // 5. 인터페이스 연결
        // 6. 부모 클래스 포인터 설정
    }
    return Z_Registration_Info_UClass_AMyCharacter.OuterSingleton;
}
```

## 5.3 CDO (Class Default Object) 생성

### CDO란?

모든 UCLASS는 `CDO(Class Default Object)`를 가진다.<br>
해당 클래스의 기본 템플릿 객체<br>

**CDO 접근 방법**

```cpp
// 수정 가능한 CDO
AMyCharacter* CDO = GetMutableDefault<AMyCharacter>();

// StaticClass()를 통한 접근
AMyCharacter* CDO = AMyCharacter::StaticClass()->GetDefaultObject<AMyCharacter>();

// const 버전
const AMyCharacter* CDO = GetDefault<AMyCharacter>();
```

**CDO의 특성**<br>

| 특성 | 설명 |
| --- | --- |
| 유일성 | 클래스당 딱 하나 존재 |
| 기본값 저장 | 모든 기본 프로퍼티 값 보유 |
| 복사 원본 | 새 객체 생성 시 `복사 원본`으로 사용 |
| Reset 기준 | 에디터의 "Reset to Default" 기준 |
| BP 기본값 | `블루프린트 기본값 편집` = CDO 편집 |

### CDO와 생성자의 관계

클래스 생성자는 CDO를 초기화할 때 한 번 호출<br>
- CDO 생성을 한 후, 나머지는 CDO를 복사하여 사용하므로<br>
  - C++ 복사 생성자가 아니라 UObject 내부의 복사 관련 옵션 사용<br>
  - UObject 를 사용하게 유도하기 위해 '복사 생성자'를 막아놓음!<br>

```cpp
AMyCharacter::AMyCharacter()
{
    // CDO 생성 시 한 번 실행됨
    Health = 100.f;
    MaxHealth = 100.f;
    WalkSpeed = 600.f;

    // 컴포넌트 생성
    MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = MeshComponent;
}

```

**새 인스턴스 생성 과정 (내부 동작)**<br>

```cpp
// 1. 메모리 할당
AMyCharacter* NewCharacter = (AMyCharacter*)StaticAllocateObject(...);

// 2. CDO의 프로퍼티 값들 복사
FMemory::Memcpy(NewCharacter, CDO, sizeof(AMyCharacter));

// 3. 컴포넌트들도 복제
DuplicateComponents(CDO, NewCharacter);

// 4. PostInitProperties 호출
NewCharacter->PostInitProperties();

```

> 주의: `생성자에서 GetWorld()나 GetOwner() 호출 금지`.` CDO 생성 시점에는 World도 Owner도 존재하지 않는다`. 이런 초기화는 BeginPlay()에서 수행
> 

## 5.4 Blueprint 클래스가 어떻게 로딩되나?

### Blueprint 클래스의 실체

블루프린트 클래스는 `UBlueprintGeneratedClass`이며,<br>
UClass를 상속한다.<br>

```
UObjectBase
└── UObjectBaseUtility
    └── UObject
        └── UField
            └── UStruct
                └── UClass
                    └── UBlueprintGeneratedClass  // 블루프린트 클래스
```

**블루프린트 로딩 과정**<br>

| 단계 | 내용 |
| --- | --- |
| 1 | .uasset 파일 로드 (`/Game/Blueprints/BP_MyCharacter.uasset`) |
| 2 | UBlueprint 객체 생성 (에디터에서 편집하는 대상) |
| 3 | UBlueprintGeneratedClass 생성 (실제 사용되는 UClass, C++ 부모 클래스 연결) |
| 4 | 프로퍼티/함수 등록 (변수 → FProperty, 함수 → UFunction) |
| 5 | CDO 생성 (블루프린트에서 설정한 기본값으로) |

### GeneratedClass

블루프린트 에셋(UBlueprint)과 생성된 클래스(UBlueprintGeneratedClass)는 다르다.<br>

```cpp
// 블루프린트 에셋 로드
UBlueprint* BP = LoadObject<UBlueprint>(nullptr,
    TEXT("/Game/Blueprints/BP_MyCharacter.BP_MyCharacter"));

// 실제 사용할 클래스 (GeneratedClass)
UClass* BPClass = BP->GeneratedClass;

// 액터 스폰 시에는 GeneratedClass를 사용
FActorSpawnParameters SpawnParams;
AActor* SpawnedActor = GetWorld()->SpawnActor<AActor>(
    BPClass, SpawnLocation, SpawnRotation, SpawnParams);

// 또는 바로 클래스 로드 (_C 접미사 주의!)
UClass* BPClass = LoadClass<AActor>(nullptr,
    TEXT("/Game/Blueprints/BP_MyCharacter.BP_MyCharacter_C"));
```

> 주의: 클래스 경로에는 _C 접미사가 붙음. BP_MyCharacter_C가 GeneratedClass의 경로이다.
> 

**Hot Reload / Live Coding**<br>

블루프린트 수정 시 GeneratedClass가 다시 생성되며,<br>
기존 인스턴스들은 새 클래스로 마이그레이션된다. 이것이 에디터에서 블루프린트 수정이 바로 반영되는 원리!<br>

## 핵심 정리

1. **모듈 로딩 순서**: Core → Engine → Plugin(LoadingPhase순) → Project → Editor<br>
2. **StartupModule()**: 모듈 로드 시 호출되며, 커스텀 초기화 코드 작성 가능<br>
3. **StaticClass()**: Lazy Initialization 방식으로 첫 호출 시 UClass 생성<br>
4. **CDO**: 클래스당 하나의 `기본 템플릿 객체`. `새 인스턴스는 CDO를 복제`하여 생성<br>
5. **생성자 주의사항**: GetWorld(), GetOwner() 호출 금지. BeginPlay()에서 수행<br>
6. **블루프린트 클래스**: UBlueprintGeneratedClass. 경로에 `_C` 접미사 필요<br>

언리얼 내부의 특이한 문제들이 일어났다면<br>
보통 엔진 내부와 연동된 것일 가능성이 높음!<br>