---
title: "김하연 튜터님 강의 - 'GoF 디자인 패턴 - 생성 패턴'"
date : "2025-10-20 14:00:00 +0900"
last_modified_at: "2025-10-20T14:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 디자인 패턴
  - 생성 패턴
---

# 게임 개발에서 자주 쓰이는 디자인 패턴에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. 디자인 패턴, 이쯤에서 솔직히 말해봅시다. 🧠

## 주니어 개발자들의 패턴 학습 단계

### **1단계: 무지의 단계**

```cpp
// 모든 것을 직접 구현
class Game {
    void Update() {
        // 1000줄의 스파게티 코드
        if (condition1 && condition2 && !condition3) {
            // 더 많은 if 문들...
        }
    }
};

개발자 심리: "코드가 돌아가기만 하면 돼"
```

- 유지보수 이전의 '구현' 우선<br>
- 일단 성공시켜야 함<br>

### **2단계: 패턴 만능주의**

```cpp
// 모든 곳에 패턴 적용
class GameObserverFactoryBuilderAdapter {
    // 실제로는 단순한 기능
    void DoSomething();
};

개발자 심리: "패턴을 많이 쓰면 고급 개발자인 것 같아!"
```

- 패턴... 보기 좋잖아?<br>
- 현실적으로 문제를 가장 많이 일으킬 수 있음<br>
  - 팀원 + 본인도 코드 보다 헷갈림<br>
  - 버그가 점점 늘어남...<br>

- 설계가 아니라... 설계 놀이..<br>
  (그래도 이 시점을 지나야 설계가 가능함)<br>
  (디자인 패턴 아예 관심 없는것 보다는 좋을수 있다)<br>

### **3단계: 패턴 회의론**

```cpp
// 패턴을 전혀 안 씀
class Game {
    void Update() {
        // 여전히 스파게티지만 "순수"함
    }
};

개발자 심리: "패턴은 오버엔지니어링이야"
```

- 2단계에서 많이 데인 후,<br>
  회의론자가 됨...<br>

- '패턴'? 괜히 성능만 먹고<br>
  대부분 간단하게 풀 수 있어<br>
  ~~(애증의 단계)~~

### **4단계: 균형점**

```cpp
// 필요한 곳에만 적절히 사용
class Game {
    IGameState* currentState; // State 패턴 (상태가 복잡함)

    void Update() {
        currentState->Update(); // 깔끔!

        // 단순한 부분은 직접 처리
        renderer.DrawFrame();
    }
};

개발자 심리: "패턴은 도구일 뿐이야"
```

- 패턴도 나름 잘 쓰면 좋구나<br>
  (깨달음)<br>

- 패턴을 '안쓰는' 이유를 알아야 함<br>

## 회사에서 한번쯤은 마주치는 **패턴 전도사들 특징**

### **1단계: 열정적 도입**

```
전도사: "Observer 패턴을 도입해봅시다!"
팀원: "좋네요! 해봅시다!"
결과: 모든 곳에 Observer 패턴 적용
```

- 대부분의 구현에 디자인 패턴 사용<br>

### **2단계: 현실 직면**

```
팀원: "버그가 생겼는데 어디서 발생했는지 모르겠어요"
전도사: "이벤트 체인을 따라가보면..."
팀원: "체인이 20단계예요..."
전도사: "음..."
```

- 디자인 패턴을 공부하며 <br>
  - '쓰기 전'이 더 좋았던것 같은데...?<br>

### **3단계: 균형점 찾기**

```
전도사: "정말 필요한 곳에만 패턴을 쓰자"
팀원: "그럼 어디가 정말 필요한 곳인가요?"
전도사: "...경험으로 알게 될 거야"
```

- 정말 필요한 곳에만 쓰도록 노력하게 됨<br>

- 일반 구현 -> 디자인 패턴으로 묶는 것을 경험으로..<br>

### **4단계: 현실적 적용**

