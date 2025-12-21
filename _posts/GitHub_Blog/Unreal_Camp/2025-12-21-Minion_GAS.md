---
title: "팀 작업 - 미니언(AI) GAS"
date : "2025-12-21 18:00:00 +0900"
last_modified_at: "2025-12-21T18:00:00"
categories:
  - Unreal
  - C++
  - 내일배움캠프
tags:
  - Unreal
  - C++
  - GAS
---

## Minion GAS 작업에 대하여

GAS의 구성 요소에 따라<br>
전반적으로 진행하되<br>

서로 유기적으로 혼용되는 부분이 많았기에<br>
정리를 한번 해보려 한다<br>

### ASC, Charcter AttributeSet

```cpp

// NpcBaseCharacter.h
UCLASS()
class PARAGONIA_API ANpcBaseCharacter : public ACharacter, public IAbilitySystemInterface
{
	GENERATED_BODY()

public:
	virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

...

protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "GAS")
	TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "GAS")
	TObjectPtr<UCharacterAttributeSet> AttributeSet;

};
```

- `IAbilitySystemInterface` & `GetAbilitySystemComponent`<br>
  : GetAbilitySystemComponent 는 IAbilitySystemInterface가<br>
    구현을 요구하는 순수 가상 함수이다<br>
    (ASC를 본격적으로 이용하기 위하여, 해당 인터페이스와 함수를 구현)<br>

- 현재 이 캐릭터는 'AI'용으로 만들 예정이기에<br>
  Character에 ASC를 넣었음<br>

- Player라면 PlayerState에 넣는 방식도 존재<br>
  - 이 경우에도 플레이어용 캐릭터과 위 요소들을 구현하되<br>
    GetAbilitySystemComponent 가 PlayerState의 ASC를 반환하는 방식을 취할 수 있음<br>
    (ASC를 원하는 곳에서 대리하여 가져올 수 있다는 점!)<br>

- 이번엔 AI 작업을 진행하지만<br>
  Player 같은 조작의 경우 'Input'에 대한 태그 형태로 ASC에<br>
  전달 확인이 가능하다 한다<br>

- 별도의 동기화 코드를 작성하지 않더라도<br>
  몇가지 세팅만으로 클라<->서버의 동기화를 대신 처리<br>
  (SetReplicationMode,SetIsReplicated 등)<br>

```cpp
#pragma once

#include "CoreMinimal.h"
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"
#include "CharacterAttributeSet.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FAttributeDataChanged, float, OldValue, float, NewValue);

UCLASS()
class PARAGONIA_API UCharacterAttributeSet : public UAttributeSet
{
	GENERATED_BODY()
	
public:
	UCharacterAttributeSet();

	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, MaxHealth);
	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, Health);
	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, Defense);
	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, AttackPower);
	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, MoveSpeed);
	ATTRIBUTE_ACCESSORS_BASIC(ThisClass, Damaged);

	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;

	virtual void PostAttributeChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue) override;

	virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data) override;

	virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

	UPROPERTY(BlueprintAssignable, Category = "Attribute")
	mutable FAttributeDataChanged OnMaxHealthChanged;

	UPROPERTY(BlueprintAssignable, Category = "Attribute")
	mutable FAttributeDataChanged OnHealthChanged;

	UPROPERTY(BlueprintAssignable, Category = "Attribute")
	mutable FAttributeDataChanged OnMaxHealthChanged_UI;

	UPROPERTY(BlueprintAssignable, Category = "Attribute")
	mutable FAttributeDataChanged OnHealthChanged_UI;

private:
	UFUNCTION()
	void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth);

	UFUNCTION()
	void OnRep_Health(const FGameplayAttributeData& OldHealth);

	UFUNCTION()
	void OnRep_Defense(const FGameplayAttributeData& OldDefense);

	UFUNCTION()
	void OnRep_AttackPower(const FGameplayAttributeData& OldAttackPower);

	UFUNCTION()
	void OnRep_MoveSpeed(const FGameplayAttributeData& OldMoveSpeed);

public:
	UPROPERTY(BlueprintReadOnly, Category = "Attribute", ReplicatedUsing = OnRep_MaxHealth)
	FGameplayAttributeData MaxHealth;

	UPROPERTY(BlueprintReadOnly, Category = "Attribute", ReplicatedUsing = OnRep_Health)
	FGameplayAttributeData Health;

	UPROPERTY(BlueprintReadOnly, Category = "Attribute", ReplicatedUsing = OnRep_Defense)
	FGameplayAttributeData Defense;

	UPROPERTY(BlueprintReadOnly, Category = "Attribute", ReplicatedUsing = OnRep_AttackPower)
	FGameplayAttributeData AttackPower;

	UPROPERTY(BlueprintReadOnly, Category = "Attribute", ReplicatedUsing = OnRep_MoveSpeed)
	FGameplayAttributeData MoveSpeed;

	UPROPERTY(BlueprintReadOnly)
	FGameplayAttributeData Damaged;
};
```

