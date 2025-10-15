---
title: "김하연 튜터님 강의 - '리팩터링의 두 얼굴 - 세분화와 통합 사이에서 살아남기'"
date : "2025-10-15 14:00:00 +0900"
last_modified_at: "2025-10-15T14:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 리팩토링
---

# 리팩토링의 서로 상반되는 핵심 결정 원칙들을 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

- 리팩토링<br>
 : 기능을 고친다 x , 버그 수정 x<br>
   코드 품질을 올린다!<br>
   (가독성, 유지보수성)<br>

- 잠재적인 기술 부채를 미리미리 털어내는 것...!<br>
  (불편하다를 넘어서서 작업 속도에 영향을 줄 수 있으므로)<br>

- 리팩토링은 중간을 찾는 것에 가까움<br>
  ex)<br>
  **함수로 많이 나누세요 vs 함수 호출은 잠재적으로 성능 영향**<br>

- 결국은 구조와의 싸움<br>
  = 끝없는 반복과 개선<br>

- 코드를 보고 한 번에 알 수 있느냐~<br>
  아니라면 리팩토링의 대상이다!<br>

# 1. 코드 세분화 vs. 코드 통합

- 나눌까 말까<br>
    - 안나누자니 함수가 너무 길어지는데?<br>
    - 나누니까 너무 함수를 많이 호출하는데?<br>

## 코드 세분화

요점은 코드를 '나누는 것'<br>

### 💡 함수 추출하기 (Extract Function)

#### 🧨 Before

```cpp
void Player::Update()
{
    if (health < 0)
    {
        isAlive = false;
        animation.Play("Death");
        sound.Play("DeathSFX");
        gameManager.NotifyPlayerDied(this);
    }
}
```

- 업데이트 안에 Player 죽음 내용 까지 포함되는데?<br>

#### ✅ After

```cpp
void Player::Update()
{
    if (health < 0)
    {
        HandleDeath();
    }
}

void Player::HandleDeath()
{
    isAlive = false;
    animation.Play("Death");
    sound.Play("DeathSFX");
    gameManager.NotifyPlayerDied(this);
}
```

- 함수 하나 파서 '의도'를 챙겨주자<br>
- 별거 아니지만 '함수명'을 통해 가독성을 올릴 수 있음<br>

### 💡 변수 추출하기 (Extract Variable)

#### 🧨 Before

```cpp
float damage = baseDamage * (1 + criticalChance * 2.0f);
```

- 저 계산 결과는 뭘 의미하는거지?<br>
- 저 수치를 디버깅으로 보기 힘들다<br>

#### ✅ After

```cpp
float critMultiplier = 1 + criticalChance * 2.0f;
float damage = baseDamage * critMultiplier;
```

- 아 크리티컬 배율이구나<br>
- 디버깅에 저 변수명 넣고 조사식 돌려도 됨<br>

### 💡 클래스 추출하기 (Extract Class)

#### 🧨 Before

```cpp
class Player
{
    FString Name;
    FString Address;
    FString Email;
};
```

- 게임 캐릭터가 이메일과 주소를 왜 가져...?<br>

#### ✅ After

```cpp
class ContactInfo
{
    FString Address;
    FString Email;
};

class Player
{
    FString Name;
    ContactInfo Contact;
};
```

- `접속 정보`로 나누어 별도의 개념으로 관리하자!<br>
- 다만 클래스 명을 너무 '세부적'으로 작성할 필요는 없음<br>

## 코드 통합

쪼갤만큼 쪼갰으면 붙여!<br>

### 💡 함수 인라인하기 (Inline Function)

#### 🧨 Before

```cpp
bool Player::IsDead()
{
    return health <= 0;
}

void Player::Update()
{
    if (IsDead())
    {
        HandleDeath();
    }
}
```

- 한줄인데? 굳이?<br>
- 이미 체력 < 0이면 죽는게 직관적이지 않나?<br>
- 이거 굳이 함수 호출해야 하나?<br>

#### ✅ After

```cpp
void Player::Update()
{
    if (health <= 0)
    {
        HandleDeath();
    }
}
```

