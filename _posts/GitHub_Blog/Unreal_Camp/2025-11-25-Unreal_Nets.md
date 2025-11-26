---
title: "Unreal Nets"
date : "2025-11-25 18:00:00 +0900"
last_modified_at: "2025-11-26T18:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - NetDriver
  - Channel
  - Packet
  - Bunch
---

## Unreal의 네트워크 구조

[![Image](https://github.com/user-attachments/assets/5d4df0c1-ff14-4ad8-ad25-2bafac30637e)](https://github.com/user-attachments/assets/5d4df0c1-ff14-4ad8-ad25-2bafac30637e){: .image-popup}<br>

[![Image](https://github.com/user-attachments/assets/9787c86d-1825-4184-af06-469deff3ec18)](https://github.com/user-attachments/assets/9787c86d-1825-4184-af06-469deff3ec18){: .image-popup}<br>


- 계층 정리도<br>

```
UWorld (월드)
 └── UNetDriver
       └── UNetConnection (Client 1개당 1개)
             └── UChannel (한 Connection에 여러 개)
                   └── FOutBunch / FInBunch (전송할 논리 단위)
                         └── Packet (실제 UDP 패킷에 여러 Bunch가 합쳐져 전송됨)
```

- UWorld<br>
  - 언리얼의 네트워크는 '월드'를 기준으로 동작<br>
  - 서버 / 클라 실행시 각각의 World가 자신의 NetDriver를 가짐<br>
  - Tick에 따라 NetDriver에게 '업데이트' 요청<br>

- UNetDriver<br>
  - 네트워크 전송 / 수신, 연결 유지, 타임 아웃 처리를 담당<br>
  - Packet을 보내고 받음<br>
  - 서버는 클라이언트 만큼 NetConnection 관리<br>

- UNetConnection (연결 객체)<br>
  - 클라이언트 1명당 서버엔 1개의 NetConnection 생성<br>
  - 서버는 접속한 클라만큼, 클라는 서버와의 통신을 위한 1개의 NetConnection만 생성<br>
  - 플레이어 1명에 대한 네트워크 상태 관리<br>

- UChannel(채널)<br>
  - 네트워크의 데이터를 '기능'별로 분리하는 단위<br>
  - NetConnection 내부에 여러 채널 존재<br>
    - ex) ControlChannel, ActorChannel 등<br>

- Bunch<br>
  - 언리얼 네트워크의 '논리적 전송 단위'<br>
  - 채널 안에 생성되며 여러 정보가 담김<br>
    - ex) RPC 호출 정보, Replication 정보 등<br>

- Packet(RUDP)<br>
  - 여러 Bunch들이 모아두었다가 한번에 Packet에 합쳐 전송<br>
    ([네트워크 관련 포스팅](https://hnjog.github.io/unreal/c++/Networks/))<br>

---

### TMI - 서버 와 클라의 World 시작

**서버의 관점**<br>

클라이언트가 접속을 하지 않더라도 진행<br>
- UWorld 생성<br>
- GameMode/State/Actor Spawn 진행<br>
- World::Beginplay 호출<br>
- GameMode가 규칙/매치 흐름 진행<br>

**클라 관점**<br>

- 서버에 접속<br>
- 클라가 같은 맵을 로딩하며 자신의 UWorld 생성<br>
- 로딩 이후, Client 내부의 Beginplay 호출<br>
  - 또한 서버에서 GameState, PlayerController, PlayerState 등과<br>
    필요한 Replicate 데이터들을 받음<br>

- 이후 받아온 데이터를 기반으로 GameState 등을 생성하고 상태 동기화<br>


[![Image](https://github.com/user-attachments/assets/da7b8534-8524-4e2a-8783-a78957fac279)](https://github.com/user-attachments/assets/da7b8534-8524-4e2a-8783-a78957fac279){: .image-popup}<br>

이러한 방식으로 '늦게 접속한' 클라이언트 역시<br>
현재 서버의 상태에 맞게 동기화하여 문제없이 진행 가능<br>

---

#### 게임 시작에 따른 전반적인 흐름

- 서버 월드/레벨 시작<br>
  : 서버 월드가 돌아가며 GameMode,GameState 등이 BeginPlay를 호출<br>

```
UGameInstance::StartGameInstance()
  → UGameEngine::LoadMap()
     → UWorld 생성
     → 기본 Actor들 스폰 (GameMode, GameState, PlayerStart 등)
  → UWorld::BeginPlay()
     → World 안의 모든 Actor에 대해 AActor::BeginPlay() 호출
```

- 서버의 게임 시작<br>
  : GameMode/GameState의 게임 시작<br>
  - GameMode::StartPlay()<br>
  - GameState::HandleBeginPlay()<br>
    (이 타이밍에 State의 Replicated 플래그를 세팅하고, 클라로 복제)<br>

- 클라 월드/레벨 시작<br>
  - 클라는 GameMode가 없기에 GameState의 복제 값과 OnRep를 통해 시작을 확인<br>
  - 레벨을 열고, 처리 자체는 생성하고 초기화하며 BeginPlay()를 호출<br>

```
서버로부터 Travel 정보 수신
  → UGameEngine::LoadMap()
     → 클라용 UWorld 생성
     → 기본 Actor들 스폰 (GameState, PlayerController, DefaultPawn 등 *클라 버전*)
  → UWorld::BeginPlay()
     → Actor::BeginPlay() (클라 쪽에서도 전체 호출)
```

- GameState의 OnRep_ReplicatedHasBegunPlay() 함수를 통해<br>
  게임 시작 확인<br>
  - GameState의 Replicate 변수의 값 변경을 통해 클라에 OnRep_ 함수가 호출<br>
  - 늦게 접속한 클라이언트들도 이 함수가 호출되며, 서버와 동기화<br>
    - OnRep 는 '이전 값 vs 이번 값'의 비교를 통해 호출되기에<br>
      새로 들어온 클라의 Replicate를 초기화하며 false인 걸 확인하고<br>
      OnRep를 바로 호출<br>


- PlayerController? PlayerState?<br>

```cpp
bool AGameModeBase::PreLogin(...);  // 접속 허가 여부 - ErrorMessage에 임의의 문자열 넣을시, 로그인 중인 플레이어의 연결이 거부됨 (설정에 따라 StandAlone으로 유지하거나, 다른 Level 로드)
APlayerController* AGameModeBase::Login(...); // PC 생성
void AGameModeBase::PostLogin(APlayerController* NewPC);
void AGameModeBase::HandleStartingNewPlayer(APlayerController* NewPC);
void AGameModeBase::RestartPlayer(AController* NewPC); // Pawn 스폰 & Possess
```

- RestartPlayer 타이밍에 PlayerController,PlayerState, Pawn 등이 서버에서 Spawn되고<br>
  NetDriver가 이들을 클라에 Replicate<br>

- 이후, Client 쪽에서 PostNetInit() 등이 호출<br>
  (이미 시작했다면 Beginplay도 바로 호출)<br>

- RestartPlayer 타이밍에 추가적으로<br>
  Pawn에 컨트롤러가 빙의되기에<br>
  - AController::Possess<br>
  - Pawn의 PossessedBy() 호출(서버에서)<br>
    - 내부에서 SetOwner를 통해 Owner 설정<br>
    - 컨트롤러 설정<br>
  - 변경된 Owner로 인해 OnRep_Owner (클라) 호출<br>

```cpp
Controller.Possess(Pawn)
    → Pawn::PossessedBy(ServerController)
    → Pawn.SetOwner(ServerController)
    → Replication 시스템에서 Owner 변경 감지
```

- 타이밍에 따라 다를수 있으나<br>
  PostNetInit() 호출 후, 추가적으로 OnRep_Owner()가 호출될 수 있음<br>
  (Owner 역시 클라에 복제되어 있다면)<br>

```cpp
(처음 Pawn 생성될 때)
Pawn.PostNetInit()
Pawn.OnRep_Owner()     // Owner가 이미 있는 상태에서 생성되었다면 호출됨

(그 이후 서버에서 Owner 변경 패킷 도착)
Pawn.OnRep_Owner()     // Possess 효과가 클라이언트에서 여기서 일어남
```

## NetLoadOnClient

[![Image](https://github.com/user-attachments/assets/412693f8-266b-4f91-bbe3-216893f231cd)](https://github.com/user-attachments/assets/412693f8-266b-4f91-bbe3-216893f231cd){: .image-popup}<br>

- 레벨 디자인을 통해<br>
  레벨에 고정적으로 배치되는 액터는 해당 옵션을 true로 지정해<br>
  모든 Client가 스스로 Spawn 시키도록 설정<br>
  (로그인 시 호출되는 PlayerController와 혼동하지 말것!)<br>

- 클라리언트가 접속할 때, 월드에 이미 존재하는 해당 옵션이 true인<br>
  Actor를 클라이언트 쪽에도 생성<br>

- Level에 이미 배치한 Actor 나<br>
  런타임 중 생성한 Actor 등에 true를 주어 사용<br>

## Replication Notify

- 서버에서 실행되지 않고 클라에서만 실행되는 특징이 있음<br>
  (서버에서 실행 시, 명시적인 호출 필요)<br>

- 속성값이 변경되어서 변경된 값이 클라에 복제되는 시점에<br>
  Callback 함수 바인딩 가능<br>
  - 이것을 Replication Notify 함수라 함<br>
  - C++ : OnRep_<br>
  - BP : RepNotify<br>

| 구분 | C++ OnRep 함수 | Blueprint RepNotify |
|------|----------------|----------------------|
| 호출 위치 | 클라이언트에서만 자동 호출 | **서버 + 클라이언트 모두 호출** |
| 명시적 호출 | C++에서 직접 호출 가능 | 명시적 호출 불가능 |
| 호출 조건 | 값이 변경된 경우에만 호출 | **서버: 항상 호출** / **클라이언트: 값 변경 시 호출** |

## NetUpdateFrequency

- 서버에서 클라이언트로 1초당 몇 번 Replication 정보를 전송할지를 표현하는 변수<br>
  - 기본값 100(초당 100번)<br>
  - 다만 '이론상'의 값이며, 이를 보장하진 않음<br>
    (서버의 Tick Rate에 따라 Replication이 발생하나,<br>
     서버의 성능에 따라 달라짐)<br>
    -> 그렇기에 렌더링 기능이 없는 Dedicated Server가 비교적 좋은 성능을 발휘 가능<br>

- Pawn, PlayerController 등의 기본값 : 100<br>
  GameState : 10<br>
  PlayerState : 1<br>

- 해당 변수의 비율을 낮추어 서버의 성능을 개선할 수 있음<br>

- 또한 액터의 처음 서버 위치와 속도 값을 안다면<br>
  이를 이용하여 '예측'하여 동기화가 가능<br>
  (보간)<br>

- MinNetUpdateFrequency?<br>
  - Replication 업데이트가 '느리게' 발생하지 않도록 제한하는 Actor 변수<br>
  - 최소한의 업데이트 빈도를 보장하는 용도의 변수이다<br>
    (실제 업데이트 Frequency = max(NetUpdateFrequency,MinNetUpdateFrequency))<br>

## Relevancy(연관성/관련성)

[![Image](https://github.com/user-attachments/assets/5d322ed2-54e1-4cbb-916f-a0ae1c306a6f)](https://github.com/user-attachments/assets/5d322ed2-54e1-4cbb-916f-a0ae1c306a6f){: .image-popup}<br>

- 레벨에 있는 모든 액터의 정보를 클라에게 실시간으로 전송하는 것은<br>
  꽤나 무거운 작업<br>
  - 그렇기에 해당 클라이언트 커넥션과 '연관성'이 있는 액터만을 Replication을 해준다<br>

### 연관성의 판단 기준(Actor)

- Owner<br>
  - 해당 Actor를 소유하고 있는 Actor(Owner)<br>
    (ex : 무기 액터를 소유한 캐릭터)<br>
  - 거리 기반 Relevancy를 무시하고 항상 Replicate 됨<br>
    (내 소유 Actor는 항상 레플리케이션)<br>

- Instigator<br>
  - 해당 Actor에 영향을 끼친 Pawn<br>
    (ex : 데미지를 가한 Pawn)<br>
  - 보통은 OwnerShip Chain을 따라 연관성에 포함되는 편<br>

- AlwaysRelevant<br>
  - 해당 Actor가 모든 Client에 연관성이 있게끔 설정<br>
    (ex : 맵 전체에 보여야 하는 보스 몬스터 등)<br>

- NetUseOwnerRelevancy<br>
  - Owner 액터의 연관성으로 해당 Actor 연관성을 대신할 때 사용<br>
    (소유한 무기 Actor 등은 장비한 캐릭터가 보일때 같이 보여야 함)<br>

- OnlyRelevantToOwner<br>
  - Owner 액터에게만 연관성을 가짐 (다른 액터와는 연관성 x)<br>
    (길찾기 Actor - 다른 플레이어가 나의 편의성 액터를 볼 필요 없음)<br>

- NetCullDistance<br>
  - View와의 거리에 따라 연관성 여부 결정<br>
    (아주 먼 곳의 고블린 Actor를 가까우면 레플리케이션, 멀면 안하기)<br>

[![Image](https://github.com/user-attachments/assets/0611676e-b5d0-464e-a2aa-7988cd3db16a)](https://github.com/user-attachments/assets/0611676e-b5d0-464e-a2aa-7988cd3db16a){: .image-popup}<br>

### 연관성의 추가 판단 기준 (Pawn/Character/PlayerController)

- Viewer<br>
  - 클라이언트 커넥션이 소유한 Player Controller<br>

- ViewTarget<br>
  - 플레이어 컨트롤러가 빙의한 Pawn<br>
  - Viewer 에 의해 빙의된 ViewTarget이 레벨을 돌아다님<br>

[![Image](https://github.com/user-attachments/assets/15d97cd7-9c08-4e0d-9305-e0e0fa64ae72)](https://github.com/user-attachments/assets/15d97cd7-9c08-4e0d-9305-e0e0fa64ae72){: .image-popup}<br>

- 관련 함수<br>

```cpp
virtual bool AActor::IsNetRelevantFor(
    const AActor* RealViewer, // Connection이 가진 Pawn (Player의 아바타)
    const AActor* ViewTarget, // PlayerCameraManager가 바라보는 대상
    const FVector& SrcLocation
) const;
```

- 플레이어가 빙의한 RealViewer는 항상 연관성이 있기에<br>
  Replicate 됨<br>

- Player의 ViewTarget 역시 항상 연관성이 있는 편<br>
  (Replicate 됨)<br>

## NetPriority

- 한정된 '대역폭'(NetBandWidth) 내에서<br>
  서버가 어떤 Actor를 먼저 Replicate 할 지 결정하는 가중치<br>
  - 순서를 정하는데만 사용<br>
    (우선순위를 정하는 것이며, 서버 상황에 따라 모두 정상적으로 보내지기도 함)<br>
    ('포화 상태'가 아니라면 기본적으로 모두 보내질 수 있음)

- 단독으로 쓰이기 보단<br>
  NetUpdateFrequency와 마지막으로 갱신되었는지 등의 요소도 같이 사용됨<br>
  - 또한 NetCullDistance 같은 '연관성' 속성도 반영될 수 있음<br>

- 포화 상태?<br>
  : 보낼 액터 데이터들의 총량이 커서 '대역폭'을 넘어선 경우를 뜻함<br>
    (패킷 용량이 꽉찼다!)<br>
    - 이 경우에 NetPriority에 따라 보낼 데이터를 결정<br>

[![Image](https://github.com/user-attachments/assets/761151d8-d114-4acd-aef1-ebb832527e90)](https://github.com/user-attachments/assets/761151d8-d114-4acd-aef1-ebb832527e90){: .image-popup}<br>

## NetDormancy (휴면)

- Actor의 Replication과 RPC를 멈추게 하는 설정(NetDormancy 설정)<br>
  - 자주 수정되지 않는 Actor에 적합<br>
  - 너무 자주 수정되는 Actor에 적용시, 오히려 오버헤드가 발생 가능함<br>

```cpp
enum class ENetDormancy : uint8
{
    DORM_Never,            // 절대 Dormant 하지 않음. 항상 Replicate.
    DORM_Awake,            // 활성 상태, Replicate 대상, 휴면 가능 (기본)
    DORM_DormantPartial,   // 일부 클라만 Dormant
    DORM_Initial,          // 휴면 상태로 시작하고, 필요할 때 깨울 수 있는 상태
    DORM_DormantAll,       // 모든 클라이언트에게 Dormant
};
```

- Conditional Property Replication<br>
  : 특정 UPROPERTY의 레플리케이트 변수가 '조건'에서만 전송되도록 하는 기능<br>
    (일반적으론 Replication 등록되면, 해제할 수 없기에 이러한 조건 분류를 통해<br>
     세밀한 조정을 함)<br>
    - 변수마다 Replication 전송 조건을 설정<br>

```cpp
COND_None                // 항상 전송(기본)
COND_InitialOnly         // Actor 최초 생성 시에만 전송
COND_OwnerOnly           // Owner 클라이언트에게만 전송
COND_SkipOwner           // Owner 제외 모든 클라이언트에게만
COND_SimulatedOnly       // SimulatedProxy에게만
COND_AutonomousOnly      // AutonomousProxy에게만
COND_SimulatedOrPhysics  // SimulatedProxy + Physics
COND_InitialOrOwner      // 초기 + Owner에게만
COND_Custom              // C++ Override 시 직접 조건 선택
```