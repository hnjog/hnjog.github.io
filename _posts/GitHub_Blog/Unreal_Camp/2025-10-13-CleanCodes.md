---
title: "김하연 튜터님 강의 - '악취가 나는 그 코드'"
date : "2025-10-13 14:00:00 +0900"
last_modified_at: "2025-10-13T14:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 클린 코드
---

# 클린 코드의 원칙들을 반면교사를 통해 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

- 버그 수정의 비용<br>
  - 개발 단계 - 0 <br>
  - QA 단계 - 1 (비용 발생)<br>
  - 라이브 단계 - 10 (매우 큰 비용)<br>

## 1. 기이한 이름 (Mysterious Name)

- 코드를 명료하게 표현하는데 가장 크게 기여하는 것은 `이름`<br>
  (개발자의 작업 중 80%가 코드를 '읽는 것'!)<br>
- 함수, 변수, 클래스, 모듈 이름만 보고도 무슨 일을 하는지 알아야 함!<br>
- 명확한 이름이 떠오르지 않는다면 설계가 잘못되었을 수 있다는 것을 명심<br>

### **나쁜 예시**

```cpp
// 이름만 보고는 이게 뭔지 알 수가 없다
void DoIt(int x);

// 의미가 전혀 안 드러나는 변수들
float AAA;
int WTF;
```

### **좋은 예시**

```cpp
// '무엇을 하기 위한 함수인지'가 분명하다
void AttackEnemy(int DamageAmount);

// 변수의 역할이 명확하다
float CurrentHealth;
int EnemyCount;
```

## 2. **중복 코드 (Duplicated Code)**

- Don't Repeat Yourself (DRY)의 원칙<br>
- 복사-붙여넣기 개발은 결국 더 많은 시간을 소비하게 함<br>
- 중복된 코드는 하나만 수정해도 되도록 모아놓을 것<br>
- 비슷하지만 조금씩 다른 코드는 공통 부분을 먼저 정리 후 분리<br>

처음 작성할땐 빠르게 해결할 수 있지만<br>
프로젝트가 진행될수록 시간을 갉아먹는 주범!<br>

### 나쁜 예시

```cpp
// 데미지 처리
void TakeDamage(float Amount)
{
    Health -= Amount;
    if (Health <= 0)
    {
        Die();
    }
}

// 보스 데미지 처리
void BossTakeDamage(float Amount)
{
    Health -= Amount;
    if (Health <= 0)
    {
        SummonMinions(); // 보스라서 특별히 미니언을 소환
        Die();
    }
}
```

### 좋은 예시

```cpp
// 공통 부모 클래스에서 데미지 로직을 통일
class AMonsterBase : public AActor
{
protected:
    virtual void OnDeath() { /* 비워두거나, 기본 처리 */ }
    
public:
    void TakeDamage(float Amount)
    {
        Health -= Amount;
        if (Health <= 0) 
        {
            OnDeath();
        }
    }
};

// 몬스터
class AFieldMonster : public AMonsterBase
{
protected:
    virtual void OnDeath() override 
    {
        // 필드 몬스터 전용 사망 처리
    }
};

// 보스
class ABoss : public AMonsterBase
{
protected:
    virtual void OnDeath() override
    {
        SummonMinions();
        // 보스 전용 사망 처리
    }
};
```

## 3. 긴 함수 (Long Function)

- 짧은 함수는 ‘무엇을 하는지’를 명확히 보여주어 코드를 쉽게 파악<br>
- 짧은 함수는 공유하기도 편함<br>
- 주석이 필요하다고 느껴지는 부분은 따로 함수로 뺄 것<br>
  의도가 드러나는 이름을 통해 작업 분리<br>

- 작은 함수로 나누면 괜한 주석을 붙이지 않아도 됨!<br>
- 긴 함수가 작성되는 것은 설계의 오류일 수 있음<br>

### 나쁜 예시

```cpp
void AMyCharacter::Tick(float DeltaTime)
{
    // 1. 이동 처리
    // 2. 점프 처리
    // 3. 공격 처리
    // 4. 버프/디버프 처리
    // 5. 체력 체크
    // 6. 애니메이션 업데이트
    // ...
    // ...
    // (500줄이 넘어가요!)
}
```

### 좋은 예시

```cpp
void AMyCharacter::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    
    HandleMovement(DeltaTime);
    HandleJump();
    HandleAttack();
    UpdateAnimation();
}

void AMyCharacter::HandleMovement(float DeltaTime)
{
    // 이동 관련 로직만 심플하게!
}

void AMyCharacter::HandleJump()
{
    // 점프 관련 로직만 모아둠
}

void AMyCharacter::HandleAttack()
{
    // 공격 로직
}
```

## 4. 긴 매개변수 목록 (Long Parameter List)

- 매개변수가 많으면 함수를 이해하고 쓰기가 너무 불편<br>
- 필요한 정보만 간결하게 전달할 수 있도록 묶거나 축소할 것<br>
- 중복된 정보가 있는지 확인하고, 불필요한 인수는 제거<br>

- 매개변수가 길면 어디에 어떤 역할을 하는 매개변수인지 헷갈리기 쉬움<br>
  -> 코드를 다시 '읽어야 함'<br>

- 구조체로 그룹화를 하면<br>
  의미가 명확해질 수 있음<br>
  (매개변수가 5개 이상이라면 연관된 구조체로<br>
  묶을 수 있는지 확인해보자)<br>

### **나쁜 예시**

