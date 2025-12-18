---
title: "팀 작업 - 미니언(AI) 작업 구조 짜기"
date : "2025-12-18 12:00:00 +0900"
last_modified_at: "2025-12-18T12:00:00"
categories:
  - Unreal
  - C++
  - 내일배움캠프
tags:
  - Unreal
  - C++
---

# 미니언(AI) 작업 구조 짜기

사용 기능(예정)<br>

- GAS<br>
- State Tree<br>
    - MassAI 의 경우, 차후 맵이 매우 넓어지는 경우 고려 가능<br>
        - 하지만 그 외의 경우는, 필요에 따라 Entity를 Actor로 승격시킬 필요가 존재함<br>
- Spline + NavMesh<br>
    - Zone Graph가 Mass AI에 특화되어 있음<br>
        - 차후, Mass AI 고려시 Zone Graph 채택 고려<br>

기능 구성<br>

**1. 길 찾기 & Line 표시**
- Spline Component를 사용할 Actor 생성 (USplineComponent)<br>
- 미니언은 다음 Spline의 위치를 받아 Move To<br>
  
**2. AI**


[![Image](https://github.com/user-attachments/assets/48e47bc8-2e42-4684-ac72-a0858fa85083)](https://github.com/user-attachments/assets/48e47bc8-2e42-4684-ac72-a0858fa85083){: .image-popup}<br>

- State Tree를 이용한 상태 구조 정의
    - ‘Death’ State
        - Condition : Health 가 0 이하
        - Task : Death 몽타주 재생 및 충돌 Collision 제거, 일정 시간 후 Destroy
    - ‘Combat’ State
        - Condition : TargetActor가 유효 및 사거리 안에 존재
        - Task : GAS Ability Activation(공격 어빌리티 실행)
            - 다만, 쿨타임 중이라면 ‘대기’해야 하는 상태 처리 필요
    - ‘Chase’ State
        - Condition : TargetActor가 유효 및 사거리 밖에 존재
        - Task : MoveTo TargetActor
            - 다만, Spline의 특정 거리만큼 벗어나게 되는 경우 다시 돌아와야 함
    - ‘Walk’ State
        - Condition : TargetActor 가 없음 (주변에 적 없음)
        - Task : FollowSpline (다음 지점을 따라가며 최종적으론 적의 보호 목표로)
        - 진입 시점에서 Target Location을 미리 저장해둘 예정


**3. Battle**

- GAS 기반
    - AttributeSet
        - Health : 체력
        - AttackDamage : 공격력
        - AttackRange : 사거리 (State Tree 의 조건 판별용)
        - Defense : 방어력?
    - GA
        - GA_MeleeAttack : 근거리 미니언 용 (다만 유사한 MeleeAttack 등이 존재하는지 체크)
            - 몽타주 재생 → Notify에서 GameplayEvent → Damage 적용
        - GA_RandedAttack : 원거리 미니언 용 (역시 유사한 GA 있는 지 체크)
            - 몽타주 재생 및 발사체(Projectile) 스폰?
        - GA_Die : 사망처리용
    - GE
        - MinionAttribute 등을 이용해 Spawn시 스탯 세팅이 가능할지 고민
            - 아니면 DataAsset 사용여부 고민

**4. Wave Manager**


[![Image](https://github.com/user-attachments/assets/2d984581-809a-4f8a-a12a-aaa39ef4f06b)](https://github.com/user-attachments/assets/2d984581-809a-4f8a-a12a-aaa39ef4f06b){: .image-popup}<br>

- 기본적으로 Server에서만 동작할 예정 (WorldSubsystem?)
- 관련 변수
    - WaveInterval : 웨이브 생성 주기
    - MinionClassess : 생성할 미니언 클래스?
    - SpawnLocation : Spawn 위치
        - 미니언들마다 Offset 을 고려?
- Timer를 이용하여 Wave를 Spawn
    - 아마 0.5초 간격으로 Spawn
- Wave 구성요소를 위한 Data?
    - 기본 근거리 3, 원거리 3, 공성 1 소환
        - 공성이 짝수 웨이브나 특정 웨이브마다 소환되는 경우는 변수 하나 추가
    - 넥서스 깨졌을때, 슈퍼 미니언이 포함되어 추가적으로 Spawn
        - 역시 변수 하나 관리
    - 다만, 차후 Spawn 방식이 다양해지는 경우는 별도의 구조체를 포함시키는 쪽으로

작업 관련?<br>

- ANpcCharacter (미니언 말고, 중립 몬스터 활용)<br>
    - 멤버<br>
        - ASC<br>
        - GameplayAbility<br>
        - AttributeSet (아마 기존의 만들어진 캐릭터 사용 예정)<br>
        - State Tree? (UStateTreeComponent)<br>
        - SkeletalMesh<br>
    - 관련 GA, GE<br>
        - GA<br>
            - Attack<br>
            - Dead<br>
        - GE<br>
            - MinionData 를 GE로? + DataTable (소환시 세팅)<br>
- WaveSpawner<br>
  : 에디터에서 배치할 Line 구조<br>
    - 멤버<br>
        - WaveSpawnData : 생성할 Minion Class + 개수에 대한 집약<br>
        - SplineComponent<br>
- MinionCharacter - 실제 생성할 캐릭터<br>
    - BP로 여럿 생성하여 MinionType 을 나누어 클래스 분류<br>
        - MinionType?<br>

