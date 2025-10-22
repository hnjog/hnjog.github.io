---
title: "김하연 튜터님 강의 - 'GoF 디자인 패턴 - 구조 패턴과 행동 패턴'"
date : "2025-10-22 18:00:00 +0900"
last_modified_at: "2025-10-22T18:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 디자인 패턴
  - 구조 패턴
  - 행동 패턴
---

# 게임 개발에서 자주 쓰이는 디자인 패턴에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. 구조 패턴 (Structural Patterns) 🌍

객체들을 어떻게 연결할 것인가!<br>

## 어댑터 패턴 (Adapter Pattern) - 서로 안 맞는 것들을 연결하기

## 📌 개념

호환되지 않는 인터페이스를 가진 클래스들을<br>
함께 작동할 수 있도록 변환해주는 패턴<br>

### 언제 필요한가?

- 외부 라이브러리를 우리 코드와 통합할 때<br>
  : 내부적으로 커스텀한 구조체를 라이브러리와 연결한다던가<br>

- 플랫폼별로 다른 API를 통일된 방식으로 사용할 때<br>

- 레거시 코드를 새로운 시스템과 연동할 때<br>

- 서로 다른 엔진/SDK를 함께 사용할 때<br>

## 🧨 문제 상황

```cpp
// 언리얼 엔진의 벡터 클래스
class FVector {
public:
    float X, Y, Z;  // 변수가 public이고 대문자
    
    float Size() const {  // 크기를 구하는 함수
        return sqrt(X*X + Y*Y + Z*Z);
    }
};

// 외부 물리 엔진 (PhysX 같은)의 벡터 클래스
class PhysicsVec3 {
private:
    float coords[3];  // 배열로 저장하네?
    
public:
    float getX() const { return coords[0]; }  // getter 함수들
    float getY() const { return coords[1]; }  
    float getZ() const { return coords[2]; }
    float magnitude() const {  // Size()가 아니라 magnitude()네?
        return sqrt(coords[0]*coords[0] + coords[1]*coords[1] + coords[2]*coords[2]);
    }
};
```

```cpp
// 언리얼 벡터를 처리하는 함수
void MoveCharacter(FVector direction) {
    float speed = direction.Size();
    // 캐릭터 이동 로직...
}

// 물리 엔진 벡터도 처리하고 싶은데...
void MoveCharacterPhysics(PhysicsVec3 direction) {
    float speed = direction.magnitude();  
    // 똑같은 로직을 또 짜야 함 ㅇㅇ
}
```

- 같은 개념이지만 호환이 안되는 상황<br>

- 같은 로직을 또 짜는 것은<br>
  코드 중복!!<br>

## ✅ Adapter 패턴 구조

### 1. 통합 인터페이스 정의

```cpp
// 우리가 원하는 공통 인터페이스
class IVector3D {
public:
    virtual float GetX() const = 0;
    virtual float GetY() const = 0;
    virtual float GetZ() const = 0;
    virtual float GetMagnitude() const = 0;
    virtual ~IVector3D() = default;
};
```

### 2. Adapter 만들기 - 변환기!

```cpp
// 언리얼 벡터를 위한 어댑터
class FVectorAdapter : public IVector3D {
private:
    FVector* vector;  // 언리얼 벡터를 가리킴
    
public:
    FVectorAdapter(FVector* vec) : vector(vec) {}
    
    // IVector3D 인터페이스 구현
    float GetX() const override { 
        return vector->X;  // 언리얼은 X로 바로 접근
    }
    
    float GetY() const override { 
        return vector->Y; 
    }
    
    float GetZ() const override { 
        return vector->Z; 
    }
    
    float GetMagnitude() const override { 
        return vector->Size();  // Size()를 GetMagnitude()로 변환!
    }
};

// 물리 엔진 벡터를 위한 어댑터
class PhysicsVectorAdapter : public IVector3D {
private:
    PhysicsVec3* physicsVector;  // 물리 엔진 벡터를 가리킴
    
public:
    PhysicsVectorAdapter(PhysicsVec3* vec) : physicsVector(vec) {}
    
    float GetX() const override { 
        return physicsVector->getX();  // getX()를 GetX()로 변환!
    }
    
    float GetY() const override { 
        return physicsVector->getY(); 
    }
    
    float GetZ() const override { 
        return physicsVector->getZ(); 
    }
    
    float GetMagnitude() const override { 
        return physicsVector->magnitude();  // magnitude()를 GetMagnitude()로!
    }
};
```

- 인터페이스를 통하여 하나로 묶어준다<br>
  그리고 인터페이스를 통해 접근시키면 상관 없음<br>

### 3. 하나의 함수로 모두 처리

```cpp
// 이제 하나의 함수로 모든 벡터 처리 가능!
void MoveCharacter(IVector3D* direction) {
    float x = direction->GetX();
    float y = direction->GetY();
    float z = direction->GetZ();
    float speed = direction->GetMagnitude();
    
    UE_LOG(LogTemp, Warning, TEXT("이동: (%.2f, %.2f, %.2f), 속도: %.2f"), 
           x, y, z, speed);
    
    // 캐릭터 이동 로직... (한 번만 구현!)
}

// 사용 예시
void GamePlay() {
    // 언리얼 벡터 사용
    FVector unrealVec(10, 20, 30);
    FVectorAdapter unrealAdapter(&unrealVec);
    MoveCharacter(&unrealAdapter);  // 잘 동작!
    
    // 물리 엔진 벡터 사용
    PhysicsVec3 physicsVec;  // (10, 20, 30)으로 초기화
    PhysicsVectorAdapter physicsAdapter(&physicsVec);
    MoveCharacter(&physicsAdapter);  // 이것도 잘 동작!
}
```

- 어댑터 클래스로 감싸준 후,<br>
  해당 타입을 전달시켜 사용<br>

- 이 경우, 차후 비슷한 개념이 등장하더라도<br>
  어댑터를 만들면 기존 코드 수정 x<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- 외부 SDK/라이브러리 통합<br>
- 크로스 플랫폼 개발<br>
- 레거시 시스템 연동<br>
- 다양한 API 통합 (렌더링, 오디오, 네트워크)<br>

