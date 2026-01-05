---
title: "김하연 튜터님 강의 - 'CPU 및 아키텍처 최적화의 기초'"
date : "2026-01-05 18:00:00 +0900"
last_modified_at: "2026-01-05T18:00:00"
categories:
  - Unreal
  - C++
  - 내배캠
  - 강의
tags:
  - Unreal
  - C++
  - 최적화
  - 강의
---

# CPU 및 아키텍처 최적화의 기초에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

- 최적화는 결국 2가지!<br>
  - CPU!<br>
  - GPU!<br>

양쪽 모두 이론적인 이야기가 필요함<br>

언리얼은 생각보다 '최적화'가 어려운 엔진<br>
(4->5 로 넘어가며 많이 바뀌기도 하였고)<br>

# 1. 프레임 예산과 시간의 중요성 🐲

## 1-1. 우리에게 주어진 프레임 예산

- 60fps 게임에서 한 프레임당 16.67밀리초가 주어지고, 120fps에서는 8.33밀리초만 주어짐.<br>
- 이 제한된 시간 안에 다음 작업들을 모두 완료<br>
    - 입력 처리<br>
    - 게임 로직<br>
    - 물리 시뮬레이션<br>
    - 애니메이션<br>
    - 오디오<br>
    - 렌더링<br>

- Lumen, Nanite 같은 것을 사용하면 GPU도 열심히 돌아감<br>
  (GPU가 다른 엔진보다 많이 작업 할당)<br>

### 언리얼의 프레임 처리 과정

```cpp
// 언리얼 엔진 내부의 Tick 처리 (매우 단순화)
void UWorld::Tick(float DeltaTime)
{
    // [1단계] 입력 수집 (Input)
    
    // [2단계] 게임 로직 (GameThread) - 여기가 최적화 대상!
    // 레벨에 있는 모든 액터를 순회
    for (AActor* Actor : AllActors)
    {
        if (Actor->CanEverTick())
        {
            // ⚠️ 여기서 비용 발생!
            // 1. 가상 함수 테이블(vtable) 조회
            // 2. 캐시 미스 (메모리 여기저기 점프)
            Actor->Tick(DeltaTime); 
        }
    }

    // [3단계] 물리, 애니메이션, 렌더링 명령 생성 등...
}
```

**💡 Tick이 느린 기술적 이유**<br>

**1. 가상 함수 비용**<br>
  : `Tick()`은 `Virtual Function` 호출 시 주소를 찾는 오버헤드 발생(VTable, Stack Frame, 데이터 복사 등..)<br>

**2. 캐시 미스 (Cache Miss)**<br>
  : 액터들은 메모리에 흩어져 있음 (Heap). CPU가 데이터를 가지러 여기저기 뛰어다녀야 함<br>
    (메인 메모리 왔다갔다...)<br>

- 병렬처리 (Worker 스레드) 로 여러 코어를 사용하지만<br>
  또, 이것을 '합치는' 과정이 필요<br>

- 이걸 '다' 해야 'Render 스레드'가 GPU에게 Draw Call을 신청<br>
  (RHI(Render Hardware Interface))<br>

-> 1프레임에 모두 진행...<br>

- 병목 현상이 발생하면 아무리 기기가 좋아도 문제가 발생함<br>
  (CPU 쪽에서 안좋은 C++ 처리하면 GPU는 멍때릴 뿐...)<br>

## 1-2. 우리가 저지르는 4가지 '고급진' 실수

**첫 번째 실수 - GAS 망각증 (Polling vs Event)**<br>

- GAS를 구축해놓고 습관적으로 `Tick`에서 검사하는 행위.<br>
  - 기술에 익숙치 않아서 '접착제' 부분을 안 좋게 구현하는 방식<br>

```cpp
// ❌ [Bad] 폴링(Polling) 방식
void AMyCharacter::Tick(float DeltaTime)
{
    // 매 프레임 검사: CPU 낭비
    if (bIsStunned) 
    {
        StunDuration -= DeltaTime; // 쿨타임도 직접 계산? NO!
    }
}
```