- 여기서만 쓰면 그냥 이렇게 쓰는게 더 좋지 않아?<br>
    - 물론 가독성을 주거나, 여러 사용처가 있다면 함수화시키는 것이 더 좋을 수 있음<br>
      (너무 복잡한 수식은 그냥 함수화해서 가독성 챙겨주는것은 괜찮음)<br>

### 💡 변수 인라인하기 (Inline Variable)

#### 🧨 Before

```cpp
int priceAfterDiscount = originalPrice - discountAmount;
int finalPrice = priceAfterDiscount + shippingFee;
```

- 한줄만 쓰이고 재사용 안하는 것 같은데?<br>
- 각각의 변수명이 명확한 것 같은데<br>

#### ✅ After

```cpp
int finalPrice = (originalPrice - discountAmount) + shippingFee;
```

- 어차피 의미가 명확한데 그냥 직접 쓰자...<br>
    - 다만 수식이 복잡하다면 의미를 부여하는 것은 옳다<br>

### 💡 클래스 인라인하기 (Inline Class)

#### 🧨 Before

```cpp
class Position
{
public:
    float X, Y;
};

class Enemy
{
    Position Pos;
};
```

- Position이 x,y만 사용하지 않아?<br>
  그리도 Enemy만 사용하는데?<br>

#### ✅ After

```cpp
class Enemy
{
public:
    float X, Y;
};
```

- 그냥 사용하거나 '구조체'로 사용하는게 깔끔하지 않아?<br>

## **결정 기준**

| 세분화가 적합한 상황 | 통합이 적합한 상황 |
| --- | --- |
| 코드가 복잡하고 이해하기 어려울 때 | 추상화가 오히려 복잡성을 증가시킬 때 |
| 재사용 가능성이 있을 때 | 간단한 로직이 불필요하게 분리되어 있을 때 |
| 변경 가능성이 높은 부분 | 사용되는 곳이 한 곳뿐인 단순한 코드 |
| 무엇과 어떻게를 분리할 필요가 있을 때 | 함수/변수 이름이 실제 로직에 가치를 더하지 않을 때 |

- 코드 중복은 죄악이기에 재사용성을 높이기에 세분화가 필요<br>

- 다만 현재 하나만 사용하고 있다면 굳이 세분화할 필요는 없음<br>

- 무엇 과 어떻게 를 구분하는 것도 방법<br>
  - 각 코드의 '목적'에 따른 분류를 잊지 말 것!<br>
  - 다만 목적성이 명확하더라도 '한 곳'만 쓰이고 있다면 통합을 고려하기<br>

- 보통 대형 프로젝트에서 고려<br>
  - 소형프로젝트라면 '공부용'으로서의 가치 정도<br>

# 2. 객체 세분화 vs. 객체 통합 🤔

- 객체 뭉칠까 말까<br>
    - 객체를 다루는 접근법에 대하여 생각해보자<br>

## 객체 세분화

### 💡 매개변수 객체 만들기 (Introduce Parameter Object)

#### 🧨 Before

```cpp
void CreateQuest(FString title, FString description, int rewardGold, int exp);
```

- 이전 강의에서 보았던 '많은 매개변수'<br>
- 데이터 추가시, 함수 매개변수가 추가...?<br>

#### ✅ After

```cpp
void CreateQuest(const FQuestData& Quest);
```

```cpp
struct FQuestData
{
    FString Title;
    FString Description;
    int RewardGold;
    int Exp;
};
```

- 순서 기억을 굳이 할 필요 없음!<br>
- 구조체 값만 추가/수정 하면 되기에 더 편리<br>
    - 구조체 이름도 어느정도 신경쓰고<br>
      구조체의 데이터 구조도 신경쓸것!<br>
      (구조체 안에 구조체 있는 건....)<br>

### 💡 단계 쪼개기 (Split Phase)

#### 🧨 Before

```cpp
void LoadAndDisplayItem()
{
    Item item = LoadItemFromDisk("item.json");
    RenderItemOnScreen(item);
}
```

- 아이템 Load 후, Render<br>
- 중간 단계 못나누나...?<br>

#### ✅ After

```cpp
Item LoadItem();
void DisplayItem(const Item& item);
```

```cpp
Item item = LoadItem();
DisplayItem(item)
```

- 함수로 쪼개자<br>
  (함수에 and 붙이지 말자..!)<br>

- 2개 이상의 역할을 같이 한다면 쪼개는 것 고려<br>

