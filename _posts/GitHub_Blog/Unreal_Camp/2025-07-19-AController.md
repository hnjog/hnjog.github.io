---
title: "Controller"
last_modified_at: "2025-07-19T12:30:00"
categories:
  - 언리얼 5
tags:
  - AController
  - Player Controller
  - AI Controller
  - Behavior Tree
  - BlackBoard
  - Perception System
---

## Controller
Pawn을 조종하는 추상적 존재이며, '유저' 혹은 'AI'의 대리하는 클래스<br>

```
UCLASS()
class AController : public AActor
{
    APawn* Pawn; // 내가 조종하는 Pawn
};

```

- Possess(APawn* InPawn)으로 Pawn에 '빙의'하여 조작 <br>
- UnPossess()로 빙의 해제<br>
- Tick() 이벤트 를 이용하여 제어 로직 사용 가능<br>
- AI 지원 함수 포함<br>

### Controller가 필요한 이유?
단순히 Actor에 '입력'을 직접 전달하지 않고,<br>
Controller라는 중간 단계를 도입하였는가<br>

'기능'의 분리를 통해<br>
'입력' <-> '움직임'을 '모듈화'할 수 있었음<br>


| 항목                | Controller가 분리됨으로써 가능한 일                       |
| ----------------- | ---------------------------------------------- |
| 🎮 입력과 움직임 분리      | 입력 로직과 실제 움직임, 충돌 로직을 완전히 분리 가능                |
| 👥 AI/플레이어 공통화    | 동일한 Pawn을 AI/Player가 번갈아 제어 가능                 |
| 🔁 Possess 기능     | 게임 중에 다른 Pawn으로 빙의 전환 (e.g., RTS 유닛 교체)        |
| 🧩 재사용성           | 하나의 Controller로 다양한 Pawn 제어 가능                 |
| 👨‍👨‍👧 멀티플레이 지원 | 네트워크 Player마다 별도의 Controller가 붙음               |
| 🧠 AI 확장          | AIController로 별도 판단 시스템(Behavior Tree 등) 구축 가능 |

- Input을 추상화하여<br>
  Controller가 '입력'을 받고, Pawn은 Controller가 준 '명령'을 따름<br>
  Controller가 캐릭터를 자유롭게 바꿀 수 있음

- 같은 Contoller로 다양한 Pawn의 제어가 가능해짐<br>
  인간 -> 전투기 탑승 -> 인간 -> 보트 탑승 -> 인간 등을<br>
  Controller가 Possess를 바꿈으로서 쉽게 구현이 가능<br>

- AI와 더 쉬운 연동<br>
  Blackvoard, BehaviorTree 등과 쉽게 연결하며<br>
  동일한 Pawn을 AI와 Player가 번갈아 사용 가능<br>

- 서버의 Pawn 제어 상태 확인<br>
  PlayerController가 클라이언트의 입력을 확인하고<br>
  서버에서 Authority Controller 기반으로 Pawn의 제어를 확인<br>

- 반대로 Controller가 없었다면<br>
  하나의 캐릭터 클래스를 AI나 Player용으로 나누어서 만들거나<br>
  내부 판단 로직이 추가되서 별도로 구현 등의 상황이 발생<br>

일반적으로는 APlayerController나 AAIController를 기반으로 사용하고<br>
AController는 공통적인 기능을 모아두는 부모 클래스의 역할로 빠져있는 편<br>

## APlayerController
입력을 받아 Pawn을 조종하는 대표적인 Controller<br>

| 기능                    | 설명                      |
| --------------------- | ----------------------- |
| `InputComponent`      | 키보드, 마우스, 게임패드 등 입력 바인딩 |
| `Possess(Pawn*)`      | 캐릭터를 조종하게 만듦            |
| `GetPawn()`           | 내가 현재 소유 중인 Pawn 반환     |
| `PlayerCameraManager` | 카메라 위치/회전/이펙트 제어        |
| `HUD`, `Widget`       | UI 표시와 상호작용             |
| `Client-Side Logic`   | 멀티플레이어에서 로컬 유저 조작 처리    |