- GAS 기반에서 Tick을 사용할일?<br>
  - 매 순간 세세한 프레임 처리가 필요한 경우<br>
  - 얼음에서 미끄러진다던가, 중심잡기 등등<br>
  - 이것도 GE나 GA로 뺄 수 있는지 상담하기<br>

- Observer 패턴임을 다시 이해하자<br>
  - Delegate를 등록해놓으면 됨!<br>

```cpp
// ✅ [Good] 이벤트(Event) 방식
void AMyCharacter::OnGameplayEffectApplied(...)
{
    // 태그가 변하거나 이펙트가 적용될 때만 호출 (평소 비용 0)
    if (AbilitySystem->HasMatchingGameplayTag(StunTag))
    {
        PlayStunAnim();
    }
}
```

**두 번째 실수 - 1회용 액터 남발 (GC Spike)**<br>

- `SpawnActor`는 매우 비싼 작업<br>
  - 상속 구조를 따라가면서 죄다 처리<br>
  - 물리 엔진에서 '새로운 액터'를 생성하여 그 연산에 '끼워넣는것'은 매우 무겁다는 것을 인식!<br>
  - GC에도 등록해야 함 + GC가 정리할때 스파이크 등이 발생 가능 -> 프레임 드랍<br>

- 해결책: 무조건 ***오브젝트 풀링*** 사용.<br>
  (Lyra의 `GameplayCue`나 `Mass Entity` 활용)<br>
  - GameplayCue도 풀링이 고려된 것<br>
  - 나이아가라 같은 이펙트 도 풀링을 고려해야 함<br>

- Object Pool 은 '초기화 상태'에 주의할 것!<br>
  - 전용 GE등을 빼두는 것도 하나의 방식임<br>

```cpp
// ❌ [Bad] 매 프레임 생성과 파괴
void AWeapon::Fire()
{
    // 1. 메모리 할당 (Malloc)
    // 2. 물리 씬 등록 (Chaos) -> 제일 비쌈!
    // 3. GC 추적 리스트 추가
    GetWorld()->SpawnActor<ABullet>(...);
}
```

**세 번째 실수 - TMap 순회 (Cache Thrashing)** <br>

- 편리함 때문에 성능을 포기한 케이스!<br>
- **해결책**<br>
  - **검색(Find):** `TMap` (ID로 찾을 때)<br>
  - **순회(Loop):** `TArray` (데이터가 연속적이라 빠름)<br>

- 데이터 순회할거면 애초에 TArray를 고려해야 함<br>
  - TMap은 '검색'에만 사용하는 점을 유의!<br>  

```cpp
// ❌ [Bad] TMap을 for문으로 돌리기
TMap<int32, FQuestData> QuestMap;

void Update()
{
    // TMap은 데이터가 메모리에 흩어져 있음 -> 캐시 미스 폭발
    for (auto& Elem : QuestMap) 
    {
        Elem.Value.CheckCondition();
    }
}
```

**네 번째 실수 - 메인 스레드 독점 (Blocking GameThread)**<br>

- 멀티 코어 CPU를 놔두고 혼자 일하는 경우.<br>
- 해결책: `UE::Tasks::Launch` 등을 사용하여 워커 스레드로 작업 분산. (Mass AI의 방식)<br>

- 메인 스레드는 '매~우' 할 일이 많음<br>
  - Mass AI 같은 기능은 '병렬처리'를 적극적으로 사용하기에<br>
    매우 빠름!<br>

- 워커 스레드들에게 일을 할당할 방법을 찾아보자<br>

```cpp
// ❌ [Bad] 메인 스레드에서 무거운 연산
void AEnemy::Tick(float DeltaTime)
{
    // 100마리가 길찾기 연산을 동시에? -> 프레임 드랍 직행
    CalculateComplexPath(); 
}
```

# 2. 수천 개 액터 관리 전략 🦕

결국 최적화는 '액터 관리!'<br>

## 2-1. 원칙 1: 중앙 집중형 제어

- **각자도생하지 말고, 통제하자.**<br>
  - **OOP (Bad):** CPU가Actor 자리를 일일이 찾아다님. -> **메모리 탐색 비용**<br>
  - **DOD (Good):** Actor들을 한 줄로 세우고 처리함. -> **캐시 적중률(Cache Hit) 상승**<br>

