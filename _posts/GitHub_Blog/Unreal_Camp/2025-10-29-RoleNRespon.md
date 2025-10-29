---
title: "김하연 튜터님 강의 - '역할과 책임 분리'"
date : "2025-10-29 12:00:00 +0900"
last_modified_at: "2025-10-29T12:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 역할과 책임
  - Framework
  - 설계
---

# 역할과 책임 분리에 대하여 알아보자

김하연 튜터님의 Notion 자료를 바탕으로 강의를 들으며<br>
수정 및 재작성한 블로깅용 글<br>
# 1. Framework 4대 클래스 🤕

### 문제 의식 - 구조를 깊게 생각하지 않은 일반적인 경우

```cpp
// 만능 플레이어 클래스
class ASuperPlayer : public APawn
{
    // 플레이어 개인 정보
    float Health = 100.0f;
    int32 Score = 0;

    float GameTimeRemaining = 300.0f;  // 이상하다...
    bool bGameStarted = false;          // 이것도...

    void UpdateHealthBar();             // UI가 왜 여기에?
    void CheckWinCondition();           // 게임 규칙도?
};
```

- 책임 혼재<br>
  : UI랑 게임 규칙에 대한 내용은 왜 있지?<br>
- 결합도 문제<br>
  : UI가 게임시간을 알고 싶을때 Player를 경유해야 함<br>

- 확장 불가능 (특히 멀티플레이)<br>
  : 서버 / 클라 구조에 부적합<br>
  (각 클라가 시간을 관리한다고..?)<br>

프로토타입/소규모 싱글 게임에선 괜찮을 순 있으나...<br>
잠재적으로 슈퍼 클래스/ 갓 클래스가 될 가능성이 높음<br>

### 언리얼의 해답은 관심사의 분리

```
🎭 GameMode     → 게임의 심판관 (규칙, 흐름 제어, 승부 판정)
📊 GameState    → 게임 상황판 (모든 플레이어가 보는 공통 정보)
👤 PlayerState  → 개인 수첩 (각 플레이어만의 정보)
🏢 GameInstance → 게임 관리자 (전체 생명주기, 레벨 간 데이터)
```

### 단일 책임 원칙

- **GameMode**: 오직 "결정"만
- **GameState**: 오직 "정보 저장"만
- **PlayerState**: 오직 "개인 데이터"만
- **GameInstance**: 오직 "전체 관리"만

# 2. GameMode - 게임 두뇌 🧠

### Base vs 일반 차이점

```cpp
GameModeBase: 기본 기능만 (가벼움)
- 단순한 게임 시작/종료
- 기본 생명주기만

GameMode: 완전한 매치 시스템 (무거움)
- EMatchState 매치 상태 관리
	  enum class EMatchState
    {
        EnteringMap,       // 맵 진입 중
        WaitingToStart,    // 시작 전 로비 단계
        InProgress,        // 실제 진행 중
        WaitingPostMatch,  // 종료 후 여운 (리플레이, 결과 등)
        LeavingMap         // 맵 전환 중
    };
- 자동 리스폰 시스템 (Pawn 재생성 및 Controller 재연결)
- 관전자 모드 지원 (멀티 플레이 관전 모드의 기반)
```

최소 기능 세트와 여러 기능 지원<br>

- EMatchState<br>
 : 게임 상태에 대한 StateMachine에 가까움<br>
   내부 상태 변화에 따른 이벤트 호출 발생<br>

가볍게 GameModeBase를 쓰는 곳도 많음<br>
(겜바겜)<br>

### 중요! 조합 규칙

```cpp
✅ GameModeBase ←→ GameStateBase
✅ GameMode     ←→ GameState
❌ GameModeBase ←→ GameState (컴파일 에러!)
❌ GameMode     ←→ GameStateBase (기능 누락!)
```

- Base라면 다른쪽도 Base로 맞춰주어야 함<br>
  (Base는 최소 기능만 존재하므로)<br>

