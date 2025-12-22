---
title: "State Tree TIL (251222)"
date : "2025-12-22 20:00:00 +0900"
last_modified_at: "2025-12-22T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - State Tree
---

## State Tree TIL

금일 작업한 내용에 대하여<br>
State Tree 관련된 TIL을 작성해보려 한다<br>

- 정확히는 삽질한 것들<br>

- [추가적으로 보면 좋은 개념](https://hnjog.github.io/unreal/c++/StateTree/)<br>

### StateTree의 Schema

작업 중, 갑자기 StateTree가 종료한 경우가 존재하였다<br>
(심지어 StartTree를 제대로 호출했음에도!)<br>

원인?<br>

- Schema가 `StateTree AI Component`!<br>
  - **StateTree AI Component**는 초기화 시점에서<br>
    자신의 주인이 AIController이거나,<br>
    자신을 가진 Pawn이 AIController를 가진지 확인<br>
    - AIController가 없으면 스스로 종료...<br>
  - 그렇기에 이러한 `StateTree AI Component`는 <br>
    AIController 내부에서 관리하는 것이 권장됨<br>
  - 그렇지 않으면 최소한 `PossessedBy` Event 타이밍 이후에 시도<br>

- 다만 내 경우는 PossessedBy에 설정하였지만<br>
  여전히 꺼졌기에 **StateTree Component**로 스키마를 바꾸어 해결하였다<br>
  - PossessedBy에 설정하여도 꺼지는 경우라면<br>
    StartLogic()을 2번 호출하거나<br>
    처음 시작시 바로 AutoStart를 켜준 상황 등을 체크하는 것을 권장<br>

### StateTree는 Root부터 시작하지만 매번 시도 되는 것이 아님

BT에 다소 익숙해져 있기에 착각한 가장 큰 오해!<br>

- BT는 매번 틱마다 Root에서부터 '다시' 내려오며<br>
  '현재 실행할 노드'를 찾음<br>
  (그렇기에 중간에 '우선순위'가 높은 노드가 있다면 그쪽으로 갈아탐)<br>

- ST는 한번 State에 진입시, '계속' 머무름<br>
  - 다음 프레임이 되어도...<br>
  - 정의된 Transition 조건이 만족될때만 떠난다!<br>
  - 그럼 Root가 왜있는걸까?<br>
    - 최초 진입용<br>
    - '전역 조건' (Global Transition)의 정의<br>
      : *Root에 '체력 0' 일시 Dead 상태로*<br>
        라는 Transition이 있다면<br>
        어떤 상태건 이 조건이 감시 됨<br>
      - 이러한 내용은<br>
        **'부모 상태의 조건'이 '자식 상태'에서도 유효**하기에<br>
        체크가 가능한 점을 알아둘 것!<br>

### StateTree의 Transition이 '시도'되는 타이밍

StateTree의 `Transition`(상태 이동)이 시도되는 경우는<br>
크게 다음과 같은 3가지가 존재한다<br>

- On State Completed (기본)<br>
  - 현재 Task의 상태가 `Succeeded` or `Failed`로 종료되었을 때 호출<br>
  - 만약 Running 상태의 Task가 FinishTask를 호출하지 않는다면<br>
    이 트리거는 발동하지 않음<br>

- On Tick(매 프레임)<br>
  - 매 프레임 조건을 검사하는 방식<br>
  - '즉시 전환' 같은 로직에 사용 가능하다<br>

- On Event<br>
  - 외부에서 `SendStateTreeEvent` 같은 이벤트를 보냈을 때 즉시 검사<br>

- StateTree의 상태변환 조건 (Transition Condition)<br>
  - 현재 상태에서 '탈출 할 수 있는지를 검사'<br>
  - Detail > Transition 항목 내의 섹션에서 결정 가능함<br>
  - 이 조건이 True여야 '타겟'으로 지정된 상태로의 이동 시도<br>

- 만약 Transition이 설정 안되어 있다면?<br>
  - '부모 상태'로 이동하여 Completed Transition이 있는지 검사<br>
    (따로 전이 조건이 없다면 점점 상위로 올라가버림)<br>
    - 결국 Root까지 올라간 후, Root에도 없다면 StateTree 자체가 '종료'됨<br>
      (Stopped)<br>
  - 그렇기에 '꺼지지 않게'하려면 계속 명시적인 Transition을 설정해주어야 함<br>

### Enter Condition은??

- '상태 선택'의 '문지기' 역할<br>
  - Transition 시도하기 전, 마지막으로 검사<br>

- 이 녀석 자체는 '이동'과는 관련이 없음!<br>
  - '검증'에 가까운 개념<br>