```
팀 규칙:
- Singleton은 진짜 하나만 있어야 하는 것에만
- Factory는 생성 로직이 복잡할 때만
- Observer는 3개 이상 반응할 때만
- 다른 사람이 이해할 수 있게 만들기
```

- 귀납적인 결과물일 수 있으나 프로젝트에 최적<br>

## 🎭 패턴 vs 성능의 영원한 딜레마

### **성능 우선 코드**

```cpp
void UpdateGame() {
    // 직접 호출 (빠름)
    player.Update();
    if (player.health <= 0) {
        ui.ShowGameOver();
        audio.PlayDeathSound();
        // ... 100줄의 직접 처리
    }
}
// 장점: 빠름
// 단점: 수정 지옥
```

- 유지보수성이 '대가'로<br>
  trade-off<br>

### **패턴 적용 코드**

```cpp
void UpdateGame() {
    player.Update();
    // Observer 패턴 (느림)
    eventManager.Dispatch(PlayerUpdated);
}
// 장점: 깔끔함, 확장성
// 단점: 성능 오버헤드
```

- 불필요한 오버헤드가 될 수 있음<br>
- 과할 경우는 장점이 퇴색됨<br>

### **현실적 해결책**

```cpp
void UpdateGame() {
    player.Update();

    // 중요한 이벤트만 패턴 사용
    if (player.healthChanged) {
        eventManager.Dispatch(HealthChanged);
    }

    // 성능 중요한 부분은 직접 처리
    if (player.health <= 0) {
        HandleDeath(); // 직접 호출
    }
}
```

- 필요할때만 쓰자...<br>

## 🎯 패턴 선택의 현실적 기준

### **이론상 선택 기준**

1. 요구사항 분석<br>
2. 확장성 고려<br>
3. 유지보수성 검토<br>
4. 팀 역량 평가<br>
5. 성능 영향 분석<br>

- 교과서적인 내용<br>

### **실제 선택 기준**

1. "예전에 써봤는데 괜찮았어"<br>
2. "시니어가 쓰라고 했어"<br>
3. "일단 돌아가면 되지 뭐"<br>
4. "데드라인이 내일이야..."<br>

- 현실(?)적인 내용<br>
- 어쨋든 선택의 큰 기준이 됨<br>
  - 그래도 왜 쓰는지 '이해'는 해야 함<br>
  - 그래야 나중에 리드 프로그래머도 해봄<br>

## 🤖 **개발자 vs ChatGPT**

```
개발자: "Observer 패턴 코드 짜줘"
ChatGPT: "완벽한 Observer 패턴 코드입니다!"
개발자: "오... 깔끔하네"

실제 사용 시
- 컴파일 에러 3개
- 메모리 누수 1개
- 스레드 안전성 문제 1개
```

- 문법적으로만 완벽인 경우가 대부분...<br>
- 좀 과하게 디자인 패턴을 적용하려 함<br>

### **미래의 패턴**

```
**예상: AI가 최적의 패턴을 자동으로 선택해줄 것
현실: AI도 Factory Factory Factory를 만들어버림**

개발자: "AI야, 이거 너무 복잡해"
AI: "패턴의 순수한 구현입니다"
개발자: ㅋ
```

- 결국 AI를 사용은 해야 함<br>
- 그래도 내가 컨트롤을 할 줄 알아야 함<br>

## 🎯 패턴 마스터로 가는 길

## **짭 패턴 마스터의 특징**

1. **패턴 이름만 안다**
    - "이거 Factory 패턴이야!" (실제로는 단순한 함수)<br>
2. **무조건 패턴을 쓴다**
    - "Singleton 안 쓰면 불안해"<br>
3. **복잡할수록 좋다고 생각한다**
    - "AbstractFactoryBuilderProxy... 완벽해!"<br>
4. **성능은 무시한다**
    - "성능보다 구조가 중요해"<br>

- 코드에 정답은 없을 수도 있음<br>

## **찐 패턴 마스터의 특징**