- 게임 흐름 진행 : Mode<br>
  (룰북, 심판)<br>
  (서버)<br>

- 게임 흐름 기록 : State<br>
  (데이터 관리)<br>
  (클라로 복제)<br>

### 기본 구조

```cpp
class AMyGameMode : public AGameModeBase
{
public:
    // 게임 규칙 (개발자만 설정, 런타임 변경 불가)
    UPROPERTY(EditDefaultsOnly, Category = "Rules")
    float MatchDuration = 300.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Rules")
    int32 ScoreToWin = 1000;

    // 게임 흐름 제어
    UFUNCTION(BlueprintCallable)
    void StartMatch();

    UFUNCTION(BlueprintCallable)
    void EndMatch(bool bPlayerWon);

private:
    bool bMatchInProgress = false;
    FTimerHandle MatchTimer;
};
```

### 핵심 구현 패턴

```cpp
// GameMode의 가장 중요한 패턴
void AMyGameMode::StartMatch()
{
    // 1. GameMode는 "결정"을 내리고
    bMatchInProgress = true;

    // 2. GameState에게 "정보 업데이트"를 위임
    if (AMyGameState* GS = GetGameState<AMyGameState>())
    {
        GS->NotifyMatchStarted(MatchDuration);
    }

    // 3. 게임 종료 타이머 설정
    GetWorldTimerManager().SetTimer(MatchTimer, [this]() {
        EndMatch(false); // 시간 초과
    }, MatchDuration, false);
}

void AMyGameMode::EndMatch(bool bPlayerWon)
{
    bMatchInProgress = false;
    UE_LOG(LogTemp, Log, TEXT("매치 종료 - 결과: %s"), bPlayerWon ? TEXT("승리") : TEXT("패배"));

    // GameState에 결과 통보
    if (AMyGameState* GS = GetGameState<AMyGameState>())
    {
        GS->NotifyMatchEnded(bPlayerWon);
    }

    // 다음 단계로 진행 (예: GameInstance로 레벨 전환 요청)
    if (bPlayerWon)
    {
        if (UGameInstance* GI = GetGameInstance())
        {
            UE_LOG(LogTemp, Log, TEXT("GameInstance로 다음 레벨 요청"));
        }
    }
}

void AMyGameMode::CheckVictoryCondition()
{
    // 모든 PlayerState를 순회하며 점수 확인
    for (APlayerState* PS : GameState->PlayerArray)
    {
        AMyPlayerState* MyPS = Cast<AMyPlayerState>(PS);
        if (MyPS && MyPS->CurrentScore >= ScoreToWin)
        {
            EndMatch(true);
            return;
        }
    }
}
```

게임 모드는 세부적인 요소들은 '위임'만 해준다!<br>
게임 규칙 등만 직접 '판정'<br>

# 3. GameState - 정보의 중앙 저장소 📊

Mode가 심판이라면<br>
GameState는 '전광판'<br>

- GameMode는 서버에만 존재하기에<br>
  클라이언트는 그 결정과 과정에 대하여<br>
  GameState를 통해 복제받음<br>

- 리플리케이트?<br>
  (UPROPERTY 등으로 설정한 변수)<br>
  - 엔진이 해당 변수들을 '클라'쪽으로 복제할 변수란 것을 알게 됨<br>

### 핵심 철학

- **정보 제공자**: 은행 전광판처럼 정보만 표시
- **읽기 전용**: UI에서 참조만, 직접 수정 불가
- **GameMode 전용 업데이트**: 다른 곳에서 함부로 수정하면 안됨

## 🎯 GameState의 올바른 사용법은?

1. **GameMode가 상태를 계산해서 GameState에 알려줌.**
2. **GameState는 그 상태를 전 클라이언트에 자동으로 복제**
3. **UI나 애니메이션은 GameState를 보고 표시만 함.**

- 그렇기에 기본적으로 멀티게임에서<br>
  GameMode에서 GameState 값을 변경하고<br>
  클라이언트는 그것들을 읽기만 해야 함<br>