### 💡 반복문 쪼개기 (Split Loop)

#### 🧨 Before

```cpp
for (const Item& item : Inventory)
{
    item.CalculateWeight();
    item.ApplyDurabilityDecay();
}
```

- 효율적으로 보이나<br>
  왜 같이 하지...?<br>

- 성능 걱정?<br>
  - 프로파일링을 보고 생각하기<br>
  - 이게 걱정될 수준이면 AI 나 파티클을 먼저 고려해야 하는 상황<br>

#### ✅ After

```cpp
for (const Item& item : Inventory)
{
    item.CalculateWeight();
}

for (const Item& item : Inventory)
{
    item.ApplyDurabilityDecay();
}
```

- 명확한 목적을 가지게 '나누는 것'을 고려하기<br>
  (분리하기 쉬우며, 멀티스레딩 환경에서 락 등을 걸기 더 쉬운 구조)<br>

## 객체 통합

쪼개다 보면<br>
합칠 것도 보인다!<br>
(비슷한 개념끼리 뭉치는 것)<br>

### 💡 여러 함수를 클래스로 묶기 (Combine Functions into Class)

#### 🧨 Before

```cpp
void StartTimer();
void StopTimer();
float GetElapsedTime();
```

- 전부 시간 관련이네?<br>
- 각자 비슷한 변수를 사용할 것 같은데...<br>

#### ✅ After

```cpp
class Timer
{
public:
    void Start();
    void Stop();
    float GetElapsed() const;

private:
    float StartTime;
    float EndTime;
};
```

- 유틸리티 클래스로 만들어서 별도로 관리하도록 하자<br>


### 💡 여러 함수를 변환 함수로 묶기 (Combine into Transform Function)

#### 🧨 Before

```cpp
float GetBaseDamage();
float GetCritBonus();
float GetTotalDamage()
{
    return GetBaseDamage() + GetCritBonus();
}
```

- 함수가 함수를 부르는...?<br>
- 쪼개놓고 어차피 다시 합치는 거 아닌가?<br>
- 이거 거의 실시간으로 돌리는 함수잖아!<br>

#### ✅ After

```cpp
float CalculateTotalDamage()
{
    return baseDamage + (baseDamage * critChance * 2.0f);
}
```

- 중간 단계가 그렇게 중요한가?<br>
- 괜한 함수 만들지 말고 하나로 합쳐서 관리하자<br>

## **결정 기준**

| 세분화가 적합한 상황 | 통합이 적합한 상황 |
| --- | --- |
| 책임이 명확히 분리될 때 | 강한 응집력이 필요할 때 |
| 다른 용도로 재사용 가능할 때 | 관련 데이터와 동작이 함께 있는 것이 자연스러울 때 |
| 독립적으로 테스트하고 싶을 때 | 공유 상태에 대한 관리가 필요할 때 |
| 복잡한 단계를 나누어 명확히 하고 싶을 때 | 여러 작은 함수들이 항상 함께 사용될 때 |

#### 세분화 고려

- 하나의 클래스가 너무 많은 일을 하지 않는가?<br>
- 다른 클래스도 이 기능이 필요해 보이는데?<br>
  (ex : 전투, 대미지 계산, 시간 관리 등등)<br>
  (협업하는데 각각 따로 만들면...)<br>

- 독립적인 기능 테스트가 필요한 경우<br>
  (이건 게임에서 돌려봐야 알 것 같아!)<br>

- 복잡한 단계가 많음<br>
  (로딩, 파싱 등)<br>

#### 통합 고려

- 단계가 너무 명확해<br>
  (중간에 나눌 필요 x)<br>

- 관련 데이터들끼리 '따로' 나뉘어져 있는 경우<br>

- 데이터와 동작이 '같이' 있는것이 당연할 때<br>
  (아이템 + 동작이 필요하다면 인벤토리 클래스 제작 고려)<br>

- 여러 작은 함수들이 사용된다면<br>
  그리고 각각 한군데만 사용된다면<br>
  일단 뭉치고 생각해보자<br>

결국 중요한 건 나중에 알아볼 수 있는 가독성!<br>

# 3. 간접 접근 vs. 직접 접근 🤤

캡슐화?<br>
불필요 중간 다리 제거?<br>

