---
title: "김하연 튜터님 강의 - 'GAS의 멀티 플레이 예측 시스템'"
date : "2025-12-17 12:00:00 +0900"
last_modified_at: "2025-12-17T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - GAS
  - Client Prediction
  - NetExecutionPolicy
---

# GAS의 멀티 플레이 예측 시스템에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>

# 1. 네트워크 기초와 GAS의 접근법 📡

## 1-1. 전통적 네트워크 방식의 문제점

**서버 권위 방식의 한계**<br>

- 너무 '늦게' 클라가 반응함<br>
- 렉에 의해 UX가 매우 나빠짐...<br>

```cpp
void AMyCharacter::Input_FireSkill()
{
    // 1. 클라이언트는 아무것도 하지 않고, 즉시 서버에 요청만 보냄.
    ServerRPC_TryFireSkill(); 
}

// ---------------------------------------------------------------
// 네트워크 레이턴시 (RTT: Round Trip Time) 발생구간
// ---------------------------------------------------------------
void AMyCharacter::ServerRPC_TryFireSkill_Implementation()
{
    // 2. 서버가 도착한 요청을 검증
    if (CurrentMana >= 10 && !bIsCooldown)
    {
        CurrentMana -= 10;
        
        // 3. 검증 통과! 이제 모든 클라이언트(나 포함)에게 연출을 보여주라고 명령
        MulticastRPC_PlaySkillEffect();
    }
}

// ---------------------------------------------------------------
// 네트워크 레이턴시 발생구간
// ---------------------------------------------------------------
void AMyCharacter::MulticastRPC_PlaySkillEffect_Implementation()
{
    // 4. 클라이언트는 이제서야 이펙트를 재생
    SpawnEmitterAtLocation(...);
    PlayAnimMontage(...);
}
```

**왜 200ms가 문제인가?**<br>

- FPS: 0.2초 동안 적이 이미 이동<br>
- 격투게임: 60fps 기준 12프레임 지연 (약손 3프레임)<br>
- 체감: "내가 누른 게 안 먹혀!"<br>

## 1-2. 클라이언트 예측 (Client Prediction)의 원리

**핵심 아이디어: "서버 응답 기다리지 말고 일단 보여주자!"**<br>

```cpp
// GAS 내부 클라이언트 예측 로직의 단순화 버전
void UMyGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, ...)
{
    // 1단계: 로컬 클라이언트인가?
    if (IsLocallyControlled()) 
    {
        // [예측 실행] 서버 응답을 기다리지 않고 즉시 저지릅니다.
        // A. 예측 키(Prediction Key) 발급
        FPredictionKey Key = GeneratePredictionKey(); 

        // B. 시각적 피드백 즉시 실행 (가짜 처리)
        PlayMontage(AttackAnim);     // 칼 휘두르기
        PlaySound(SwingSound);       // 소리 재생
        ModifyAttribute(Mana, -50);  // 내 화면의 마나만 일단 깎음 (서버값 아님)
        
        // C. 서버로 전송 (RPC)
        ServerRPC_TryActivateAbility(Key);
    }
}

// ---------------------------------------------------------------
// 서버 측 처리 (Server Authority)
// ---------------------------------------------------------------
void UMyGameplayAbility::ServerRPC_TryActivateAbility_Implementation(FPredictionKey Key)
{
    // 서버가 검증
    if (CanActivateAbility()) 
    {
		    // [승인]
        // 실제 서버 데이터(마나, 쿨타임)를 갱신하고 다른 유저들에게 전파합니다.
        ClientRPC_PredictionSuccess(Key);
    }
    else 
    {
        // [거부]
        ClientRPC_PredictionFailed(Key);
    }
}
```

- 원래 있는 '네트워크 이론'의 일종이며<br>
  이를 GAS가 채택!<br>

- 클라가 '먼저 스킬을' 발동시킨 것처럼 연출<br>
  서버가 검증하여 성공시, 다른 클라에 전파<br>
  실패시, 반환하기<br>

- Key를 통해 서버의 처리 결과를 연동받을 수 있음<br>

- 이러한 Key에 실행한 클라의 행동을 묶어두어<br>
  실패 시, 아래의 '롤백'을 실행함<br>

## 1-3. 롤백(Rollback) 메커니즘

**예측이 틀렸을 때의 처리**<br>