- Game 상태는 내부 로직을 통해 변경<br>


### 기본 구조

```cpp
// MyGameState.h
UCLASS()
class AMyGameState : public AGameStateBase
{
    GENERATED_BODY()

public:
    // 서버에서만 수정, 클라이언트는 읽기 전용
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_RemainingTime)
    float RemainingTime = 0.0f;

    // GameMode → GameState 통신용
    UFUNCTION()
    void NotifyMatchStarted(float Duration);

    // UI 친화 함수
    UFUNCTION(BlueprintPure)
    FString GetFormattedTime() const;

    UFUNCTION(BlueprintPure)
    FLinearColor GetTimeColor() const;

protected:
    // 서버에서 RemainingTime이 갱신될 때 클라에서 자동 호출됨
    UFUNCTION()
    void OnRep_RemainingTime();

private:
    float InitialDuration = 0.0f;
    FTimerHandle CountdownHandle;
};
```

- 클라는 OnRep_RemainingTime가 호출됨으로<br>
  상태 변경을 공지받음<br>

### 핵심 구현 패턴

```cpp
// 서버가 타이머를 관리
void AMyGameState::NotifyMatchStarted(float Duration)
{
    if (GetLocalRole() != ROLE_Authority) return; // 서버만 실행
    InitialDuration = Duration;
    RemainingTime   = Duration;

    UE_LOG(LogTemp, Log, TEXT("GameState: 매치 시작 (%.0f초)"), Duration);

    // 1초마다 RemainingTime 감소시키고 자동 복제
    GetWorldTimerManager().SetTimer(
        CountdownHandle,
        [this]()
        {
            RemainingTime = FMath::Max(RemainingTime - 1.f, 0.f);

            // 모든 클라가 동시에 업데이트됨 (Replication Trigger)
            if (RemainingTime <= 0.f)
            {
                GetWorldTimerManager().ClearTimer(CountdownHandle);
            }
        },
        1.0f, true
    );
}

void AMyGameState::OnRep_RemainingTime()
{
    UE_LOG(LogTemp, Verbose, TEXT("클라이언트: 남은 시간 갱신 %.0f"), RemainingTime);

    // UI 업데이트 신호를 던질 수도 있음
    OnTimeUpdated.Broadcast(RemainingTime);
}

// UI 친화적 시간 포맷팅
FString AMyGameState::GetFormattedTime() const
{
    int32 Minutes = RemainingTime / 60;
    int32 Seconds = (int32)RemainingTime % 60;
    return FString::Printf(TEXT("%02d:%02d"), Minutes, Seconds);
}

// 시간에 따른 색상 변화 (UI 활용)
FLinearColor AMyGameState::GetTimeColor() const
{
    float Ratio = RemainingTime / InitialDuration;
    if (Ratio > 0.5f) return FLinearColor::Green;   // 여유
    if (Ratio > 0.2f) return FLinearColor::Yellow;  // 주의
    return FLinearColor::Red;                       // 위험
}
```

- 클라쪽에서는 Delegate가 BroadCast한 것을 통해<br>
  자체 UI 업데이트 등을 진행<br>

- 데이터는 서버 -> 클라 (단방향)<br>

- 게임 서버/클라가 '같은 코드'를 사용하지만<br>
  `권한`을 체크하여<br>
  서버/클라 에서 도는지를 분리하여 로직 적용<br>

# 4. PlayerState - 개인 정보의 관리자 👤

### 생명주기 특징 - 레벨 전환 시

```
          Level_Menu → Level_Game1 → Level_Game2
              ↓             ↓            ↓
GameMode    [파괴]   → [새로 생성]  → [새로 생성]
GameState   [파괴]   → [새로 생성]  → [새로 생성]
PlayerState [유지]   → [유지]       → [유지]   ←  얘는 지속됨
```

플레이어 정보는 유지되기에<br>
플레이어 자체가 가져야 할 정보라면 PlayerState에 보존<br>