1. **패턴을 안 쓸 줄 안다**
    - "이 상황엔 패턴보다 단순한 해결책이 나아"<br>
2. **패턴을 변형할 줄 안다**
    - "Observer에 우선순위를 추가해보자"<br>
3. **패턴의 한계를 안다**
    - "Singleton은 테스트하기 어려워"<br>
4. **팀을 고려한다**
    - "신입이 이해할 수 있는 수준으로 쓰자"<br>
5. **성능을 고려한다**
    - "이 부분은 핫패스니까 직접 호출하자"<br>

- 패턴은 `도구`!<br>
- 자료구조, 알고리즘처럼 '장점'과 '단점'을 정확하게 파악해야 함<br>

## 🎪 마무리: 패턴과 함께하는 개발 인생

### **패턴 학습의 진실**

- 패턴을 배우는 이유
    1. 좋은 코드를 쓰기 위해서 (10%)<br>
    2. 면접에서 써먹으려고 (30%)<br>
    3. 다른 개발자와 소통하기 위해 (40%)<br>
    4. 뭔가 있어 보이려고 (20%)<br>

### **패턴 사용의 현실**

- **실제 개발에서**
    - 90%: 간단한 해결책이 최선<br>
    - 9%: 패턴이 정말 도움됨<br>
    - 1%: 패턴 때문에 더 복잡해짐...<br>
- **그래도 패턴을 배워야 하는 이유**
    - 그 9%가 정말 중요하기 때문(91점이 상한이 되버리기 싫음!)<br>
    - 다른 사람 코드를 읽을 수 있기 때문<br>
    - 면접관이 물어보기 때문 (...)<br>

### **최종 조언**

- 패턴은 도구다. 목적이 아니다.<br>
- 단순함이 복잡함보다 낫다.<br>
- 동료가 이해할 수 있어야 한다.<br>
- 성능도 중요하다.<br>
- 완벽한 코드는 없다.<br>

<aside>
💡

**"완벽한 패턴은 없다. 완벽한 타이밍만 있을 뿐이다." - 김하연 튜터** *뇌피셜 명언 (?)*

</aside>

# 2. GoF와 23개 패턴 👽

- 클린 코드 : 건물을 깔끔하게 지음<br>
- 리팩토링 : 건물을 깔끔하게 '다시' 지음<br>
- 디자인 패턴 : 건물 이미 '지은' 사람이 남긴 조언<br>
  (다 비슷한 문제를 겪는데 이거 '패턴'으로 만들 수 있지 않을까?)<br>

## Gang of Four (1994)

- Erich Gamma
- Richard Helm
- Ralph Johnson
- John Vlissides

## 🏗️ 생성 패턴 (5개)

- 객체 생성 메커니즘을 다루는 패턴 (HOW to create)<br>
  - GameCharacter를 누가, 언제, 어떻게 만들지?<br>
    : 이걸 누가 전문적으로 다루면 편할 것 같은데?<br>

- **Singleton, Factory Method, Abstract Factory, Builder, Prototype**<br>

### 🔗 구조 패턴 (7개)

- 클래스나 객체를 조합하는 패턴 (HOW to compose)<br>
  - 각 객체들을 어떻게 '연결'할 지<br>
  - 기능이 많아질수록 복잡해지며,<br>
    그것을 어떻게 깔끔하게 만들지를 고민<br>

- **Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy**<br>

### 🎯 행동 패턴 (11개)

- 객체 간 상호작용과 책임 분배 패턴 (HOW to interact) ← 다음 강의<br>
  - 클래스들끼리 어떻게 '일'하게 할까<br>

- **Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor**<br>

# **3. 실무 핵심 10개 패턴 📌**

### 사용 빈도 분석

- **매우 자주 사용**: Singleton, Factory, Observer
- **자주 사용**: Builder, Decorator, Command, State
- **가끔 사용**: Adapter, Facade, Strategy
- **거의 안씀**: 나머지 13개 (게임 쪽에서 잘 안쓰임 + 공간 복잡도를 너무 고려한 패턴)<br>