```cpp
// 📜 GAS 네트워크 타임라인 시뮬레이션
// 상황: 마나 100 보유. 50 소모 스킬(Fireball) 시전.
// 변수: 0.1초 전 피격되어 실제 마나는 10임 (클라이언트는 모름)

// [T+0.00s] Client (예측 시작)
// ---------------------------------------------------------
1. PredictionKey [101] 생성
2. UI 업데이트: 마나 100 -> 50 (가짜 값)
3. GameplayEffect 적용 (Predicted): "Cost_Fireball" (마나 -50)
4. 애니메이션: "Montage_CastFireball" 재생 시작
5. 서버로 전송: ServerRPC_ActivateAbility(Key: 101)

// [T+0.10s] Server (검증 및 판결)
// ---------------------------------------------------------
1. 요청 수신: Key [101] 확인
2. 검증(Validate): 
   - 현재 서버 마나: 10
   - 필요 비용: 50
   - 결과: 실패 (Not Enough Resources)
3. 처형(Punishment): 
   - ClientRPC_PredictionFailed(Key: 101) 전송
   - "야, 너 마나 10이야." (Attribute 동기화 패킷 전송)

// [T+0.20s] Client (롤백 집행)
// ---------------------------------------------------------
1. 실패 통보 수신: Key [101]은 무효임.
2. 뒷수습 (Cleanup):
   - Key [101]에 묶여있던 GameplayEffect("Cost_Fireball")를 찾아서 강제 제거.
   - 마나 값 보정: 50이었던 마나가 서버 값인 10으로 '튀면서' 변경.
   - 실행 중이던 어빌리티(Ability) 강제 종료 (CancelAbility).
```

1. Clinet가 Key 생성 및 연출 처리 (UX를 위해)<br>
2. Server가 Key 확인 및 가능여부 확인후 최종 판정<br>
3. Client가 Key를 다시 확인하여 현재 관련된 작업을 전부 취소 함<br>

- 이러한 롤백이 UX에 좋을까...?<br>
  - 일단 CharacterMovement 에서, '미끄러진' 느낌으로 위치를 조정<br>
  - GAS에서 StopMontage를 하더라도, blend Time을 주어 애니메이션을 블렌드함<br>
  - Particle 등도 투명도를 서서히 낮추는 방식으로 취소<br>

- 물론 렉 특유의 불쾌함은 있지만 여러 처리를 통해 불쾌함을 최소화 하려 함<br>

## 1-4. Prediction Key - 예측의 추적 시스템

**FPredictionKey의 실제 구조**<br>

```cpp
// FPredictionKey 구조체 (개념적 단순화)
struct FPredictionKey
{
public:
    // [1] 현재 내 예측 번호 (My Hope)
    int16 Current;       

    // [2] 서버가 마지막으로 인정한 번호 (Server's Ack)
    int16 Base;          

    // [3] 서버 주도 여부
    bool bIsServerInitiated; 
};
```

- 일종의 '영수증'<br>

- bIsServerInitiated 가 true이면<br>
  강력한 권한에 따르어야 함<br>
  반대로 false면 보통 요청 단계를 뜻함<br>

```cpp
// 실제 GAS 어빌리티 발동 내부 로직 (Conceptual Code)
void UGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, ...)
{
    // 1. 예측 키 생성 (ASC가 알아서 해줌)
    // Current Key: 41 -> 42
    FPredictionKey ActivationKey = GetPredictionKey();

    // 2. [중요] 예측 윈도우(창구) 열기!
    // "지금부터 이 중괄호 { } 안에서 일어나는 모든 일은 Key #42 소관이다!"
    {
        FScopedPredictionWindow ScopedWindow(GetAbilitySystemComponent(), ActivationKey);

        // --- 이 아래 모든 함수는 자동으로 Key #42를 달고 실행됩니다 ---

        // (A) 마나 감소
        // 내부적으로: ApplyGameplayEffect(..., PredictionKey=42);
        CommitAbility(Handle, ActorInfo, ActivationInfo);

        // (B) 몽타주 재생
        // 내부적으로: PlayMontage(..., PredictionKey=42);
        MontageTask->ReadyForActivation();

        // (C) 이벤트 전송
        // 내부적으로: SendGameplayEvent(..., PredictionKey=42);
        SendGameplayEvent(...);

    } // <--- 윈도우 종료. Key #42의 영향력 끝.

    // 3. 서버로 전송
    // "Key #42로 이것들 저질렀습니다. 확인 부탁해요."
    ServerRPC_TryActivateAbility(ActivationKey);
}
```

- ASC가 Key를 자동으로 생성함<br>
  - Scope를 활용하여 해당 key와 연동될 작업을 결정<br>
    RAII!<br>