```cpp
void InitWeapon(FString Name, float Damage, float FireRate, int32 AmmoCount, float ReloadTime, USkeletalMesh* Mesh, USoundBase* Sound)
{
    // 와, 많다 ...
}

// 나중에 보면 0.25f가 뭐였더라? 할 수 있다
InitWeapon("AK47", 42.0f, 0.25f, 30, 2.5f, MeshAsset, FireSound);
```

### **좋은 예시**

```cpp
// 구조체로 묶자
struct FWeaponData
{
    FString Name;
    float Damage;
    float FireRate;
    int32 AmmoCount;
};

// 구조체로 또 묶자
struct FWeaponAssets
{
    USkeletalMesh* Mesh;
    USoundBase* Sound;
};

void InitWeapon(const FWeaponData& InData, const FWeaponAssets& InAssets)
{
    // 훨씬 깔끔!
}

// 이제 이렇게 호출해서 쓰면 됨
FWeaponData WeaponInfo = { "AK47", 42.0f, 0.25f, 30, 2.5f };
FWeaponAssets Assets = { MeshAsset, FireSound };
InitWeapon(WeaponInfo, Assets);
```

## 5. 전역 데이터의 남용 (Global Data)

- 전역 데이터의 남용은 프로그램의 악취중 가장 독한 악취 중의 하나<br>
- 어디서든 접근 가능해 디버깅과 유지보수가 복잡<br>
  (모든 전역변수가 바뀌는 지점에 디버깅 포인트를 잡아야 함...)<br>
- 값이 바뀔 때 추적이 어려워 에러가 발생하기 쉬움<br>
- 데이터 범위를 최소화하고, 꼭 필요한 곳에서만 사용하도록 통제할 것!<br>

- 언리얼이라면 Subsystem을 고려해보자<br>
  (Subsystem : 엔진에서 관리하는 싱글톤과 비슷)<br>

### **나쁜 예시**

```cpp
// 글로벌 관리자
UGameManager* GGameManager; // 전역 변수!

// 아무 함수에서나 직접 접근해 값 변경
void IncreaseScore()
{
    GGameManager->Score += 10;
}
```

### **좋은 예시**

```cpp
// 언리얼 Subsystem을 이용한 예
UCLASS()
class UScoreSystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

private:
    int32 Score;

public:
    void AddScore(int32 Amount)
    {
        Score += Amount;
        // 점수가 변경됐음을 알리는 로직
    }

    int32 GetScore() const { return Score; }
};

// 사용은 이렇게 함.
void AEnemy::OnDefeated()
{
    if (UGameInstance* GI = GetGameInstance())
    {
		    // GetSubsystem<UScoreSystem>() 쓰는 곳만 접근 가능
        if (UScoreSystem* ScoreSys = GI->GetSubsystem<UScoreSystem>())
        {
            ScoreSys->AddScore(50);
        }
    }
}
```

- [Subsytem 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/programming-subsystems-in-unreal-engine){:target="_blank"}<br>

## 6. 가변 데이터 (Mutable Data)

- 값이 자주 바뀌면 예기치 못한 오류나 복잡도가 증가<br>
  (함부로 public으로 매개변수에 접근하게 두지 말자...)<br>
  (Getter,Setter도 무작정 만들기 보다 한번 더 생각하면 더 깔끔해짐)<br>

- 변경 가능한 범위를 최소화하고, 가급적 불변 데이터를 활용<br>
  (Clamp, Max, Min 등 제한하는 함수를 사용해보자)<br>

- 수정이 필요한 부분을 명확히 나누는 습관<br>

- 변수는 일단 private에 넣은 후<br>
  상황에 따라 protected, public으로 변경할 것<br>

### **나쁜 예시**

```cpp
class APlayerCharacter : public ACharacter
{
public:
    // 마음대로 바꿀 수 있는 공공재(!)
    float Health;
    int32 Level;
};

void SomeRandomFunc(APlayerCharacter* Player)
{
    Player->Health = 99999.f;
    Player->Level = 999;
    // 이걸 발견하면, 팀원들 열받음.
}
```

### **좋은 예시**

```cpp
class APlayerCharacter : public ACharacter
{
private:
		UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Stats") // 언리얼 예시
    float Health;
    int32 Level;

public:
    float GetHealth() const { return Health; }
    int32 GetLevel() const { return Level; }

    void TakeDamage(float Amount)
    {
        Health = FMath::Max(0.0f, Health - Amount);
        // 데미지 받은 로직은 여기에만!
    }

    void LevelUp()
    {
        Level++;
        Health = 100.f * Level;
    }
};
```

## 7. 뒤엉킨 변경 (Divergent Change)

- 한 모듈이 여러 이유로 자주 수정되어야 하면 복잡<br>
- 다른 맥락의 동작은 각각 다른 모듈로 분리해 단일 책임을 지킬 것<br>
- 필요에 따라 단계를 나누고 클래스를 쪼개 이해하기 쉽게 만들기<br>

- 하나에 '몰아두면' 협업할때<br>
  다들 그 클래스를 수정하므로 Git도 같이 뒤엉킨다<br>

- 클래스는 창고가 아니다<br>
  일단 때려놓지 말고 명확한 책임을 하나만 가져야 함<br>

- 지나치게 하나가 자주 수정된다면<br>
  구조가 이상한지 의심할 것<br>

### 나쁜 예시

```cpp
class AGameManager : public AActor
{
public:
    // (1) 데이터 관련
    void LoadPlayerData();
    void SavePlayerData();

    // (2) 게임플레이 관련
    void StartNewGame();
    void SpawnEnemies();

private:
    // (1) 데이터 관련 필드
    FString SaveFilePath;

    // (2) 게임플레이 관련 필드
    TArray<AEnemy*> ActiveEnemies;
};
```