구조가 예쁜 것이 아니라 성능상 이점이 존재함<br>

- 가상 함수가 오버헤드가 존재하다는 점은 항상 유의하자<br>
  (Tick 사용이 성능과 직결되는 이유)<br>

- 서브시스템에 Tick 사용시<br>
  `FTickableGameObject`를 고려하자<br>
  + `GetStatId()`<br>

- Weakptr 로 해야 GC가 잡아갈 수 있다!<br>
  (Mark & Sweep 이기에 ObjectPtr로 잡으면 World에 영원히 남을 수 있음...)<br>

- Tick 을 여러 적에게 돌려야 하는 상황이라면 훌륭한 옵션<br>

```cpp
// ✅ [Good] UWorldSubsystem을 활용한 매니저 패턴
UCLASS()
class UMonsterManagerSubsystem : public UWorldSubsystem, public FTickableGameObject
{
    GENERATED_BODY()

    // 몬스터들을 한곳에 모아 관리 (Cache Friendly)
    UPROPERTY()
    TArray<TWeakObjectPtr<AMonster>> ActiveMonsters;

public:
    virtual void Tick(float DeltaTime) override
    {
        // 가상 함수(Virtual Function) 오버헤드 없이 직접 루프!
        for (int32 i = ActiveMonsters.Num() - 1; i >= 0; --i)
        {
            if (AMonster* Monster = ActiveMonsters[i].Get())
            {
                // 인라인 처리 가능한 가벼운 로직 호출 (내부 틱 대신 매니저의 Tick 사용)
                Monster->UpdateMovementLogic(DeltaTime); 
            }
            else
            {
                // 죽은 몬스터는 Swap으로 빠르게 제거 (O(1))
                ActiveMonsters.RemoveAtSwap(i, 1, false);
            }
        }
    }
    // Subsystem도 Tick을 돌리려면 필수
    virtual TStatId GetStatId() const override { return RETURN_QUICK_DECLARE_CYCLE_STAT(UMonsterManagerSubsystem, STATGROUP_Tickables); }
};
```

## 2-2. 원칙 2: 중요도 관리 (Significance Manager)

- **보이지 않는 곳에서는 연기하지 말자. (Logic LOD)**<br>
  - **Significance 1.0 (주연):** 카메라 앞 -> Full Tick, 고품질 애니메이션.<br>
  - **Significance 0.0 (엑스트라):** 화면 밖 -> Tick Off, 로직 동면.<br>

그렇다고 '수학 연산'을 Tick에 넣진 말자...<br>

```cpp
// ❌ [Bad] 직접 거리 계산 (매 프레임 제곱근 연산)
void Tick(float DeltaTime) {
    if (GetDistanceTo(Player) < 1000.f) ... // CPU 사망 원인
}
```

- Mesh 뿐 아니라 내부 로직 등도 꺼버리기<br>
- 대규모의 개발 상황에서 사용할때의 기준을 잘 생각해보자<br>
  - 최적화는 '성능 측정' 후, 원인 파악하고 진행해야 함<br>
  - 그렇지 않으면 시간 낭비가 될 가능성 존재<br>

- GAS를 '끄더라도' 쿨타임 등은 정상적으로 계산됨<br>
  - 마지막 작동 시간을 기록해두었다가<br>
    다시 켜졌을때, 시간을 비교하여 계산함<br>

```cpp
// ✅ [Good] Significance Manager 연동 (엔진이 계산해줌)
void AMyCharacter::OnSignificanceChanged(float Old, float New)
{
    // 애니메이션 최적화 (엔진 내장)
    GetMesh()->SetSignificance(New);

    // 로직 최적화 (우리의 영역)
    if (New > 0.8f) 
    {
        SetActorTickInterval(0.0f); // 매 프레임
        AbilitySystem->SetComponentTickEnabled(true); // GAS 켜기
    }
    else 
    {
        SetActorTickInterval(1.0f); // 1초에 한 번 (생존 신고만)
        AbilitySystem->SetComponentTickEnabled(false); // GAS 끄기 (GAS도 무거운 편임!)
    }
}
```

