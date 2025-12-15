---
title: "김하연 튜터님 강의 - 'State Tree와 Mass AI 차세대 군중 시뮬레이션'"
date : "2025-12-15 12:00:00 +0900"
last_modified_at: "2025-12-15T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - State Tree
  - Mass AI
---

# State Tree와 Mass AI에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

- Mass Entity 라고도 표현함<br>

- MassAI, MassEntity, Mass Crowd, MassGameplay 등의 플러그인을 설치<br>
  - MassEntity : 본체<br>

- ZoneGraph도 켜야 함<br>
  (5.5 기준?)<br>

## 1. Mass AI - Zone Graphs 🗺️

### Mass AI란?

**대규모 군중 시뮬레이션 시스템**으로,<br>
Entity Component System (ECS) 아키텍처를 사용<br>
([ECS?](https://hnjog.github.io/unreal/c++/DataBaseStruct/))<br>

**전통적인 AI vs Mass AI**<br>

| 구분 | 전통적인 AI | Mass AI |
| --- | --- | --- |
| 구조 | 개별 액터 + AI Controller | 경량 Entity + ECS |
| 적합 규모 | 수십 명 | 수천 명 |
| 성능 | 개별 처리 (느림) | 일괄 처리 (빠름) |

- Entity : AI와 유사하다고 일단 생각<br>

### Zone Graph의 개념

**Zone Graph는 NPC의 이동 경로와 영역을 정의하는 내비게이션 시스템**입니다.<br>

- **NavMesh**<br>
  : "어디를 걸을 수 있는가?"<br>
    (어디까지 갈 수 있는가?)<br>
  
- **Zone Graph**<br>
  : "어떤 경로를 따라 걸을 것인가?"<br>
    (ex - 고속도로 차선)<br>
    - 규칙이 있는 하나의 길<br>
      (도시에서 AI들이 규칙에 따라 이동(인도, 횡단보도 등))<br>

### Zone Graph 구성 요소

Alt를 눌러 추가 조작이 가능<br>
- Show -> Zone graph를 킴으로서<br>
  선택하지 않은 Zone Graph도 볼 수 있음<br>

**1) Zone Shape (형태)**<br>

- **Spline (스플라인)**<br>
  : 길, 보도, 복도 같은 `선형 경로`<br>

- **Polygon (폴리곤)**<br>
  : 광장, 공원, 건물 내부 같은 넓은 영역<br>

**2) Lanes (차선)**<br>

각 Zone Shape 내의 세부 이동 경로:<br>
(Project Settings - Engine - ZoneGraph에서 설정 가능)<br>

- **Width**: 차선의 폭<br>
- **Direction**: Forward(전진), Backward(후진), Both(양방향)<br>
- **Tags**: 접근 권한 제어<br>
- **Speed**: 속도 제한 (선택사항)<br>

**3) Tags (태그)**<br>

`NPC 유형`과 `경로`를 '연결'하는 분류 시스템:<br>
(Project Settings - Engine - ZoneGraph에서 설정 가능)<br>

- `Pedestrian`: 보행자용 경로<br>
- `Vehicle`: 차량용 도로<br>
- `HighClass`: VIP 전용 구역<br>

### 태그 시스템 활용

**계층화된 세계 구축**<br>

```
일반 농민 → Pedestrian 태그만 → 공공 도로만 접근
상인 → Pedestrian + Merchant 태그 → 상점 내부도 접근
귀족 → All 태그 → 성과 정원까지 접근
```

- 엔진에서 설정하는 것 말고<br>
  ZoneShape 내부의 Tag 자체도 존재함<br>
  - 세부적인 설정용도의 Tag<br>
    - 같은 보행자용 도로이지만,<br>
      마을 안/ 위험지대 도로 구분 등의 용도로 구분 가능<br>

## 2. Mass Entity Config Asset ⚙️

도로를 깔았으니 그 규칙을 따를 Entity를 만들기<br>

### Config Asset이란?

- *Mass Entity의 `설계도`<br>
- Blueprint가 액터를 정의하듯, Config Asset은 *Mass 엔티티*를 정의<br>

- MassEntityConfigAsset 으로 생성<br>

### 핵심 개념

여러 '특성'을 조합하여 Entity를 만듦<br>

**1) Fragments (프래그먼트)**<br>

엔티티가 보유하는 `데이터` 조각:<br>

- **Transform Fragment**: 위치와 회전<br>
- **Velocity Fragment**: 속도<br>
- **Health Fragment**: 체력<br>

