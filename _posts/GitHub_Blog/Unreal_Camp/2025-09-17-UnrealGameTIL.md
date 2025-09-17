---
title: "Unreal TIL (0917)"
date : "2025-09-17 17:00:00 +0900"
last_modified_at: "2025-09-17T17:00:00"
categories:
  - Unreal
  - C++
tags:
  - GameMode
  - Pawn
  - Character
  - Input Mapping Context
  - Player Controller
  - Input Action
---

# **GameMode 이해하기**

## **GameMode란?**

- **GameMode**는 게임의 전반적인 규칙과 흐름을 총괄 관리하는, 일종의 **컨트롤 타워** 역할을 하는 클래스<br>
- **싱글 플레이**에서는 ‘서버’와 ‘클라이언트’ 개념이 나뉘지 않으므로, **GameMode가 온전히 로컬에서 동작**하여 게임 전체를 제어<br>
- 플레이어가 spawn할 **Pawn** (or Character), **PlayerController**를 지정하거나, 승패 조건/점수 계산 방법에 대한 설정 등 **게임 플레이의 핵심 로직**을 담당<br>
    - 프로젝트 전역(기본) 혹은 레벨별 필요한 GameMode를 구분해 설정 가능 (예: 튜토리얼 맵 전용 GameMode, 일반 맵 전용 GameMode 등)<br>

## GameMode의 주요 기능과 책임

- **플레이어 Pawn/Character 스폰**<br>
    - 게임이 시작될 때 (or 플레이어가 리스폰될 때), **DefaultPawnClass** 또는 지정한 Pawn 클래스를 스폰시킴<br>
    - 스폰된 Pawn을 플레이어가 조작 가능하도록 **PlayerController**와 연동<br>
- **PlayerController 지정**<br>
    - 플레이어의 입력(키보드, 마우스, 게임패드 등)을 전달/처리할 **PlayerController** 클래스를 결정<br>
- **게임 규칙 관리**<br>
    - 점수 계산, 타이머, 라운드 제어, 난이도 등 **게임 전반의 규칙**을 정의 및 유지<br>
    - 특정 점수 달성, 보스 몬스터 처치, 제한 시간 종료 등 **승리/패배를 결정하는 조건**을 관리<br>
    - 승리/패배가 확정에 따라, 게임 오버 화면을 띄우거나 다음 레벨로 전환하는 식의 후속 처리<br>
- **GameState / PlayerState 사용**<br>
    - GameState는 전체 게임 흐름 (타이머, 전역 변수 등), PlayerState는 플레이어별 정보 (체력, 점수 등)를 관리<br>
    - 멀티플레이만큼 복잡하게 쓰진 않더라도, 상태 저장과 관리를 좀 더 체계적으로 하고 싶을 때 유용<br>

## GameMode vs GameModeBase

- 언리얼 엔진 5에는 `GameMode`와 `GameModeBase` 두 종류가 존재<br>
- **GameMode**<br>
    - 언리얼에서 제공하는 멀티플레이 기능 (세션, 플레이어 연결 로직 등)을 일부 포함, 싱글 플레이에서도 사용 가능<br>
    - 필요에 따라 **GameState**, **PlayerState** 등 연동이 활성화<br>
- **GameModeBase**<br>
    - 좀 더 단순화된 형태로, **멀티플레이 관련 로직**이 거의 포함 X <br>
    - 간단한 싱글 플레이 게임 또는 직접 멀티플레이 로직을 구현하고 싶을 때 사용<br>

# **Pawn과 Character 클래스 이해하기**

## **Pawn 클래스란?**

- **Pawn** : 플레이어 혹은 AI가 “소유( Possess )”할 수 있는 **가장 상위 클래스**<br>
- Pawn에는 **이동 로직**이나 **충돌 처리**, **중력**, **네트워크 이동**을 위한 기능들이 **기본적으로 포함되어 있지 않음**<br>
    - **보행** (걷기, 달리기, 점프 등)에 필요한 시스템 (캡슐 콜리전, 중력, 지형 따라가기)을 모든 단계에서 직접 구현 필요<br>
- 그렇기에, **비행기**, **드론**, **카메라**처럼 기존 Character의 이동과 다른 움직임을 구현할 때 사용<br>

## **Character 클래스란?**