- 동료분이 만들어주신 '캐릭터' 전반에 대한 AttributeSet<br>
  - 캐릭터 전반에 대한 'Stat' 정의<br>

- `ATTRIBUTE_ACCESSORS_BASIC`<br>
  : 스탯에 관련된 FGameplayAttributeData 에 대한 Getter와 Setter, 초기화 함수 등을 만들어주는 매크로 함수<br>

- pre,post 등을 통해 스탯이 변하기 전/후 에 확인 및 보정 가능<br>
  - pre : 적용 전에 미리 값 보정하기 (ex : 1회 대미지 최댓값은 100으로!)<br>
  - post : 값이 변한후, 상태 변화 등 (ex : 체력이 0이하네? 사망 처리!)<br>

- `FGameplayAttributeData`를 이용한 능력치 관리<br>
  - *BaseValue*(기본값) : 캐릭터의 '영구적' 능력치<br>
  - *CurrentValue*(현재값) : 버프,디버프 등의 다양한 효과가 합산된 '최종값'<br>

- `GameplayEffectExecutionCalculation`? <br>
  - Execute_Implementation 함수를 구현함으로서<br>
    AttributeData를 '계산'하는 방식을 조정 가능<br>
    (아래 코드는 방어력에 따른 대미지 감소 예시)<br>

```cpp
void UExecCalc_Damage::Execute_Implementation(
    const FGameplayEffectCustomExecutionParameters& ExecutionParams,
    FGameplayEffectCustomExecutionOutput& OutExecutionOutput
) const
{
    const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();

    FAggregatorEvaluateParameters EvaluationParams;

    float AttackPower = 0.0f;
    ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(
        GetDamageCapture().AttackPowerDef,
        EvaluationParams,
        AttackPower
    );

	float Defense = 0.0f;
    ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(
        GetDamageCapture().DefenseDef,
        EvaluationParams,
        Defense
	);

	const float AbilityBaseDamage = Spec.GetSetByCallerMagnitude(TAG_Data_Damage_Base, false, 0.0f);
    const float AttackPowerMultiplier = Spec.GetSetByCallerMagnitude(TAG_Data_Damage_Multiplier, false, 0.0f);
	const float RawDamage = FMath::Max(0.0f, AbilityBaseDamage + (AttackPower * AttackPowerMultiplier));

	const float SafeDefense = FMath::Max(0.0f, Defense);
	const float DamageMultiplier = K / (K + SafeDefense);

	const float FinalDamage = RawDamage * DamageMultiplier;

    OutExecutionOutput.AddOutputModifier(FGameplayModifierEvaluatedData(UCharacterAttributeSet::GetDamagedAttribute(), EGameplayModOp::Additive, FinalDamage));
}
```


### Gameplay Ability

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GA/PGGameplayAbilityBase.h"
#include "Struct/FAttackData.h"
#include "GA_NpcAttackBase.generated.h"

/**
 * 
 */
UCLASS(Abstract)
class PARAGONIA_API UGA_NpcAttackBase : public UPGGameplayAbilityBase
{
	GENERATED_BODY()

public:
	UGA_NpcAttackBase();

	virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;

protected:

	UFUNCTION()
	void OnMontageCompleted();

	UFUNCTION()
	void OnMontageInterrupted();

	UFUNCTION()
	void OnMontageCancelled();