## 간접 접근

### 💡 변수 캡슐화하기 (Encapsulate Variable)

#### 🧨 Before

```cpp
player->health = 100;
```

- 이 중요한걸 외부 클래스가 왜 만져...?<br>

#### ✅ After

```cpp
player->SetHealth(100);
```

```cpp
void Player::SetHealth(int NewHealth)
{
    health = FMath::Clamp(NewHealth, 0, maxHealth);
}
```

- 마음대로 바뀌면 매우 곤란한 데이터는<br>
  반드시 간접 접근!<br>

- Setter 로 유용한 범위를 검사하고 보장해주자<br>
  (디버깅 가능성을 만들어두기)<br>
  (수정 부분만 디버깅 포인트 잡아주면 됨)<br>

### 💡 레코드 캡슐화하기 (Encapsulate Record)

#### 🧨 Before

```cpp
struct PlayerData
{
    FString Name;
    int Level;
};

PlayerData player;
player.Level = 5; // 아무나 막 건드림
```

- 중요한 데이터를 구조체로 사용하면<br>
  좀...<br>

#### ✅ After

```cpp
class Player
{
public:
    int GetLevel() const { return Level; }
    void SetLevel(int NewLevel) { Level = FMath::Max(1, NewLevel); }

private:
    int Level;
};
```

- 중요한 데이터라면 그냥 클래스에 넣어두어<br>
  캡슐화의 대상으로 만들기<br>

### 💡 컬렉션 캡슐화하기 (Encapsulate Collection)

#### 🧨 Before

```cpp
TArray<Item*> Inventory;

Inventory.Add(Sword);
Inventory.Add(Potion);
```

- 이거 null도 들어가지 않나?<br>

#### ✅ After

```cpp
class InventorySystem
{
public:
    void AddItem(Item* NewItem)
    {
        if (!NewItem || IsFull()) return;
        Items.Add(NewItem);
    }

    const TArray<Item*>& GetItems() const { return Items; }

private:
    TArray<Item*> Items;

    bool IsFull() const { return Items.Num() >= MaxSlots; }
};
```

- 버그가 많이 나올수록 단단하고 까다로운 접근 방식을 세워야 안전함<br>
  (게임의 핵심 상태 관리이므로)<br>

## 직접 접근

### 💡 중개자 제거하기 (Remove Middle Man)

#### 🧨 Before

```cpp
FString name = gameManager->GetPlayerManager()->GetMainPlayer()->GetName();
```

- 호출 체이닝...<br>

- nullcheck를 하더라도 길어지면 호출 체이닝은 맞음<br>

#### ✅ After

```cpp
FString name = gameManager->GetMainPlayerName();
```

```cpp
FString GameManager::GetMainPlayerName() const
{
    return PlayerManager->GetMainPlayer()->GetName();
}
```

- 필요에 따라 Getter를 통해 바로 반환하는 내용이 필요<br>

### 💡 위임 숨기기 (Hide Delegate)

#### 🧨 Before

```cpp
FString zip = player.ContactInfo.Address.ZipCode;
```

- 내부 구조가 너무 노출되는 상황<br>
- 만약 내부 구조가 바뀌면 이 코드는 바로 수정되어야 함<br>

#### ✅ After

```cpp
FString zip = player.GetZipCode();
```

```cpp
FString Player::GetZipCode() const
{
    return ContactInfo.Address.ZipCode;
}
```

- 그냥 Player 쪽에서 반환받는 값을 사용하는 것이<br>
  유지보수적으로 좋음<br>

## **결정 기준**

| 간접 접근이 적합한 상황 | 직접 접근이 적합한 상황 |
| --- | --- |
| 데이터 검증이나 부가 처리가 필요할 때 | 과도한 래퍼가 복잡성만 증가시킬 때 |
| 변경 추적이 필요할 때 | 성능이 중요한 핫스팟일 때 |
| 향후 구현 변경 가능성이 있을 때 | 단순한 데이터 구조에서 |
| 중복된 접근 로직이 여러 곳에 있을 때 | 위임 체인이 너무 길어질 때 |

### 간접 접근

- 중요한 데이터는 기본적으로 간접 접근 추천<br>
  (Gold, HP 등)<br>