```cpp
// 상황: 0.1초 간격으로 스킬 3개를 난사함 (콤보)
// Client State: Base(40) / Current(43)

// [Time 0.00s] 대시 (Key #41) -> 예측 실행
// [Time 0.10s] 공격 (Key #42) -> 예측 실행
// [Time 0.20s] 점프 (Key #43) -> 예측 실행

// ... 통신 지연 (Ping 100ms) ...

// [Time 0.30s] 서버의 판결 도착!
void OnServerResponse()
{
    // [판결 1] "대시(#41)? 문제없어. 승인(ACK)."

    // [판결 2] "공격(#42)? 너 그때 기절(Stun) 상태였잖아. 거절(NACK)!"

    // [판결 3] "점프(#43)? ...어?" // Re-simulation
    
    // Case A: 점프가 공격의 후딜레이 캔슬이었다면? -> 점프도 불가능해짐 -> 취소
    // Case B: 공격과 상관없는 이동 점프였다면? -> 점프는 살아남음!
}
```

- 여러 Client가 보낸 다양한 Key들을 처리<br>

- Re-simulation?<br>
  : 특정 시점에서 상태를 재확인<br>

key를 기반으로<br>
- UX를 위한 Client 먼저 연출 출력<br>
- key를 바탕으로 server에서 상태 처리<br>
- 서버의 로직을 따라 roll-back<br>

-> 동기화 코드를 처음부터 고려할 필요없어짐!<br>

# 2. Ability Activation 정책 심화 🎮

- GAS 동기화 문제가 나오기 매우 쉬운 부분!<br>

## 2-1. NetExecutionPolicy 완벽 이해

```cpp
enum class EGameplayAbilityNetExecutionPolicy : uint8
{
    LocalOnly,          // 로컬 전용
    LocalPredicted,     // 클라이언트 예측
    ServerOnly,         // 서버만 실행
    ServerInitiated     // 서버가 시작
};
```

**각 정책의 사용처**<br>

| 정책 | 용도 | 예시 |
| --- | --- | --- |
| **LocalOnly** | 네트워크 무관 | UI 조작, 카메라 이동 |
| **LocalPredicted** | 즉각 반응 필요 | 기본 공격, 이동 스킬 |
| **ServerOnly** | 보안 중요 | 아이템 구매, 레벨업 |
| **ServerInitiated** | 서버 이벤트 | 보스 패턴, 환경 피해 |

- UI나 카메라는 애초에 Server에서 생성이 안됨<br>
  (아예 통신 시도를 안함)<br>

- 이펙트 출력이 필요하다면, 먼저 Local 처리가 필요함<br>
  (클라에서 보여야 하며, UX가 중요함)<br>

- 보안이 중요한 로직은 무조건 서버에서!<br>

- 모든 플레이어에게 발생시킬 이벤트는 서버가 직접 발생시켜야 함<br>

**1. LocalOnly와 LocalPredicted**<br>

```cpp
// 1. 인벤토리 열기 (LocalOnly)
UInventoryAbility::UInventoryAbility()
{
    // 정책 설정: 나만 실행함
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalOnly;
}

void UInventoryAbility::ActivateAbility(...)
{
    // 서버 체크 불필요
    // RPC를 보내지 않으므로 즉각 실행됨. 핑(Ping) 0ms.
    if (IsLocallyControlled())
    {
        ShowInventoryUI();
    }
}

// -----------------------------------------------------------

// 2. 대시 스킬 (LocalPredicted)
UDashAbility::UDashAbility()
{
    // 정책 설정: 예측 실행
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UDashAbility::ActivateAbility(...)
{
    // 클라이언트의 선 실행 (Prediction)
    if (IsLocallyControlled())
    {
        // 1. 일단 이동시킴 (예측)
        PerformDashMovement(); 
        // 2. 쿨타임 UI 돌림 (가짜)
        CommitAbilityCost(Handle, ...); 
    }

    // 서버의 검증 (Authority)
    // (이 코드는 서버에서 0.1초 뒤에 실행됨)
    if (HasAuthority())
    {
        if (!CanActivateAbility(...))
        {
            // 롤백 명령 전송
            CancelAbility(Handle, ...);
        }
    }
}
```

- 인벤토리 UI 여는 것은 사실상 그 Client만 알면 됨<br>

- 대쉬 같은 기능은 Server에 보내주긴 해야 함<br>
  (그래도 UX를 위해 클라에서 먼저 연출)<br>
  (반응속도가 중요함)<br>

**2. ServerOnly와 ServerInitiated: 권위의 영역**