**2) Traits (특성)**<br>

엔티티에 `기능`을 추가하는 모듈:<br>

- **Movement Trait**: 이동 기능<br>
- **Avoidance Trait**: 충돌 회피<br>
- **Animation Trait**: 애니메이션<br>

MassAI 관련 Traits<br>
- MassActorFragment : Mass AI 관련하여 필요한 프래그먼트<br>
- AgentRadiusFragment : NPC의 충돌 범위를 지정<br>
- MassViewInfoFragment : LOD 관련 정보를 저장 (최적화)<br>
- AgentCapsuleCollisionSync : 아래의 '동기화' 관련하여 필요한 기능<br>

Crowd 관련<br>

- CrowdMember<br>
- Crowd Visualization<br>
  
이런 대량의 AI 를 다루는 경우 LOD 같은 최적화 옵션을 잘 체크해야 함<br>

**3) Processors (프로세서)**<br>

*Fragments를 처리하고 업데이트*하는 시스템<br>

- 이러한 Entity들을 '한번에' 처리하는 거대한 로직<br>
  (Entity는 '정보'만, 로직을 프로세서가 처리)<br>

### Traits와 Fragments의 관계

```
Movement Trait 추가
  ↓
자동으로 추가되는 Fragments:
  - Transform Fragment
  - Velocity Fragment
  - Movement Parameters Fragment
  ↓
Movement Processor 활성화
```

**모듈식 설계의 장점**<br>

- 필요한 기능만 선택적 추가<br>
- 메모리 효율성 극대화<br>
- 다양한 NPC 유형 쉽게 생성<br>

### 자동으로 작동하는 시스템들

### Config Asset에서 추가한 Traits 덕분에

- **Avoidance Trait**: 다른 NPC와 충돌 예측 및 회피<br>
- **Movement Trait**: 부드러운 이동 처리<br>
- **Steering Trait**: 자연스러운 회전<br>
- **Navigate Obstacle Trait**: 장애물 우회<br>

**추가 코드 없이 자동으로 작동!**<br>

## 3. Sync (동기화) 🔄

- MassEntity : Entity 기준으로 동작<br>
- Actor : Actor 기반의 실제 동작<br>

이 둘의 행동을 실제로 동기화 해야 함<br>

### Sync란?

**Mass Entity(데이터)와 Visual Actor(3D 캐릭터) 사이의 정보 동기화**<br>

```
Mass Entity (두뇌)
    ↕️ Sync
Visual Actor (몸)
```

- Sync Transform은 끄는편이 좋을 수 있음<br>

### 동기화 방향

- **Mass to Actor**: Mass 계산 → 액터 적용 (가장 일반적)<br>
- **Actor to Mass**: 액터 상태 → Mass 전달<br>
- **Both Ways**: 양방향 동기화<br>

### 예시

**Agent Movement Sync (Mass to Actor):**<br>

```
Mass가 새 위치 계산 → Visual Actor를 그 위치로 이동
```

**Player Navigation Obstacle (Actor to Mass):**<br>

```
플레이어 이동 → Mass 시스템에 위치 알림 → NPC가 회피
```

여기까지 왔으면<br>
도로와 그 위의 데이터 세팅은 어느정도 한 편<br>

- 적당한 캐릭터 BP 생성<br>
- Config 쪽의 Visualization에서<br>
  고/저해상도 캐릭터에 해당 캐릭터를 넣어줌<br>
  - Params의 LOD 세팅 중 StaticMeshInstance로 설정하는 경우, 멀리 있으면 T-Pose 같은 표시가 될 수 있으므로 주의<br>

- *ZoneGraph Navigation*<br>
  : Lane을 실제로 걸을 수 있게 하는 Trait<br>
    Tag를 조합<br>
    Any : or 조건 (하나라도 만족하면 가능)<br>
    All : and 조건 (전부 만족할때 가능)<br>
    Not : Not 조건 (이 태그 있으면 절대 안감)<br>

- Movement를 설정하여 움직임에 대한 수치 추가 설정<br>
  (있어야 함!)<br>

- Mass Spawner를 이용하여 실제로 생성<br>
  - config와 수량을 세팅<br>
  - ZoneGraph를 통해 Spawn Data 설정하기<br>

- Zone Graph 빌드까지 하면 Zone Graph 위를 걸을 수 있음<br>

- 다만 State Tree 설정은 아직 안했기에 실제로 걷지 않음<br>
  (기능을 알지만, 실제 판단 주체가 없는 상황)<br>
  (BT는 사용 x)<br>
  - 성능 이슈<br>
    (AI 한마리당 AI Controller가 필요)<br>