	UFUNCTION()
	virtual void OnAttackEventReceived(FGameplayEventData Payload);

	FGameplayAbilityTargetDataHandle MakeTargetDataHandleFromActor(AActor* TargetActor);

protected:
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Attack")
	FAttackData AttackData;
};

// cpp

// Fill out your copyright notice in the Description page of Project Settings.


#include "GA/Npc/GA_NpcAttackBase.h"
#include "AbilitySystemComponent.h"
#include "AbilitySystemBlueprintLibrary.h"
#include "Abilities/Tasks/AbilityTask_PlayMontageAndWait.h"
#include "Abilities/Tasks/AbilityTask_WaitGameplayEvent.h"
#include "Abilities/GameplayAbilityTargetTypes.h"

UGA_NpcAttackBase::UGA_NpcAttackBase()
{
	InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
	// 서버에서만 돌게 하도록 (AI)
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::ServerOnly;
}

void UGA_NpcAttackBase::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (CommitAbility(Handle, ActorInfo, ActivationInfo) == false)
	{
		EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		return;
	}

	if (IsValid(AttackData.Montage) == false)
	{
		EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		return;
	}

	UAbilityTask_PlayMontageAndWait* MontageTask = 
		UAbilityTask_PlayMontageAndWait::CreatePlayMontageAndWaitProxy(
		this, NAME_None, AttackData.Montage
	);

	if (IsValid(MontageTask))
	{
		MontageTask->OnCompleted.AddDynamic(this, &UGA_NpcAttackBase::OnMontageCompleted);
		MontageTask->OnInterrupted.AddDynamic(this, &UGA_NpcAttackBase::OnMontageInterrupted);
		MontageTask->OnCancelled.AddDynamic(this, &UGA_NpcAttackBase::OnMontageCancelled);
		MontageTask->ReadyForActivation();
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("UGA_NpcAttackBase::ActivateAbility - Failed to create Ability Task"));
		EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		return;
	}

	UAbilityTask_WaitGameplayEvent* WaitEventTask = 
		UAbilityTask_WaitGameplayEvent::WaitGameplayEvent(
		this,
		FGameplayTag::RequestGameplayTag(FName("Event.Npc.HitResult")),
		nullptr, false, false
	);

	if (IsValid(WaitEventTask))
	{
		WaitEventTask->EventReceived.AddDynamic(this, &UGA_NpcAttackBase::OnAttackEventReceived);
		WaitEventTask->ReadyForActivation();
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("UGA_NpcAttackBase::ActivateAbility - Failed to create HitResult Ability Task"));
		EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
	}
}

void UGA_NpcAttackBase::OnMontageCompleted()
{
	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, false, false);
}

void UGA_NpcAttackBase::OnMontageInterrupted()
{
	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, false, true);
}

void UGA_NpcAttackBase::OnMontageCancelled()
{
	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, false, true);
}

void UGA_NpcAttackBase::OnAttackEventReceived(FGameplayEventData Payload)
{
	// Melee,Range 에서 override
}

FGameplayAbilityTargetDataHandle UGA_NpcAttackBase::MakeTargetDataHandleFromActor(AActor* TargetActor)
{
	if(TargetActor == nullptr)
		return FGameplayAbilityTargetDataHandle();

	FGameplayAbilityTargetData_SingleTargetHit* NewData = new FGameplayAbilityTargetData_SingleTargetHit();
	NewData->HitResult.HitObjectHandle = FActorInstanceHandle(TargetActor);
	NewData->HitResult.bBlockingHit = true;

	FGameplayAbilityTargetDataHandle Handle;
	Handle.Add(NewData);

	return Handle;
}