### 좋은 예시

```cpp
// (1) 데이터 전용 클래스
class UPlayerDataManager : public UGameInstanceSubsystem
{
public:
    void LoadPlayerData();
    void SavePlayerData();
    // ...
};

// (2) 게임플레이 전용 클래스
class UGameplayManager : public UGameInstanceSubsystem
{
public:
    void StartNewGame();
    void SpawnEnemies();
    // ...
};
```

## 8. 샷건 수술 (Shotgun Surgery)

- 위의 내용과 유사<br>
  하나만 고치려 했는데 이상하게 많이 영향을 받음<br>

- 작은 변경을 위해 여러 곳을 동시에 수정해야 하는 상황은 피해야 함<br>
  (변경이 누락될 수 있음)<br>

- 산재된 수정 포인트가 많을수록 버그가 쉽게 발생하고 찾기 어려움<br>
  (손을 많이 댈수록 휴먼 에러의 발생 가능성 증가)<br>

- 관련된 것들은 한 군데로 모아 수정 범위를 좁힐 것<br>

- 비슷한 개념이 모이면 하나의 클래스를 생성하여<br>
  '단순'하게 접근할 것<br>
  -> 모듈 설계<br>



### 나쁜 예시

```cpp
class APlayerCharacter : public ACharacter
{
public:
    void TakeDamage(float Amount)
    {
        // 데미지 로직 1
    }
};

class AWeapon : public AActor
{
public:
    float CalculateDamage()
    {
        // 데미지 로직 2
        return 0.0f;
    }
};

class AMyGameMode : public AGameModeBase
{
public:
    void UpdateDamageLeaderboard()
    {
        // 데미지 로직 3
    }
};
```

### 좋은 예시

```cpp
class UDamageSystem : public UObject
{
public:
    // 데미지 계산 로직을 한 군데 모음!
    float CalculateDamage(AWeapon* Weapon, ACharacter* Target);
    void ApplyDamage(AWeapon* Weapon, ACharacter* Target);
    void UpdateDamageLeaderboard(ACharacter* Damager, ACharacter* Target, float Amount);
};
```

## 9. 기능 편애 (Feature Envy)

- 어떤 함수가 자기 객체보다 남의 객체 기능이나 데이터와 더 많이 소통한다면?<br>
- 그 함수를 데이터가 있는 곳으로 옮겨 의존성을 줄일 것<br>
- 서로 가까운 기능끼리 모여야 코드가 자연스럽고 관리도 수월<br>

- 다른 클래스의 멤버 변수에 지나치게 많이 접근한다고 생각하면<br>
  데이터를 가진쪽에서 처리하는 것이 더 객체 지향적임<br>

- 해당 멤버를 가지는 클래스에서 함수를 만들고<br>
  호출을 시키는 방식<br>

- 다만 잘못 적용하면 위의 '뒤엉킨 변경'과 '샷건 수술'과 연관되어<br>
  이상한 책임 전가가 발생 가능하므로 주의<br>
  (캐릭터가 대미지를 계산하게 했더니<br>
  대미지 계산 공식을 수정할때 여러번 해야 한다?)<br>
  (대미지 계산 공식을 관리하는 클래스가 존재해야 할것)<br>

### 나쁜 예시

```cpp
class UDamageCalculator : public UObject
{
public:
    float CalculateDamageReduction(AMyCharacter* Character, float Damage)
    {
        // Character의 정보를 훨씬 더 많이 사용!
        float HealthPercent = Character->GetHealth() / Character->GetMaxHealth();
        float ArmorFactor   = Character->GetArmor() * 0.1f;
        // ...
        return Damage * (1.0f - ArmorFactor * HealthPercent);
    }
};
```

### 좋은 예시

```cpp
class AMyCharacter : public ACharacter
{
public:
    float CalculateDamageReduction(float Damage) const
    {
        float HealthPercent = Health / MaxHealth;
        float ArmorFactor   = Armor * 0.1f;
        // ...
        return Damage * (1.0f - ArmorFactor * HealthPercent);
    }
};

class UDamageCalculator : public UObject
{
public:
    float CalculateDamageReduction(AMyCharacter* Character, float Damage)
    {
        // 캐릭터가 스스로 계산하게끔 위임!
        return Character->CalculateDamageReduction(Damage);
    }
};
```

## 10. 데이터 뭉치 (Data Clumps)

- 자주 함께 쓰이는 데이터는 하나로 묶으면 의미가 명확<br>
- 비슷한 데이터끼리는 모아놓을 것<br>
- 중복되는 필드나 매개변수 그룹은 별도 구조로 분리<br>

- 보통 이러한 원인은 '귀찮아서'...<br>
  구조체를 통해 묶으면 매우 깔끔해짐<br>

- 구조체가 확장되더라도 기본적으로 함수 내부에서<br>
  '사용' 하던 녀석들만 사용하기에<br>

- 또한 USTRUCT를 통해 디자이너들에게<br>
  Open 시킬 수 있음<br>
  (데이터 테이블과 연관됨)<br>
  (관리 측면에서 좋아짐)<br>

### 나쁜 예시

```cpp
void FireWeapon(float Damage, float Range, float Accuracy);
void ShowWeaponStats(float Damage, float Range, float Accuracy);
void UpgradeWeapon(float& Damage, float& Range, float& Accuracy);
```

### 좋은 예시

```cpp
// 무기 스탯 구조체
USTRUCT(BlueprintType)
struct FWeaponStats
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Damage;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Range;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Accuracy;
};

void FireWeapon(const FWeaponStats& Stats);
void ShowWeaponStats(const FWeaponStats& Stats);
void UpgradeWeapon(FWeaponStats& Stats);
```