비슷한 개념이지만<br>
실제 클래스가 다른 경우 등에 응용이 가능<br>
어댑터 클래스만 추가하면 되기에 유연<br>

### ❌ 피해야 할 경우

- 인터페이스가 이미 호환되는 경우<br>
- 단순한 데이터 변환만 필요한 경우<br>
- 성능이 극도로 중요한 경우<br>

래핑 클래스이기에<br>
성능적으로 주의할 필요는 있음<br>
또한, 클래스 수가 증가한다는 점도 유의<br>

## 데코레이터 패턴 (Decorator Pattern) - 무기에 인챈트를 걸기

## 📌 개념

객체에 동적으로 새로운 책임 (기능)을 추가할 수 있게 하는 패턴.<br>
기능 확장이 필요할 때 서브클래싱 대신 사용<br>

### 언제 필요한가?

- 무기에 여러 인챈트를 조합하고 싶을 때<br>
- 버프/디버프를 중첩해서 적용할 때<br>
- 기본 기능에 옵션을 동적으로 추가/제거할 때<br>
- 상속으로는 조합이 폭발적으로 증가하는 경우<br>

## 🧨 문제 상황

```cpp
// 상속으로 모든 조합을 만들면... 😱
class Sword {};
class FireSword : public Sword {};           // 화염 검
class IceSword : public Sword {};            // 얼음 검
class LightningSword : public Sword {};      // 번개 검

// 2개 조합
class FireIceSword : public Sword {};        // 화염+얼음
class FireLightningSword : public Sword {};  // 화염+번개
class IceLightningSword : public Sword {};   // 얼음+번개

// 3개 조합
class FireIceLightningSword : public Sword {};  // 화염+얼음+번개

// 만약 인챈트가 10개라면?
// 2^10 = 1024개의 클래스를 만들어야 함 ^^...
```

- 늘 새로운 조합 클래스를 만드는 것은...<br>

## ✅ 데코레이터 패턴 구조

### 1. 무기 인터페이스 - 모든 무기의 기본

```cpp
// 모든 무기가 가져야 할 기본 기능
class IWeapon {
public:
    virtual float GetDamage() const = 0;        // 데미지
    virtual float GetAttackSpeed() const = 0;   // 공격 속도
    virtual FString GetDescription() const = 0;  // 설명
    virtual float GetPrice() const = 0;         // 가격
    virtual ~IWeapon() = default;
};
```

- 기본이 될 인터페이스<br>

### 2. 기본 무기들 - 인챈트 없는 순수한 무기

```cpp
// 기본 검
class BasicSword : public IWeapon {
public:
    float GetDamage() const override { 
        return 50.0f;  // 기본 데미지 50
    }
    
    float GetAttackSpeed() const override { 
        return 1.0f;   // 기본 공속 1.0
    }
    
    FString GetDescription() const override { 
        return "Basic Sword";  // 그냥 기본 검
    }
    
    float GetPrice() const override { 
        return 100.0f;  // 100골드
    }
};

// 기본 활
class BasicBow : public IWeapon {
public:
    float GetDamage() const override { return 40.0f; }
    float GetAttackSpeed() const override { return 1.5f; }  // 활은 빨라!
    FString GetDescription() const override { return "Basic Bow"; }
    float GetPrice() const override { return 150.0f; }
};
```

- 기본 상태를 정의<br>

### 3. Decorator 베이스 - 인챈트의 부모

```cpp
// 모든 인챈트의 부모 클래스
class WeaponDecorator : public IWeapon {
protected:
    IWeapon* baseWeapon;  // 감쌀 무기를 저장
    
public:
    WeaponDecorator(IWeapon* weapon) : baseWeapon(weapon) {}
    
    virtual ~WeaponDecorator() {
        delete baseWeapon;  // 메모리 정리
    }
    
    // 기본 동작은 그냥 전달 (위임)
    float GetDamage() const override { 
        return baseWeapon->GetDamage();  // 안쪽 무기의 데미지 그대로
    }
    
    float GetAttackSpeed() const override { 
        return baseWeapon->GetAttackSpeed(); 
    }
    
    FString GetDescription() const override { 
        return baseWeapon->GetDescription(); 
    }
    
    float GetPrice() const override { 
        return baseWeapon->GetPrice(); 
    }
};
```

- Weapon 클래스를 감쌀 데코레이터 베이스 클래스<br>

### 4. 구체적인 인챈트들

```cpp
// 🔥 화염 인챈트
class FireEnchantment : public WeaponDecorator {
private:
    float fireDamage = 20.0f;  // 화염 추가 데미지
    
public:
    FireEnchantment(IWeapon* weapon) : WeaponDecorator(weapon) {}
    
    float GetDamage() const override {
        // 기존 데미지 + 화염 데미지!
        return baseWeapon->GetDamage() + fireDamage;
    }
    
    FString GetDescription() const override {
        // 기존 설명 + 화염 설명!
        return baseWeapon->GetDescription() + " [Fire +20]";
    }
    
    float GetPrice() const override {
        // 인챈트 비용 추가!
        return baseWeapon->GetPrice() + 500.0f;
    }
};

// ❄️ 얼음 인챈트
class IceEnchantment : public WeaponDecorator {
private:
    float iceDamage = 15.0f;
    float slowEffect = 0.2f;  // 20% 둔화 효과
    
public:
    IceEnchantment(IWeapon* weapon) : WeaponDecorator(weapon) {}
    
    float GetDamage() const override {
        return baseWeapon->GetDamage() + iceDamage;
    }
    
    FString GetDescription() const override {
        return baseWeapon->GetDescription() + " [Ice +15, Slow 20%]";
    }
    
    float GetPrice() const override {
        return baseWeapon->GetPrice() + 400.0f;
    }
};

// ⚡ 번개 인챈트
class LightningEnchantment : public WeaponDecorator {
private:
    float lightningDamage = 25.0f;
    
public:
    LightningEnchantment(IWeapon* weapon) : WeaponDecorator(weapon) {}
    
    float GetDamage() const override {
        return baseWeapon->GetDamage() + lightningDamage;
    }
    
    float GetAttackSpeed() const override {
        // 번개는 공속도 올려줘요!
        return baseWeapon->GetAttackSpeed() * 1.2f;  // 20% 증가
    }
    
    FString GetDescription() const override {
        return baseWeapon->GetDescription() + " [Lightning +25, Speed +20%]";
    }
    
    float GetPrice() const override {
        return baseWeapon->GetPrice() + 800.0f;
    }
};

// ⚔️ 예리함 인챈트 (곱셈 효과!)
class SharpnessEnchantment : public WeaponDecorator {
private:
    float damageMultiplier = 1.5f;  // 1.5배!
    
public:
    SharpnessEnchantment(IWeapon* weapon) : WeaponDecorator(weapon) {}
    
    float GetDamage() const override {
        // 곱셈이라 순서가 중요해요!
        return baseWeapon->GetDamage() * damageMultiplier;
    }
    
    FString GetDescription() const override {
        return baseWeapon->GetDescription() + " [Sharpness x1.5]";
    }
    
    float GetPrice() const override {
        return baseWeapon->GetPrice() + 1000.0f;
    }
};
```