```


- `ActivateAbility` 자체는<br>
  GA의 '진입점'에 해당하는 역할이다<br>
  GA가 작동하는 경우 이 함수로 들어오게 됨<br>

- EndAbility 는<br>
  GA가 '끝났다' 것을 알려주는 함수로<br>
  해당 함수가 호출되지 않으면 게임 어빌리티가 '종료'되지 않은것으로 판단되어<br>
  차후 호출 시, 제대로 동작하지 않을 수 있으므로 주의!<br>

- 현재는 초기 Target을 저장하는 방식을 채용<br>
  - 원래는 `OnAttackEventReceived` 처럼 'Animation'의 특정한 montarge에서<br>
    돌려준 값을 기반으로 목표를 찾거나 trace를 돌리는 방식이 일반적이다<br>
  - 지금은 미니언의 '평타'가 빗나가는 일이 없도록 진행하기 위하여<br>
    해당 방식으로 진행<br>

- `FGameplayEventData`<br>
  - 해당 이벤트 데이터에 EventTag(어떤 이벤트), Instigator(나), Target(맞은 놈), Magnitude(세기) 등의 정보를 담을 수 있음<br>
    (일종의 택배 상자)<br>
  - 더 세부적으로 진행하는 경우, '판정'을 별도로 진행하며<br>
    GA가 return 받는 payload에는 그 결과이며, 그것에 따라 수치 조정만 하는 것도 가능<br>
  - TargetData와 연동하여 Location,Normal 같은 상세 정보를 담을 수 있음<br>

- `FGameplayAbilityTargetDataHandle`<br>
  - Target을 다루는 표준 데이터 (위 구조체의 TargetData도 이 형식)<br>
  - 다형성을 기반으로 한 추가적인 구조체 존재<br>
  - 이 구조체를 통해 서버<->클라이언트의 타겟 정보를 효율적으로 전송 가능<br>

- NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::ServerOnly;<br>
  - AI의 공격은 '서버'에서만 진행될 필요가 있음<br>
    (굳이 클라에서 Minion의 공격을 일일이 계산하여 서버로 전송할 필요가 없음!)<br>
  - 반대로 플레이어의 GA는 'LocalPredicted' 옵션으로 설정하여<br>
    클라에서 '미리' 반응하고, 서버에서 온 결과에 따라 취소하는 등의 방식으로 진행<br>

### Gameplay Effect

[![Image](https://github.com/user-attachments/assets/185ba0dd-def0-4af9-a74d-d12524977c75)](https://github.com/user-attachments/assets/185ba0dd-def0-4af9-a74d-d12524977c75){: .image-popup}<br>

- GameplayEffect는 전반적인 'Attibute'와 상태 변화를 정의하는 **데이터 에셋**<br>

- 구체적인 '로직'보단 '데이터'에 가까우며<br>
  이를 ASC가 '해석'하여 적용함<br>
  -> '데이터 기반 설계'<br>

- 지속성(Duration) 정책<br>
  - 우리의 'NPCDamage' 용도의 GE는 '즉발성'이기에<br>
    Instant로 설정<br>

- Modifier를 추가하는 방식도 존재하나<br>
  위에서 동료가 만든 ExecCalc_Damage가 존재하기에<br>
  해당 계산 방식을 적용하도록 설정<br>


[![Image](https://github.com/user-attachments/assets/2f2b1399-a1cb-477a-ae04-0beea372d2ba)](https://github.com/user-attachments/assets/2f2b1399-a1cb-477a-ae04-0beea372d2ba){: .image-popup}<br>

- 이후 GA의 Effect Class에 적용<br>
  (상속받은 GA의 내부 Effect 적용에 사용)<br>


### Gameplay Tag

```cpp
// .ini
;METADATA=(Diff=true, UseCommands=true)

[/Script/GameplayTags.GameplayTagsList]
GameplayTagList=(Tag="AI.NPC.Minion.Melee",DevComment="근거리 미니언 태그")
GameplayTagList=(Tag="AI.NPC.Minion.Range",DevComment="원거리 미니언 태그")
GameplayTagList=(Tag="AI.NPC.Minion.Seige",DevComment="공성 미니언 태그")
GameplayTagList=(Tag="AI.NPC.Minion.Super",DevComment="슈퍼 미니언 태그")

GameplayTagList=(Tag="AI.NPC.Stat.Health",DevComment="Health 스탯 태그")
GameplayTagList=(Tag="AI.NPC.Stat.Attack",DevComment="Attack 스탯 태그")
GameplayTagList=(Tag="AI.NPC.Stat.Defense",DevComment="Defense 스탯 태그")
GameplayTagList=(Tag="AI.NPC.Stat.Speed",DevComment="Speed 스탯 태그")

