---
title: "김하연 튜터님 강의 - 'State Tree로 만드는 차세대 AI 시스템'"
date : "2025-12-12 12:00:00 +0900"
last_modified_at: "2025-12-12T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - State Tree
---

# State Tree에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

- Unreal '5.3 ~ 5.4' 버전부터 본격적으로 사용 가능해진 기능<br>
- 사용 시, 활성화할 플러그인 필요<br>
  - `StateTree` , `GameplayStateTree` 등<br>

- 좋은 범용성을 가짐<br>
  - State tree는 AI뿐 아니라<br>
    UI, Mission 등의 '상태' 개념을 가진 요소에 구현 도움<br>
  - GAS 와도 궁합이 좋음<br>
    (Native 연동 - 별도의 연동 없이 GA,GE 가 발동)<br>

- BT가 점점 커질수록 까다로워짐<br>
  - 트리가 깊어질수록 '왜 이 행동을?' 싶은 동작의 원인을 찾기 힘듦<br>

- 차세데 AI 구현 개념에 가까워짐<br>
  - 큰 흐름을 '상태'로<br>
    세부 동작은 'task'가<br>
    (이러한 Task가 Gas와 연동하기 쉽게 해줌)<br>

- Smart Object 와 EOS 와 연동하여 더 세부적인 AI 구축도 가능함<br>

- 처음 공부할 때, 'BT' 관련 내용은 잊어버리기!<br>

# 1. State Tree 소개 및 기본 개념 😶‍🌫️

## 1-1. State Tree란?

State Tree는 Unreal Engine에서 제공하는 **상태 기반 제어 프레임워크**로,<br>
AI 행동뿐만 아니라 다양한 게임플레이 로직을 구현할 수 있는 시스템입니다.<br>

**핵심 특징**<br>

- **Behavior Tree의 유연한 태스크 실행 구조**<br>

- **State Machine의 직관적인 상태 전환 구조**<br>

- 두 시스템의 장점을 결합한 하이브리드 접근 방식<br>

## 1-2. 플러그인 종류

- **Gameplay State Tree**<br>
    - 게임플레이 로직 및 **AI 행동 제어**에 특화<br>
    - `Behavior Tree`를 **대체**하거나 **보완**하는 용도<br>
    - 일반적으로 *"AI State Tree"*라고 지칭<br>

- **State Tree (일반)**<br>
    - *범용*적인 '**상태 기계**' 프레임워크<br>
    - UI 애니메이션, 미션 진행, 게임플로우 등 **비AI 영역**에서도 활용<br>
    - `상태 전환이 필요한 거의 모든 상황`에 적용 가능<br>

## 1-3. Behavior Tree vs. State Tree

| 특징 | Behavior Tree | State Tree |
| --- | --- | --- |
| 구조 | 트리 기반 태스크 실행 | 상태 + 전환 기반 |
| 상태 표현 | 암묵적 | 명시적 |
| 전환 로직 | 복잡한 조건 분기 필요 | 직관적인 상태 전환 |
| 유지보수 | 복잡해질수록 어려움 | 구조적으로 명확 |
| GAS 연동 | 수동 구현 필요 | 자연스러운 연동 지원 |

# 2. State Tree 구성 요소 🛹

## 2-1. Schema (스키마)

- State Tree의 실행 컨텍스트를 정의하는 설정<br>

### StateTree Component

- **Actor 자체에 붙은 State Tree Component**에서 실행<br>
- **AI Controller 불필요**<br>
- 독립적으로 동작하는 Actor 단위 상태머신에 적합<br>
- 예: 상호작용 오브젝트의 상태 관리 (꺼짐 → 켜짐 → 파괴됨)<br>

### StateTree AI Component

- **AI Controller에서 실행**되는 State Tree<br>
- `Controller 하나가 여러 Pawn을 제어` 가능<br>
- `여러 Pawn`이 **동일한 AI 로직을 공유**할 때 유용<br>
- 예: 플레이어 근처에서 동시에 도망가는 여러 동물, 근접 미니언들 등<br>

### 커스텀 스키마 (C++)