## 2-3. 원칙 3: 이벤트 드리븐 (Event-Driven)

- **문 열어보지 말고 (Polling), 초인종을 달자 (Event).**<br>
  - **Polling:** 매 프레임 `if (HP < 30)` 검사 -> 99%는 낭비 (**Branch Misprediction**)<br>
  - **Event:** `TakeDamage` 함수 내에서만 검사 -> 평소 비용 0<br>

```cpp
// ❌ [Bad] 스토커 스타일 (Polling)
void Tick(float DeltaTime) {
    if (Health < 30.0f && !bIsEnraged) EnterEnrage(); // 매 프레임 낭비
}
```

```cpp
// ✅ [Good] 초인종 스타일 (Event)
void TakeDamage(float DamageAmount, ...) {
    Super::TakeDamage(...);
    Health -= DamageAmount;
    
    // "체력이 변했을 때"만 검사
    if (Health < 30.0f && !bIsEnraged) EnterEnrage(); 
}
```

## 2-4. 원칙 4: 오브젝트 풀링 (Object Pooling)

- 그릇을 깨지 말고 (Destroy), 설거지해서 쓰자 (Reset)<br>
- Spawn/Destroy 반복<br>
    - GC Hitching: 가비지 컬렉터가 60초마다 게임을 멈춤.<br>
    - Memory Fragmentation: 힙 메모리가 스위스 치즈처럼 구멍 남.<br>
      (단편화로 인해 잠재적으로 성능을 '갉아먹음')<br>

- 💡 실전 전략: Create -> Activate -> Deactivate -> Reuse<br>
    - Pre-allocate: 로딩 화면에서 총알 500개 미리 생성.<br>
    - Sleep Mode: `SetActorHiddenInGame(true)`, `SetActorTickEnabled(false)`, `Collision(false)`.<br>
    - Reuse: 필요할 때 꺼내고 `Reset` (변수 초기화 필수).<br>
- ⚠️ 주의: 액터뿐만 아니라 Niagara Component, Audio Component도 반드시 풀링 할 것! (생성 비용이 더 비쌈)<br>