### 10개 패턴만 제대로 알면

- 실무의 90% 이상 커버<br>
- 코드 품질 향상<br>
- 팀 커뮤니케이션 개선<br>
- 면접 대비 완료<br>

# 4. 생성 패턴 (Creational Patterns) 🪐

## 싱글톤 패턴 (Singleton Pattern) - 게임 매니저의 필수품

## 📌 개념

클래스의 인스턴스가 단 하나만 존재하도록 보장하고, 어디서든 접근할 수 있게 하는 패턴

### 언제 필요한가?

- GameManager처럼 게임 전체를 관리하는 객체<br>
- 여러 개 있으면 혼란을 일으키는 시스템<br>
- 모든 곳에서 접근해야 하는 중앙 관리자<br>

## 🧨 문제 상황

```cpp
// 문제: GameManager가 여러 개 생성될 수 있음
GameManager* manager1 = new GameManager();
manager1->SetScore(100);

// 다른 곳에서...
GameManager* manager2 = new GameManager();  // 또 다른 GameManager!
int score = manager2->GetScore();  // 0점 (엥?)
```

- 왜 여러개 만들어서 쓰지...?<br>
- 서로 다른 데이터를 가지고, 통일성이 없어지는데?<br>

## ✅ Singleton 구조

```cpp
class GameManager {
private:
    // 1. static 인스턴스
    static GameManager* instance;
    
    // 2. private 생성자 (외부에서 new 금지)
    GameManager() {
        score = 0;
        gameTime = 0.0f;
    }
    
public:
    // 3. 접근 메서드
    static GameManager* GetInstance() {
        if (instance == nullptr) {
            instance = new GameManager();
        }
        return instance;
    }
};

// static 멤버 초기화
GameManager* GameManager::instance = nullptr;
```

- 중요한 로직/데이터는 한 녀석만 진행하도록!<br>

## 😌 실제 사용 예제는?

```cpp
class GameManager {
private:
    static GameManager* instance;
    
    // 게임 데이터
    int score;
    float gameTime;
    bool isPaused;
    int currentLevel;
    
    // private 생성자
    GameManager() {
        score = 0;
        gameTime = 0.0f;
        isPaused = false;
        currentLevel = 1;
    }
    
public:
    static GameManager* GetInstance() {
        if (instance == nullptr) {
            instance = new GameManager();
        }
        return instance;
    }
    
    // 게임 관리 메서드들
    void AddScore(int points) { score += points; }
    int GetScore() const { return score; }
    void PauseGame() { isPaused = true; }
    void ResumeGame() { isPaused = false; }
    void NextLevel() { currentLevel++; }
};

// GameManager.cpp
GameManager* GameManager::instance = nullptr;  // 반드시 cpp 파일에!
```

```cpp
// 어디서든 동일한 인스턴스에 접근
void OnCoinCollected() {
    GameManager::GetInstance()->AddScore(10);
}

void OnEnemyKilled() {
    GameManager::GetInstance()->AddScore(50);
}

void UpdateUI() {
    int score = GameManager::GetInstance()->GetScore();
    // UI 업데이트
}
```

## 🧨 보너스: 언리얼 엔진 스타일

```cpp
UCLASS()
class MYGAME_API UMyGameInstance : public UGameInstance {
    GENERATED_BODY()
    
private:
    UPROPERTY()
    int32 TotalScore;
    
public:
    // 편의 메서드
    static UMyGameInstance* Get() {
        return Cast<UMyGameInstance>(
            UGameplayStatics::GetGameInstance(GetWorld())
        );
    }
    
    void AddScore(int32 Points) { TotalScore += Points; }
};

// 사용
UMyGameInstance::Get()->AddScore(100);
```

- GameInstanceSubsystem 방식 고려 가능<br>
- 그 외에도 WorldSubsystem 같은<br>
  다양한 Subsystem (생존 기간이 다름)을<br>
  통해 다양한 매니저를 구현 가능함<br>