- 특정 Actor 타입 전용 고유 데이터가 필요할 때 사용<br>
- 대부분의 경우 기본 스키마로 충분<br>
  - 카메라 로직 관련 커스텀 스키마도 존재<br>

## 2-2. Context (컨텍스트)

- State Tree가 동작할 대상 Actor를 지정하는 설정<br>
  (Class Type 지정)<br>

### **Context Actor Class**

- State Tree가 `어떤 Actor를 대상`으로 동작할지 지정<br>
- Task 내에서 자동으로 참조 가능<br>
- 구체적으로 지정할수록 Task 작성이 편리함<br>

### **AIController Class** (AI Component 전용)

- AI Controller가 `State Tree를 실행`<br>
- Controller = 로직을 실행할 '두뇌'<br>
- Context Actor = 조종하는 '몸체(Actor/Pawn)'<br>

## 2-3. Parameters (파라미터)

- State Tree 에셋에 정의하는 입력 변수<br>

- **특징**<br>
    - 외부에서 값을 주입할 수 있는 설정값<br>
    - 트리 시작 전에 세팅하는 `초기 설정`값<br>
    - 런타임 중간에 자주 변경하는 용도가 아님<br>

- **Behavior Tree Blackboard와의 비교**<br>
    - Blackboard = '실시간' 값 공유소 (동적)<br>
    - Parameters = 시작 전 주는 설정값 (정적)<br>

- **사용 예시**<br>
    - 순찰 거리, 탐지 반경, 기본 속도 등의 기본 세팅<br>
    - 같은 트리를 여러 캐릭터에서 다른 값으로 재사용<br>

- **Theme**?<br>
  : State에 '색'과 설명을 지정하여<br>
    차후 유지보수에 도움을 줌<br>
    (개발자를 위한 편의 기능)<br>

## 2-4. State (상태)

- 캐릭터 또는 오브젝트가 `현재 어떤 상태`에 있는가를 나타내는 노드<br>

- **특징**<br>
    - State Tree의 **기본 구성 단위**<br>
    - '트리 구조'로 배치되어 *상황에 따라 전환*<br>
    - `State` 자체는 '상황'을 나타내고, `Task`가 '행동'을 담당<br>

- **State 타입**<br>
    - **State:** Task를 실행할 수 있는 기본 단위 (주로 사용)<br>
    - **Group:** 상태를 묶어서 정리하는 용도<br>
    - **Linked/Linked Asset/Subtree:** 다른 트리를 불러오는 고급 기능<br>

- 내부에 GameplayTag 등을 지정이 가능함<br>
  - ex) AI.State.Content<br>
  - 보통 자식 State가 Tag를 물려받음<br>

- State Type?<br>
  - State : 일반적 사용<br>
  - Group : 자식별 그룹을 깔끔하게 관리하는 용도<br>
  - Linked : 서브 트리를 지정 (재사용할 트리 로직을 지정)<br>
    (함수 호출처럼 파라미터 넣고 그 트리 로직으로 던져버림)<br>
  - Linked Asset : 다른 StateTree를 '통째로' 불러와서 던져버림<br>
    (보스의 패턴이 페이즈마다 완벽히 달라질때 등)<br>
  - SubTree : 재사용을 위한 트리 용도<br>

## 2-5. Root State

- State Tree의 **시작 지점**이 되는 최상위 State<br>

- **특징**<br>
    - State Tree 생성 시 `자동 생성`<br>
    - **모든 실행은 Root에서 시작**<br>
    - Root 자체는 *보통 Task를 포함하지 않음*<br>
      (보통 Task를 넣어놓지 않는, 로직의 시작점 역할)<br>
    - Child State들을 관리하는 컨테이너 역할<br>

- **실행 흐름**<br>
    1. State Tree 실행 시작<br>
    2. Root 활성화<br>
    3. Root의 Child 중 하나 선택<br>
    4. 선택된 Child에서 행동 시작<br>

## 2-6. Child State vs Sibling State

### Child State (자식 상태)