```cpp
UPurchaseAbility::UPurchaseAbility()
{
    // 정책 설정: 서버 전용
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::ServerOnly;
}

void UPurchaseAbility::ActivateAbility(...)
{
    // 클라이언트가 이 함수를 실행하려고 하면?
    if (!HasAuthority())
    {
        // 아무 일도 안 일어나거나, 로딩 UI만 띄우고 종료.
        // 실제 로직은 절대 실행 불가.
        ShowLoadingSpinner(); 
        return; 
    }

    // 서버 로직 (여기만 진짜)
    if (PlayerMoney >= Cost)
    {
        PlayerMoney -= Cost;
        GrantItem();
        // 결과만 클라이언트에게 통보 (RPC)
        ClientRPC_PurchaseSuccess(); 
    }
}
```

- 서버에서 진행하는 것이 좋은 경우에 대한 처리<br>
  (경제 관련된 매우 중요한 로직)<br>

- 시스템 & 보스 & 기믹 & 컷씬 등은 서버가 재생을 해주는 것이 좋음<br>

## 2-2. LocalPredicted 심화

**실제 구현 패턴**<br>

```cpp
void UMyMeleeAbility::ActivateAbility(...)
{
    // 예측 윈도우 열기
    FScopedPredictionWindow PredictionWindow(ASC, true);

    // 즉시 실행
    PlayMontage(AttackMontage);
    ExecuteGameplayCue("GameplayCue.Swing");
    PlaySound(SwingSound);

    // CommitAbility가 비용과 쿨다운 처리
    if (!CommitAbility(...))
    {
        EndAbility(...);
        return;
    }
}
```

**FScopedPredictionWindow 사용법**<br>

- `FScopedPredictionWindow(ASC, true)` - 새 키 생성<br>
- `FScopedPredictionWindow(ASC, ExistingKey)` - 기존 키 사용<br>

LocalPredicted 사용 시<br>
키 발급 후, 여러 스탯, 자원 처리 등을 처리하면 됨!<br>
- 어차피 실패시, 서버가 취소해버리므로<br>
  자동 복구<br>

```cpp
void MagicShow()
{
    // 1. 상자를 연다 (Scope 시작)
    { 
        FScopedPredictionWindow Box(ASC, true); // <--- Key: #101 발급
        
        DoMagic1(); // Key #101 태그 부착
        DoMagic2(); // Key #101 태그 부착
        
    } // <--- 2. 상자를 닫는다 (Scope 끝)
      // 자동으로 서버 전송: "Key #101로 Magic1, Magic2 했습니다. 확인 바람."
}
```

### 2-3. 서버 권위와 클라이언트 자유의 균형

```cpp
// 경쟁 게임 (서버 권위 90%)
UMyCompetitiveAbility::UMyCompetitiveAbility()
{
    NetExecutionPolicy = ServerOnly;
    bServerRespectsRemoteAbilityCancellation = false;
    bReplicateInputDirectly = false;
}

// 캐주얼 게임 (클라이언트 자유 70%)
UMyCasualAbility::UMyCasualAbility()
{
    NetExecutionPolicy = LocalPredicted;
    bServerRespectsRemoteAbilityCancellation = true;
    bReplicateInputDirectly = true;
}

// 배틀로얄 (하이브리드)
void UBattleRoyaleAbility::ActivateAbility(...)
{
    float DistanceToEnemy = GetDistanceToNearestEnemy();

    if (DistanceToEnemy < 1000.0f)  // 10미터 이내
    {
        SetNetExecutionPolicy(ServerOnly);  // 근접전: 서버 권위
    }
    else
    {
        SetNetExecutionPolicy(LocalPredicted);  // 원거리: 예측
    }
}
```

- ESport 게임에서 가장 중요한건 '공정성'<br>
  - 정말 중요한 판정이라면 서버 권한으로 처리<br>

- 하드코어한 경쟁, 스포츠, FPS 등이라면<br>
  서버에 권위를 주는 것이 더 공정함<br>
  (내가 했는데 왜 안되는거야?)<br>

- 캐주얼은 반대로 LocalPredict를 이용하여<br>
  UX를 극대화<br>
  (사소한 렉은 가볍게 넘기지만, 게임 자체가 느리면 불쾌)<br>

- 상황에 따라 양측을 번갈아가면서 써야하는 경우 또한 있음<br>
  - 근처에 아무도 없다면 LocalPredicted<br>
  - 전투 상황에 따라 서버에 권위를 주어 공정성 판별<br>