## 11. 기본형 집착 (Primitive Obsession)

- 복잡한 데이터를 단순한 기본형 (int, string)에 과도하게 의존하는 경향<br>
  (타입의 의미에 의미가 있나?)<br>
  (float 선언하더라도 체력은 '음수'가 될 수 있음)<br>
  (때로는 유효성 검사를 필요할 수 있으나<br>
   깜빡하거나 귀찮아서 안넣는 경우 존재...)<br>

- 복잡한 개념은 기본형 대신 클래스나, 구조체를 사용해서 차라리 해결할 것<br>

- 좋은 타입 설계는 복잡한 로직과 버그 발생을 줄여줌<br>
  (타입 형을 보고 의미를 깨닫기 쉬움)<br>
  (내부적으로 값 제한을 걸 수 있기에 디버깅 시간이 줄어듦)<br>

- UAttributeSet 과 같은 스탯형으로 사용하기 좋은 클래스도 존재<br>
  (귀찮더라도 개념마다 클래스를 생성하는게<br>
  개념적/설계적 으로 좋을 수 있음)<br>

### 나쁜 예시

```cpp
float Health;
float MaxHealth;

FString PhoneNumber; // 형식 검증이 전혀 없음
```

### 좋은 예시

```cpp
// 체력을 표현하는 클래스
class FHealth
{
public:
    FHealth(float InCurrent, float InMax)
        : Current(FMath::Clamp(InCurrent, 0.f, InMax)), Max(InMax) {}

    void ApplyDamage(float Amount)
    {
        Current = FMath::Max(0.f, Current - Amount);
    }

    float Get() const { return Current; }

private:
    float Current;
    float Max;
};

// FHealth를 사용해보자
class AMyCharacter : public ACharacter
{
public:
		// 이렇게 FHealth를 씀.
    FHealth Health = FHealth(100.f, 100.f);

    void TakeHit(float Damage)
    {
        Health.ApplyDamage(Damage);

        if (Health.Get() <= 0.f)
        {
            Die();
        }
    }

private:
    void Die()
    {
        // 사망 처리 로직
    }
};
```

## 12. 반복되는 스위치문 (Repeated Switches)

- 새로운 분기가 생길 때마다 여러 switch문을 전부 수정해야 한다면 비효율적<br>
  (Enum 500개 넘어가면 진짜 어지럽다)<br>
- 다형성 구조를 적용해 중복되는 분기 로직을 없앨 것<br>
  처음에 귀찮더라도 나중 가면 함수 수정을 안해도 됨<br>

- 그냥 Switch문 제발 좀 쓰지말것<br>
  (개방 폐쇄 원칙을 지키는 쉬운 방법은<br>
   Switch 문을 없애고 다형성을 이용하는 것)<br>

### 나쁜 예시

```cpp
switch (WeaponType)
{
    case EWeaponType::Sword:
        return DoSwordAttack();
    case EWeaponType::Bow:
        return DoBowAttack();
    case EWeaponType::Gun:
        return PewPew();
}
```

### 좋은 예시

```cpp
// 다형성 활용...!
// 무기 베이스
class AWeapon : public AActor
{
public:
    virtual void Attack();
};

// 무기별 클래스
class ASword : public AWeapon
{
public:
    virtual void Attack() override { /* 칼 공격 로직 */ }
};

class ABow : public AWeapon
{
public:
    virtual void Attack() override { /* 활 공격 로직 */ }
};
```

```cpp
// 그리고 캐릭터 쪽에서는 더 이상 switch 안 씀.
void AMyCharacter::UseWeapon()
{
    if (EquippedWeapon)
    {
        EquippedWeapon->Attack(); // 알아서 잘함
    }
}
```

## 13. 반복문 (Loops)

- 루프 안에 비즈니스 로직을 다 넣지 말것<br>
- 반복문은 성능 저하의 원인<br>
  (중첩 반복문은 말할 것도 없음)<br>

- 핵심 동작을 '나누어' 의도를 명확히 들어내는 것<br>
  -> `'반복문' 보면서 '의도 추론'`할 **필요 없음**<br>

- 반복문을 별도로 분리하여<br>
  그 의미를 명확히 이해하도록 할 것<br>
  (가독성 향상)<br>

### 나쁜 예시

```cpp
// 인벤토리에서 무거운 아이템을 찾아서 무게를 계산하는 과정
void ProcessHeavyItems()
{
    TArray<UItem*> Items = GetAllItems();
    TArray<UItem*> HeavyItems;

    // (1) 무거운 아이템 골라내기
    for (int32 i = 0; i < Items.Num(); i++)
    {
        if (Items[i]->Weight > 10.f)
        {
            HeavyItems.Add(Items[i]);
        }
    }

    // (2) 무게 총합 계산
    float TotalWeight = 0.f;
    for (int32 j = 0; j < HeavyItems.Num(); j++)
    {
        TotalWeight += HeavyItems[j]->Weight;
    }

    // (3) 너무 무거우면 효과 적용
    if (TotalWeight > 50.f)
    {
        ApplySlowEffect();
    }
}
```

### 좋은 예시