```cpp
// ✅ 효율적인 오브젝트 풀
UCLASS()
class MYGAME_API AProjectilePool : public AActor
{
    GENERATED_BODY()
    
private:
    // 사용 가능한 발사체들 (비활성 상태)
    UPROPERTY()
    TArray<class AProjectile*> AvailableProjectiles;
    
    // 현재 사용 중인 발사체들 (활성 상태)
    UPROPERTY()
    TArray<class AProjectile*> ActiveProjectiles;
    
    // 풀 설정
    UPROPERTY(EditAnywhere, Category = "Pool Settings")
    int32 InitialPoolSize = 100;  // 시작할 때 미리 만들 개수
    
    UPROPERTY(EditAnywhere, Category = "Pool Settings")
    int32 MaxPoolSize = 500;      // 최대 풀 크기
    
    UPROPERTY(EditAnywhere, Category = "Pool Settings")
    TSubclassOf<AProjectile> ProjectileClass;  // 발사체 클래스
    
public:
    AProjectilePool()
    {
        PrimaryActorTick.bCanEverTick = false;  // 풀 자체는 Tick 필요 없음
    }
    
    void BeginPlay() override
    {
        Super::BeginPlay();
        
        // 게임 시작할 때 미리 발사체들 생성
        PreAllocateProjectiles();
    }
    
    void PreAllocateProjectiles()
    {
        if (!ProjectileClass)
        {
            return;
        }
        
        // 배열 크기 미리 예약 (재할당 방지)
        AvailableProjectiles.Reserve(InitialPoolSize);
        ActiveProjectiles.Reserve(InitialPoolSize);
        
        for (int32 i = 0; i < InitialPoolSize; ++i)
        {
            // 발사체 생성 (화면 밖 어딘가에)
            FVector HiddenLocation = FVector(0, 0, -10000);  // 땅 밑에 숨기기
            
            AProjectile* NewProjectile = GetWorld()->SpawnActor<AProjectile>(
                ProjectileClass, 
                HiddenLocation, 
                FRotator::ZeroRotator
            );
            
            if (NewProjectile)
            {
                // 비활성 상태로 설정
                NewProjectile->SetActorHiddenInGame(true);      // 화면에 안 보이게
                NewProjectile->SetActorEnableCollision(false);  // 충돌 끄기
                NewProjectile->SetActorTickEnabled(false);      // Tick 끄기
                
                // 사용 가능한 풀에 추가
                AvailableProjectiles.Add(NewProjectile);
            }
        }
    }
    
    // 발사체 대여하기
    AProjectile* GetProjectile()
    {
        AProjectile* Projectile = nullptr;
        
        if (AvailableProjectiles.Num() > 0)
        {
            // 사용 가능한 발사체가 있으면 재사용
            Projectile = AvailableProjectiles.Pop();
            ActiveProjectiles.Add(Projectile);
            
            // 활성화
            Projectile->SetActorHiddenInGame(false);      // 보이게 하기
            Projectile->SetActorEnableCollision(true);    // 충돌 켜기
            Projectile->SetActorTickEnabled(true);        // Tick 켜기
            
            // 발사체 초기화 (이전 상태 제거)
            Projectile->Reset();
        }
        else if (ActiveProjectiles.Num() + AvailableProjectiles.Num() < MaxPoolSize)
        {
            // 풀이 비었지만 최대 크기에 도달하지 않았으면 새로 생성
            Projectile = GetWorld()->SpawnActor<AProjectile>(ProjectileClass);
            if (Projectile)
            {
                ActiveProjectiles.Add(Projectile);
            }
        }
        else
        {
            // 풀이 완전히 가득 참 - 가장 오래된 발사체 재사용
            UE_LOG(LogTemp, Warning, TEXT("Pool at maximum capacity! Recycling oldest projectile"));
            
            if (ActiveProjectiles.Num() > 0)
            {
                Projectile = ActiveProjectiles[0];  // 가장 오래된 것
                ActiveProjectiles.RemoveAt(0);
                ActiveProjectiles.Add(Projectile);  // 맨 뒤로 이동
                
                // 강제로 초기화
                Projectile->Reset();
            }
        }
        
        return Projectile;
    }
    
    // 발사체 반납하기
    void ReturnProjectile(AProjectile* Projectile)
    {
        if (!Projectile) return;
        
        // 활성 목록에서 제거
        int32 RemovedCount = ActiveProjectiles.RemoveSingle(Projectile);
        
        if (RemovedCount > 0)
        {
            // 비활성화
            Projectile->SetActorHiddenInGame(true);       // 숨기기
            Projectile->SetActorEnableCollision(false);   // 충돌 끄기
            Projectile->SetActorTickEnabled(false);       // Tick 끄기
            
            // 위치 초기화 (메모리 절약)
            Projectile->SetActorLocation(FVector(0, 0, -10000));
            Projectile->SetActorRotation(FRotator::ZeroRotator);
            
            // 사용 가능한 풀에 다시 추가
            AvailableProjectiles.Add(Projectile);
        }
    }
    
    // 현재 풀 상태 확인
    void LogPoolStatus() const
    {
        UE_LOG(LogTemp, Log, TEXT("Pool Status - Available: %d, Active: %d, Total: %d"), 
               AvailableProjectiles.Num(), 
               ActiveProjectiles.Num(),
               AvailableProjectiles.Num() + ActiveProjectiles.Num());
    }
    
    // 게임 끝날 때 정리
    void EndPlay(const EEndPlayReason::Type EndPlayReason) override
    {
        // 모든 발사체 파괴 (언리얼이 자동으로 해주지만 명시적으로)
        for (AProjectile* Projectile : AvailableProjectiles)
        {
            if (Projectile)
            {
                Projectile->Destroy();
            }
        }
        
        for (AProjectile* Projectile : ActiveProjectiles)
        {
            if (Projectile)
            {
                Projectile->Destroy();
            }
        }
        
        AvailableProjectiles.Empty();
        ActiveProjectiles.Empty();
        
        Super::EndPlay(EndPlayReason);
    }
};
```

**발사체 클래스도 풀링에 맞게 수정**<br>