- 데코레이터 베이스 클래스를 상속받아 여러 기능을 커스텀<br>

### 5. 사용 방법

```cpp
void CreateMyUltimateWeapon() {
    // 1단계: 기본 검 생성
    IWeapon* mySword = new BasicSword();
    UE_LOG(LogTemp, Warning, TEXT("%s - 데미지: %.1f, 가격: %.1f"), 
           *mySword->GetDescription(), 
           mySword->GetDamage(), 
           mySword->GetPrice());
    // 출력: "Basic Sword - 데미지: 50.0, 가격: 100.0"
    
    // 2단계: 화염 인챈트 추가! 🔥
    mySword = new FireEnchantment(mySword);
    UE_LOG(LogTemp, Warning, TEXT("%s - 데미지: %.1f, 가격: %.1f"), 
           *mySword->GetDescription(), 
           mySword->GetDamage(), 
           mySword->GetPrice());
    // 출력: "Basic Sword [Fire +20] - 데미지: 70.0, 가격: 600.0"
    
    // 3단계: 번개 인챈트 추가! ⚡
    mySword = new LightningEnchantment(mySword);
    UE_LOG(LogTemp, Warning, TEXT("%s - 데미지: %.1f, 공속: %.1f"), 
           *mySword->GetDescription(), 
           mySword->GetDamage(), 
           mySword->GetAttackSpeed());
    // 출력: "Basic Sword [Fire +20] [Lightning +25, Speed +20%] - 데미지: 95.0, 공속: 1.2"
    
    // 4단계: 예리함 추가! (순서 중요!)
    mySword = new SharpnessEnchantment(mySword);
    // 최종: 데미지 = 95.0 * 1.5 = 142.5!
}
```

- 조합할 때<br>
  기본 무기를 사용할 때, Decorator를 만든 후<br>
  반환받음<br>
  (Decorator도 Interface가 존재하기에 위와 같은 방식으로 적용 가능)<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- 무기/방어구 인챈트 시스템<br>
- 버프/디버프 시스템<br>
- UI 스타일 중첩<br>
- 스킬 강화 시스템<br>
- 런타임에 기능 추가/제거<br>

어찌되었든<br>
기존 클래스를 건들지 않으면서,<br>
런타임 중에 여러 조합이 가능<br>

기존 객체의 데이터를 건들지 않는 점도 장점<br>
(상속은 컴파일 타임에 결정)<br>
(if 문 같은 분기문은 유지보수 지옥이 펼쳐질 가능성 존재)<br>

상속을 끔찍하게 할 바에는 훨씬 좋다<br>
(단점을 고려하더라도 클래스 1000개 만드는 것보다는...)<br>

### ❌ 피해야 할 경우

- 기능이 고정적인 경우<br>
- 조합이 많지 않은 경우<br>
- 성능이 극도로 중요한 경우 (많은 래핑)<br>

순서가 결과에 영향을 줄 수 있음!<br>
(ex : 1.5배를 앞에서 더하느냐 뒤에서 더하느냐에 따른 결과가 크게 달라짐)<br>
(기획적인 문제 유발 가능함)<br>

계속 New로 감싸기에<br>
Delete 할때 바깥쪽만 지우기가 어려워짐<br>
(스마트 포인터 등을 이용하여 메모리 관리를 잘 해야 한다)<br>

어떤 데코레이터가 값을 바꾸었는지 디버깅이 까다로울 수 있음<br>

## 퍼사드 패턴 (Facade Pattern) - 복잡한 것을 간단하게!

## 📌 개념

복잡한 서브시스템들을 하나의 간단한 인터페이스로 통합하여 제공하는 패턴<br>
복잡함을 숨기고 사용하기 쉽게 만듦<br>

### 언제 필요한가?

- 게임 시작 시 수많은 시스템을 초기화할 때<br>
- 복잡한 라이브러리를 간단하게 사용하고 싶을 때<br>
- 여러 서브시스템을 조합해서 사용해야 할 때<br>
- 클라이언트가 내부 구조를 몰라도 되게 하고 싶을 때<br>

## 🧨 문제 상황

```cpp
// 게임 시작하려면 이 모든 걸 직접 해야 함ㅋㅋㅋㅋㅋ
void StartGame() {
    // 1. 렌더링 시스템 초기화
    RenderingSystem* renderer = new RenderingSystem();
    renderer->Initialize();
    renderer->LoadShaders();            // 셰이더 로딩
    renderer->SetupRenderTargets();     // 렌더 타겟 설정
    renderer->InitializePostProcessing(); // 포스트 프로세싱
    renderer->SetResolution(1920, 1080); // 해상도 설정
    renderer->EnableVSync(true);         // 수직동기화
    
    // 2. 오디오 시스템 초기화
    AudioSystem* audio = new AudioSystem();
    audio->Initialize();
    audio->InitializeAudioDevice();     // 오디오 장치 초기화
    audio->LoadAudioBanks();            // 사운드 파일들 로딩
    audio->SetupAudioChannels();        // 채널 설정
    audio->SetMasterVolume(0.8f);       // 볼륨 설정
    
    // 3. 입력 시스템 초기화
    InputSystem* input = new InputSystem();
    input->Initialize();
    input->DetectControllers();         // 컨트롤러 감지
    input->LoadKeyBindings();           // 키 설정 로드
    input->SetupInputContexts();        // 입력 컨텍스트
    
    // 4. 리소스 로딩
    ResourceSystem* resources = new ResourceSystem();
    resources->LoadTextures();          // 텍스처 로딩
    resources->LoadMeshes();            // 3D 모델 로딩
    resources->LoadAnimations();        // 애니메이션 로딩
    resources->LoadSounds();            // 사운드 로딩
    
    // 5. 네트워크 초기화 (멀티플레이어라면)
    // 6. 물리 엔진 초기화
    // 7. AI 시스템 초기화
    // ... 아직도 더 있어요!
}
```