```cpp
void ProcessHeavyItems()
{
    // 모든 아이템 가져오기
    TArray<UItem*> Items = GetAllItems();

    // 무게 10 이상인 아이템만 필터링
    TArray<UItem*> HeavyItems = GetHeavyItems(Items);

    // 필터링된 아이템의 총 무게 계산
    float TotalWeight = GetTotalWeight(HeavyItems);

    // 총 무게가 기준치를 초과하면 느려지는 효과 적용
    if (IsTooHeavy(TotalWeight))
    {
        ApplySlowEffect();
    }
}

// 무거운 아이템만 골라내는 함수
TArray<UItem*> GetHeavyItems(const TArray<UItem*>& Items)
{
    TArray<UItem*> Result;
    for (UItem* Item : Items)
    {
        if (Item && Item->Weight > 10.f)
        {
            Result.Add(Item);
        }
    }
    return Result;
}

// 아이템 배열의 총 무게를 계산하는 함수
float GetTotalWeight(const TArray<UItem*>& Items)
{
    float Total = 0.f;
    for (UItem* Item : Items)
    {
        if (Item)
        {
            Total += Item->Weight;
        }
    }
    return Total;
}

// 너무 무거운지 판단하는 기준 함수
bool IsTooHeavy(float Weight)
{
    return Weight > 50.f;
}
```

## 14. 게으른 요소 (Lazy Element)

- 하는 일 없이 존재만 하는 메서드나 클래스는 오히려 혼동을 준다<br>
  (패스만 하는 코드가 왜 필요하지..?)<br>

- 코드 흐름상 실제로 필요 없는 구조는 과감히 없애자. 지우기 귀찮아도 삭제할 것<br>
  (이거 어떤 의도가 있는거지? 싶은 상황을 없앰)<br>

- ‘단순화’ 라는 것을 명심<br>

- 실제 '필요'할 때 다시 만들것!<br>

- 그렇다고 꼭 안좋다거나 지워야 하는 것은 아님<br>
  (계획이 확실한 경우라면 내버려 두는 것도 방법)<br>
  (혹은 인터페이스 같이 반드시 구현하는 경우)<br>

### 나쁜 예시

```cpp
// 과도하게 중간함수만 존재
class AProjectile : public AActor
{
public:
    void Launch(const FVector& Dir, float Speed)
    {
        // 여기서 다시 다른 함수를 호출만 함
        LaunchProjectile(Dir, Speed);
    }

private:
    void LaunchProjectile(const FVector& Dir, float Speed)
    {
        // 실제 로직
        ProjectileMovement->Velocity = Dir * Speed;
    }
};
```

### 좋은 예시

```cpp
class AProjectile : public AActor
{
public:
    void Launch(const FVector& Dir, float Speed)
    {
        ProjectileMovement->Velocity = Dir * Speed;
    }

private:
    UProjectileMovementComponent* ProjectileMovement;
};
```

## 15. 추측성 일반화 (Speculative Generality)

- 현재 필요한 기능에 집중해 불필요한 추상화를 걷어낼 것<br>
  (확장성을 너무 고려??)<br>
  (TODO : 유령...)<br>
  (나중에 생각보다 안쓰게 되는 경우가 많다 함)<br>

- 미래 대비보다 현재 문제 해결이 우선<br>

- “나중에 필요할 수도 있어”라는 생각으로 만든 코드는 대부분 짐이 됨<br>
  : 자동완성에 걸리기에 '이게 뭐지' 싶은 경우가 종종 있음<br>

- 그렇다고 다른 사람들이 이거 지우는 것도 아님<br>
  (뭐 이유가 있지 않을까?)<br>

- 확장성은 필요할때 고려해도 늦지는 않으며<br>
  처음에 기능 구현할때는 필요한 것만 구현하기<br>
  (프로토 타입 구현)<br>
  (나중에 기획이 엎어질수도...)<br>

### 나쁜 예시

```cpp
// 엄청나게 확장 가능한 무기 클래스... 그런데 전혀 안 씀
class AWeapon : public AActor
{
public:
    virtual void APlayer::PlayWeaponSound()
{
    USoundBase* AttackSound = GetEquippedWeaponSound();
    if (AttackSound)
    {
        UGameplayStatics::PlaySound2D(this, AttackSound);
    }
}

USoundBase* APlayer::GetEquippedWeaponSound()
{
    // 아래 호출부에서 직접 소리를 반환
    return Inventory ? Inventory->GetAttackSound() : nullptr;
}

USoundBase* UInventoryComponent::GetAttackSound()
{
    if (!EquippedWeapon) return nullptr;
    return EquippedWeapon->GetAttackSound();
}

USoundBase* AWeapon::GetAttackSound()
{
    return SoundData ? SoundData->AttackSound : nullptr;
}ttack();
    virtual void SpecialAttack();   // 안 씀
    virtual void UltimateAttack();  // 안 씀
    virtual void ElementalAttack(); // 안 씀
    // ...

    void SetDamage(float BaseDamage, float Crit, float Splash, float Chain, float Summon);
    // TODO: 추후에 쓸 수도?
};
```

### 좋은 예시

```cpp
class AWeapon : public AActor
{
public:
    // 필요한 기능만
    void Attack();
    void SetDamage(float InDamage);

private:
    float Damage;
};

// 필요할 때 다른 무기 타입을 '상속'해서 만듦
class AMagicWeapon : public AWeapon
{
    void ElementalAttack();
};
```

## 16. 임시 필드 (Temporary Field)

- 목적이 분명치 않은 필드는 코드 복잡도를 높이는 원인<br>
  (다른 클래스는 안쓰는 필드를 왜 상속 받지...?)<br>

- 특정 상황에서만 쓰이는 필드는 다른 상황에선 쓸데없는 혼란을 부른다<br>
  (Null 검사 + 불필요한 메모리 낭비)<br>