GameplayTagList=(Tag="AI.NPC.State.Dead",DevComment="Npc 사망 태그")
GameplayTagList=(Tag="AI.NPC.State.Cooldown.Attack", DevComment="일반 공격 후 딜레이")

GameplayTagList=(Tag="Event.Montage.AttackHit", DevComment="근접 공격 타격 시점")
GameplayTagList=(Tag="Event.Montage.ProjectileFire", DevComment="원거리 발사 시점")

GameplayTagList=(Tag="Ability.Attack.Melee", DevComment="근접 공격 GA")
GameplayTagList=(Tag="Ability.Attack.Range", DevComment="원거리 공격 GA")

GameplayTagList=(Tag="GameplayCue.NPC.HitReact", DevComment="피격 시 연출")

---
#include "GameplayTag/PGGameplayTags.h"

UE_DEFINE_GAMEPLAY_TAG(TAG_Data_Damage_Base, "Data.Damage.Base");
UE_DEFINE_GAMEPLAY_TAG(TAG_Data_Damage_Multiplier, "Data.Damage.Multiplier");

```

- Tag는 매우 다양한 방식으로 설정이 가능하다<br>
  ini 파일에 설정하는 방식도 있으며<br>
  GameplayTagContainer나 매크로 함수를 이용할수도 있다<br>
  - Editor에서 매니저를 통해 설정도 가능<br>


[![Image](https://github.com/user-attachments/assets/38184e88-154a-4b75-b661-a56165122d46)](https://github.com/user-attachments/assets/38184e88-154a-4b75-b661-a56165122d46){: .image-popup}<br>

- 헷갈리기 쉬운 부분으로는<br>
  AttributeSet의 데이터와 GameplayTag가<br>
  반드시 1:1 매칭되어야 하는 것은 아니다!<br>
  - 동료가 만들어둔 AttributeSet + GameplayTag가 있으나<br>
    나는 별도의 작업을 위해 AI.NPC.Stat 태그를 제작하였다<br>
  - 이후, 별도의 GE에서 마킹하는 방식을 통해 미니언 스탯을 초기화 하는 방식을 사용하였다<br>

### TMI - AnimInstance에서 넘겨주기

```cpp

void UNpcAnimInstance::AnimNotify_AttackHit()
{
	SendGameplayEventToOwner(FGameplayTag::RequestGameplayTag(FName("Event.Montage.AttackHit")));
}

void UNpcAnimInstance::AnimNotify_ProjectileFire()
{
	SendGameplayEventToOwner(FGameplayTag::RequestGameplayTag(FName("Event.Montage.ProjectileFire")));
}

void UNpcAnimInstance::SendGameplayEventToOwner(FGameplayTag EventTag)
{
	if (IsValid(OwnerCharacter) == false)
	{
		return;
	}

	AActor* CurrentTarget = OwnerCharacter->GetAttackTarget();
	if (IsValid(CurrentTarget) == false)
	{
		return;
	}

	FGameplayEventData Payload;
	Payload.Instigator = OwnerCharacter;
	Payload.Target = CurrentTarget;

	FGameplayAbilityTargetData_SingleTargetHit* NewData = new FGameplayAbilityTargetData_SingleTargetHit();
	NewData->HitResult = FHitResult(CurrentTarget, nullptr, FVector::ZeroVector, FVector::ZeroVector);
	NewData->HitResult.bBlockingHit = true;

	Payload.TargetData.Add(NewData);

	UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(OwnerCharacter, EventTag, Payload);
}
```

- 몽타주에서 해당 AnimNotify 를 설정하기<br>
  - NewNotify를 통해 설정<br>
  - 위 경우 AttackHit 노티파이를 설정하였다<br>

- 이후 SendGameplayEventToOwner 를 통해<br>
  OwnerCharacter의 ASC에 맞는 EventTag를 기다리는 어빌리티에게 전송<br>
  (PayLoad가 전송될 결과물)<br>

- 위에서 말하였듯<br>
  TargetData를 이곳에서 별도로 설정하여 전송한다<br>
  - 필요에 따라 GA 쪽에서 설정할수도 있을지도...?<br>
    (다만 그 경우는 별도의 타겟 설정 로직 필요)<br>