- StartGame 보는 개발자 : 이걸 내가 다 알아야 해...?<br>

- 초기화 순서 달라지게 되면 버그 발생 가능<br>

- 새로운 순서 추가시, 모든 코드를 수정해야 할수도...<br>

## ✅ 퍼사드 패턴 구조

### 1. 복잡한 서브시스템들

```cpp
// 렌더링 시스템 - 그래픽 담당
class RenderingSystem {
public:
    void Initialize() {
        UE_LOG(LogTemp, Warning, TEXT("렌더링 시스템 초기화 중..."));
        LoadShaders();
        SetupRenderTargets();
        InitializePostProcessing();
    }
    
    void LoadShaders() { 
        UE_LOG(LogTemp, Warning, TEXT("셰이더 로딩..."));
    }
    
    void SetupRenderTargets() { 
        UE_LOG(LogTemp, Warning, TEXT("렌더 타겟 설정..."));
    }
    
    void InitializePostProcessing() { 
        UE_LOG(LogTemp, Warning, TEXT("포스트 프로세싱 초기화..."));
    }
    
    void SetResolution(int width, int height) { 
        UE_LOG(LogTemp, Warning, TEXT("해상도 설정: %dx%d"), width, height);
    }
    
    void EnableVSync(bool enable) { 
        UE_LOG(LogTemp, Warning, TEXT("수직동기화: %s"), enable ? TEXT("ON") : TEXT("OFF"));
    }
};

// 오디오 시스템 - 소리 담당
class AudioSystem {
public:
    void Initialize() {
        UE_LOG(LogTemp, Warning, TEXT("오디오 시스템 초기화 중..."));
        InitializeAudioDevice();
        LoadAudioBanks();
        SetupAudioChannels();
    }
    
    void InitializeAudioDevice() { /* 오디오 장치 설정 */ }
    void LoadAudioBanks() { /* 사운드 파일 로딩 */ }
    void SetupAudioChannels() { /* 5.1채널, 스테레오 등 */ }
    void SetMasterVolume(float volume) { /* 마스터 볼륨 */ }
    
    void PlayBGM(const FString& musicName) { 
        UE_LOG(LogTemp, Warning, TEXT("BGM 재생: %s"), *musicName);
    }
};

// 그 외 시스템들도 마찬가지...
```

- 일단 얘내는 그대로 둘 것<br>

### 2. Facade 클래스 - 간단한 인터페이스

```cpp
class GameSystemFacade {
private:
    // 모든 복잡한 시스템들을 내부에 숨겨요
    RenderingSystem* renderer;
    AudioSystem* audio;
    InputSystem* input;
    NetworkSystem* network;
    ResourceSystem* resources;
    
public:
    GameSystemFacade() {
        // Facade가 모든 시스템을 관리
        renderer = new RenderingSystem();
        audio = new AudioSystem();
        input = new InputSystem();
        network = new NetworkSystem();
        resources = new ResourceSystem();
    }
    
    ~GameSystemFacade() {
        // 메모리 정리도 Facade가 알아서
        delete renderer;
        delete audio;
        delete input;
        delete network;
        delete resources;
    }
    
    // 🎮 싱글플레이어 게임 시작 - 한 줄로 끝!
    void StartSinglePlayerGame() {
        UE_LOG(LogTemp, Warning, TEXT("=== 싱글플레이어 게임 시작 ==="));
        
        // 1. 필수 시스템만 초기화
        renderer->Initialize();
        audio->Initialize();
        input->Initialize();
        // 네트워크는 싱글플레이어니까 스킵!
        
        // 2. 싱글플레이어에 맞는 설정 자동 적용
        ApplyGraphicsSettings(EGraphicsQuality::High);  // 높은 그래픽
        ApplyAudioSettings(0.8f, 0.6f, 0.7f);          // 볼륨 설정
        
        // 3. 리소스 로딩
        resources->LoadGameResources();
        resources->PreloadLevel("Tutorial");  // 튜토리얼 레벨 준비
        
        // 4. 게임 시작!
        audio->PlayBGM("MainTheme");
        
        UE_LOG(LogTemp, Warning, TEXT("=== 게임 준비 완료! ==="));
    }
    
    // 🌐 멀티플레이어 게임 시작
    void StartMultiplayerGame(const FString& serverIP, bool isHost) {
        UE_LOG(LogTemp, Warning, TEXT("=== 멀티플레이어 게임 시작 ==="));
        
        // 1. 멀티플레이어는 네트워크도 필요!
        renderer->Initialize();
        audio->Initialize();
        input->Initialize();
        network->Initialize();  // 네트워크 추가!
        
        // 2. 호스트인지 클라이언트인지에 따라 다르게
        if (isHost) {
            network->SetConnectionType(true);
            UE_LOG(LogTemp, Warning, TEXT("호스트로 게임 생성"));
        } else {
            network->SetConnectionType(false);
            network->ConnectToServer(serverIP);
            UE_LOG(LogTemp, Warning, TEXT("서버 %s에 접속"), *serverIP);
        }
        
        // 3. 멀티플레이어는 성능이 중요! 그래픽 낮춤
        ApplyGraphicsSettings(EGraphicsQuality::Medium);
        ApplyAudioSettings(0.8f, 0.6f, 0.7f);
        
        // 4. 멀티플레이어 전용 리소스
        resources->LoadGameResources();
        resources->PreloadLevel("MultiplayerLobby");
        
        audio->PlayBGM("MultiplayerTheme");
    }
    
    // 🎨 그래픽 품질 설정도 간단하게!
    enum class EGraphicsQuality { Low, Medium, High, Ultra };
    
    void ApplyGraphicsSettings(EGraphicsQuality quality) {
        switch (quality) {
            case EGraphicsQuality::Low:
                // 저사양 PC용
                renderer->SetResolution(1280, 720);
                renderer->EnableVSync(false);
                UE_LOG(LogTemp, Warning, TEXT("그래픽: 낮음 (720p)"));
                break;
                
            case EGraphicsQuality::High:
                // 일반 게이밍 PC용
                renderer->SetResolution(1920, 1080);
                renderer->EnableVSync(true);
                UE_LOG(LogTemp, Warning, TEXT("그래픽: 높음 (1080p)"));
                break;
                
            case EGraphicsQuality::Ultra:
                // 고사양 PC용
                renderer->SetResolution(3840, 2160);
                renderer->EnableVSync(true);
                UE_LOG(LogTemp, Warning, TEXT("그래픽: 울트라 (4K)"));
                break;
        }
    }
    
    // 🔊 오디오 설정도 간단하게!
    void ApplyAudioSettings(float master, float sfx, float bgm) {
        audio->SetMasterVolume(master);
        // SFX 볼륨, BGM 볼륨 등도 설정...
        UE_LOG(LogTemp, Warning, TEXT("오디오 설정 완료"));
    }
    
    // 🎯 빠른 테스트용 메서드
    void QuickTestMode() {
        UE_LOG(LogTemp, Warning, TEXT("=== 빠른 테스트 모드 ==="));
        
        // 최소한의 초기화만!
        renderer->Initialize();
        renderer->SetResolution(1280, 720);  // 낮은 해상도로
        input->Initialize();
        
        // 테스트 레벨 바로 로드
        resources->PreloadLevel("TestArena");
        
        UE_LOG(LogTemp, Warning, TEXT("테스트 준비 완료! (3초 걸림)"));
    }
};
```