- 사용되지 않는 시점이 더 많다면 다른 구조로 옮기거나 클래스로 분리<br>
  (필요한 녀석만 기능 분리)<br>

- 기능을 '컴포넌트'로 분리시키기<br>
  이후 HasA 를 통해 기능 확인 가능<br>
  (컴포넌트 유무로 기능 파악)<br>

### 나쁜 예시

```cpp
class AEnemy : public ACharacter
{
public:
    // 일반 공격
    float Health;

    // 원거리 공격 전용 (근접 적은 안 씀)
    float ProjectileSpeed;
    UParticleSystem* ProjectileEffect;

    // 텔레포트 전용 (다른 적은 안 씀)
    float TeleportCooldown;
    float LastTeleportTime;
};

```

### 좋은 예시

```cpp
// "컴포넌트"로 분리
class URangedAttackComponent : public UActorComponent
{
    float ProjectileSpeed;
    void ExecuteAttack();
};

class UTeleportComponent : public UActorComponent
{
    float TeleportCooldown;
    void ExecuteTeleport();
};

// 적 캐릭터
class AEnemy : public ACharacter
{
    float Health;
    URangedAttackComponent* RangedComp;   // 원거리 적만 붙임
    UTeleportComponent* TeleportComp;     // 텔레포트 적만 붙임
};
```

## 17. 메시지 체인 (Message Chains)

- 클래스도 프라이버시가 존재<br>

- 객체를 줄줄이 호출하면 내부 구조가 노출돼 결합도가 커짐<br>
  (이것만 붙여야지 -> 반복 -> 누더기 골렘 탄생...)<br>

- 필요하다면 최종 로직을 호출부 가까이로 옮겨 의존을 줄일 것<br>

- Null 검사를 '명확'하게 해야 하는 경우<br>
  함수로 나누어 책임을 나눌 것!<br>
  (단계를 나누어, 직접적으로 필요한 부분만 요청할 것!)<br>
  (한 함수에서 Null체크 여러번 하지 않고<br>
  그냥 함수를 통해 요청하면 책임이 분산)<br>

- 이러한 같은 함수에서 '여러 요소'를 가져다 사용하는 것을<br>
  '메시지 체인'이라 하며 피해야 하는 방식<br>

### 나쁜 예시

```cpp
// 길~~게 이어진 참조
void APlayer::PlayWeaponSound()
{
    if (Inventory
        && Inventory->EquippedWeapon
        && Inventory->EquippedWeapon->SoundData
        && Inventory->EquippedWeapon->SoundData->AttackSound)
    {
        UGameplayStatics::PlaySound2D(this, Inventory->EquippedWeapon->SoundData->AttackSound);
    }
}
```

### 좋은 예시

```cpp
void APlayer::PlayWeaponSound()
{
    USoundBase* AttackSound = GetEquippedWeaponSound();
    if (AttackSound)
    {
        UGameplayStatics::PlaySound2D(this, AttackSound);
    }
}

// 플레이어는 인벤토리한테만 물어봄
USoundBase* APlayer::GetEquippedWeaponSound()
{
    // 아래 호출부에서 직접 소리를 반환
    return Inventory ? Inventory->GetAttackSound() : nullptr;
}

// 인벤토리는 무기한테만 물어봄
USoundBase* UInventoryComponent::GetAttackSound()
{
    if (!EquippedWeapon) return nullptr;
    return EquippedWeapon->GetAttackSound();
}

// 무기는 사운드만 알고 있음
USoundBase* AWeapon::GetAttackSound()
{
    return SoundData ? SoundData->AttackSound : nullptr;
}
```

## 18. 중재자 (Middle Man)

- 실질적 로직 없이 위임만 하는 클래스는 존재 가치가 의심<br>
  (캡슐화를 너무 의식한 경우)<br>

- 직접 연결해도 문제가 없다면 중간 단계를 제거<br>
  (이 함수 필요 없지 않나?)<br>
  (그냥 포장만 하는거 아닌가?)<br>
  (디버깅이 귀찮아지며<br>
  책임이 이상해짐)<br>
  (이 클래스에 요청했는데 왜 하청을 맡기지?)<br>

- 늘 직관적 구조로 수정할 것<br>

- 중재자는 '흐름을 바꾸는 경우'에 유용함<br>
  (우선순위 구분, 조건 검사)<br>

- 언리얼에서 IMC를 바꾼다던가 할 때<br>
  '중재자' 로직이 유용해질 수 있음<br>


### 나쁜 예시

```cpp
class AMyPlayerController : public APlayerController
{
public:
    void MoveForward(float Value)  { Character->MoveForward(Value); }
    void MoveRight(float Value)    { Character->MoveRight(Value); }
    void Jump()                    { Character->Jump(); }
    void StartFire()               { Character->StartFire(); }
    void StopFire()                { Character->StopFire(); }
    // ...
private:
    AMyCharacter* Character;
};
```

### 좋은 예시

```cpp
// 직접 캐릭터에 입력 바인딩
void AMyPlayerController::SetupInputComponent()
{
    Super::SetupInputComponent();

    // 현재 캐릭터 가져오기
    AMyCharacter* MyChar = Cast<AMyCharacter>(GetCharacter());
    if (MyChar && InputComponent)
    {
        // 캐릭터가 필요한 입력을 직접 바인딩
        MyChar->SetupPlayerInput(InputComponent);
    }
}

void AMyCharacter::SetupPlayerInput(UInputComponent* PlayerInputComponent)
{
    PlayerInputComponent->BindAxis("MoveForward", this, &AMyCharacter::MoveForward);
    PlayerInputComponent->BindAxis("MoveRight", this, &AMyCharacter::MoveRight);
    // ...
}
```