- 바뀔 가능성이 많은 것들<br>

- 호출 체이닝이 많다면<br>
  함수를 만드는 것을 고려<br>

### 직접 접근

- 과잉 설계<br>
  (래퍼가 지나치게 많은 경우)<br>

- 성능이 정말 중요한 경우<br>

- 단순 데이터 구조, 위임 접근이 너무 길어진다는 등에 고려<br>

# 4. 조건문 vs. 다형성

프로그램 흐름 분기의 다양한 방식<br>

- 어느 쪽이 무조건 좋지는 않음<br>

## 조건문

- 조금 더 예쁜 조건문을 써보자<br>
  (내부 조건이 복잡하면 대충 읽고 이해하지 않고 넘어가는 편...)<br>

### 💡 조건문 분해하기 (Decompose Conditional)

#### 🧨 Before

```cpp
if (player.HasKey() && player.IsAlive() && !player.IsStunned())
{
    OpenDoor();
}
```

- 나쁘지는 않지만 조금 더 좋은 가독성을 챙길순 없나?<br>

#### ✅ After

```cpp
bool Player::CanOpenDoor() const
{
    return HasKey() && IsAlive() && !IsStunned();
}

if (player.CanOpenDoor())
{
    OpenDoor();
}
```

- 위의 조건들은 전부 '문'을 열 수 있는지를 확인하는 것이므로<br>
  명확한 함수명을 준다<br>

- 요점은 '문'을 열 수 있는가에 가독성 확보<br>
  (의도 + 재사용성 가능성)<br>

### 💡 조건식 통합하기 (Consolidate Conditional Expression)

#### 🧨 Before

```cpp
if (player.IsDead())
{
    return false;
}

if (player.IsDisconnected())
{
    return false;
}

return true;
```

- 저 위에 2개 어차피 return false인데<br>
  굳이 떨어트려야 하나?<br>

- 검사도 Player에서만 하네?<br>

#### ✅ After

```cpp
bool Player::CanParticipate() const
{
    return !(IsDead() || IsDisconnected());
}
```

- 그냥 플레이어에서 묶어서 처리하자<br>

## 다형성

- 난 조건문 싫어...<br>
- if문이 10줄 넘어가면 개편해야 해...!<br>

### 💡 조건부 로직을 다형성으로 바꾸기 (Replace Conditional with Polymorphism)

#### 🧨 Before

```cpp
float Enemy::GetDamage()
{
    if (Type == "Orc") return Strength * 1.2f;
    if (Type == "Goblin") return Strength * 0.8f;
    return Strength;
}
```

- 적 타입에 따라 대미지 적용이 다르게 됨<br>
- 이건 잠재적으로 Switch 구문 처럼 된다!<br>

#### ✅ After

```cpp
class Enemy { virtual float GetDamage() const = 0; };

class Orc : public Enemy
{
    float GetDamage() const override { return Strength * 1.2f; }
};

class Goblin : public Enemy
{
    float GetDamage() const override { return Strength * 0.8f; }
};
```

```cpp
// 호출부
Enemy* enemy = GetRandomEnemy();
float damage = enemy->GetDamage();
```

- 각각의 적들이 별도의 타입을 가지게 하는것이 더 OOP적이야!<br>
  (타입을 굳이 몰라도 됨)<br>

- 다른 Enemy 클래스 추가되더라도<br>
  이 코드는 수정될 필요 없으므로<br>

- 다만 적 타입이 300개가 넘어가게 되면<br>
  클래스의 남용...임<br>
  (전략 패턴 or 데이터 기반으로 문제를 해결해야 함)<br>

### 💡 특이 케이스 추가하기 (Introduce Special Case)

#### 🧨 Before

```cpp
if (player)
{
    ShowPlayerName(player->Name);
}
else
{
    ShowPlayerName("Guest");
}
```

- nullCheck에 따른 변화법<br>

- 이게 코드 곳곳에 뿌려져 있다면<br>
  생각보다 머리 아픔<br>
  (기획자 : Guest 말고 다른 이름으로 바꾸죠)<br>

#### ✅ After

```cpp
class NullPlayer : public Player
{
public:
    FString GetName() const override { return "Guest"; }
};
```