- 여러 시스템을 관리하는 팀장 같은 느낌의 클래스<br>

### 3. 사용 방법

```cpp
// ❌ Facade 없이 - 50줄 이상의 복잡한 코드
void OldWayToStartGame() {
    RenderingSystem* renderer = new RenderingSystem();
    renderer->Initialize();
    renderer->LoadShaders();
    // ... 50줄 더
}

// ✅ Facade 사용 - 단 2줄!
void NewWayToStartGame() {
    GameSystemFacade* gameFacade = new GameSystemFacade();
    
    // 싱글플레이어? 한 줄!
    gameFacade->StartSinglePlayerGame();
    
    // 멀티플레이어 호스트? 한 줄!
    gameFacade->StartMultiplayerGame("", true);
    
    // 클라이언트로 접속? 한 줄!
    gameFacade->StartMultiplayerGame("192.168.1.100", false);
    
    // 테스트? 한 줄!
    gameFacade->QuickTestMode();
}
```

- 파사드로 대표되는 클래스가 알아서 해준다<br>

- 내부 순서를 굳이 알 필요 없음<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- 복잡한 서브시스템 통합<br>
- 게임 초기화/종료 시스템<br>
- 세이브/로드 시스템<br>
- 설정 관리 시스템<br>
- 외부 SDK 통합<br>

내부를 몰라도 된다는 점은<br>
책임 관리 및 실수 방지에 유리<br>

여러 시스템들의 '특정한 공통된 시점'을 잡아<br>
하나로 묶어 관리<br>

### ❌ 피해야 할 경우

- 단순한 기능<br>
- 세밀한 제어가 필요한 경우<br>
- 서브시스템이 자주 변경되는 경우<br>

너무 심취하면 God 클래스를 만들어버림!<br>
최소한의 '역할'을 부여하여 '클래스의 존재 의의'를 정의해야 함<br>

자주 수정되는 클래스 라면 파사드도 같이 영향을 받으니 주의<br>

# 2. 행동 패턴 (Behavioral Patterns) 🎯

## 옵저버 패턴 (Observer Pattern) - 이벤트 시스템의 핵심

## 📌 개념

- 객체의 상태 변화를 관찰하던 객체들에게 자동으로 알림을 보내는 패턴<br>
- 1:N 의존 관계를 정의하여 한 객체의 상태가 변하면 모든 의존 객체들이 알림을 받고 자동으로 업데이트 됨<br>

### 언제 필요한가?

- 플레이어 체력이 변할 때 UI, 사운드, 이펙트가 동시에 반응해야 할 때<br>
- 게임 상태 변화를 여러 시스템이 감지해야 할 때<br>
- 느슨한 결합을 유지하면서 이벤트를 전파하고 싶을 때<br>
- MVC 패턴에서 모델의 변화를 뷰에 알려야 할 때<br>

## 🧨 문제 상황

```cpp
class Player {
    HealthBarUI* healthBar;
    AudioManager* audio;
    
public:
    void TakeDamage(float damage) {
        health -= damage;
        
        // 😱 모든 시스템을 직접 호출해야 함
        healthBar->UpdateHealth(health);
        audio->PlayDamageSound();
        // 새 시스템 추가할 때마다 여기 수정...
    }
};
```

- 플레이어가 너무 많은 외부 클래스를 알아야 하며<br>
  결합도가 높아짐<br>

## ✅ Observer 패턴 구조

### 1. Subject 인터페이스

```cpp
class IHealthObserver {
public:
    virtual void OnHealthChanged(float health) = 0;
};
```

- 플레이어 체력에 관심있는 클래스들이 상속받을 클래스<br>

### 2. Concrete Subject (Player)

```cpp
class Player {
private:
    float health = 100.0f;
    vector<IHealthObserver*> observers;
    
public:
    void AttachObserver(IHealthObserver* obs) {
        observers.push_back(obs);
    }
    
    void NotifyHealthChanged() {
        for (auto obs : observers) {
            obs->OnHealthChanged(health);
        }
    }
    
    void TakeDamage(float damage) {
        health -= damage;
        NotifyHealthChanged(); // 모든 관찰자에게 자동 알림!
    }
};
```

- Attach 된 옵저버들에게 알림을 보낸다<br>

### 3. Concrete Observers

