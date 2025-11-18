---
title: "Unreal NetMode, NetConnection, NetDriver"
date : "2025-11-18 18:00:00 +0900"
last_modified_at: "2025-11-18T18:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Server
  - NetMode
  - NetDriver
  - NetConnection
  - NetRole
---

## Unreal의 네트워크 구조 이해를 위해 필요한 것들

NetMode, NetDriver, NetConnection, NetRole 등의 구조를 이해해야<br>
Dedicated Server 기반의 게임 제작하는데 문제가 없다<br>

기본 개념 정리<br>

``` arduino
NetMode → 이 클라이언트/서버 프로그램이 어떤 상태인가? (Dedicated? Client?)
NetRole → 이 Actor가 현재 어떤 권한(Role)을 가지고 있는가?
NetDriver → 실제 네트워크를 관리하는 엔진의 드라이버(네트워크 백엔드)
NetConnection → 한 클라이언트와 서버 사이의 실제 연결 객체
```

## NetMode

현재 프로세스의 `네트워크 모드`를 뜻함<br>
우리가 작성중인 로직이 '서버'에서 도는지 '클라'에서 도는지를 분류하기 위해<br>
필요한 개념!<br>

- 멀티 플레이에선 '같은 코드'가 여러 PC에서 동작하며<br>
  이를 위하여 각 PC의 역할 구분이 필요함<br>

### NetMode의 종류

| NetMode                | 의미                                  |
| ---------------------- | ----------------------------------- |
| **NM_Standalone**      | 싱글플레이 or 로컬 플레이                     |
| **NM_ListenServer**    | Listen Server (서버+호스트 플레이어가 같이 있음). |
| **NM_DedicatedServer** | Dedicated Server (호스트 없는 순수 서버).    |
| **NM_Client**          | 네트워크 클라이언트.                         |

- StandAlone<br>
  : 원격 클라 연결을 허용하지 않는 서버<br>
    (싱글 or 로컬 플레이 게임에 주로 사용)<br>

- Client<br>
  : 게임이 네트워크 멀티 플레이 세션에서 '클라이언트'로 실행됨<br>
    서버측 로직 실행 x, 서버로부터 복제된 proxy를 보여줌<br>

- Listen Server<br>
  : 원격 클라이언트의 연결 수락하고, 자신의 로컬 플레이어도 서버에 배치<br>
    서버 프로세스도 게임에 참여함<br>
    캐주얼 협동 같은 멀티 플레이어에 자주 사용<br>

- Dedicated Server<br>
  : 원격 클라이언트의 연결을 허용하나, 자신의 로컬 플레이어는 x<br>
    그래픽, 사운드, 입력 및 기타 플레이어 위주의 기능을 삭제한 방식<br>
    지속적이고 안전한 대규모 멀티 플레이 게임에 사용<br>

#### NetMode에 따른 액터 위치

- 서버에만 존재하는 Actor<br>
  : GameMode<br>
  - 클라에서 GetGameMode() 함수 호출시 Nullptr 반환이 됨<br>

- 서버 + 모든 클라에 존재하는 액터<br>
  : 배경용 Actor와 Pawn<br>

- 서버와 클라에만 존재하는 액터<br>
  : Player Controller, ABP<br>
  - ABP는 동작 종료 등을 체크할 때, 사용할 수 있음<br>

- 클라에만 존재하는 오브젝트들<br>
  : UI 등 렌더링에 필요한 요소들<br>


### 예시 코드

```cpp
ENetMode Mode = GetWorld()->GetNetMode();
```

이 외에도<br>
IsDedicatedServer 와 같은 다양한 함수가 존재<br>

## NetDriver

`언리얼의 네트워크 엔진 (소켓, 전송 기능)`<br>

네트워크 연결을 실제로 관리하는 백엔드 드라이버<br>

- 언리얼 네트워크 통신에서 로우레벨 동작을 관리하는 클래스<br>
- 싱글 플레이에선 `UNetDriver` 객체 생성 x<br>
- 멀티 플레이에서만 `UWorld::Listen()` 함수를 통해 `UNetDriver` 객체가 생성<br>
- 각 PC마다 `UNetDriver` 객체 생성<br>

``` scss
World
 └── NetDriver
       ├── ServerConnection (서버에서만)
       └── ClientConnections[] (클라이언트 목록)
```

### NetDriver의 종류

| NetDriver            | 용도                     |
| -------------------- | ---------------------- |
| **GameNetDriver**    | 게임 멀티플레이를 위한 기본 드라이버   |
| **IpNetDriver**      | IP 기반의 네트워크 TCP/UDP 담당 |
| **DemoNetDriver**    | 리플레이/데모 재생용            |
| **PendingNetDriver** | 접속 시도 중인 네트워크          |


## NetConnection

`클라이언트와 서버의 1:1 연결`<br>

서버는 여러 ClientConnections 를 가짐<br>
클라는 하나의 ServerConnection만 가짐<br>

- 다른 PC와의 연결이 발생하면 `UNetConnection` 객체가 생성<br>
- 서버에 클라이언트가 접속하면 서버에 `ClientConnection` 객체 추가<br>
  - 반대로 클라엔 `ServerConnection` 객체 생성<br>