- 계층적으로 "아래"에 있는 상태<br>
- Child가 선택되면 **부모도 함께 활성화**<br>
- 부모의 Task + 자식의 Task 동시 실행 가능<br>
- **용도:** 상위/공통 로직 + 세부 행동 레이어링<br>

**예시**<br>

```
Combat (부모: 타겟 유지/버프 관리)
├─ Chase (자식: 추적)
├─ Attack (자식: 공격)
└─ Retreat (자식: 이탈)
```

- Chase State가 돌아가고 있다면<br>
  Root - Combat - Chase '상태'가 모두 '활성화'!<br>

### Sibling State (형제 상태)

- Parent의 Selection Behavior에 따라 **하나만 선택**되어 활성화<br>
- 서로 **상호 배타적**인 모드/단계 표현<br>
- **용도:** 같은 맥락 아래 전환되는 여러 케이스 배치<br>

**선택 기준**<br>

- '공통 로직 유지' + '세부 행동' 변경 → **Child 구조**<br>
- 같은 단계의 `여러 대안 중 하나`만 실행 → **Sibling 구조**<br>

## 2-7. Selection Behavior

- Parent State가 어떤 Child State를 `선택하고 실행`할지 결정하는 규칙<br>

- **옵션**<br>

| 옵션 | 설명 | 용도 |
| --- | --- | --- |
| None | 선택하지 않음 | 특수 상황 |
| Try Enter | Parent 자신만 실행, *자식 무시* | Parent Task만 실행 |
| Try Select Children In Order | *위에서부터 순서*대로 조건 검사 (기본값) | **일반적인 선택** - Selector처럼 1개 선택 |
| Try Select Children At Random | 조건 통과한 Child 중 **랜덤 선택** | 행동 다양성 |
| Try Select Children With Highest Utility | Utility `점수가 가장 높은 Child` 선택 | Utility AI |
| Try Select Children At Random Weighted By Utility | Utility `점수 비율로 가중 랜덤` 선택 | Utility + 랜덤 |
| Try Follow Transitions | Child 선택 없이 `Transition 조건 우선` 검사 | 전환 허브 |

- Utility 부턴 5.5 버전부터 추가됨<br>
  - EQS처럼 점수를 부여하고 그에 따른 선택<br>

# 3. Task 시스템 💥

- `state tree task` 클래스를 통해 제작 가능<br>

## 3-1. Task 정의

- State 안에서 *실제로 실행*되는 동작 단위<br>

- **관계**<br>
    - State = 상황 (Idle, Patrol, Combat)<br>
    - Task = 그 상황에서 하는 일 (애니메이션, 이동, 공격)<br>

- **특징**<br>
    - 한 State에 '여러 Task'를 '넣을 수 있음'<br>
      - 그 경우, '동시에' Task가 실행함<br>
      - Delay 와 행동 Task를 넣으면<br>
        어느게 먼저 실행될지도 모르고, 행동이 불안정해짐<br>
        (그렇기에 별도의 State를 분리하는 것을 권장)<br>
    - State *진입 시 Task 시작*, *이탈 시 Task 종료*<br>
    - State에 Task가 없으면 실제 행동이 없음<br>

## 3-2. Task 생명주기 함수

| 함수 | 호출 시점 | 용도 |
| --- | --- | --- |
| **EnterState** | State '진입' 시 '한 번' | 행동 시작 로직 (이동, 애니메이션, 초기화) |
| **ExitState** | State '이탈' 시 즉시 | 빠른 정리 작업 (순서 보장 안됨) |
| **StateCompleted** | State '완전 종료' 시 | **안정적인 종료** 처리 (순서 보장) |
| **Tick** | State 활성화 중 '매 프레임' | 지속적 값 체크/업데이트 (성능 주의) |
| **GetDescription** | '에디터 표시'용 | 디버깅/가독성용 설명 문자열 |

- **권장 사항**<br>
    - 중요한 종료 로직은 **StateCompleted** 사용<br>
      - 성공/실패/취소 등의 상황에서 순서를 보장해 호출하기에 안전<br>
      - ExitState는 '최대한 빠르게' 정리함<br>
    - Tick은 꼭 필요한 경우에만 사용 (성능 고려)<br>