### 소유권 구조

```
GameMode (서버만 존재)
    ↓
PlayerController (각 유저마다 1개, 서버+클라 공존)
    ↓
PlayerState (서버+모든 클라 복제)
```

플레이어를 구분할 수 있기에<br>
(폰 변경 등에도 유지)<br>

PlayerController에 종속함<br>

### Controller와 PlayerState의 관계

```cpp
AMyPlayerState* PS = GetPlayerState<AMyPlayerState>();
```

```cpp
PlayerController  ↔  PlayerState
       ↓
      Pawn (게임 속의 몸체)
```

### 올바른 접근 방법

```cpp
// ❌ 잘못된 접근
AMyPlayerState* PS = GetWorld()->GetGameState()->PlayerArray[0];
```

```cpp
// ✅ 올바른 방식 – 클라이언트에서 서버에게 요청하기
MyPlayerState->Server_AddScore(50);
```

서버에서는 누가 0번인지를 모르는 위험한 접근!<br>

서버에서도 복제본이 존재함<br>

- 또한 PlayerState 수정시 서버에게 '요청'하여<br>
  바꾸어야 함<br>
  (기본적으로는 Server에서 바꾸는 것이 기본)<br>
  (클라에서의 요청을 굳이?)<br>

- Unreal은 기본적으로 '단방향'인 점을 다시 인지하자<br>

### 기본 구조

```cpp
// MyPlayerState.h
UCLASS()
class AMyPlayerState : public APlayerState
{
    GENERATED_BODY()

public:
    // 플레이어별 개인 데이터
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_Score)
    int32 CurrentScore = 0;

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_Lives)
    int32 Lives = 3;

    // 안전한 데이터 변경 (서버 전용 RPC)
    UFUNCTION(Server, Reliable)
    void Server_AddScore(int32 Points);

    UFUNCTION(Server, Reliable)
    void Server_TakeDamage(int32 Amount);

    // 상태 조회
    UFUNCTION(BlueprintPure)
    bool IsGameOver() const { return Lives <= 0; }

    // UI 업데이트용 Delegate
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnScoreChanged, int32, Old, int32, New);
    UPROPERTY(BlueprintAssignable)
    FOnScoreChanged OnScoreChanged;

protected:
    UFUNCTION()
    void OnRep_Score();

    UFUNCTION()
    void OnRep_Lives();

private:
    int32 HighScore = 0;
};

```

- 역시 Replicate를 설정하여<br>
  클라에게 복제되도록 설정<br>

### 핵심 구현 패턴

```cpp
// MyPlayerState.cpp
void AMyPlayerState::Server_AddScore_Implementation(int32 Points)
{
    if (Points <= 0) return;
    int32 Old = CurrentScore;
    CurrentScore += Points;

    if (CurrentScore > HighScore)
        HighScore = CurrentScore;

    // 점수 변경 브로드캐스트 (서버 → 클라)
    OnScoreChanged.Broadcast(Old, CurrentScore);

    // GameMode에 승리 조건 검사 요청
    if (AMyGameMode* GM = GetWorld()->GetAuthGameMode<AMyGameMode>())
    {
        GM->CheckVictoryCondition();
    }
}

void AMyPlayerState::OnRep_Score()
{
    // 클라이언트에서 자동 호출됨 (Replication 이벤트)
    UE_LOG(LogTemp, Verbose, TEXT("클라: 점수 갱신 %d"), CurrentScore);
    OnScoreChanged.Broadcast(CurrentScore, CurrentScore);
}

void AMyPlayerState::Server_TakeDamage_Implementation(int32 Amount)
{
    Lives = FMath::Max(Lives - Amount, 0);
    OnRep_Lives();
}

void AMyPlayerState::OnRep_Lives()
{
    UE_LOG(LogTemp, Verbose, TEXT("클라: 남은 목숨 %d"), Lives);
}
```

GameMode에게 보고<br>

GameMode - 룰<br>
PlayerState - 각 플레이어 들의 개인정보<br>