## 📌 사용 가이드라인

전역적이고 제한적인 접근, 생성 통제<br>
다양한 설정 및 로직 등에 사용 가능<br>

다만,<br>

싱글톤은 전역적인 접근이기에<br>
상태 변화에 취약 -> 테스트가 어려움<br>

또한, 전역 접근의 큰 특징으로<br>
대부분의 클래스가 싱글톤을 '의존'하기에<br>
결합성이 커짐<br>

멀티 스레드 환경에서 문제가 발생할 수 있음<br>
- GetInstance()가 동시에 호출되어<br>
  2개가 만들어질 가능성이 존재하며<br>
  Lock을 걸어주어야 안전<br>

싱글톤끼리 호출하는 것은 피할 것!<br>
순서 꼬이면 Crash가 발생하기 딱 좋음<br>

- 전역 변수/함수 보단 좋지만<br>
  자주 보이면 별다를 것은 없어진다<br>

### ✅ 적합한 경우

- GameManager (게임 상태 관리)
- AudioManager (사운드 시스템)
- SaveManager (저장 시스템)
- NetworkManager (네트워크 연결)
- ResourcePool (리소스 캐시)

공용으로 읽고 씀<br>
'중앙 관리자'의 역할<br>

### ❌ 피해야 할 경우

- Player, Enemy (게임 오브젝트)
- Weapon, Item (여러 개 존재)
- UI 요소들
- 임시 오브젝트

아무데서나 GetInstance를 사용하기 시작한다면<br>
구조가 망가질 수 있음<br>

테스팅이 필요한 로직이라면 싱글톤은 자제할 것<br>

## 팩토리 패턴 (Factory Method Pattern) - 몬스터 생성의 정석

## 📌 개념

객체 생성을 별도의 메서드로 캡슐화하여, 생성 로직을 한 곳에서 관리하는 패턴

### 언제 필요한가?

- 다양한 종류의 몬스터/아이템을 생성할 때<br>
- 조건에 따라 다른 객체를 생성해야 할 때<br>
- 생성 로직이 복잡하고 자주 변경될 때<br>

## 🧨 문제 상황

```cpp
// 문제: 직접 생성하면 구체적인 클래스에 의존
class Game {
    void SpawnEnemy() {
        // 구체적인 클래스를 직접 알아야 함
        Enemy* enemy = new Zombie();  // Zombie 클래스에 의존
        enemy->Spawn();
    }
};
```

- Zombie 클래스에 의존<br>

- Game 클래스가 매번 Enemy를 생성해줌<br>
  (책임적 문제)<br>

- 확장성 문제<br>

## ✅ Factory 구조

GameManager는 추상적인 Create,Spawn Enemy 같은 함수만 호출<br>
상세적인 일은 하위 클래스에서 작업하면 됨<br>

### 1. Product - 만들어질 몬스터들

```cpp
// 모든 적의 기본 인터페이스
class Enemy {
public:
    // 모든 적이 할 수 있는 행동들
    virtual void Attack() = 0;      // 공격하기
    virtual void Move() = 0;        // 이동하기
    virtual void TakeDamage(float damage) = 0;  // 피해 입기
    virtual ~Enemy() = default;     // 가상 소멸자 (중요!)
    
    // 모든 적이 가지는 공통 속성
    float Health = 100.0f;
    float Speed = 100.0f;
    int ExpReward = 10;
};
```

- 기본 제품<br>

```cpp
// 진짜로 게임에 등장할 몬스터들
class Zombie : public Enemy {
public:
    Zombie() {
        Health = 50.0f;
        Speed = 150.0f;
        ExpReward = 10;
    }
    
    void Attack() override {
        // 좀비 공격: 물어뜯기
    }
    
    void Move() override {
        // 느리지만 꾸준한 이동
    }
    
    void TakeDamage(float damage) override {
        Health -= damage;
        // 좀비 특유의 피격 효과
    }
};

class Skeleton : public Enemy {
public:
    Skeleton() {
        Health = 30.0f;
        Speed = 200.0f;
        ExpReward = 15;
    }
    
    void Attack() override {
        // 스켈레톤 공격: 화살 쏘기
    }
    
    void Move() override {
        // 빠른 이동
    }
    
    void TakeDamage(float damage) override {
        Health -= damage * 1.5f;  // 뼈라서 물리 공격에 약함
    }
};
```