- TMI : 기본적으로 AI Controller는 직접 안만들어도 Default 가 존재하기에<br>
  Task 내부에서 어찌되었든 가져올 수 있음<br>

## 3-3. Finish Task의 진실

- `Finish Task`는 Task만 끝내는 것이 아니라 **State 전체를 종료**시킵니다.<br>

- **동작 방식**<br>
    - 한 State에 여러 Task가 동시 실행 가능<br>
    - 그중 하나라도 `Finish Task` 호출 시 **State 전체 종료**<br>
    - 사실상 `Finish State`와 동일한 효과<br>

- **주의사항**<br>
    - 'Delay Task'를 같은 State에 넣으면 **의도대로 동작하지 않음**<br>
    - 먼저 끝나는 Task가 State를 종료시키기 때문<br>
    - 대기 동작은 별도 State로 분리 필요<br>

## 3-4. Task 간 데이터 공유 (바인딩)

Task의 변수 값의 Category 설정 가능<br>

### Input 변수

- Category를 `input`으로 설정<br>
- 반드시 다른 값에 바인딩해야 함<br>
- 수동 입력 필드 없음 (실수 방지)<br>

**바인딩 가능 대상:**<br>

1. **Parameters** - 'State Tree Asset'의 설정값<br>
2. **Context Actor 변수** - 캐릭터에 '선언된 변수'<br>
3. **다른 Task Output** - 앞선 Task의 '계산 결과'<br>

### Output 변수

- Category를 `output`으로 설정<br>
- 입력 필드가 사라지고 출력 전용이 됨<br>
- 다른 Task의 Input으로 바인딩 가능<br>

**데이터 흐름**

```
Task A [Output] → Task B [Input]
앞 Task의 결과 → 뒤 Task의 입력
```

## 3-5. Context 자동 바인딩

- **방법**<br>
    1. Task 변수의 Category를 `context`로 설정<br>
    2. 타입이 Context Actor Class와 일치하면 자동 연결<br>
    3. Actor Reference는 하나만 사용 권장<br>

- **장점**<br>
    - 매번 Context Actor를 수동으로 연결할 필요 없음<br>
    - 타입만 맞으면 자동으로 채워짐<br>
    - 코드 간소화<br>

# 4. Transition과 Condition 🤪

## 4-1. Transition 개요

- State 간의 '전환'을 제어하는 시스템<br>

- **필요성**<br>
    - 상황에 따라 '다른 행동'으로 전환<br>
    - 조건부 행동 구현<br>
    - 복잡한 AI 로직 구성<br>

## 4-2. Transition Trigger

- **언제 전환을 검사할 것인가?**<br>

| Trigger | 발동 시점 | 용도 |
| --- | --- | --- |
| **On State Completed** | State 종료 시 (성공/실패 `무관`) | *기본 연결* |
| **On State Succeeded** | State `성공 종료` 시만 | '성공 케이스' 처리 |
| **On State Failed** | State `실패` 종료 시만 | '실패 케이스' 보정 |
| **On Tick** | 매 프레임 '조건 체크' | *즉시 반응* (플레이어 발견 등) |
| **On Event** | *외부 이벤트* 수신 시 | *외부 신호 반응* (경보, 대화 등) |

- Event 등을 통해 '외부 반응'에 따른 처리 가능<br>
  (Gameplay Tag를 전달함으로서 '어떤 이벤트'가 발생했는지를 전달 가능)<br>

## 4-3. Transition To

- **어디로 전환할 것인가?**<br>

| 옵션 | 동작 | 용도 |
| --- | --- | --- |
| **None** | 전환 안 함 | 테스트/비활성화 |
| **Next State** | 같은 레벨의 다음 State | 순차 실행 |
| **Next Selectable State** | 조건 만족하는 다음 State | 조건부 순차 실행 |
| **Tree Succeeded** | State Tree 전체 성공 종료 | 임무 성공 |
| **Tree Failed** | State Tree 전체 실패 종료 | 임무 실패 |