샘플 코드<br>

```
void AMyPlayerController::SetupInputComponent()
{
    Super::SetupInputComponent();

    // "MoveForward"라는 입력 축(Input Axis)이 발생했을 때
    // AMyPlayerController::MoveForward 함수를 호출하도록 바인딩
    InputComponent->BindAxis("MoveForward", this, &AMyPlayerController::MoveForward);
}

void AMyPlayerController::MoveForward(float Value)
{
    // Pawn 소유 중인지 확인한후, Pawn에게 이동 요청
    if (GetPawn())
        GetPawn()->AddMovementInput(GetPawn()->GetActorForwardVector(), Value);
}

```

## AAIController
AI 전용 판단 로직을 수행<br>
Behavior Tree, Blackboard, Perception 등과 함께 동작<br>

| 구성 요소                    | 설명                 |
| ------------------------ | ------------------ |
| `UBehaviorTreeComponent` | 트리 구조의 AI 행동 제어 로직 |
| `UBlackboardComponent`   | 변수 저장소 (위치, 타겟 등)  |
| `UAIPerceptionComponent` | 시야, 청각 등 센서        |
| `RunBehaviorTree()`      | BT 실행 시작 지점        |
| `Tick()`                 | 매 프레임 행동 업데이트 가능   |
| `MoveTo()`               | 내비게이션 기반 이동 명령     |


---

### Behavior Tree?

행동 트리라 불리며,<br>
AI가 특정한 조건에 맞게 '행동'을 결정하고<br>
그 행동을 실행<br>

```
Selector (?)
├── Sequence (순차 실행)
│   ├── MoveTo(Target)
│   └── Attack(Target)
└── Wait(3초)

```

- Selector: 자식 노드 중 하나라도 성공하면 종료<br>

- Sequence: 자식 노드 모두 성공해야 성공<br>

- Task: 실제 행동을 수행 (MoveTo, Attack)<br>

- Decorator: 조건문 (e.g., Blackboard에 Target이 있을 때만)<br>

- Service: 반복 실행되는 감시 로직 (e.g., 주변 감지)<br>

---

### Blackboard
AI의 '기억장치'<br>

| 키(Key)        | 타입     | 사용 예시    |
| ------------- | ------ | -------- |
| `TargetActor` | Object | 추적/공격 대상 |
| `Destination` | Vector | 이동 목적지   |
| `HasLOS`      | Bool   | 시야 확보 여부 |

Behavior Tree의 Task/Decorator 들이 이 키 값들을 읽고 쓰면서 '행동'을 결정한다<br>

---

### AI Perception System
AI의 '감각'<br>

| 감각      | 기능                      |
| ------- | ----------------------- |
| Sight   | 시야각, 거리, 시야 유효 시간 설정 가능 |
| Hearing | 소리 발생 시 이벤트 처리          |
| Damage  | 공격받을 때 감지 가능            |

-> AI가 특정한 상황에 따라 '수치값'을 조정하여(BlackBoard에 값 수정 및 저장)<br>
   이후 Behavior Tree에서 행동을 결정<br>


## 정리


| 항목         | AController     | APlayerController        | AAIController               |
| ---------- | --------------- | ------------------------ | --------------------------- |
| 역할         | Pawn 조종 추상화     | 입력 기반 캐릭터 조종             | AI 기반 캐릭터 조종                |
| 생성 시점      | GameMode가 자동 생성 | Player가 접속 시 생성          | AI 스폰 시 생성                  |
| Possess 기능 | O               | O                        | O                           |
| 입력 처리      | 직접 처리 안 함       | 키보드/패드 입력 처리             | 없음 (BT/Perception 사용)       |
| AI 지원      | X               | X                        | ✅ Behavior Tree, Blackboard |
| UI/HUD     | X               | ✅                        | X                           |
| 네트워크 대응    | 서버 권한 기반        | 클라이언트 소유                 | 서버 전용                       |
| 카메라 제어     | X               | ✅ PlayerCameraManager 사용 | X                           |