- 구체적 제품<br>

### 2. Factory (공장 클래스) - 몬스터를 만드는 공장

```cpp
class EnemySpawner {
public:
    virtual Enemy* CreateEnemy() = 0;

    void SpawnEnemy(FVector location) {
        Enemy* enemy = CreateEnemy();   // 실제 몬스터 종류는 모름
        enemy->Move();
        RegisterToGameWorld(enemy);
    }

protected:
    void RegisterToGameWorld(Enemy* enemy) {
        // 월드에 등록하는 코드
    }
};
```

- 기본 제품 공장<br>

```cpp
// 좀비 공장
class ZombieSpawner : public EnemySpawner {
public:
    Enemy* CreateEnemy() override {
        return new Zombie();  // 좀비를 만들어요!
    }
};

// 스켈레톤 공장
class SkeletonSpawner : public EnemySpawner {
public:
    Enemy* CreateEnemy() override {
        return new Skeleton();  // 스켈레톤을 만들어요!
    }
}
```

- 세부 제품 공장<br>

### 3. 실제 사용

```cpp
class GameMode {
    std::unique_ptr<EnemySpawner> currentSpawner;
    
    // 스테이지에 따라 다른 스포너 사용
    void SetupStage(int stageNumber) {
        switch (stageNumber) {
            case 1:
                // 1스테이지는 좀비만!
                currentSpawner = std::make_unique<ZombieSpawner>();
                UE_LOG(LogTemp, Warning, TEXT("1스테이지: 좀비 공장 가동!"));
                break;
                
            case 2:
                // 2스테이지는 스켈레톤!
                currentSpawner = std::make_unique<SkeletonSpawner>();
                UE_LOG(LogTemp, Warning, TEXT("2스테이지: 스켈레톤 공장 가동!"));
                break;
        }
    }
    
    // 적 스폰하기
    void SpawnEnemyWave() {
        if (currentSpawner) {
            // 현재 공장에서 적 생산!
            currentSpawner->SpawnEnemy(GetRandomLocation());
            // 어떤 적이 나올지는 currentSpawner가 결정!
        }
    }
};
```

- 여기서 더 나아가면 Wave를 클래스 + 데이터 화를 통하여<br>
  GameManager가 Wave에게 적 Spawn을 일임하는 방식도 존재<br>
  (다만, 다소 과하게 확장성을 의식한 설계일수도 있음)<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- 몬스터/아이템/무기 생성 시스템<br>
- 난이도별 오브젝트 생성<br>
- UI 요소 동적 생성<br>
- 파티클 이펙트 생성<br>

생성이 '복잡'한 경우<br>
클래스 하나를 파서 위임하여 처리<br>

### ❌ 피해야 할 경우

- 단순한 객체 (한 종류만)<br>
- 생성 로직이 없는 경우<br>
- 성능이 극도로 중요한 경우<br>

단순히 New 하나로 끝나는 경우도 분명 존재함<br>
이 경우는 괜한 '복잡도'의 증가<br>
(2~3개 라면 그냥 사용하는 것도...)<br>

기본 + 세부 클래스가 늘어나기에<br>
괜한 처리가 될 수 있음<br>
(단계가 늘어나기에 디버깅 난이도 상승)<br>

## 빌더 패턴 (Builder Pattern) - 캐릭터를 조립하듯이 만들기

## 📌 개념

복잡한 객체를 단계별로 생성할 수 있게 하는 패턴. 생성 과정과 표현을 분리하여 동일한 생성 절차에서 다른 표현을 만들 수 있음

