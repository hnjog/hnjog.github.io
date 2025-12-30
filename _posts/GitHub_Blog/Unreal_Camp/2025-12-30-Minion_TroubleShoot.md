---
title: "Minion Trobule Shooting - 불사 버그"
date : "2025-12-30 20:00:00 +0900"
last_modified_at: "2025-12-30T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - AddLooseGameplayTag
  - HasMatchingGameplayTag
---

## 미니언 버그 - 불사?

[![Image](https://github.com/user-attachments/assets/6539249a-7e8d-44e4-a064-2ee47bcb8e02)](https://github.com/user-attachments/assets/6539249a-7e8d-44e4-a064-2ee47bcb8e02){: .image-popup}<br>

팀 프로젝트 테스트 중 미니언이 사망하지 않는 버그가 발생하였다!<br>
미니언이 사망하지 않아, 라인이 계속 뭉쳐버려 게임 진행을 할 수 없는 버그였다<br>

- 미니언 & Wave 작업을 담당하였기에 빠르게 버그의 원인을 파악하였다<br>


## 버그의 원인?

처음에는 `AddLooseGameplayTag`가<br>
클라이언트에 '전달'되지 않아서<br>
서버 <-> 클라 간 '사망 상태' 동기화가 안되었나?<br>
싶었다<br>

그런데 곰곰히 생각하니 조금 이상하였다<br>

- Destory 패킷은 결국 반드시 '클라'에 전달됨<br>
  - 그렇다는 것은 '현재' 서버에서 '죽지 않는다는 뜻'!<br>
  - 지금 파괴 처리를 어떻게 하고 있지?<br>


```cpp
void ANpcBaseCharacter::Multicast_HandleDeath_Implementation()
{
	...

	if (HasAuthority())
	{
		SetLifeSpan(DeathDuration + 0.1f);
		SetAttackTarget(nullptr);
	}
}
```

- SetLifeSpan이네?<br>
  -> 이게 '중복' 호출 된다면 영원히 죽지 않는다!<br>

따라서 이러한 '중복' 처리를 막을 필요가 있다<br>

```cpp
void ANpcBaseCharacter::HandleDeath(AActor* KillerActor)
{
	if (HasAuthority())
	{
		bool bIsDead = ASC->HasMatchingGameplayTag(DeadTag);
		if (bIsDead == true)
		{
			return;
		}
    ...
	}
}
```

지금은 DeadTag를 처리하기에 '괜찮'을줄 알았으나<br>
'미니언'이 많아진 상황에서 내부 로직 정리에 혼선이 있을 가능성이 존재함<br>


*결국 '타이밍 문제'일 가능성이 유력*해 보임<br>

**문제의 흐름**<br>

1. HasMatchingTag 만으로 사망 처리 확인<br>
2. AddLooseGameplayTag 로 DeadTag 부여<br>
3. StateTree의 이벤트 호출 -> 사망용 GA 실행<br>
4. GA에서 SetLifeSpan<br>

이것이 'Atomic'하게 실행되는 것이 아님!<br>

- ST의 '이벤트 큐'<br>
  - 또한 '동시에' 같은 이벤트가 여러번 들어올 수 있음!<br>
  - 따라서 '단 시간'에 사망 처리 이벤트가 와르르 들어오고<br>
    처리 타이밍이 꼬여서 '무한 루프'가 아닌<br>
	'매우 늦게 죽었을 가능성'도 존재함<br>
	(DeadState 여도 다시 Root->Dead 로 전이하여 GA 재 호출)<br>

- HasMatchingTag가 Tag를 확인하는 것과<br>
  AddLooseGameplayTag가 Count를 '부여'하는 타이밍이 엇갈릴 수 있음<br>
  (GAS가 태그 변경을 '모았다가' 한번에 계산하려 시도할 수 있음)<br>

- GA도 TryAbility를 통해 호출되기에 '곧바로' 실행되지 않을 가능성이 존재<br>

**유의할 점들**<br>

- GAS의 `"태그 집계(Aggregation)"와 "스코프(Scope)"`<br>
  처리 방식으로 인하여 '과부하' 상황에서 오류가 발생 가능할 수 있음<br>
  - 성능 저하를 막기 위하여 Tag 변경에 Lock을 걸어놓는 등<br>
    '태그 추가/삭제' 등에 대한 순간적인 신뢰성을 잃을 수 있음<br>
  - 이후 다른 GA의 `AbilityTask_WaitGameplayTag` 같은 이벤트들도<br>
    같이 들어오게 되면 로직이 '예측'한대로 호출되지 않을 수 있음<br>

- 매프레임 호출(`Tick`) 되거나<br>
  매우 많은 객체가 접근할 가능성이 있는 로직이라면<br>
  `HasMatching` 같은 함수 대신 신뢰성이 있는 bool 변수 등을 통해<br>
  이중 관리를 고려할 것!<br>
  (DeadTag를 붙여도 사망처리이지만 bool bIsDead 같은 변수를 통해 확실하게 상태 처리!)<br>

## 수정 코드

```cpp
// h

{
protected:

	UPROPERTY(Replicated)
	bool bIsDead;
}
// cpp
void ANpcBaseCharacter::HandleDeath(AActor* KillerActor)
{
	if (HasAuthority())
	{
		bool bAlreadyDead = ASC->HasMatchingGameplayTag(DeadTag);
		if (bAlreadyDead == true ||
			bIsDead == true)
		{
			return;
		}
    ...
	}
}
```

- 기존의 'Tick' 머테리얼 변화용 함수를<br>
  아예 bIsDead 용으로 승격시켰다!<br>

```cpp
AActor* ANpcBaseCharacter::GetAttackTarget() const
{
	if (IsTargetValid(CurrentAttackTarget.Get()) == false)
	{
		return nullptr;
	}

	return CurrentAttackTarget.Get();
}

bool ANpcBaseCharacter::CanAttack() const
{
	if (IsTargetValid(CurrentAttackTarget.Get()) == false)
	{
		return false;
	}

	float DistSq = GetSquaredDistanceTo(CurrentAttackTarget.Get());
	float AttackRangeSq = AttackRange * AttackRange;

	return DistSq <= AttackRangeSq;
}

bool ANpcBaseCharacter::IsTargetValid(AActor* TargetActor) const
{
	if (IsValid(TargetActor) == false)
	{
		return false;
	}

	if (TargetActor->GetClass()->ImplementsInterface(UPGTeamStatusInterface::StaticClass()))
	{
		bool bTargetDead = IPGTeamStatusInterface::Execute_GetIsDead(TargetActor);
		if (bTargetDead)
		{
			return false;
		}

		int32 TargetTeamId = IPGTeamStatusInterface::Execute_GetTeamID(TargetActor);
		if (TeamId == TargetTeamId)
		{
			return false;
		}
	}

	return true;
}
```

- 또한 이 이외에도<br>
  AttackTarget의 '공격 대상'의 '사망처리'용 bool 변수를 확인하도록 수정하였다<br>

## 문제 해결

[![Image](https://github.com/user-attachments/assets/bf5a29c5-1e80-45ce-bce6-d38785bbce54)](https://github.com/user-attachments/assets/bf5a29c5-1e80-45ce-bce6-d38785bbce54){: .image-popup}<br>

이제 비교적 많은 미니언들이 치고 받아도<br>
이전과 같이 '무적' 미니언 버그가 나타나지 않았다!<br>

### 배운 점

1. 대규모로 사용될 코드가 있다면 '동기화 타이밍'을 더 고려할 것<br>
2. 확실한 '상태 체크'용 변수는 bool 등을 고려<br>