- **Character**는 Pawn을 상속받아 만들어진 자식 클래스 중 하나로, 기본적으로 `UCharacterMovementComponent`를 포함<br>
    - **이동**, **회전**, **점프**, **중력**, **지형 따라가기**, **네트워크 동기화** 등 **보행형 캐릭터**에게 필요한 기능이 이미 구현<br>
    - 미리 정의된 대표적인 함수들(예: `MoveForward`, `MoveRight`, `Jump`)이 존재<br>
- 캐릭터를 구성하는 전형적인 요소들이 표준화되어 있어, **일반적인 인간형 캐릭터**를 만드는 데 최적화<br>
    - **자동차**나 **비행기**처럼 다른 이동을 구현할 땐, 불필요한 '기능'이 탑재되어 있는 상황이므로 Pawn 사용 고려<br>


### 플레이어 Anim Instance C++ 세팅 중 만난 일

```cpp
ATaskPlayer::ATaskPlayer()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	SpringArmComp = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	SpringArmComp->SetupAttachment(RootComponent);
	SpringArmComp->TargetArmLength = 300.0f;
	SpringArmComp->bUsePawnControlRotation = true;

	CameraComp = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	CameraComp->SetupAttachment(SpringArmComp, USpringArmComponent::SocketName);
	CameraComp->bUsePawnControlRotation = false;

	ConstructorHelpers::FObjectFinder<USkeletalMesh> MeshAsset(TEXT("/Game/Resource/Robot_scout_R_21/Mesh/SK_Robot_scout_R21.SK_Robot_scout_R21"));
	if (MeshAsset.Succeeded())
	{
		GetMesh()->SetSkeletalMesh(MeshAsset.Object);
		GetMesh()->SetRelativeLocation(FVector(0.0, 0.0, -90.0));
		GetMesh()->SetRelativeRotation(FRotator(0.0, -90.0, 0.0));

		// Anim Class를 찾아 세팅
		// Class를 찾는것이기에 마지막에 _C 를 붙인다!
		ConstructorHelpers::FClassFinder<UAnimInstance> MeshAnimAsset(TEXT("/Game/Resource/Robot_scout_R_21/Demo/Animations/ThirdPerson_AnimBP.ThirdPerson_AnimBP"));
		if (MeshAnimAsset.Succeeded())
		{
			GetMesh()->SetAnimationMode(EAnimationMode::AnimationBlueprint);
			GetMesh()->SetAnimInstanceClass(MeshAnimAsset.Class);
		}
	}
}
```

- Player 세팅 중 FClassFinder 를 사용하여 AnimBP를 세팅하려 하였으나<br>
  이상하게도 '찾지 못한다'는 로그가 계속 발생하였다<br>
  (그런데 Copy Ref를 통해 가져온 것과 동일한 상황)<br>

#### 해결

```cpp
ConstructorHelpers::FClassFinder<UAnimInstance> MeshAnimAsset(TEXT("/Game/Resource/Robot_scout_R_21/Demo/Animations/ThirdPerson_AnimBP.ThirdPerson_AnimBP_C"));
```

- 뒤 쪽에 `'_C'` 를 붙이니 정상적으로 Anim Instance Class를 가져올 수 있었다<br>

### 발생 원인?

에디터에서 만든 BP 에셋은 2가지 '형태'로 나뉘게 된다<br>

| 개념                   | 예시 경로                              | 설명                                                                                                  |
| -------------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------- |
| **블루프린트 에셋(Object)** | `/Game/Characters/Hero/ABP_Hero`   | 에디터에서 보이는 `.uasset`. 단순한 데이터 오브젝트(UAnimBlueprint)로서 “어떤 클래스인지”에 대한 정의를 **포함**하지만 **실제 UClass가 아님**. |
| **생성된 클래스(UClass)**  | `/Game/Characters/Hero/ABP_Hero_C` | 에디터가 블루프린트를 **컴파일**할 때 생성하는 **런타임 UClass 객체**. 게임 실행 시 이 `_C`가 **실제 코드 클래스**처럼 동작.                  |

- BP를 저장하면 에디터는 내부적으로 `'컴파일'`하여 **UClass** 를 만듦<br>
- BP 클래스가 'BP' 이름 뒤에 '_C'를 붙인 이름으로 만들어진다<br>
  그렇기에 C++에서 BP용 클래스를 '가져올 때'는 `_C` 를 붙여 가져오는 것이 권장<br>

