---
title: "Mass Entity"
date : "2026-01-10 18:00:00 +0900"
last_modified_at: "2026-01-10T18:00:00"
categories:
  - Unreal
  - Mass AI
tags:
  - Unreal
  - Mass AI
  - Mass Entity
---

## Mass Entity에 대하여

Unreal의 Mass AI 시스템은 '매~우' 많은 양의 Npc 등을 구현할 때 사용한다<br>

- 좀비 같은 적을 100마리 정도만 사용한다면,<br>
  Actor로도 충분할 수 있음<br>
  - 그러나 이 이상 넘어가면 점점 처리하기 힘들어짐<br>

그렇기에 사용하는 것이 `Mass Entity`!<br>

- Mass Entity는 이러한 계산을 위하여<br>
  데이터 지향 계산 방식을 채택한<br>
  일종의 프레임워크<br>
  (DOD, Data-Oriented Design : 데이터 지향 설계)<br>

- Entity는 단순히 '데이터'만 갖는 일종의 요소<br>
  - 각각의 '구성'을 통해 속성을 부여(name,helath 등)<br>
    (Fragment!)<br>
  - 그리고 '시스템'을 통해 로직을 처리 (이동, 체력 등)<br>
    (Processor!)<br>
    - Entity는 단순히 ID이지만 여기에 데이터를 레고처럼 조립하는 방식<br>
      -> ECS(Entity Component System)!<br>

- [언리얼 5.6 기준 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/overview-of-mass-entity-in-unreal-engine?application_version=5.6)<br>

## Entity, Archetype, Fragment

