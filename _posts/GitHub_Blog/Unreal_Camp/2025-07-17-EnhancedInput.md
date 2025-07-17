---
title: "Enhanced Input System"
last_modified_at: "2025-07-17T16:30:00"
categories:
  - 언리얼 5
tags:
  - Input Actions
  - Input Mapping Contexts
  - Input Modifiers
  - Input Triggers
---

## 향상된 입력 시스템(Enhanced Input System)
UE5에선 새로운 입력 시스템을 제공하며<br>
기존 UE4의 입력 시스템을 보완하여 복잡한 입력 처리 요구 사항에<br>
대응하기 위하여 설계되었다<br>


| 시스템     | UE4 (Legacy Input)                   | UE5 (Enhanced Input)                  |
| ------- | ------------------------------------ | ------------------------------------- |
| 키 매핑 관리 | `Project Settings > Input`에서 키 이름 등록 | `Input Mapping Context` 에셋에서 액션 기반 설정 |
| 바인딩 방식  | 키 이름 직접 비교 (`"Jump"` 등)              | `InputAction` 객체를 이벤트에 바인딩            |
| 유연성     | 고정된 축/액션, 키만 연결                      | 입력 종류, 조건, 장치, 상태 기반 처리 가능            |
| 장점      | 단순, 직관적                              | 계층적, 강력한 시스템 확장성                      |
| 단점      | 커스터마이징 어려움                           | 구조가 복잡하고 진입장벽 ↑                       |


---

'입력 시스템'에 기반하기에<br>
'Local Player Subsystem'과 'Player Controller'와도 밀접한 관계를 가진다<br>

| 요소                         | 역할            | 비고              |
| -------------------------- | ------------- | --------------- |
| **Enhanced Input System**  | 입력 액션 정의 및 매핑 | 데이터 중심 구조       |
| **Local Player Subsystem** | 입력 매핑의 적용과 제어 | 플레이어 단위         |
| **Player Controller**      | 입력의 최종 바인딩 처리 | 입력 이벤트를 수신하는 주체 |


---

### SubSystem?
 : 특정한 범위(엔진, 게임, 레벨, 플레이어 등)에 '종속'된 유일무이한 개체(Singlton)<br>
  전역적인 접근이 가능하며, 해당 범위 안에서 '상태', '기능', '서비스' 등을 관리<br>
  (다양한 범위의 상태를 Subsystem으로 분리하여 관리)<br>


| 서브시스템 종류                    | 클래스                      | 설명                           |
| --------------------------- | ------------------------ | ---------------------------- |
| **Engine Subsystem**        | `UEngineSubsystem`       | 엔진 전체에서 하나, 모든 게임 전역 공유      |
| **Game Instance Subsystem** | `UGameInstanceSubsystem` | 게임 실행 단위 (메뉴 ↔ 인게임 사이 공유 가능) |
| **World Subsystem**         | `UWorldSubsystem`        | 월드 단위 (레벨마다 개별 인스턴스)         |
| **Local Player Subsystem**  | `ULocalPlayerSubsystem`  | **플레이어별** 상태 관리              |
| **Editor Subsystem**        | `UEditorSubsystem`       | 에디터 전용 기능 확장용 (툴 작성용)        |

그 중 '입력'과 관계있는 LocalPlayer SubSystem은<br>

-  **플레이어 전용 상태/서비스 관리**       : Enhanced Input Context, HUD 설정, 플레이어 설정 <br>
-  **로컬 입력과 관련된 작업 처리**        : 키 매핑 컨텍스트 추가/제거                         <br>
-  **멀티플레이에서 각 플레이어 별로 동작 분리** : Player1은 UI 띄우고, Player2는 게임 입력 유지    <br>

등의 기능을 제공한다<br>


## Input Action
 입력 동작(ex : Jump, Move, Look 등) 그 자체를 '객체'화한 개념으로서<br>
 '동작의 의미'와 '입력 장치'를 분리하고<br>
 하나의 'Input Action'에 여러 키 바인딩이 가능하게 한다<br>

  - Trigger<br>
    : 이벤트를 언제 발동시킬지에 대한 조건 지정<br>
     (ex : pressed, Released, Hold, Double Tap 등)<br>
  
  - Modifiers<br>
    : 입력값을 어떻게 변형/보정 할지에 대한 설정<br>
     (ex : Invert Axis, Scale by Constant,Deadzone 등)<br>


## Input Mapping Contexts
 여러 InputAction들을 어떤 키나 장치 입력에 연결할지 정의하는 에셋<br>

예시를 들자면<br>

| Input Mapping Context | Input Action | 키 바인딩                |
| --------------------- | ------------ | -------------------- |
| IMC\_Player           | IA\_Jump     | Spacebar / Gamepad A |
|                       | IA\_Move     | WASD / Left Stick    |
|                       | IA\_Shoot    | Left Mouse / RT      |

위와 같은 방식으로 Input Action을 '한 묶음(Context)'로 만들어<br>
'게임 중'(Dynamic)에 '적용/제거'를 통해 입력 방식을 바꿀 수 있다<br>

| 상황             | 예시                                        |
| -------------- | ----------------------------------------- |
| **게임 ↔ UI 전환** | IMC\_Player ↔ IMC\_UI (게임 입력 끄고 UI 키기)    |
| **캐릭터 ↔ 탈것**   | IMC\_Character ↔ IMC\_Vehicle             |
| **스킬 휠 모드**    | IMC\_Normal ↔ IMC\_SkillWheel (조작 다르게 구성) |
| **플랫폼 전용**     | Gamepad 전용 컨텍스트, VR 전용 컨텍스트 등 분리 가능       |

'우선순위(Priority)'가 존재하기에<br>
여러 Context가 동시에 적용되는 경우 우선순위가 높은 Context가 우선 적용되기에<br>
명확한 관리가 필요한 기능<br>

## 정리
UE4에선 하나의 '고정'키에 1:1 바인딩이 되며<br>
특정 상황에 맞게 각 키마다 예외처리를 했어야 하였음<br>
(전진 역할인 w 키가 UI 상황에는 입력을 무시해야 한다던가)<br>

그렇기에 UE5에선<br>
입력과 키 바인딩을 분리하여 좀 더 모듈화된 구조로 개선<br>

UE5에서 실제로 적용되는 예시는 다음과 같음<br>

1. InputAction 객체: "Jump" 같은 행동 정의<br>

2. InputMappingContext: Spacebar → JumpAction 등의 키 바인딩 구성<br>

3. Subsystem에서 Context를 플레이어에게 적용<br>

4. EnhancedInputComponent에서 InputAction 바인딩<br>

5. 입력 발생 → 해당 액션 발동 → 캐릭터/로직 작동<br>