```cpp
Player* GetPlayerOrNull(int id)
{
    Player* player = FindPlayer(id);
    if (!player)
    {
        static NullPlayer nullPlayer;
        return &nullPlayer;
    }
    return player;
}

// 이제 어디서든 null 걱정 없이 사용 가능
Player* player = GetPlayerOrNull(playerId);
ShowPlayerName(player->GetName());  // 항상 안전!
```

- 지역 static을 통해 nullPlayer를 그냥 가볍게 만들어서<br>
  계속 반환<br>
  (객체 하나 비용)<br>

- GetName을 그냥 바로 반환하면 됨<br>

- 지저분한 if문보단 null용 개체를 만드는 것이 좋을 수는 있음<br>
  (물론 케바케)<br>

## **결정 기준**

| 조건문이 적합한 상황 | 다형성이 적합한 상황 |
| --- | --- |
| 단순한 분기 로직일 때 | 타입별 동작 차이가 뚜렷할 때 |
| 일회성이거나 지역적인 결정일 때 | 타입 추가가 자주 발생할 때 |
| 성능이 매우 중요한 곳일 때 | 타입별 코드가 반복적으로 나타날 때 |
| 타입 배열이 고정적일 때 | 동작이 확장될 가능성이 높을 때 |

### 조건문 고려

- 분기가 간단하다면 굳이 다형성을?<br>
  -> 오버 엔지니어링<br>

- 성능이 중요할 때, 일회성일 때 등<br>

- 난이도 쉬움,중간,어려움<br>
  (새로 추가 안될 것 같음)<br>

### 다형성 고려

- 타입별 나눠야 하는 동작이 '뚜렷'한 경우<br>
  (오크는 물리 공격, 드래곤은 브레스~)<br>

- 신규 타입 추가 가능성<br>

- 비슷한 타입에 대한 처리가 '반복적'으로 호출되는 경우<br>
  (유지보수가 걱정되면 다형성이 좋다)<br>

- 동작의 확장처리 가능성 존재<br>

# 5. 상속 vs. 위임 😱

- 상속<br>
  : 쉬운 확장 + 책임을 같이짐<br>
  (부모 바뀌면 반드시 자식이 바뀜)<br>
  - 물려 받는것은 편함<br>
  - 책임을 모두 지기에 문제 터지면...<br>

- 위임<br>
  : 깔끔한 구조, 책임을 따로<br>
  (계약 관계)<br>
  - 유연하며 느슨한 관계<br>

함께한다면 상속!<br>
기능만 빌려 쓴다면 위임!<br>

## 상속

### 💡 메서드 올리기 (Pull Up Method)

#### 🧨 Before

```cpp
class Orc : public Enemy
{
    void Die() { /* 공통 죽음 처리 */ }
};

class Goblin : public Enemy
{
    void Die() { /* 공통 죽음 처리 */ }
};
```

- 코드 중복...!<br>

#### ✅ After

```cpp
class Enemy
{
public:
    virtual void Die() { /* 공통 죽음 처리 */ }
};
```

- 부모 클래스에서 처리하여 코드 중복 제거<br>

다만, 이것도 예외가 존재함<br>

- 유령 같은 불사 몬스터?<br>

### 💡 필드 올리기 (Pull Up Field)

#### 🧨 Before

```cpp
class Orc : public Enemy
{
protected:
    int Health;
};

class Goblin : public Enemy
{
protected:
    int Health;
};
```

- 변수의 비슷한 상황<br>

#### ✅ After

```cpp
class Enemy
{
protected:
    int Health;
};
```

- 역시 올리기!<br>
- 공통 인터페이스를 올려놓으면 '전투 로직'등의 통합에 편리<br>

### 💡 슈퍼클래스 추출하기 (Extract Superclass)

#### 🧨 Before

```cpp
class Player
{
    FString Name;
    FVector Position;
};

class NPC
{
    FString Name;
    FVector Position;
};
```

- 다른 클래스이지만 둘다 공통된 특성은 존재함<br>
- 그런데 둘다 진짜 다른 경우???<br>

#### ✅ After

```cpp
class ActorBase
{
    FString Name;
    FVector Position;
};

class Player : public ActorBase {};
class NPC : public ActorBase {};
```

- 공통된 부분을 개념으로 빼내<br>
  하나의 클래스로 만들어주기<br>