## 4. Wander (배회) State 구현 🚶

### Wander의 두 단계

1. **목표 찾기**: 어디로 갈지 결정<br>
2. **이동하기**: 그곳까지 경로 따라 이동<br>

### Task 구성

**Task 1: ZG Find Wander Target**<br>

- 주변 도로의 정보를 읽을 수 있고<br>
  임의의 위치 선택<br>

```
현재 위치에서 Zone Graph 검색
  ↓
태그 필터링 (Pedestrian, HighClass)
  ↓
무작위 지점 선택
  ↓
Output: Wander Target Location

```

**Task 2: ZG Path Follow**<br>

- 실제 이동<br>
- Task 하나당 State 하나씩 파는게 좋지 않나?<br>
  - 내부적으로 자동 연동이 됨<br>
  - 따로 파면 전역 변수 하나 파야 함<br>
  - 단순한 무한루프 용이기에 그 부분은 괜찮음<br>

```
Input: Wander Target Location (Task 1의 출력)
  ↓
경로 계산
  ↓
경로 따라 이동

```

### Transition 설정

**On State Completed → Transition to Root**<br>

```
Wander 완료 → Root로 돌아가기 → Wander 재시작 → 무한 반복
```

## 5. Mass AI 디버깅 🎨

### 디버그 명령어 (게임 내 ' 키)

- **기본 뷰**: Mass 데이터 + 위치<br>
- **Shift + V**: 속도, 상태 등 추가 정보<br>
- **Shift + O**: 회피(Avoidance) 시각화<br>
- **Shift + C**: 경로(Zone Graph) 표시<br>
- **Shift + S**: Shape 표시<br>

### 시각적 요소

- **원**: Mass 엔티티 위치<br>
- **메시**: Visual Actor 위치<br>
- **화살표**: 이동 방향과 의도<br>
- **노란색 선**: 목표 방향 (Smooth Orientation)<br>
- **작은 선들**: 회피 벡터<br>
- **Maroon 화살표**: 원하는 목적지<br>

## 6. 플레이어-NPC 상호작용 🤪

### 문제<br>

- NPC는 다른 Mass 엔티티만 인식<br>
- 플레이어는 Mass 엔티티가 아님<br>
- 결과: NPC가 플레이어를 "보지 못함"<br>

### 해결: Navigation Obstacle<br>

- Navigation Obstacle?<br>
  : 장애물로 인식하는 Trait<br>

**플레이어를 Mass 시스템에 등록**<br>

```
Player Config Asset 생성
  ↓
Traits 추가:
  - Agent Capsule Collision Sync (Actor to Mass)
  - Navigation Obstacle
  ↓
플레이어 위치를 매 프레임 Mass에 전달
  ↓
NPC가 플레이어를 회피
```

#### Mass AI 는 CharacterMovement를 이용하지 않음

- 그렇기에 ABP 설정 시엔 해당 부분에 유의할 것<br>

#### AI Controller는 사용하지 않음

- BT를 사용하지 않는 이유가 AI Controller를 개당 사용하게 되므로<br>
  AI Controller 자체를 하나씩 생성하지 않기 위해 사용하는 기능<br>

- Entity라는 데이터 덩어리를 Processer가 움직여 주는 방식임<br>

- 세부 로직은 State Tree를 이용하여 로직 처리가 가능함<br>

#### 추가 질문들

- **Mass Entity 의 요소를 런타임 중에 추가/제거 등도 가능함**<br>

- Mass Spawn으로 태어나는 요소들은 Actor가 아니며 Entity라는 매우 가벼운 요소임<br>
  - 거듭 말하듯 Data(Fragment)들을 기반으로 Processor가 움직여 주는 것<br>
  - 다만 필요에 따라 MassActorTrait 등을 통해 실제 Actor 등으로 변환 가능함<br>

- 그래도 State Tree를 이용하여 전투 로직을 짤 순 있음<br>
  (GAS 연동시 ASC 를 달아야 하기에 Actor 승격이 필요함)<br>
  (AI끼리만 싸운다면 GAS 대신, 자체적인 대미지 처리를 고려)<br>
  - AI 끼리는 자체적인 대미지 처리<br>
  - AI <-> Player 의 경우에, AI를 Actor로 승격하여 GAS를 통해 대미지 처리하기<br>

- Mass Entity끼리 공격이 가능함<br>
