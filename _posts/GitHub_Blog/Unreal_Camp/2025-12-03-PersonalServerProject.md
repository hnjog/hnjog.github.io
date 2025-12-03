---
title: "Unreal/C++ 개인 과제 - 경찰과 도둑"
date : "2025-12-03 20:00:00 +0900"
last_modified_at: "2025-12-03T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
---

## Unreal/C++ 개인 과제 - 경찰과 도둑

[Git Page](https://github.com/hnjog/Sparta_Task_Personal)<br>

### 과제 소개 및 목표

이번 프로젝트는 '개인 프로젝트'로서<br>
이전 챕터의 복습을 위한 과제이다<br>

- 개인 프로젝트 주제<br>
    - CH03 챕터에서 배운 언리얼 게임플레이 프레임워크 클래스를 활용하고,<br>
    CH04 멀티플레이가 되게끔 로직을 작성합니다.<br>
    - 레벨에 선량한 시민 AI NPC들이 돌아다닙니다.<br>
    경찰과 도둑, 시민 AI NPC는 모두 같은 모습을 하고 돌아다닙니다.<br>
    - 경찰은 도둑일거 같은 캐릭터를 때려잡고,<br>
    도둑은 계속해서 도망다니거나 시민 AI NPC 사이에 숨어다닙니다.<br>
    - 제한 시간 내에 도둑을 잡지 못하면 도둑 승리. 잡으면 경찰 승리입니다.<br>

- 개인 프로젝트 이유<br>
    - 프로젝트 준비부터 프로젝트 마무리까지 혼자서 경험해보기 위함.<br>
    - CH04 내용을 복습하기 위함.<br>

## 구현 기능

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/j35FsiR8ONQ"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

구현요소<br>
- 이동 동기화<br>
- Match State 동기화<br>
- 공격/피격 동기화<br>
- 게임 결과 동기화<br>
- 아이템 생성 및 효과 적용 동기화<br>

---

### 클래스 설명

```less
TagNChase
- 로깅용 매크로를 넣어둔 파일

Animation
- BaseAnimInst : NPC가 사용하기 위하여 부모용 AnimInstance를 분리
- TaskAnimInstance : Player가 사용하는 AnimInstance (AimOffset)

Character
- BaseCharacter 
 : Player와 NPC가 같이 사용하기 위한 부모 캐릭터 클래스 (ACharacter)
   TakeDamage, OnDeath(Pure), StatusComponent를 가짐

- TaskCharacter
  : Player 용의 캐릭터 클래스, 입력 처리, 공격 등에 대한 처리 포함

- AICharacter
  : Ragdoll 용 함수가 포함된 NPC 클래스

UI
- UTNHPTextWidgetComponent 
  : 월드에 위젯을 띄우기 위한 클래스 
   (SetOnlyOwnerSee(true) / SetOwnerNoSee(false)) 설정을 통해
   자기 자신만이 볼 수 있도록 수정

- UTNStatusComponent
  : 체력 , 역할 등에 대한 데이터가 담긴 컴포넌트 클래스
    (변경에 따른 Delegate 포함)

Controller
- TaskTitlePlayerController : Title Level에서 활용하는 컨트롤러
- TaskPlayerController : UI 와 연동하는 기능을 추가한 PlayerController 클래스
- NPCController : AIController를 상속받은 클래스 (기초적인 BT 등에 대한 세팅 용도)

GameMode
- ATaskGameModeBase
  : 로그인 및 매치 흐름에 대한 관리용 게임모드 클래스
    로그인 한 플레이어들을 모으고, 역할 분리 및 게임 흐름 진행
    (다만, 게임 흐름 부분은 별도의 매니저를 두어도 좋았을 듯)

GameState
- ATaskGameStateBase
  : 매치 상태와 매치 시간을 가진 GameState 클래스

Item
- UItemSpawnSubsystem : 아이템을 Spawn 시키는 용도의 WorldSubsystem 클래스
- AItemSpawnVolume : 아이템을 Spawn 시킬 임의의 위치를 찾는 볼륨용 액터 클래스
- ATaskItem 
: 아이템 클래스 
  Player와 접촉 시, 사라지고 Role 에 따른 효과 부여
  (다만, 별도의 효과 클래스를 구현하지 않아 고정된 효과만 적용)

```

### 트러블 슈팅 1 - Enemy가 사망하였으니 Ragdoll이 발생하지 않는 상황

```cpp
void AAICharacter::OnDeath()
{
    if (AAIController* AICon = Cast<AAIController>(GetController()))
    {
        AICon->StopMovement();

        if (UBrainComponent* Brain = AICon->GetBrainComponent())
        {
            Brain->StopLogic(TEXT("Dead"));
        }
    }

    if (UCharacterMovementComponent* MoveComp = GetCharacterMovement())
    {
        MoveComp->DisableMovement();
    }

    USkeletalMeshComponent* MeshComp = GetMesh();
    if (MeshComp)
    {
        GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);

        MeshComp->SetCollisionProfileName(TEXT("Ragdoll"));
        MeshComp->SetSimulatePhysics(true);
        MeshComp->SetAllBodiesSimulatePhysics(true);
        MeshComp->WakeAllRigidBodies();
    }

    DetachFromControllerPendingDestroy();

    SetLifeSpan(10.0f);

	ATaskGameModeBase* GameMode = Cast<ATaskGameModeBase>(UGameplayStatics::GetGameMode(this));
	if (HasAuthority() == true && IsValid(GameMode) == true)
	{
		GameMode->PaneltyPolice();
	}
}
```

- NPC가 사용하는 코드로서<br>
  경찰이 NPC를 공격할 시<br>
  대미지를 받고 사망하여 호출되는 코드이다<br>

- 다만, '패널티'를 통한 경찰의 체력감소는 적용되었으나<br>
  클라이언트에선 lagdoll 이 발생하지 않는 현상이 있었다<br>
  - '동기화' 문제!<br>
  - 서버에선 처리되었으나, 클라이언트에선 처리되지 않은 문제!<br>

해결방법<br>

```cpp
// AICharacter.h

protected:
	UPROPERTY(ReplicatedUsing = OnRep_IsDead)
	uint8 bIsDead : 1;

	UFUNCTION()
	void OnRep_IsDead();

// cpp
void AAICharacter::OnDeath()
{
    if (!HasAuthority())
        return;

    if (bIsDead)
        return;

    bIsDead = true;

    // 래그돌 관련 코드
    StartRagdoll();

    ATaskGameModeBase* TGM = Cast<ATaskGameModeBase>(UGameplayStatics::GetGameMode(this));
    if (IsValid(TGM))
    {
        TGM->PaneltyPolice();
    }
}

void AAICharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    DOREPLIFETIME(ThisClass, bIsDead);
}

void AAICharacter::OnRep_IsDead()
{
    if (bIsDead)
    {
        StartRagdoll();
    }
}
```

- 죽음과 관련된 상태를<br>
  저장하고, 해당 상태 변화에 따라<br>
  OnRep 를 호출하여<br>
  '클라이언트'에도 죽음의 상황을 공유하여 해결!<br>

### 트러블 슈팅 2 - 설정한 모자가 보이지 않는 상황

```cpp
//TaskCharacter.h

#pragma region Hats
	void ApplyRoleHat(ERoleType InRole);

protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Role|Hat")
	TObjectPtr<UStaticMeshComponent> RoleHatMesh;

	UPROPERTY(EditDefaultsOnly, Category = "Role|Hat")
	TObjectPtr<UStaticMesh> PoliceHatMesh;

	UPROPERTY(EditDefaultsOnly, Category = "Role|Hat")
	TObjectPtr<UStaticMesh> ThiefHatMesh;
#pragma endregion

//cpp

// BeginPlay
StatusComponent->OnRoleChanged.AddUObject(this, &ThisClass::ApplyRoleHat);

```

- Role 에 따라 보여질 모자를 '다르게' 수정하였으나<br>
  모자 자체가 보이지 않는 상황!<br>

- 분명 BroadCast를 통해 Role 변화시 동적으로 호출되게 하였는데??<br>
  -> Broadcast는 '해당 장치' 내에서만 Broadcast 됨!<br>
  - 어찌보면 당연하지만, BroadCast 되며 서버->클라 연동이 될것이라 생각...<br>
    (Net Multicast 와 헷갈린듯...??)<br>

- 그렇기에 Status의 BroadCast를 클라에서도 적용되도록 수정하였다

```cpp
// StatusComponent.h

UFUNCTION()
void OnRep_Role();

...

UPROPERTY(EditAnywhere, BlueprintReadOnly, ReplicatedUsing = OnRep_Role)
ERoleType Role;

// cpp
void UTNStatusComponent::OnRep_Role()
{
	OnRoleChanged.Broadcast(Role);
}
```

- 이를 통해<br>
  각 클라이언트에서 BroadCast를 호출하여<br>
  모자에 대한 Mesh 설정을 제대로 할 수 있었다!<br>

### 트러블 슈팅 3 - 스폰한 아이템이 보이지 않는 상황

- TaskItem를 SpawnActor 로 생성하였으나<br>
  보이지 않는 상황이었다<br>

- 처음에는 '생성'에 실패하였나?<br>
  싶었으나 로그를 찍어보니 '서버'에서는 생성에 성공하였다<br>
  - 그렇다... 또 클라이언트 동기화 문제였다<br>

```cpp
ATaskItem::ATaskItem()
{
...
	bReplicates = true;
	bAlwaysRelevant = true;
}
```

- Replicate 를 true로 설정하며<br>
  또한, 연관성이 있도록 설정<br>
  - 아이템은 어떤 플레이어도 섭취할 수 있으므로<br>


- 아이템이 잘 보이게 되었다!<br>

[![Image](https://github.com/user-attachments/assets/d2a09f28-864c-415c-984e-3101e9b970fc)](https://github.com/user-attachments/assets/d2a09f28-864c-415c-984e-3101e9b970fc){: .image-popup}<br>


- 모자와 마찬가지로<br>
  주어지는 역할에 따라<br>
  메시와 효과적용을 다르게 하였다<br>