```cpp
class HealthBarUI : public IHealthObserver {
public:
    void OnHealthChanged(float health) override {
        cout << "UI: Health updated to " << health << endl;
    }
};

class AudioObserver : public IHealthObserver {
public:
    void OnHealthChanged(float health) override {
        if (health < 20) cout << "Audio: Low health warning!" << endl;
    }
};
```

- 알림이 왔으니 작업을 처리!<br>
  플레이어는 이들의 존재를 몰라도 됨<br>
  (그냥 알림 보내면 누가 받겠지)<br>

### 4. 사용 방법

```cpp
Player player;
HealthBarUI ui;
AudioObserver audio;

player.AttachObserver(&ui);
player.AttachObserver(&audio);

player.TakeDamage(30); // UI와 Audio가 자동으로 반응!
```

- Init 할 시점에 등록을 해둔다<br>

### etc. Delegate?

개념상으로 Delegate는 '대표 객체'에 위임하는 방식<br>
(1 : 1)<br>

Observer는 1 : N 형식<br>

- Unreal에서 존재하는 Multi Delegate는 Observer와 유사함<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- **플레이어 상태 변화**: 체력, 마나, 경험치 변화 시 여러 UI 요소가 동시에 업데이트<br>
- **이벤트 기반 시스템**: 아이템 획득, 퀘스트 완료, 레벨업 등의 이벤트 발생 시<br>
- **실시간 알림**: 길드 채팅, 친구 접속, 경매장 알림 등<br>
- **성취/업적 시스템**: 특정 조건 달성 시 여러 시스템이 반응해야 할 때<br>

이벤트 기반 시스템에 적합<br>
특정한 '이벤트'에 반응함으로서 각 객체가 전반적으로 행동할 수 있음<br>

### ❌ 피해야 할 경우

- 성능 문제 상황<br>
    - **극도로 빈번한 호출**: 매 프레임 발생하는 이벤트 (예: 마우스 이동)<br>
    - **대량의 관찰자**: 수천 개의 Observer가 등록된 경우<br>
    - **복잡한 알림 로직**: Observer 내부에서 무거운 연산을 수행하는 경우<br>
    - **체인 반응**: Observer가 또 다른 이벤트를 발생시켜 무한 루프 위험<br>

너무 자주 발생하는 이벤트에는 적합하지 않음<br>
일일이 호출하는 편을 고려할 수도 있다<br>

- 단순한 상황<br>
    - **1:1 관계**: 오직 하나의 객체만 반응하는 경우<br>
    - **직접 호출이 간단**: `healthBar->Update()` 한 줄로 충분한 경우<br>
    - **즉시 처리 필요**: 이벤트 발생과 동시에 즉시 처리되어야 하는 경우<br>

오버 엔지니어링은 항상 유의할 것!<br>


## 커맨드 패턴 (Command Pattern) - 행동의 캡슐화

## 📌 개념

요청을 객체로 캡슐화하여 매개변수로 전달하고,<br>
요청을 큐에 저장하거나 로그를 남기고,<br>
실행 취소를 지원할 수 있게 하는 패턴<br>

### 언제 필요한가?

- 플레이어 행동을 되돌리기 (Undo) 기능이 필요할 때<br>
- 키 바인딩을 동적으로 변경하고 싶을 때<br>
- 행동을 큐에 저장해서 순차 실행하고 싶을 때<br>
- 매크로나 리플레이 시스템을 만들 때<br>
- AI의 행동을 계획하고 실행할 때<br>

## 🧨 문제 상황

```cpp
class Player {
public:
    void MoveUp() { y++; }
    void Attack() { /* 공격 */ }
};

// 문제: 되돌리기 불가능, 키 바인딩 변경 어려움
void HandleInput(char key) {
    if (key == 'W') player.MoveUp();
    if (key == 'A') player.Attack();
}
```

- 전략 게임이라면 되돌리기 기능이 필요할 수 있음<br>

## ✅ Command 패턴 구조

### 1. Command 인터페이스

```cpp
class ICommand {
public:
    virtual void Execute() = 0;
    virtual void Undo() = 0;
};
```

- '특정 행동'을 '명령'으로 감싸기<br>

### 2. Receiver (실제 작업을 수행하는 객체)

```cpp
class Player {
public:
    int x = 0, y = 0;
    void MoveUp() { y++; }
    void MoveDown() { y--; }
};
```

- 명령 수행 객체를 분리<br>

### 3. Concrete Commands

```cpp
class MoveUpCommand : public ICommand {
private:
    Player* player;
    
public:
    MoveUpCommand(Player* p) : player(p) {}
    
    void Execute() override { player->MoveUp(); }
    void Undo() override { player->MoveDown(); }
};
```

- 구체적인 명령 수행 객체<br>

- 명령 수행 객체에게 명령을 '적용'할 객체<br>
  (인터페이스 상속 클래스)<br>

### 4. Invoker (커맨드 실행자)

```cpp
class CommandManager {
private:
    vector<ICommand*> history;
    int currentIndex = -1;
    
public:
    void ExecuteCommand(ICommand* cmd) {
        cmd->Execute();
        history.push_back(cmd);
        currentIndex++;
    }
    
    void Undo() {
        if (currentIndex >= 0) {
            history[currentIndex]->Undo();
            currentIndex--;
        }
    }
};
```

- 히스토리를 저장하여<br>
  Undo가 가능하도록<br>

### 5. 사용 방법

```cpp
Player player;
CommandManager cmdManager;

auto moveCmd = new MoveUpCommand(&player);
cmdManager.ExecuteCommand(moveCmd); // 실행
cmdManager.Undo();                  // 되돌리기!
```

- 명령 객체를 생성하여 적용하고<br>
  관리자를 통해 취소할 수 있도록 설정<br>

## 📌 사용 가이드라인

### ✅ 적합한 경우

- **실행 취소/재실행**: 턴제 게임, 퍼즐 게임, 레벨 에디터<br>
- **매크로 시스템**: 복잡한 조작을 하나의 버튼으로 실행<br>
- **리플레이 기능**: 플레이어 행동을 기록하고 재생<br>
- **키 바인딩**: 사용자가 키 설정을 자유롭게 변경<br>