```cpp
// 풀링을 고려한 발사체 클래스
class AProjectile : public AActor
{
private:
    UPROPERTY()
    class AProjectilePool* OwnerPool;  // 자신이 속한 풀

    float LifeTime = 5.0f;             // 생존 시간
    float ElapsedTime = 0.0f;          // 경과 시간

public:
    void SetOwnerPool(AProjectilePool* Pool) { OwnerPool = Pool; }

    // 풀에서 대여될 때 초기화
    void Reset()
    {
        ElapsedTime = 0.0f;
        // 기타 초기화 작업...
    }

    void Tick(float DeltaTime) override
    {
        Super::Tick(DeltaTime);

        // 이동 로직
        AddActorWorldOffset(GetActorForwardVector() * Speed * DeltaTime);

        // 시간 체크
        ElapsedTime += DeltaTime;
        if (ElapsedTime >= LifeTime)
        {
            // 수명이 다하면 풀로 돌아가기
            ReturnToPool();
        }
    }

    // 충돌했을 때도 풀로 돌아가기
    void OnHit()
    {
        // 히트 이펙트 등...
        ReturnToPool();
    }

private:
    void ReturnToPool()
    {
        if (OwnerPool)
        {
            OwnerPool->ReturnProjectile(this);
        }
        else
        {
            // 풀이 없으면 그냥 파괴
            Destroy();
        }
    }
};

```

**사용법**

```cpp
// 무기에서 총알 발사
void AWeapon::Fire()
{
    if (ProjectilePool)
    {
        // 풀에서 발사체 가져오기 - new/delete 없음!
        AProjectile* Bullet = ProjectilePool->GetProjectile();

        if (Bullet)
        {
            // 발사 위치와 방향 설정
            Bullet->SetActorLocation(GetMuzzleLocation());
            Bullet->SetActorRotation(GetMuzzleRotation());
            Bullet->SetOwnerPool(ProjectilePool);

            // 발사!
            Bullet->Fire();
        }
    }
}
```

# 3. Tick 병합과 매니저 패턴 🦖

## 3-1. 개별 액터의 비극

- **상황:** 총알 1,000개가 각자 `Tick`을 돌며 이동하고 수명을 체크함.<br>
- **결과:** 프레임 드랍 발생 (CPU 병목).<br>

```cpp
// ❌ [Bad] 각자도생 (개별 Tick)
void AMyBullet::Tick(float DeltaTime)
{
    // 1. 가상 함수 테이블(vtable) 조회 비용 발생 (매우 비쌈)
    // 2. 메모리 점프(Cache Miss) 발생
    AddActorWorldOffset(Velocity * DeltaTime); 
    
    LifeTime -= DeltaTime;
    if (LifeTime <= 0) Destroy();
}
```

**💡 Under the Hood: 왜 느린가?**<br>

1. **Instruction Cache Miss:** `Tick`은 가상 함수. 1,000번의 Context Switch(함수 진입/복귀)가 발생.<br>
2. **Data Cache Miss:** 총알 액터들이 힙 메모리 여기저기에 흩어져 있음.<br>

## 3-2. 해결책: 서브시스템 & SoA (Manager Pattern)

- 데이터를 뭉쳐야 빨라진다. (DOD + SIMD)<br>
  - AoS (Array of Structures): `[ {Pos, Vel}, {Pos, Vel} ]` -> 객체지향적, 캐시 효율 낮음.<br>
  - SoA (Structure of Arrays): `[Pos, Pos], [Vel, Vel]` -> 데이터 지향적, `SIMD` 최적화.<br>

- 성능 비교<br>
  - 개별 액터: 3.8ms (Hitching)<br>
  - 매니저 패턴: 0.3ms (쾌적) -> 10배 이상 성능 향상!<br>

Actor로 Editor에 배치하는 것은 Old 함..<br>
World가 사라질때 '같이 사라짐'<br>

- 편의를 고려하자..<br>

- `SIMD(Single Instruction, Multiple Data)`?<br>
  - 하나의 명령어로 여러 데이터를 동시에 처리하는 병렬 기법<br>
  - ParallelFor!<br>