### 언제 필요한가?

- 캐릭터 커스터마이징처럼 옵션이 많을 때
- 생성자 매개변수가 너무 많을 때
- 선택적 매개변수가 많을 때
- 객체 생성 과정이 여러 단계일 때

## 🧨 문제 상황

```cpp
// 문제: 생성자가 너무 복잡함
class Character {
public:
    // 😱 매개변수가 너무 많다!
    Character(
        string name, 
        string className,
        int level,
        float health,
        float mana,
        float strength,
        float intelligence,
        float agility,
        string weapon,
        string armor,
        string accessory,
        FLinearColor skinColor,
        FLinearColor hairColor,
        int hairStyle
        // ... 더 많은 매개변수
    );
};

// 사용할 때도 혼란
Character* hero = new Character(
    "Arthas",      // 이게 뭐였지?
    "Warrior",     // 순서가...
    50,            // 레벨인가 체력인가?
    500,           // ???
    100,           // ???
    // ... 떼잉
);
```

- 어지러운 생성자 매개 변수<br>

## ✅ Builder 패턴 구조

### 1. 생성될 복잡한 객체

```cpp
class Character {
private:  // private으로 숨겨둬요!
    // 기본 정보
    FString Name;
    FString Class;
    int Level;
    
    // 외형
    FLinearColor SkinColor;
    FLinearColor HairColor;
    int HairStyle;
    float Height;
    
    // 능력치
    float Health;
    float Mana;
    float Strength;
    float Intelligence;
    float Agility;
    
    // 장비
    FString Weapon;
    FString Armor;
    
    // 스킬
    TArray<FString> Skills;
    
public:
    // Builder를 friend로 선언 - Builder만 접근 가능!
    friend class CharacterBuilder;
    
    void PrintInfo() {
        UE_LOG(LogTemp, Warning, TEXT("=== 캐릭터 정보 ==="));
        UE_LOG(LogTemp, Warning, TEXT("이름: %s"), *Name);
        UE_LOG(LogTemp, Warning, TEXT("직업: %s"), *Class);
        UE_LOG(LogTemp, Warning, TEXT("레벨: %d"), Level);
        UE_LOG(LogTemp, Warning, TEXT("체력: %.1f"), Health);
    }
};
```

- 만들 결과물이나<br>
  닫혀 있고, Builder에만 friend로 열어둠<br>

- 그리고 Builder로만 캐릭터를 세팅하게 함<br>

### 2. Builder 클래스

```cpp
class CharacterBuilder {
private:
    Character* character;  // 조립 중인 캐릭터
    
public:
    CharacterBuilder() {
        character = new Character();
        
        // 기본값 세팅 - 이러면 일부만 설정해도 OK!
        character->Level = 1;
        character->Height = 1.8f;
        character->Health = 100.0f;
        character->Mana = 100.0f;
        character->Strength = 10.0f;
        character->Intelligence = 10.0f;
        character->Agility = 10.0f;
    }
    
    // 이름 설정하고 자기 자신을 반환!
    CharacterBuilder& WithName(const FString& name) {
        character->Name = name;
        return *this;  // 이게 핵심! 체이닝을 위해!
    }
    
    // 직업 설정
    CharacterBuilder& WithClass(const FString& className) {
        character->Class = className;
        return *this;
    }
    
    // 레벨 설정
    CharacterBuilder& WithLevel(int level) {
        character->Level = level;
        return *this;
    }
    
    // 외형을 한 번에 설정
    CharacterBuilder& WithAppearance(
        FLinearColor skin, 
        FLinearColor hair, 
        int hairStyle
    ) {
        character->SkinColor = skin;
        character->HairColor = hair;
        character->HairStyle = hairStyle;
        return *this;
    }
    
    // 능력치를 한 번에 설정
    CharacterBuilder& WithStats(
        float health, 
        float mana, 
        float strength, 
        float intelligence
    ) {
        character->Health = health;
        character->Mana = mana;
        character->Strength = strength;
        character->Intelligence = intelligence;
        return *this;
    }
    
    // 무기 장착
    CharacterBuilder& WithWeapon(const FString& weapon) {
        character->Weapon = weapon;
        return *this;
    }
    
    // 스킬 추가 (여러 번 호출 가능!)
    CharacterBuilder& AddSkill(const FString& skill) {
        character->Skills.Add(skill);
        return *this;
    }
    
    // 최종 빌드! 완성된 캐릭터를 반환
    Character* Build() {
        // 마지막 검증
        if (character->Name.IsEmpty()) {
            character->Name = "Unknown Hero";  // 이름 없으면 기본값
        }
        
        // 직업별 보너스 적용
        ApplyClassDefaults();
        
        return character;  // 완성!
    }
    
private:
    void ApplyClassDefaults() {
        if (character->Class == "Warrior") {
            // 전사는 체력과 힘 보너스!
            character->Health *= 1.5f;
            character->Strength *= 1.3f;
            
            if (character->Weapon.IsEmpty()) {
                character->Weapon = "Iron Sword";  // 기본 무기
            }
        }
        else if (character->Class == "Mage") {
            // 마법사는 마나와 지능 보너스!
            character->Mana *= 2.0f;
            character->Intelligence *= 1.5f;
            
            if (character->Weapon.IsEmpty()) {
                character->Weapon = "Wooden Staff";  // 기본 무기
            }
        }
    }
};
```