턴제, 퍼즐 계열 게임에 유용한 디자인 패턴<br>
(실행 취소 등이 필요한 경우)<br>
(캐릭터 공격 버튼 누르고, 다시 다른 캐릭 공격 누르러 갔다가 취소한다던가)<br>

EnhancedInputSystem도 Command 패턴의 일종<br>
(행동 단위로 정의된 객체 - InputAction이 Command 객체 역할)<br>


### ❌ 피해야 할 경우

- 성능 중시 상황<br>
    - **실시간 액션 게임**: 즉시 반응이 필요한 조작 (점프, 공격)<br>
    - **물리 연산**: 매 프레임 실행되는 물리 시뮬레이션<br>
    - **렌더링 명령**: 그래픽 카드에 직접 전달되는 명령들<br>
    - **메모리 제약**: 모바일 등 메모리가 극도로 제한적인 환경<br>

별도의 명령 객체를 관리하기에<br>
오버헤드가 존재함<br>

- 단순한 기능<br>
    - **일회성 실행**: 되돌리기가 불필요한 간단한 기능<br>
    - **상태 없음**: 매개변수가 없는 단순한 메서드 호출<br>
    - **즉시 실행만**: 큐잉이나 지연 실행이 불필요한 경우<br>

일회성 동작에 이런 것을 고려할 필요는 당연히 없음<br>

## 상태 패턴 (State Pattern) - AI와 게임 상태의 달인

## 📌 개념

객체의 내부 상태가 변할 때, 객체의 행동도 함께 변하도록 하는 패턴.<br>
상태를 별도의 클래스로 캡슐화하고 위임을 통해 행동을 변경<br>

### 언제 필요한가?

- AI 캐릭터의 상태 (Idle, Patrol, Chase, Attack)가 복잡할 때<br>
- 게임 상태 (Menu, Playing, Paused, GameOver)를 관리할 때<br>
- 플레이어 상태 (Normal, Stunned, Invisible, Flying)에 따라 행동이 달라질 때<br>
- 복잡한 if-else 문을 깔끔하게 정리하고 싶을 때<br>

## 🧨 문제 상황

```cpp
class Enemy {
    enum State { IDLE, CHASE, ATTACK };
    State currentState = IDLE;
    
public:
    void Update() {
        switch(currentState) {
            case IDLE:
                if (SeePlayer()) currentState = CHASE;
                // 😱 모든 상태 로직이 한 곳에 몰림
                break;
            case CHASE:
                if (NearPlayer()) currentState = ATTACK;
                else if (!SeePlayer()) currentState = IDLE;
                break;
            // 상태가 늘어날수록 복잡해짐...
        }
    }
};
```

- 하나의 클래스에 Switch 문으로<br>
  상태 관리<br>

- 하나의 상태 추가 -> Switch 추가<br>

## ✅ State 패턴 구조

### 1. State 인터페이스와 Context

```cpp
class IEnemyState {
public:
    virtual void Update(class Enemy* enemy) = 0;
};

class Enemy {
private:
    IEnemyState* currentState;
    
public:
    void ChangeState(IEnemyState* newState) {
        delete currentState;
        currentState = newState;
    }
    
    void Update() {
        if (currentState) currentState->Update(this);
    }
    
    bool SeePlayer() { return true; }  // 임시
    bool NearPlayer() { return true; } // 임시
};
```

- `상태`를 하나의 클래스로 정의<br>

- Enemy 클래스는 현재 상태를 딱히 알 필요가 없음<br>
  현재 '상태'가 알아서 작동하도록 함수만 호출<br>

### 2. Concrete States

```cpp
class IdleState : public IEnemyState {
public:
    void Update(Enemy* enemy) override {
        cout << "Idling..." << endl;
        if (enemy->SeePlayer()) {
            enemy->ChangeState(new ChaseState());
        }
    }
};

class ChaseState : public IEnemyState {
public:
    void Update(Enemy* enemy) override {
        cout << "Chasing player!" << endl;
        if (enemy->NearPlayer()) {
            enemy->ChangeState(new AttackState());
        }
    }
};

class AttackState : public IEnemyState {
public:
    void Update(Enemy* enemy) override {
        cout << "Attacking!" << endl;
    }
};
```

- 각각의 상태를 통하여<br>
  Update 에서 각 상태의 특정 행동을 구현함<br>

- Enemy와 행동 로직이 분리됨!<br>

- 내부에서 상태 변환도 할 수 있음<br>

- Delete / New 는 할당 비용이 높아짐<br>
  차라리 Object Pool 을 고려하는 것도 하나의 방법임<br>
  (아니면 지역 Static 변수를 통한 응용)<br>

### 3. 사용 방법

```cpp
Enemy enemy;
enemy.ChangeState(new IdleState());

for (int i = 0; i < 10; i++) {
    enemy.Update(); // 상태에 따라 다른 행동!
}
```

## 📌 사용 가이드라인

### ✅ 적합한 경우

- **NPC 행동**: Idle → Patrol → Alert → Attack → Flee<br>
- **몬스터 AI**: 평상시, 경계, 추적, 공격, 도망, 사망<br>
- **보스 패턴**: 페이즈별로 완전히 다른 행동 패턴<br>
- **동물 AI**: 배고픔, 목마름, 수면, 경계 등 복합 상태<br>

BT와 AI Controller도 이러한 패턴의 응용에 가까움<br>
(State Tree)<br>

ABP의 상태 변화도 이것과 유사<br>

상태가 하나의 '책임'을 가지고<br>
그러한 '상태'를 구현하여 사용<br>

복잡한 '상태' 구현에 유용<br>
(게임의 전체 흐름 제어에도 응용이 가능함)<br>
(하나의 클래스에서 일일이 비교할 필요 없음)<br>

### ❌ 피해야 할 경우

- 단순한 상태
    - **2-3개의 단순 상태**: boolean 변수로도 충분한 경우
    - **상태 전환이 거의 없음**: 초기 설정 후 변경이 드문 경우
    - **로직이 매우 단순**: 각 상태에서 하는 일이 한두 줄인 경우

과설계 (오버 엔지니어링)<br>

- 성능이 중요한 경우
    - **매 프레임 실행**: 물리 연산처럼 극도로 빈번한 호출
    - **대량의 객체**: 수천 개의 객체가 각각 상태를 가지는 경우
    - **실시간 처리**: 지연이 허용되지 않는 시스템