```cpp
// ✅ [Good] 현대적인 매니저 패턴 (SoA)
UCLASS()
class UProjectileSubsystem : public UWorldSubsystem, public FTickableGameObject
{
    GENERATED_BODY()

private:
    // 데이터를 끼리끼리 모음 -> 캐시 적중률 100%
    TArray<FVector> Locations;
    TArray<FVector> Velocities;
    TArray<float> LifeTimes;
    
    // 렌더링은 ISM(Instanced Static Mesh)으로 한 번에 처리
    UPROPERTY()
    TObjectPtr<UInstancedStaticMeshComponent> ProjectileRenderer;

public:
    virtual void Tick(float DeltaTime) override
    {
        int32 Count = Locations.Num();
        
        // [핵심] ParallelFor로 병렬 처리 + SIMD 자동화
        // 1,000번의 함수 호출이 '단 1번'의 루프로 바뀜
        ParallelFor(Count, [&](int32 i)
        {
            if (LifeTimes[i] > 0.0f)
            {
                // CPU가 한 번에 4개씩 계산 (SIMD)
                Locations[i] += Velocities[i] * DeltaTime; 
                LifeTimes[i] -= DeltaTime;
            }
        });

        // GPU에 한 번에 전송 (Draw Call 1회)
        ProjectileRenderer->BatchUpdateInstancesTransforms(...);
    }
    
    // Subsystem 필수 설정
    virtual TStatId GetStatId() const override { return RETURN_QUICK_DECLARE_CYCLE_STAT(UProjectileSubsystem, STATGROUP_Tickables); }
};
```

## 3-3. 타이밍 이슈 (Tick Groups & Dependency)

### 이슈 1: 벽뚫핵 (Wall-Hack)

- **원인:** 총알 이동(Manager)이 물리 엔진(Physics)보다 **늦게** 실행됨.<br>
  - Physics: "충돌 없음" 판정 -> Manager: "이동" (벽 통과) -> Render: 벽 뒤에 그림.<br>

- **해결:** 매니저를 물리 엔진보다 **앞**에 배치.<br>
  (`TG_PrePhysics`로 설정하기)<br>

```cpp
// TickGroup 종류 (실행 순서대로)
enum ETickingGroup : uint8
{
    TG_PrePhysics,        // 물리 시뮬레이션 전
    TG_StartPhysics,      // 물리 시뮬레이션 시작
    TG_DuringPhysics,     // 물리 시뮬레이션 중
    TG_EndPhysics,        // 물리 시뮬레이션 끝
    TG_PostPhysics,       // 물리 시뮬레이션 후
    TG_PostUpdateWork,    // 업데이트 작업 후
    TG_LastDemotable,     // 마지막
    TG_NewlySpawned       // 새로 생성된 액터들
};

// ✅ TickGroup 설정
void UProjectileSubsystem::Initialize(...)
{
    // "물리 엔진 계산하기 전에 제가 먼저 움직일게요."
    // 액터라면: PrimaryActorTick.TickGroup = TG_PrePhysics;
}
```

### 이슈 2: 카메라 떨림 (Jittering)

- **원인:** 카메라가 플레이어보다 **먼저** 움직여서, 과거의 위치를 찍음.<br>
- **해결:** `AddPrerequisite` (전제 조건 설정).<br>

- 스프링 암의 속도 등을 조정하여 임시 수정은 가능하지만<br>
  실행 순서를 명확히 하는 것이 제대로 된 해결!<br>

```cpp
// ✅ 의존성 설정 (줄 서기)
void ACameraManager::BeginPlay()
{
    if (APlayer* Player = GetPlayer())
    {
        // "플레이어 틱이 끝나야, 내가 움직인다."
        this->PrimaryActorTick.AddPrerequisite(Player, Player->PrimaryActorTick);
    }
}
```

# 4. 메모리와 컨테이너 최적화 🐺

## 4-1. TArray의 배신: 재할당과 복사

- **이사는 한 번만, 조립은 집에서.**<br>

### (1) Reserve: 이사 비용 아끼기

`TArray`가 꽉 찰 때마다 메모리를 늘리는 과정(`Malloc` -> `Memcpy` -> `Free`)은 매우 비쌈.<br>