## 19. 내부자 거래 (Insider Trading)

- 모듈 간에 비공개 데이터가 과하게 오가면 결합도가 높아진다<br>
  (다른 클래스를 '니'가 왜 수정해...?)<br>
  (과하면 '해킹' 스크립트와 닮아짐)<br>

- 필요한 정보만 교환할 수 있게 인터페이스 범위를 명확히 정의<br>
  (내 할일만 하고, 해당 클래스가 작업하도록 함수를 호출)<br>

- 모듈 간 벽을 두껍게 유지해 각자 책임을 분리<br>
  (남의 클래스를 민감한 부분을 건드리면 안된다)<br>

### 나쁜 예시

```cpp
// AEnemy가 APlayerCharacter의 내부 변수까지 막 참조
void AEnemy::Attack(APlayerCharacter* Player)
{
    if (!Player->bIsInvulnerable)
    {
        float Damage = AttackDamage - Player->EquippedArmor->DamageReduction;
        Player->CurrentHealth -= Damage;

        // UI도 직접 갱신?!
        Player->PlayerHUD->UpdateHealthBar(Player->CurrentHealth, Player->MaxHealth);
    }
}
```

### 좋은 예시

```cpp
void AEnemy::Attack(APlayerCharacter* Player)
{
    if (Player && Player->CanBeAttacked())
    {
        Player->ReceiveDamage(AttackDamage);
    }
}

// Player 쪽 내부 함수들 1
bool APlayerCharacter::CanBeAttacked() const
{
    return !bIsInvulnerable;
}

// Player 쪽 내부 함수들 2
void APlayerCharacter::ReceiveDamage(float IncomingDamage)
{
    float FinalDamage = EquippedArmor ? EquippedArmor->ApplyReduction(IncomingDamage) : IncomingDamage;
    CurrentHealth = FMath::Clamp(CurrentHealth - FinalDamage, 0.f, MaxHealth);

    UpdateHUD();
}

// Player 쪽 내부 함수들 3
void APlayerCharacter::UpdateHUD()
{
    if (PlayerHUD)
    {
        PlayerHUD->UpdateHealthBar(CurrentHealth, MaxHealth);
    }
}
```

## 20. 거대한 클래스 (Large Class)

- 너무 많은 책임을 지는 클래스는 필드와 메서드가 폭발적으로 늘어남<br>
  (갓 클래스...)<br>
  (협업 하기 힘들어지며, 책임 구분하기 힘들어짐)<br>

- 중복이 생기고 관리가 어려워지므로 역할이나 기능별로 분리<br>
  (컴포넌트가 유용한 이유!)<br>
  (분리하여 작업하면 협업도 유용)<br>

- 사용 패턴을 분석해 클래스를 쪼개면 유지보수가 훨씬 수월<br>

### 나쁜 예시

```cpp
class AGameCharacter : public ACharacter
{
public:
    // 이동 처리
    void MoveForward(float Value);
    void MoveRight(float Value);
    // 전투 처리
    void Attack();
    void Reload();
    // 인벤토리 처리
    void AddItem(UItem* Item);
    void RemoveItem(UItem* Item);
    // 퀘스트 처리
    void AcceptQuest(UQuest* Quest);
    void CompleteQuest(UQuest* Quest);
    // 대화 처리
    void StartDialogue();
    void EndDialogue();
    // ... 500줄 넘게 계속 ...
};
```

### 좋은 예시

```cpp
class AGameCharacter : public ACharacter
{
public:
    AGameCharacter();
    // 핵심 동작만 유지, 나머지는 컴포넌트에 맡김
private:
    UPROPERTY()
    UMovementComponent* MovementComp;

    UPROPERTY()
    UCombatComponent* CombatComp;

    UPROPERTY()
    UInventoryComponent* InventoryComp;

    UPROPERTY()
    UQuestComponent* QuestComp;
    // ...
};
```

## 21. 서로 다른 인터페이스의 대안 클래스들 (Alternative Classes with Different Interfaces)

- 클래스를 교체하려면 인터페이스가 호환되어야 함<br>
  (같은 기능이라면 '통일'해야 함)<br>

- 유사 기능 클래스끼리 일관된 형식을 갖추는 것이 좋음<br>
  (일관성이 없어지면, '분기'가 생기며<br>
  위의 Switch 같은 문제를 유발할 수 있음)<br>

- 메서드 시그니처를 통일해 교체 가능성을 높일 것<br>
  (다형성은 아주 훌륭한 도구)<br>

### 나쁜 예시

```cpp
class ARangedWeapon
{
public:
    void FireProjectile();
    void Reload();
};

class AMeleeWeapon
{
public:
    void PerformAttack();
    void SharpenBlade();
};

// 플레이어 캐릭터
void APlayerCharacter::Attack()
{
    if (CurrentRangedWeapon)
        CurrentRangedWeapon->FireProjectile();
    else if (CurrentMeleeWeapon)
        CurrentMeleeWeapon->PerformAttack();
}
```

### 좋은 예시

```cpp
class AWeapon : public AActor
{
public:
    virtual void Attack() = 0;  // 추상 메서드
    virtual void Reload() {}    // 기본 구현(근접 무기는 비워둘 수도)
};

class ARangedWeapon : public AWeapon
{
public:
    virtual void Attack() override { /* 원거리 공격 */ }
    virtual void Reload() override { /* 탄약 보충 */ }
};

class AMeleeWeapon : public AWeapon
{
public:
    virtual void Attack() override { /* 근접 공격 */ }
    // Reload()는 오버라이드 안 해도 됨(불필요)
};

// 캐릭터는 이제 딱 한 줄로 호출
void APlayerCharacter::Attack()
{
    if (CurrentWeapon)
    {
        CurrentWeapon->Attack(); // 무기 종류 관계없이 한 번에 호출
    }
}
```