수천 개 사용되는 AI라면 다시 고려해볼 것<br>
(미니언 같은 애들은 매우 '많이' 생성될 수 있는 AI임)<br>

## 전략 패턴 (Strategy Pattern) - 알고리즘의 자유로운 교체

## 📌 개념

동일한 문제를 해결하는 여러 알고리즘을 캡슐화하고,<br>
런타임에 알고리즘을 선택할 수 있게 하는 패턴<br>

알고리즘의 변경이 클라이언트에 영향을 주지 않음<br>

- 사진을 찍을 때, 다양한 모드를 적용하여 찍지만<br>
  전부 하나의 '사진' 찍기<br>

### 언제 필요한가?

- AI의 행동 패턴을 동적으로 바꾸고 싶을 때<br>
- 난이도에 따라 적의 움직임이 달라져야 할 때<br>
- 무기마다 다른 공격 패턴을 가져야 할 때<br>
- 정렬, 탐색 등의 알고리즘을 상황에 맞게 선택하고 싶을 때<br>

## 🧨 문제 상황

```cpp
class Enemy {
    enum MovementType { AGGRESSIVE, DEFENSIVE, RANDOM };
    MovementType moveType;
    
public:
    void Move() {
        switch(moveType) {
            case AGGRESSIVE:
                // 😱 모든 알고리즘이 한 클래스에
                MoveTowardsPlayer();
                break;
            case DEFENSIVE:
                RunAway();
                break;
            // 새 전략 추가 시 여기 수정...
        }
    }
};
```

- 난이도에 따라 적의 행동 패턴이 달라지는 경우?<br>

- Switch를 통하여 만들면 Move가 점점 커지게 된다<br>

## ✅ Strategy 패턴 구조

### 1. Strategy 인터페이스

```cpp
class IMovementStrategy {
public:
    virtual void Execute(class GameCharacter* character) = 0;
};
```

- 움직임 전략 클래스<br>

### 2. Context 클래스

```cpp
class GameCharacter {
private:
    IMovementStrategy* movementStrategy;
    
public:
    void SetMovementStrategy(IMovementStrategy* strategy) {
        delete movementStrategy;
        movementStrategy = strategy;
    }
    
    void Move() {
        if (movementStrategy) {
            movementStrategy->Execute(this);
        }
    }
};
```

- 기존 전략을 삭제하고 새로운 전략으로 교체<br>

- 기능은 해당 전략 클래스에 위임<br>

- State와 유사한 패턴?<br>

### 3. Concrete Strategies

```cpp
class AggressiveMovement : public IMovementStrategy {
public:
    void Execute(GameCharacter* character) override {
        cout << "Moving aggressively towards player!" << endl;
        // 플레이어를 향해 빠르게 이동
    }
};

class DefensiveMovement : public IMovementStrategy {
public:
    void Execute(GameCharacter* character) override {
        cout << "Moving defensively, keeping distance" << endl;
        // 거리를 유지하며 이동
    }
};

class RandomMovement : public IMovementStrategy {
public:
    void Execute(GameCharacter* character) override {
        cout << "Moving randomly" << endl;
        // 랜덤한 방향으로 이동
    }
};
```

- 각각의 전략을 패턴 생성<br>
  (서로의 존재도 모른다)<br>

### 4. 사용 방법

```cpp
GameCharacter enemy;

// 전략 변경
enemy.SetMovementStrategy(new AggressiveMovement());
enemy.Move(); // "Moving aggressively towards player!"

enemy.SetMovementStrategy(new DefensiveMovement());
enemy.Move(); // "Moving defensively, keeping distance"
```

- 필요에 따라 전략을 변경<br>

## 📌 사용 가이드라인

State 와 Strategy?<br>
`변화의 주체`가 다른 편<br>

- State : 자체적으로 상태가 변경됨<br>

- Strategy : 외부의 환경 변화에 따라 '전략'을 변경함<br>

### ✅ 적합한 경우

- 게임 AI 다양성<br>
    - **AI 행동 패턴**: 공격적, 수비적, 균형잡힌 등 다양한 성향<br>
    - **난이도별 AI**: 같은 캐릭터라도 난이도에 따라 다른 전략<br>
    - **상황 적응 AI**: 플레이어 플레이 스타일에 따라 전략 변경<br>
    - **팀 전술**: 상황에 따라 공격, 수비, 밸런스 전술 전환<br>

- 전투 시스템<br>
    - **무기별 공격 패턴**: 검, 활, 마법 등 각각 다른 공격 방식<br>
    - **스킬 발동 조건**: 체력, 마나, 상황에 따른 다른 스킬 선택<br>
    - **방어 전략**: 방패, 회피, 반격 등 다양한 방어 방식<br>
    - **버프/디버프**: 상황에 맞는 최적의 버프 선택<br>

동적인 무언가를 조정하기에 유용한 디자인 패턴<br>
(런타임에 테스트할때, 동적으로 바꿔끼면 되므로 아주 좋다)<br>

BT 도 전략 패턴의 응용이라 볼수도 있음<br>
(전략 : Task)<br>

단순히 AI 라기 보단<br>
특정 조건에 따라 변경되어야 한다면 고려할 수 있음<br>
(자동 전투에 유용할 듯)<br>

### ❌ 피해야 할 경우

- 단일 알고리즘
    - **고정된 로직**: 항상 같은 방식으로만 처리되는 경우
    - **변경 불필요**: 런타임에 전략을 바꿀 필요가 없는 경우
    - **단순한 분기**: if-else 한두 개로 충분한 경우
    - **성능이 최우선**: 함수 포인터 직접 호출이 더 빠른 경우

매 프레임 호출된다면<br>
함수 포인터 나 람다 를 고려할 수 있음<br>

- 과도한 추상화
    - **과도한 복잡성**: 전략 패턴으로 인해 코드가 더 복잡해지는 경우
    - **이해하기 어려움**: 팀원들이 이해하기 어려운 수준의 추상화
    - **디버깅 어려움**: 어떤 전략이 실행되는지 추적하기 어려운 경우

오~버 엔지니어링 주~의!!<br>