[![Image](https://github.com/user-attachments/assets/e5c06250-f7e8-4f4c-98a6-a576b6b30b08)](https://github.com/user-attachments/assets/e5c06250-f7e8-4f4c-98a6-a576b6b30b08){: .image-popup}<br>

- Fragment<br>
  : 주요 데이터 구조<br>
    (Processor의 계산에 사용되는 최소 단위의 데이터 '조각')<br>
    - Transform, Velocity, LOD 등등이 존재<br>
    - 별도의 '로직'을 포함하지 않음!<br>
    - 태그?<br>
    : 데이터가 없는 '사소'한 Fragment<br>
        (존재 여부 자체를 데이터로 활용하는 용도)<br>
    - Chunk Fragment?<br>
    : Entity 대신, '청크'와 연결된 Fragment<br>
        (LOD 계산 등, 관리 프로세스에 사용되는 청크 단위의 데이터를 저장)<br>

- Archetype(아키타입)<br>
  : Fragment의 '목록'이 같은 엔티티의 모음!<br>
    - 내부의 엔티티는 '메모리 청크(Chunk)'로 조직<br>
      (이들은 '연속적'으로 메모리에 배치됨)<br>
      - 동일 아키타입의 엔티티와 관련된 Fragment를 참조할 때 훌륭한 퍼포먼스를 발휘<br>
        (캐시 적중률 상승! + SIMD 처리 가능)<br>

- 이렇기에 Entity 자체는 매우 가볍고<br>
  아키타입에서 보관된 자신의 정보를 쉽게 찾을 수 있음<br>

예시)<br>

```cpp

Entity A : {TransformFragment} , {VelocityFragment}
Entity B : {TransformFragment} , {VelocityFragment}
Entity C : {TransformFragment} , {VelocityFragment} , {HealthFragment}

-> C는 별개의 아키타입으로 분류!

+ 혹시 Processor 등으로 Fragment가 제거/추가 될 경우, 아키타입이 바뀌게 됨
  -> 이건 비싼 작업이므로, 매 프레임(Tick) 실행되는 것은 피할 것!

```


## Processor

[![Image](https://github.com/user-attachments/assets/142e01f7-6a49-4307-8330-a481a38b289a)](https://github.com/user-attachments/assets/142e01f7-6a49-4307-8330-a481a38b289a){: .image-popup}<br>

- Processor(프로세서)<br>
  : `프로세싱` 로직을 공급하는 '상태 없는 클래스'<br>
    (프로세싱 작업하는 클래스가 프로세서...)<br>
    - 프로세서는 데이터(Fragment) 값을 갱신!<br>
    - Fragment를 추가 or 제거('구조적 변경')<br>
      - 다만 이 작업은 아키타입 자체가 이동해야 하기에<br>
        특정 상황에서만 호출을 권장함<br>
        (ex : 사망, 무기 획득 등)<br>
   
- Entity Query<br>
  : 프로세서가 사용하며, '작업 수행'에 필요한 Fragment 타입을 지정함<br>
    (엔티티의 식별자에 관계 없이, 프로세서에 Fragment 배치를 제공)<br>

- `MassCommandBuffer`<br>
  : 위에서 말한 '구조적 변경'을 '안전'하게 수행<br>
     - 아키타입을 옮기는 것을 '바로' 수행하면 다른 프로세서에서 크래시가 발생 가능!<br>
       그렇기에 커맨드 버퍼에게 '예약'하고, 일반적인 데이터 프로세싱의 진행 이후, 구조 변경 진행<br>

- `EntityView`<br>
  : 현재 프로세스가 아닌 다른 '엔티티'에 엑세스 해야 할때 사용<br>
    - 엔티티의 데이터(Fragment)를 직접 확인하기 위해 사용<br>
    - 일반적으로 프로세서가 쿼리를 통해 받는 데이터는 '나 자신'(Current Entity)<br>
      그러나 '다른 엔티티'의 상태를 확인해야 한다면<br>
      그 엔티티의 ID를 통해 EntityView를 생성하고 데이터를 체크해야 함<br>

### Entity Manager (FMassEntityManager)

- UMassEntitySubsystem<br>
  - 역할 : 엔진에서 `UWorld` 내부에 Mass AI의 자리를 마련해주는 컨테이너 이자 관리자<br>
  - `FMassEntityManager`의 인스턴스를 하나 들고 있고<br>
    다른 시스템들이 이에 접근할 수 있도록 함<br>

- FMassEntityManager<br>
  - 실질적인 매니저 로직이 들어간 구조체<br>
    - 아키타입 관리, Fragment 조작, 메모리 관리 등<br>

```cpp

// 월드 서브시스템을 통해 매니저를 가져옴
UWorld* World = ...;
UMassEntitySubsystem* EntitySubsystem = World->GetSubsystem<UMassEntitySubsystem>();
FMassEntityManager& EntityManager = EntitySubsystem->GetMutableEntityManager();
```

## Trait

[![Image](https://github.com/user-attachments/assets/faa9d3ce-5151-4535-8a1c-911f11f36ac9)](https://github.com/user-attachments/assets/faa9d3ce-5151-4535-8a1c-911f11f36ac9){: .image-popup}<br>

- Trait<br>
  : Entity 생성의 블루프린트 같은 역할을 함<br>
   - Mass 시스템이 실제 엔티티를 생성(Spawn)할 때, <br>
     Trait을 보고 '정의'된 Fragment를<br>
     Entity에 넣어 생성함<br>

- MassEntityConfig<br>
  : 이러한 Trait를 가지는 '데이터 에셋'<br>
   - 정확히는 Spawner가 이것을 기반으로 엔티티를 생성함<br>
   - 내부에 여러 Trait이 존재<br>

- Entity Template<br>
  : 위의 MassEntityConfig로부터 생성<br>
   - 시스템이 MassEntityConfig를 읽은 후,<br>
     Trait들을 해석하여 Fragment가 필요한지 체크<br>
     이후, 목록을 확정하고, `FMassEntityTemplate`로 데이터를 만들어놓음<br>
     -> 이후 DataAsset이 아니라, 미리 만들어둔 데이터에서 바로 복사해 사용<br>

Trait들을 통해<br>
'아키타입'을 만드는 것!<br>

### 흐름 정리

[![Image](https://github.com/user-attachments/assets/bddfb430-7e59-4cc6-a915-1ed9432ada18)](https://github.com/user-attachments/assets/bddfb430-7e59-4cc6-a915-1ed9432ada18){: .image-popup}<br>

- Trait에 Fragment의 정의를 넣어둠<br>

- MassEntityConfig 라는 데이터 에셋을 통해<br>
  사용할 Trait을 정의함<br>

- 게임 시작 시, EntityTemplate 를 빌드함<br>
  (차후 사용할 녀석들)<br>

- 이후 매니저가 아키타입을 준비<br>
  - 다만 LazyLoading 방식이기에<br>
    Spawner가 실제로 엔티티 생성을 요청하고<br>
    그러한 아키타입이 없다면 새로이 아키타입을 생성하는 방식<br>

- 참고로 Processor는 이들과 연결되지 않음<br>
  Query를 이용하여 매칭됨<br>
  (MassEntityManager!)<br>

- 매니저의 중앙집권체제<br>

## 그 외에도 여러 서브 시스템 존재

- 관련한 월드 서브시스템들이 존재<br>
  - 그렇기에 World의 수명에 귀속<br>

- 이건 플러그인과 모듈로 분리되어 있음!<br>
  (MassGameplay 등)<br>

- Mass Simulation Subsystem (UMassSimulationSubsystem)<br>
  - 역할 : Mass AI의 심장<br>
  - 기능 : MassProcessor들의 실행 순서와 단계를 관리<br>
    프로세서들이 멀티스레드에서 안전하게 돌아가도록 스케줄링<br>
    게임 루프(Tick)에 맞춰 시뮬레이션을 동작<br>

- Mass Spawner Subsystem (UMassSpawnerSubsystem)<br>
  - 역할: 엔티티의 생성을 담당<br>
  - 기능 :레벨에 배치된 MassSpawner 액터들과 통신<br>
    MassEntityConfig에 정의된 대로 엔티티를 실제로 찍어내는(Spawn) 역할<br>

- Mass Representation Subsystem (UMassRepresentationSubsystem)<br>
  - 역할: 엔티티의 **시각화(렌더링)**를 담당<br>
  - 기능: 수만 개의 엔티티를 모두 액터로 그리면 느려지므로, <br>
  이를 ISM(Instanced Static Mesh)으로 그려줄지,<br>
  가까이 오면 액터로 바꿔줄지(LOD 처리) 등을 결정하고 관리<br>

- Mass Actor Subsystem (UMassActorSubsystem)<br>
  - 역할: 기존 액터(Actor)와 Mass 엔티티 간의 연결을 담당<br>
  - 기능: Mass AI를 쓴다고 해서 모든 것을 ECS로만 할 수는 없음<br>
    기존의 AActor와 데이터를 주고받거나 동기화해야 할 때의 중간 다리 역할<br>

- Mass Signal Subsystem (UMassSignalSubsystem)<br>
  - 역할: 엔티티 간의 이벤트/신호 전달을 담당<br>
  - 기능: 특정 엔티티에게 "너 데미지 입었어" 같은 신호를 보낼 때 사용<br>
    직접 함수를 호출하는 대신 신호를 큐에 넣고 처리하는 방식(비동기적)을 지원<br>