# 5. GameInstance - 게임 최고 관리자 🏢

### 절대적 특징

- **게임 전체 생명주기**: 시작부터 종료까지 살아있음
- **레벨 간 데이터 유지**: 레벨 바뀌어도 절대 안 사라짐
- **레벨 전환 권한**: 오직 GameInstance만 레벨 바꿀 수 있음

| 역할 | 생존 범위 | 책임 |
| --- | --- | --- |
| **GameMode** | 레벨 단위 | 규칙·승패·플로우 |
| **GameState** | 레벨 단위 | 현재 진행 데이터 |
| **PlayerState** | 세션 단위 | 개인 정보 |
| **GameInstance** | **게임 전체** | 데이터 영속 + 레벨 전환 제어 |

여기에 데이터를 넣으면 계속 유지됨<br>
레벨 전환도 제어 가능<br>

- GameMode는 결국 Level별로 교체가 가능함<br>
- GameState는 그 GameMode의 데이터를 관리<br>
- PlayerState는 각 Player들의 데이터<br>
- GameInstance는 게임 전체 데이터를 관리해야 함<br>
  (정말 영속적인 데이터를 저장해야 함)<br>
  (아니면 레벨/세션 관리 를 진행)<br>

### 기본 구조

```cpp
UCLASS()
class UMyGameInstance : public UGameInstance
{
public:
    virtual void Init() override;
    
    // 영속적 데이터 (레벨 바뀌어도 절대 안 사라짐)
    UPROPERTY(BlueprintReadWrite)
    FString PlayerName = TEXT("Player");
    
    UPROPERTY(BlueprintReadWrite)
    int32 TotalScore = 0;           // 전체 누적 점수
    
    UPROPERTY(BlueprintReadWrite)
    int32 CompletedLevels = 0;
    
    // 레벨 전환 중앙 관리
    UFUNCTION(BlueprintCallable)
    void LoadGameLevel(int32 LevelIndex);
    
    UFUNCTION(BlueprintCallable)
    void LoadNextLevel();
    
    // 게임 진행 상황 통합
    UFUNCTION(BlueprintCallable)
    void ReportLevelCompleted(int32 LevelIndex, int32 Score);

private:
    int32 CurrentLevelIndex = 0;
    bool bIsChangingLevel = false;
};
```

지금까지의 결과물을 레벨에 상관없이 관리하는 용도<br>

### GameInstance의 스마트한 레벨 전환

```cpp
// MyGameInstance.cpp - 레벨 전환의 핵심
void UMyGameInstance::LoadGameLevel(int32 LevelIndex)
{
    if (bIsChangingLevel) return; // 중복 방지
    
    UE_LOG(LogTemp, Warning, TEXT("레벨 %d 로딩 시작"), LevelIndex);
    
    bIsChangingLevel = true;
    CurrentLevelIndex = LevelIndex;
    
    // 현재 상태 백업 (중요!)
    if (APlayerController* PC = GetWorld()->GetFirstPlayerController())
    {
        if (AMyPlayerState* PS = PC->GetPlayerState<AMyPlayerState>())
        {
            TotalScore += PS->GetCurrentScore(); // 백업
        }
    }
    
    // 실제 레벨 로딩
    FString LevelName = FString::Printf(TEXT("Level_%d"), LevelIndex);
    UGameplayStatics::OpenLevel(GetWorld(), FName(*LevelName));
}

void UMyGameInstance::ReportLevelCompleted(int32 LevelIndex, int32 Score)
{
    UE_LOG(LogTemp, Warning, TEXT("레벨 %d 완료! 점수: %d"), LevelIndex, Score);
    
    TotalScore += Score;
    CompletedLevels = FMath::Max(CompletedLevels, LevelIndex);
    
    // 2초 후 다음 레벨로
    FTimerHandle Timer;
    GetWorldTimerManager().SetTimer(Timer, [this]() {
        LoadNextLevel();
    }, 2.0f, false);
}

void UMyGameInstance::LoadNextLevel()
{
    // 그냥 다음 인덱스 기반으로 호출만 위임
    LoadGameLevel(CurrentLevelIndex + 1);
}
```