- 클라 <-> 서버는 `UNetConnection` 객체를 통해 통신함<br>
- `UNetDriver`는 생성된 `UNetConnection` 객체를 소유, 관리<br>
- 서버 PC에 생성된 `UNetDriver`는 접속한 클라이언트의 수만큼 관리하게 됨<br>

- 클라 PC의 UNetDriver는 ServerConnection 하나만을 관리함<br>

용어 정리<br>
- ServerConnection : 클라이언트가 소유한 UNetConnection 객체<br>
- ClientConnection : 서버가 소유한 UNetConnection 객체<br>

결국 UNetConnection을 관리하며<br>
객체의 '네이밍'을 저렇게 한 것임<br>

즉,<br>

클라쪽의 상황<br>

``` scss
UWorld
 └── GameNetDriver (UNetDriver)
        └── ServerConnection : UNetConnection *
```

서버 쪽의 상황<br>

``` scss
UWorld
 └── GameNetDriver (UNetDriver)
        ├── ClientConnections[] : TArray<UNetConnection*>
        │        [0] → 플레이어1
        │        [1] → 플레이어2
        │        [2] → 플레이어3
        └── ServerConnection = nullptr
```

## NetRole

`이 Actor가 이 머신(서버/클라이언트)에서 어떤 권한을 가졌는가?`<br>

- Actor의 권한 상태(Role)을 나타내는 개념<br>
  - Actor의 진짜 주인에 대한 이야기이기도 하다<br>

예시를 들자면)<br>

이 Actor의 주인은 Server,<br>
이 Actor는 서버의 복제를 받아 움직이는 상황,<br>
이 Actor는 클라이언트가 입력을 할 수 있는 Actor이다 등<br>

-> 이 Actor의 '권한'은 누구에게 있는지를 표현하는 개념<br>

### NetRole의 2가지 Role

- LocalRole<br>
  : 현재 머신(서버/클라)에서의 Actor 권한<br>
   - 서버는 기본적으로 모든 Actor에 대하여 '절대 권한'을 가지는 편<br>
   - 클라에선 각 Actor의 역할이 크게 바뀌는 편이다<br>
     - 플레이어 조종 캐릭터 : ROLE_AutonomousProxy<br>
     - 다른 사람의 조종 캐릭터 : ROLE_SimulatedProxy<br>

- RemoteRole<br>
  : 현재 반대편의 네트워크 구조에서의 Actor의 권한<br>
    (클라 -> 서버 / 서버 -> 클라)<br>
   
### NetRole의 종류

| Role                     | 의미                        | 사용처                      |
| ------------------------ | ------------------------- | ------------------------ |
| **ROLE_Authority**       | 서버에서만 가지는 절대 권한           | 게임의 실제 상태 변경             |
| **ROLE_AutonomousProxy** | 클라이언트가 주도적으로 조작하는 Actor   | PlayerController, 본인 캐릭터 |
| **ROLE_SimulatedProxy**  | 서버의 복제를 “수동으로 따라가는” Actor | 다른 플레이어의 캐릭터             |


ex)<br>
서버에 A,B,C가 존재할때의 NetRole 예시<br>

| Actor    | LocalRole       | RemoteRole |
| -------- | --------------- | ---------- |
| A(내 캐릭터) | AutonomousProxy | Authority  |
| B(타인)    | SimulatedProxy  | Authority  |
| C(타인)    | SimulatedProxy  | Authority  |


- AutonomousProxy의 존재를 통해 해당 클라이언트에서<br>
  '입력'을 처리한 후, 서버에 RPC 요청을 할 수 있음<br>
  - 서버와 클라 모두 이동 처리를 진행하여 '부드럽게' 보일 수 있음<br>
  - 네트워크 지연(Latency)를 최소화하는 방식<br>

- AutonomousProxy는 '입력'으로 조종되는 단 하나의 Pawn 정도에만 쓰임<br>
  - 플레이어가 대규모 RTS 등을 통해 유닛을 대량으로 조정하는 경우도<br>
    '그 입력'을 받아 서버에서 처리 후, 클라에 동기화 하는 것임<br>

## OwnerShip

[![Image](https://github.com/user-attachments/assets/976a6ec7-9de0-4499-9fdc-d7ac7574ce21)](https://github.com/user-attachments/assets/976a6ec7-9de0-4499-9fdc-d7ac7574ce21){: .image-popup}<br>

- 하나의 `ClientConnection`은 하나의 `PlayerContoller`를 소유<br>
- 즉, `PlayerContoller`의 **Owning Connection**은 `ClientConnection`<br>
- `PlayerController`가 빙의하는 폰의 `Owner` 속성은 해당 플레이어 컨트롤러로 설정됨<br>
- 폰에 무기 Actor가 생성되고, 무기 액터의 Owner 속성에 해당 폰을 설정할 수 있음<br>
- ClientConnection ~ 무기 Actor 까지의 소유 관계를 '패밀리' 라고도 부름<br>
- 소유 관계 속에 있는 액터가 '본인'의 **Owning Connection**을 얻으려면<br>
  `AActor::GetNetConnection()` 호출<br>

- 이러한 소유 관계가<br>
  RPC와 Property Replication 과 연관됨<br>
  - OwnerShip에 대한 추가적인 내용은 해당 개념들을 포스팅할때 더 다룰 예정<br>