```cpp
// ❌ [Bad] 아무 생각 없는 Add (재할당 반복)
void BadExample()
{
    TArray<FVector> Points;
    // 4 -> 8 -> 16 ... 계속 이사 다님 (System Call 발생)
    for (int32 i = 0; i < 10000; ++i) Points.Add(FVector(i, i, i));
}
```

- `std::vector` 처럼 자동 재할당을 함<br>

```cpp
// ✅ [Good] 선견지명 (Reserve)
void GoodExample()
{
    TArray<FVector> Points;
    Points.Reserve(10000); // 관리실, 방 10,000개 미리 빼놔주세요.
    for (int32 i = 0; i < 10000; ++i) Points.Add(FVector(i, i, i));
}
```

### (2) Emplace vs Add: 택배 vs 방문 조립

- **Add:** 임시 객체 생성 -> **복사/이동** -> 임시 객체 파괴.<br>
- **Emplace:** 배열 메모리 내에서 **직접 생성** (복사 비용 0).<br>
- `int`, `float`, `ptr`: **Add** (상관없음)<br>
- `struct`, `FVector`, `FString`: **Emplace** (필수)<br>

- 그럼 `Emplace`만 사용해도 되는거 아닌가?<br>
  - 이미 객체가 생성된 경우는 Add가 좀더 가시성이 좋음<br>
  - 성능상 차이가 적다면 Add가 더 '직관적'...<br>


```cpp
// ❌ [Bad] 구조체 복사 발생
Points.Add(FVector(i, i, i));

// ✅ [Good] 제자리 생성 (In-Place Construction)
Points.Emplace(i, i, i);
```

## 4-2. 컨테이너 춘추전국시대 (Container Selection)

적재적소에 맞는 도구를 쓰자.<br>

**1. TArray: 캐시의 제왕**<br>

- 특징: 메모리 연속적. 순회 속도 최강.<br>
- 용도: `Tick` 업데이트, 데이터 순차 처리.<br>
- 주의: `RemoveAt(0)` 금지! (모든 데이터를 앞으로 당겨야 함). -> `RemoveAtSwap` 사용.<br>

**2. TMap: 검색의 제왕**<br>

- 특징: `Key`로 O(1) 검색. 메모리 사용량 높음. 데이터 흩어짐.<br>
- 용도: ID로 특정 데이터 찾기.<br>
- 주의: `for`문으로 전체 순회 금지 (Cache Miss 유발).<br>

**3. TSet: 문지기**<br>

- 특징: `Key`만 존재. 중복 허용 안 함.<br>
- 용도: "이미 방문했나?", "버프 걸려있나?" 확인용.<br>
- 비교: `TArray::Contains` (O(N)) vs `TSet::Contains` (O(1))<br>

**4. TSparseArray: 구멍 난 배열**<br>

- 특징: 중간 요소 삭제 시 당겨오지 않음 (구멍 유지). 인덱스 불변 (Stable Index).<br>
- 용도: 자체적인 오브젝트 풀 구현, 엔티티 시스템.<br>

## 4-3. 구조체 패딩 (Struct Padding)

```cpp
// ❌ [Bad] 뚱뚱한 구조체 (16 bytes)
struct FBad
{
    bool bDead;    // 1 byte
    // [Padding 3 bytes] 낭비! 💨
    int32 Hp;      // 4 bytes
    bool bAngry;   // 1 byte
    // [Padding 3 bytes] 낭비! 💨
    float Dmg;     // 4 bytes
};
```

```cpp
// ✅ [Good] 다이어트 구조체 (12 bytes)
// 큰 것부터 작은 순서로 정렬 (Sort by Size)
struct FGood
{
    int32 Hp;      // 4 bytes
    float Dmg;     // 4 bytes
    bool bDead;    // 1 byte
    bool bAngry;   // 1 byte
    // [Padding 2 bytes] (최소화됨)
};
```

- 메모리 테트리스를 잘해야 한다.<br>
    - CPU는 데이터를 4byte/8byte 단위 (Word)로 읽음. 순서를 막 짜면 공기(Padding)가 들어감.<br>
- 정렬 순서 공식`Pointer` (8) -> `double` (8) -> `int`/`float` (4) -> `short` (2) -> `bool`/`byte` (1)<br>