- 레벨을 직접 바꿀 수 있으므로<br>
  로딩 등도 이쪽에서 고려<br>

# 6. Framework 로딩 순서 (중요!) ⚠️

## 6-1. 게임 시작부터 레벨 로딩까지 순서

```
🎮 엔진 부팅 단계
1. UEngine 초기화
2. UGameInstance 생성 → Init() 호출 ⭐ 가장 먼저!
   - 이 시점: World 없음 / GameMode 없음
   - 가능: 글로벌 설정, 서브시스템 초기화, SaveGame 로드
   - 불가: GetWorld(), UI 생성, GameMode 접근

🌍 첫 레벨 로딩 단계
3. World 생성 (Persistent Level 포함)
4. GameModeBase 생성 → InitGame() 호출
5. GameStateBase 생성 (GameMode 내부에서 스폰)
6. PlayerController 생성
7. PlayerState 생성 (Controller 소유)
8. Pawn 생성 → Controller 가 Possess()

9. 모든 BeginPlay() 호출 (순서 보장 ❌)
   - 대상: GameMode, GameState, PlayerController,
            PlayerState, Pawn, 기타 모든 Actor
   - ⚠ 의존성 있는 접근은 주의!
```

- 로딩 순서가 아주 중요하다<br>
  (초기화 순서를 알아야 Nullptr 등을 피할 수 있음)<br>
  (이거 아직 생성 안되었네?)<br>

- BeginPlay 시점에서 '다른 무언가'를 참조하는 경우<br>
  '아직 없거나' '설정'이 제대로 되지 않을 수 있음<br>

- Post 계열의 함수를 써야 안전함<br>

- Beginplay에 주의할 것!<br>

## **6-2. 레벨 전환 시의 특별한 메커니즘**

```cpp
=== 현재 레벨 정리 단계 ===
1. 모든 Actor의 EndPlay() 호출
2. GameMode / GameState 파괴
3. 일반 Actor 파괴
4. PlayerState 데이터 보존 ⭐ (실제 복사 이주)

=== 새 레벨 로딩 단계 ===
5. 새 World 생성
6. PlayerState 데이터가 새 인스턴스로 복사 ⭐ (CopyProperties)
7. 새 GameMode / GameState 생성
8. PlayerController 가 새 PlayerState에 재연결
9. 새 Pawn 생성
10. 모든 BeginPlay() 호출
```

전부 파괴를 하고<br>
새로운 레벨에 맞게 재생성<br>

- 참고: PlayerState는 완전히 ‘같은 객체’가 이주하는 게 아니라,<br>
  이전 World의 데이터를 새 PlayerState로 복사 (CopyProperties) 하는 방식<br>
  따라서 끊김 없이 이어지는 것처럼 보이는 새 객체<br>

- PlayerState 복사 과정 시점에서 위험할 수는 있음<br>
  OpenLevel 직후, PlayerState를 참조하면 Null 참조 하지 말 것!<br>
  (BeginPlay이후에 접근해야 안전)<br>
  (따라서 정 그 타이밍을 원한다면 GameInstance에 필요한 데이터를 넣을 것)<br>

- OpenLevel 이후의 로직은 실행이 불분명하므로 마지막에 호출할 것<br>
  (정보 저장 등은 그 이전에 호출하자)<br>

## 6-3. 각 단계에서 할 수 있는 것/없는 것

### GameInstance::Init()

```cpp
// ✅ 가능
- 글로벌 설정 로드
- 서브시스템 초기화
- SaveGame 데이터 로드

// ❌ 금지
- GetWorld() 호출 (아직 World 없음)
- GameMode / PlayerController 접근
- UI 생성
```

### GameMode 생성자

