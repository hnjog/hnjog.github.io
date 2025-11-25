---
title: "Unreal Nets"
date : "2025-11-25 18:00:00 +0900"
last_modified_at: "2025-11-25T18:00:00"
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
bool AGameModeBase::PreLogin(...);  // 접속 허가 여부
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