## 22. 데이터 클래스 (Data Class)

- 필드와 게터/세터만 있는 클래스는 다른 곳에서 함부로 조작되기 쉬움<br>
  (있는것은 좋은데 근데 이러면 그냥 public이랑 같지 않나?)<br>

- 변경될 필요가 없는 필드는 세터를 제거해 안전성을 높이기<br>
  ('값'만 있다면 클래스라기 보단 CSV...)<br>

- 필요 기능이 있다면 이 클래스 안에 직접 구현해 응집도를 높일 것<br>
  (행동 로직을 통해 '책임'을 구분 가능)<br>

### 나쁜 예시

```cpp
class FPlayerStats
{
public:
    float GetHealth() const { return Health; }
    void SetHealth(float H) { Health = H; }

private:
    float Health;
    float MaxHealth;
};

// 플레이어가 데미지를 주면서 stats를 수동 조작
void APlayerCharacter::TakeDamage(float Damage)
{
    float NewHealth = PlayerStats.GetHealth() - Damage;
    PlayerStats.SetHealth(FMath::Max(0.f, NewHealth));
}
```

### 좋은 예시

```cpp
class FPlayerStats
{
public:
    // 함수 안에서 로직 처리
    void ApplyDamage(float Damage)
    {
        float ActualDamage = Damage * (1.0f - Defense / 100.f);
        Health = FMath::Max(0.f, Health - ActualDamage);
    }

    bool IsDead() const { return Health <= 0.f; }

private:
    float Health;
    float Defense;
};
```

```cpp
// 플레이어
void APlayerCharacter::TakeDamage(float Damage)
{
    PlayerStats.ApplyDamage(Damage);
    if (PlayerStats.IsDead())
    {
        Die();
    }
}
```

## 23. 상속 포기 (Refused Bequest)

- 서브클래스가 부모의 기능 중 일부만 필요하거나 인터페이스가 맞지 않는다면?<br>
  (IsA 관계를 해치지 말것)<br>
  (상속은 '반드시 쓸 기능'을 건네줘야 함)<br>

- 꼭 상속하지 않더라도, 필요한 부분만 다른 방식으로 얻을 수 있음<br>
  (모든 자식이 안쓴다면, 그 기능을 쓰는 자식쪽에서 구현해야 함)<br>

- 위임 등으로 불필요한 유산을 거부해 구조를 단순화<br>
  (쓰는 쪽에서 구현 해!)<br>

### 나쁜 예시

```cpp
class AWeapon
{
public:
    virtual void Attack();
    virtual void Reload(); // 근접 무기는 재장전 필요 X
};

class AMeleeWeapon : public AWeapon
{
public:
    virtual void Reload() override
    {
        // 근접 무기에선 의미가 없으니 비워둠
    }
};
```

### 좋은 예시

```cpp
class AWeapon : public AActor
{
public:
    virtual void Attack() = 0;
};

class ARangedWeapon : public AWeapon
{
public:
    virtual void Attack() override { FireProjectile(); }
    void Reload() { /* 탄약 충전 */ }
};

class AMeleeWeapon : public AWeapon
{
public:
    virtual void Attack() override { SwingBlade(); }
    // Reload 없음! 필요 없으니까!
};
```

## 24. 주석 (Comments)

- 물론 올바른 주석은 아주 좋다. 그러나 코드만으로 명확하게 이해가 되는게 훨씬 더 중요<br>
  (책임을 나누는 것을 두려워하지 말 것!)<br>

- 주석은 사실 코드를 변명하기 위한 장치에 가까움<br>
  (함수 나누기 귀찮니..?)<br>

- 주석이 필요없는 좋은 코드가 되도록 노력<br>
  (코드 수정은 하지만 주석 수정은 안하는 경우가 태반)<br>

- 진짜 필요한 경우에만 TODO를 남길 수 있으나<br>
  Git에 올릴 필요는 없음<br>

- 아니면 진짜 그렇게 작성한 이유를 적기<br>
  (성능 이슈 / 버그로 인한 회피)<br>
  (문서 자동화를 위하여)<br>

- 주석은 필요할 때만 쓰자!<br>
  (개인 공부용 등)<br>

### 나쁜 예시

```cpp
void AEnemy::UpdateBehavior()
{
    // 1. 플레이어 위치 가져오기
    // 2. 시야 범위 확인
    // 3. 시야 각도 계산
    // 4. 라인 트레이스 해서 장애물 있는지
    // 5. 없으면 공격, 있으면 패트롤
    // 50줄짜리 함수에 각 단계별 설명이 잔뜩 → 너무 복잡.
    ...
}
```

### 좋은 예시

```cpp
void AEnemy::UpdateBehavior()
{
    if (CanSeePlayer())
    {
        EngagePlayer();
    }
    else
    {
        PatrolArea();
    }
}

bool AEnemy::CanSeePlayer()
{
    return IsWithinSightRange() && IsInFieldOfView() && HasLineOfSight();
}
```

## 마치며

단숨에 하라는 것이 아니니 안심할 것<br>
이러한 내용을 '기억'하고<br>

- 구현한 이후, 다시 고쳐보기<br>
  (나중에 고치면 진짜 문제 터지기 쉬움)<br>

- 시간 때문에 빠르게 작성하더라도 나중에 리팩토링할때 체크할 것<br>