```cpp
// ✅ 가능
- 기본 클래스 설정
- 게임 룰 변수 초기화

// ❌ 금지
- GameState 접근 (아직 없음)
- PlayerController 접근
```

- 생성자에 접근 코드를 함부로 넣지 말것!<br>

### GameMode::PostInitializeComponents()

```cpp
// ✅ 이 시점부터 GameState 접근 가능
// ⚠ PlayerController / PlayerState는 아직 안 생김
```

다만 PlayerController 등에 대한 로직은 집어넣지 말것...<br>

## 시점 별 안전성 표

| 타이밍 | ✅ 가능한 작업 | ❌ 금지 사항 |
| --- | --- | --- |
| `GameInstance::Init()` | 글로벌 설정, SaveGame, 네트워크 초기화 | `GetWorld()`, GameMode 접근 |
| `GameMode 생성자` | 클래스 설정, 룰 초기화 | GameState, PlayerController 접근 |
| `PostInitializeComponents()` | GameState 접근 가능 | PlayerController, PlayerState 접근 |
| `BeginPlay()` | 게임 로직 시작(⚠ 순서 랜덤) | 의존 객체 접근 금지 |
| `PostLogin()` | ✅ 모든 플레이어 데이터 확실 준비 | 없음 (가장 안전) |

- Init에 대한 접근은 항상 신중히<br>

### 생명주기 함수 안전성

| 위험도 | 함수 | 설명 |
| --- | --- | --- |
| 🔴 높음 | 생성자 | 의존성 객체 아직 없음 |
| 🟠 중간 | PostInitializeComponents | GameState 정도는 존재 |
| ⚠ 주의 | BeginPlay | 호출 순서 비보장 → 의존성 접근 주의 |
| 🟢 안전 | PostLogin | PlayerController / PlayerState 모두 준비됨 |
| 🟢 안전 | StartPlay | 모든 BeginPlay 이후 완전 준비 상태 |

<주의사항>

- PlayerState는 “복사 이주(CopyProperties)” 과정 동안 일시적으로 접근 불가.<br>
- GameMode / GameState는 항상 레벨 단위로 다시 생성됨.<br>
- GameInstance만이 세션 전체 연속성 보장.<br>
- 레벨 전환 중간에 PlayerState나 World 데이터에 접근하면 크래시 위험.<br>

## 7. 언리얼 설계 사고법 - 왜 이렇게 구조가 되어 있을까? 🙄

`게임은 '상태'의 집약이라는 철학`을 가진 엔진<br>
그렇기에 프레임워크들이 그렇게 맞추어져 있음<br>

관리 및 유지보수, 협업을 고려한 설계<br>
(상태 관리만 잘된다면)<br>

## 설계 판단 기준 - 데이터 위치 결정 3요소

1. **생명주기**: "언제까지 살아있어야 하는가?"<br>
    - 레벨 끝까지: GameMode/GameState<br>
    - 게임 끝까지: PlayerState/GameInstance<br>
2. **수정 권한**: "누가 바꿀 수 있는가?"<br>
    - 게임 시스템: GameMode가 관리<br>
    - 플레이어 개인: PlayerState가 관리<br>
3. **참조 대상**: "누가 봐야 하는가?"<br>
    - UI 표시: GameState/PlayerState<br>
    - 내부 로직: private 멤버<br>


## 🎯 기억할 것 - 싱글→멀티 전환의 마법 (올바른 구조의 힘)

```cpp
// 기존 싱글플레이어 코드
void AMyPlayerState::AddScore(int32 Points)
{
    Score += Points;
    OnScoreChanged.Broadcast(Score);
}

// 멀티플레이어 - 한 줄만 추가!
void AMyPlayerState::AddScore(int32 Points)
{
    if (HasAuthority()) // 이 줄만 추가!
    {
        Score += Points;
        OnScoreChanged.Broadcast(Score);
    }
}
```

- “싱글플레이어라서 대충”이 아니라, “싱글플레이어부터 제대로"<br>

- 변화에 강한 구조<br>