- 잘못 사용하면 로직이 튈 수 있기에 주의하여 사용할 것<br>

## 4-4. Enter Condition

- State에 들어가기 직전에 검사하는 조건<br>
  - 50% 등의 조건을 발생시키는 경우도 가능함<br>
    (이 경우는 '실패' 등에 대한 처리 Transitions를 고려하는 것이 좋음)<br>

- **Transition Condition과의 비교**<br>
    - **Transition Condition:** State가 끝날 때 검사 (나갈 때)<br>
      - 다음 상태 진입에 대한 고려(사용하기 편함)<br>
    - **Enter Condition:** State에 들어가기 전 검사 (들어갈 때)<br>
      - 현 상태 진입전 고려<br>
        (특별한 전제조건 필요시 고려)<br>
        (ex : 특정한 지역에 들어가려면 어떤 이벤트를 보고 오세요~)<br>
        (아래의 Required Event to Enter!)<br>

- **기본 제공 Condition:**<br>
    - Bool 비교<br>
    - Distance Compare<br>
    - Enum Compare<br>
    - Float/Integer Compare<br>
    - Random Chance (확률 기반)<br>

## 4-5. Enter Condition 상세 설정

### Required Event to Enter

- 특정 이벤트 발생 필수<br>
- 이벤트 없이는 State 진입 불가<br>
- 예: "On Player Detected" 이벤트 필요<br>

### Check Prerequisites when Activating Child Directly

- Child State로 직접 진입 시에도 Parent 조건 검사<br>
- 조건 무시 방지 (안전 장치)<br>
- 체크 시: 직접 진입해도 정직하게 검사<br>

#### priority

- normal이 default이기에<br>
  우선순위를 정하고 싶다면 high부터 고려할 것<br>
  (똑같으면 다른 작업이 우선될 수 있음)<br>

## 5. Evaluator와 Global Task 🌏

## 5-1. Evaluator와 Global Task 개념

- **공통점**<br>
    - 트리 전체에 걸쳐 실행되는 `전역 코드`<br>
    - Enter, Tick, Exit 라이프사이클 보유<br>

- **차이점**<br>

| 구분 | Evaluator | Global Task |
| --- | --- | --- |
| 주 목적 | **값 계산/생산** | **전역 행동/감시** |
| Output | 있음 ('값' 내보냄) | 없음 ('행동' 수행) |
| Finish 호출 | 안 함 | 할 수 있음 ('주의') |
| 사용 예 | 거리 계산, 시작 위치 저장 | 디버그 HUD, 버프 관리, 이벤트 리스닝 |

## 5-2. Evaluator 특징

state tree evaluator 로 생성 가능<br>

- **역할**<br>
    - **순수 계산만** 수행<br>
    - Output으로 값을 노출<br>
    - 다른 State/Task/Condition에서 바인딩하여 `재사용`<br>

- **장점**<br>
    - 복잡한 계산을 Evaluator에서 전담<br>
    - 나머지는 단순 비교만 수행<br>
    - 코드 간소화 및 재사용성 향상<br>

- **사용 예시**<br>
    - 시작 위치 기억 (`StartLocation`)<br>
    - Target까지 거리 계산 (`DistanceToTarget`)<br>
    - 시야각 내 타깃 수 세기<br>

- 생명주기<br>

| 함수 | 호출 시점 | 용도 |
| --- | --- | --- |
| **Tree Start** | 트리 `시작 시 한 번` | '초기 값' 설정/계산 |
| **Tree Stop** | 트리 종료 시 | 정리 작업 |
| **Tick** | 실행 중 '매 프레임' | 동적 값 갱신 ('성능' 주의) |

## 5-3. Global Task vs Evaluator 선택 기준

- **Evaluator 사용**<br>
    - '계산된 값을 공유'하는 경우<br>
    - Output이 필요한 경우<br>
    - 순수 데이터 생산자 역할<br>

- **Global Task 사용**<br>
    - `전역 행동 감시/수행`<br>
    - 디버그 메시지 출력<br>
    - *주기적 효과* 적용 (체력 회복 등)<br>
    - *이벤트 리스닝*<br>