- 기본 세팅만 가능<br>
- *this 반환으로 체이닝이 가능하도록 함<br>

## 😌 사용 방법은?

```cpp
// 방법 1: 풀 커스터마이징
Character* myHero = CharacterBuilder()
    .WithName("Arthas")
    .WithClass("Warrior")
    .WithLevel(50)
    .WithAppearance(
        FLinearColor(1, 0.8f, 0.6f),    // 피부색
        FLinearColor(0.2f, 0.2f, 0.2f), // 검은 머리
        3                                // 헤어스타일 3번
    )
    .WithStats(500, 100, 50, 20)        // 체력, 마나, 힘, 지능
    .WithWeapon("Frostmourne")
    .AddSkill("Death Coil")
    .AddSkill("Army of the Dead")
    .Build();

// 방법 2: 간단하게 필요한 것만!
Character* simpleHero = CharacterBuilder()
    .WithName("John")
    .WithClass("Archer")
    .Build();  // 나머지는 다 기본값!

// 방법 3: 순서 상관없이!
Character* wizard = CharacterBuilder()
    .AddSkill("Fireball")        // 스킬부터 추가해도 OK
    .WithClass("Mage")           // 순서 바꿔도 OK
    .WithName("Gandalf")         // 편한 대로!
    .AddSkill("Teleport")
    .WithLevel(99)
    .Build();
```

- 상황에 따라 커스터마이징 하는 방식<br>
  (마지막에 Build()만 호출)<br>

- 가독성 좋음<br>
  (함수명을 통한 직관성)<br>

- 유연성<br>
  (기본 세팅에 필요한 함수만 호출)<br>
  (순서 상관 x, 마지막에 Build 만)<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- 캐릭터 커스터마이징 시스템<br>
- 복잡한 아이템 제작<br>
- UI 레이아웃 구성<br>
- 게임 설정 구성<br>
- 던전/맵 생성 시스템<br>

옵션이 많은 복잡한 구조에 고려 가능<br>
(복잡한 객체에 적합한 패턴)<br>

### ❌ 피해야 할 경우

- 매개변수가 적고 단순한 객체<br>
- 모든 값이 필수인 경우<br>
- 객체 구조가 자주 변경되는 경우<br>

간단한 경우는 애초에 필요 없는 패턴<br>
파일이 길어짐<br>
(오버 엔지니어링!)<br>

런타임 상의 안전성 보장이 '낮음'<br>
그렇기에 값이 자주 수정되는 경우는<br>
매번 저걸 사용해서 다시 만들수도...<br>