- 기본적으로 언리얼 엔진이 켜진 상황에서 생성되는 클래스들에 붙으므로<br>
  '직접' 클래스 경로를 가져올때는 _C 를 이용해야 한다<br>
  (ex: 문자열 경로로 사용 가능한 `LoadClass`, `FClassFinder`, `SpawnActor` 등)<br>

- 그래도 평소에는 `AssetManager` 나 `TObjectPtr`, `TSubClassOf` 등을 사용한다면<br>
  일반적으로는 사용할 일 자체는 없긴 하다<br>

---

# **PlayerController 이해하기**

## **PlayerController란?**

- **PlayerController**는 사용자가 키보드, 마우스, 게임패드 등에서 입력을 받으면, 그 입력을 해석하여 캐릭터나 다른 오브젝트에게 동작을 명령하는 클래스<br>
- 언리얼 엔진의 중요한 철학 중 하나는 “**플레이어 입력은 PlayerController에서 처리**한다”는 것입니다. 이를 통해 입력 처리 로직과 실제 캐릭터의 동작 로직을 분리하여 구조적인 관리가 쉬워짐<br>
- **입력이 처리되는 기본 흐름**
    1. 키보드, 마우스, 게임패드 등 입력 장치로부터 사용자 조작 신호가 들어옴<br>
    2. 이 신호는 **PlayerController**가 받아서 해석<br>
    3. PlayerController가 현재 소유 (Possess)하고 있는 Pawn에게 이동, 회전, 공격 등의 구체적인 명령<br>
- 특히 멀티플레이 환경에서는, 각 플레이어마다 개별 **PlayerController**가 생성, 여러 사용자의 입력을 충돌 없이 분리하고 관리<br>

## PlayerController의 주요 기능

- **입력 처리**<br>
    - 키보드, 마우스, 게임패드, 터치 등 다양한 입력 장치의 이벤트를 처리<br>
    - 언리얼 엔진 5에서 제공하는 **Enhanced Input** 시스템을 사용하면, 액션/축 매핑을 보다 체계적으로 설정 가능<br>
    - C++에서는 `SetupInputComponent()` 함수를 오버라이드하여, 블루프린트에서는 이벤트 그래프를 통해 입력 로직을 구현<br>
- **카메라 제어 로직**
    - 마우스나 게임패드의 축 입력을 받아 캐릭터의 시점 회전이나 줌 인/아웃 같은 카메라 동작을 수행<br>
- **HUD 및 UI와의 상호작용**
    - 언리얼의 UMG (언리얼 모션 그래픽) 기반 UI를 통해 버튼 클릭, 드래그, 터치 등의 이벤트를 PlayerController에서 받을 수 있음<br>
    - 예를 들어 인벤토리 열기, 스킬 사용 등의 명령을 UI에서 트리거하면 PlayerController가 이를 해석해 Pawn 또는 GameMode 등 다른 시스템으로 전달 가능<br>
- **Possess / UnPossess**
    - PlayerController는 특정 Pawn에 “빙의 (Possess)”하여 해당 Pawn을 제어함<br>
    - 필요할 때 `UnPossess()` 함수를 호출하여 Pawn과의 연결을 해제한 뒤, 다른 Pawn에 빙의 가능<br>
    - 멀티플레이 시 각 플레이어마다 고유의 PlayerController가 있고, 이 컨트롤러가 특정 Pawn을 소유함으로써 서로 다른 캐릭터 조작이 가능한 것<br>


# Enhanced Input System의 이해 및 Input Action 설정하기

## **Enhanced Input System란?**

- **언리얼 엔진 5**에는 이전 버전(UE4 등)에서 사용하던 전역 `Project Settings → Input` 시스템을 대체하거나 확장하기 위해 **Enhanced Input** 시스템이 제공<br>
- Enhanced Input은 입력 설정을 “입력 맵(Input Mapping Context, IMC)”과 “입력 액션(Input Action, IA)”이라는 개념으로 나누어 관리<br>

## **Input Action (IA) 생성**