- 그렇다고 Name 같은 걸 죄다 뭉치는 불필요한 클래스를 만들필욘 없다...<br>

## 위임

- 클래스가 다른 클래스 객체를 멤버로 갖고<br>
  특정 작업을 위임하는 것<br>

- 책임을 전가<br>

### 💡 서브클래스를 위임으로 바꾸기 (Replace Subclass with Delegate)

#### 🧨 Before

```cpp
class Enemy
{
    virtual FString GetSound() const = 0;
};

class Orc : public Enemy
{
    FString GetSound() const override { return "Roar"; }
};
```

- 런타임의 행동 변경이 까다로움<br>

- 언리얼에서 상속 구조에서 컴포넌트 구조로 가려는 이유<br>
  (상속은 지나치게 굳어있는 구조)<br>

#### ✅ After

```cpp
class SoundBehavior
{
public:
    virtual FString GetSound() const = 0;
};

class Enemy
{
    TUniquePtr<SoundBehavior> Sound;
    FString MakeSound() const { return Sound->GetSound(); }
};
```

- Sound를 갈아끼는 방식으로 구현 가능<br>
- 또한 '기능'이 별도의 클래스이기에<br>
  다른 Enemy 하위 클래스도 해당 Sound 클래스를 사용 가능<br>

- 전략 패턴에 가까움<br>
    - 필요한 요소를 갈아끼워<br>
      행동이 다른 객체를 만들어냄<br>
    - 한번만 쓰는 고정 로직이라면 굳이 필요없긴 하지만...<br>
      (오버 엔지니어링)<br>

### 💡 슈퍼클래스를 위임으로 바꾸기 (Replace Superclass with Delegate)

#### 🧨 Before

```cpp
class UIWidget
{
public:
    void Render();
};

class ScoreWidget : public UIWidget
{
    void Render() override;
};
```

- 처음에는 죄다 Render하기에 만들었지만<br>
  점점 별도의 기능을 Render에 넣다가<br>
  '다중 상속'의 길로 가게 됨<br>
  (UIWidget, UIAnimation, 등)<br>

#### ✅ After

```cpp
class WidgetRenderer
{
public:
    void Render();
};

class ScoreWidget
{
private:
    WidgetRenderer Renderer;
public:
    void Show() { Renderer.Render(); }
};
```

- 계보에서 독립<br>
- Render에게 자신의 일을 위임<br>

- IsA -> HasA로 변경<br>

## **결정 기준**

| 상속이 적합한 상황 | 위임이 적합한 상황 |
| --- | --- |
| 명확한 "is-a" 관계가 있을 때 | "has-a" 관계나 행동 공유일 때 |
| 공통 기능이 많고 타입 계층이 필요할 때 | 런타임에 동작을 변경해야 할 때 |
| 다형성을 활용한 확장이 자연스러울 때 | 다중 상속과 같은 효과가 필요할 때 |
| 코드 재사용이 수직적일 때 | 기존 클래스 변경 없이 기능 확장이 필요할 때 |

### 상속 고려

- 명확한 Is-A 관계<br>
  정말 '하위' 관계가 확실하고 변하지 않음<br>

- 공통 기능이 많고 '명확한 계층'이 필요<br>
  (진짜 공통 기능이 많음)<br>

- 다형성을 통해 해결하고 싶은 경우<br>
  (인터페이스 등도 포함)<br>
  
- 강화 등 코드 재사용이 고려 가능<br>

### 위임 고려

- Has-A 관계<br>
  '무기'를 '가지는 것'은 Has-A임<br>
  (플레이어가 무기를 소유하는 것)<br>

- 런타임 중 행동 변경<br>
  (플레이어의 무기 변경)<br>
  (그에 따른 공격 방식도 변경해야 함)<br>

- 다중 상속과 비슷한 효과를 고려해야 함<br>

- 기존 클래스의 변경 x<br>

#### ex

```
ABaseCharacter :: Attack
|- AEnemy :: Attack
|- APlayer :: Attack
   -> AWeapon :: Attack
```

플레이어와 적 모두 Attack을 가짐 - 상속<br>
다만 플레이어는 무기에 따른 공격을 변경하고 싶다면<br>
Weapon을 만든 후, Attack과 연결해줌 - 위임<br>