- **Input Action (IA)**
    - **Input Action**은 캐릭터의 이동, 점프, 발사, 줌 등과 같이 **특정 동작**을 추상화한 단위<br>
    - 예를 들어 WASD 이동을 담당하는 `IA_Move`, 스페이스바 점프를 담당하는 `IA_Jump`, 마우스 회전을 담당하는 `IA_Look` 등을 만들 수 있음<br>
    - Value Type, Trigger, Modifier 등을 이용하여 입력의 세부 속성 조절<br>

- **Value Type**은 Input Action (IA)이 입력 동작을 발생시킬 때, 어떤 유형의 값을 제공할지 결정하는 옵션<br>
    - **`Bool` (참/거짓)**<br>
        - 단순 On/Off 토글 입력에 사용<br>
        - 예) 점프(스페이스바), 공격(마우스 왼쪽 버튼)<br>
    - **`Axis1D` (1차원 축 값)**<br>
        - 단일 축 (-1~1 범위)의 입력에 사용<br>
        - 예) 게임패드 트리거(가속 페달), 전진/후진(W/S)<br>
    - **`Axis2D` (2차원 축 값)**<br>
        - X, Y 두 축을 동시에 처리할 때 사용<br>
        - 예) 캐릭터 이동(WASD), 마우스 이동(가로+세로)<br>
    - **`Axis3D` (3차원 축 값)**<br>
        - X, Y, Z 세 축을 동시에 처리<br>
        - 예) 비행 시뮬레이션에서 3축 제어<br>
- **트리거 (Trigger)**는 입력이 활성화되는 특정 조건을 표현<br>
    - **Pressed Trigger:** 키를 누르는 순간에만 작동.<br>
    - **Hold Trigger:** 키를 일정 시간 눌렀을 때 작동.<br>
    - **Released Trigger:** 키를 뗄 때 작동.<br>
- **모디파이어 (Modifier)는 입력 값을 수정하거나 변환**하기 위한 설정<br>
    - **Scale** : 입력 값에 일정 배율을 곱해줌 (마우스 이동 속도 2배)<br>
    - **Invert** : 입력 값을 반전 (상하 반전 카메라)<br>
    - **Deadzone** : 일정 임계값보다 작은 입력은 무시 (게임패드 조이스틱 미세 떨림 방지)<br>

# Input Mapping Context 설정하기

## **Input Mapping Context (IMC) 생성**

- **Input Mapping Context (IMC)**<br>
    - IMC는 여러 개의 IA들을 한데 모아놓은 매핑 설정 파일<br>
    - 예를 들어, “플레이어 기본 이동, 점프, 시점 전환”을 하나의 IMC에 넣고, “UI 전용 입력”을 다른 IMC로 분리하는 식으로 관리 가능<br>
    - 게임 진행 중 특정 상황에서 IMC를 활성 (Enable)하거나 비활성 (Disable)하여 입력을 제어할 수 있음<br>

- **Swizzle Input Axis Values란?**<br>
    - **Swizzle Input Axis Values**는 입력 축 (Axis)을 변환하거나 재구성하는 기능<br>
    - 언리얼 엔진의 입력 시스템에서는 입력 데이터를 X, Y, Z 축 중 하나로 매핑함<br>
    - Swizzle은 이 입력 값이 올바른 축에 맞지 않을 경우, 특정 축으로 재배치하거나 변환할 수 있게 보조<br>

- 이러한 IMC를 활성화하려면 `Local Player Subsystem`를 이용<br>

- **Local Player Subsystem이란?**<br>
    - 게임이 실행되면 언리얼은 각 플레이어를 표현하기 위해 **Local Player** 객체를 생성<br>
        - 싱글플레이어 상황에서는 하나의 Local Player<br>
        - 로컬 멀티플레이 (하나의 화면에서 여러 명이 플레이)라면 플레이어 수만큼 Local Player가 생성<br>
    - **UEnhancedInputLocalPlayerSubsystem** 은 Local Player에 부착되어, 해당 플레이어가 사용할 입력 매핑 (IMC)을 관리<br>
        - 이를 통해, 플레이 중에 동적으로 다른 IMC를 추가·제거하여 입력 모드를 전환 가능<br>
        - 예) 전투 중 (IMC_Character) → UI창 열림 (IMC_UI) → 전투 종료 후 다시 (IMC_Character)<br>

