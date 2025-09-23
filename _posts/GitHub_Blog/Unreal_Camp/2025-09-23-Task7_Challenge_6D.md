---
title: "Unreal/C++ 과제 7 - 도전 구현 - 6D"
date : "2025-09-23 20:00:00 +0900"
last_modified_at: "2025-09-23T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - 오일러 각도
  - 쿼터니언
  - 사원수
  - 짐벌락
---

## Unreal/C++ 과제 7 - 6자유도 (6 DOF) 드론/비행체 구현

### 과제 소개 및 목표

- **6축 이동 및 회전 액션 구현**
    - **이동**<br>
        - 전/후 (W/S) - 로컬 X축 이동<br>
        - 좌/우 (A/D) - 로컬 Y축 이동<br>
        - 상/하 (Space/Shift) - 로컬 Z축 이동<br>
    - **회전**<br>
        - Yaw - 좌우 회전, 마우스 X축 이동<br>
        - Pitch - 상하 회전, 마우스 Y축 이동<br>
        - Roll - 기울기 회전, 마우스 휠 또는 별도 키<br>
- **Orientation 기반 이동 구현**
    - 현재 Pawn의 회전 상태에 따라 이동 방향이 결정되는 비행체 움직임을 구현<br>
    - 단순 월드 좌표계 이동이 아닌, Pawn의 로컬 좌표계 기준 이동을 구현<br>

## 구현 기능

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/B-StxcaJRkA"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

- 6축 이동 및 회전 액션 구현<br>
  - 상/하, 좌/우, 전/후 이동<br>
  - Yaw , Pitch, Roll 회전 <br>
- Orientation 기반 이동<br>

---

### 클래스 설명

- 추가 구현 클래스

```
ATaskHoverPawn
- 6D 구현용 Pawn 클래스
- Move 와 Look 은 방식과 유사하며
  입력으로 UpMove, DownMove 그리고 RotateZ 를 추가

AHoverController
- 6D 구현용 PlayerController 클래스
```

## 코드 구현부

```cpp
void ATaskHoverPawn::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (MoveVec.Size() > 0)
	{
		FVector TempMoveVec = MoveVec.GetSafeNormal() * MoveSpeed * DeltaTime;

		AddActorLocalOffset(TempMoveVec);
		MoveVec = FVector::Zero();
	}

	if (RotateR.IsNearlyZero() == false)
	{
		AddActorLocalRotation(RotateR);
	}

	RotateR = FRotator::ZeroRotator;
}

---

void ATaskHoverPawn::UpMove(const FInputActionValue& value)
{
	const float& UpValue = value.Get<float>();
	const FRotator RollRot(0.f, 0.f, UpValue);
	if (FMath::IsNearlyZero(UpValue) == false)
	{
		const FVector Up = FRotationMatrix(RollRot).GetUnitAxis(EAxis::Z);
		MoveVec += (Up * UpValue);
	}
}

void ATaskHoverPawn::DownMove(const FInputActionValue& value)
{
	const float& DownValue = value.Get<float>();
	const FRotator RollRot(0.f, 0.f, DownValue);
	if (FMath::IsNearlyZero(DownValue) == false)
	{
		const FVector Down = FRotationMatrix(RollRot).GetUnitAxis(EAxis::Z);
		MoveVec -= (Down * DownValue);
	}
}

void ATaskHoverPawn::RotateZ(const FInputActionValue& value)
{
	const float& RotateValue = value.Get<float>();
	RotateR.Roll = -RotateValue;
}
```

- Tick에서 SpringArm만 회전시키는 것을<br>
  다시 Actor 전체로 수정<br>
  (이번에는 YawPitchRoll을 모든 요소가 적용받아야 하므로)<br>

## TMI - FRotator(Euler) vs FQuat(Quaternion)

게임에서 '회전'과 관련해서 일반적으로는 2개의 표현법을 사용<br>

- 오일러 각(Euler Angles)<br>
  : 회전을 '세 축' (XYZ)을 기준으로 순차적으로 적용하는 방식<br>
  - 장점<br>
    : 직관적이기에 사용 및, Tool 제작, 입력 매핑이 간편함<br>
  - 단점<br>
    : '짐벌락' 현상 발생 가능, 각 축 간의 회전 순서에 종속, 보간 문제<br>

- 쿼터니언(Quaternion,사원수)<br>
  : 회전을 4차원 벡터 (x,y,z,w)로 표현하는 방식<br>
  - 장점<br>
    : '짐벌락' 없음, 누적값에 대한 안정적(정규화), 안전한 보간 가능<br>
  - 단점<br>
    : 비직관적, 수학적 개념 필요<br>

보통 게임 엔진에서는 양측의 장점을 혼용하여 사용<br>

- Tool, 에디터, UI 등에서 '오일러' 표현을 통해 사용자가 편하게 사용<br>
- 내부 회전에서는 쿼터니언을 사용하여, '안정적'인 처리<br>

### 짐벌락(Gimbal Lock) 현상?

[![Image](https://github.com/user-attachments/assets/44a8bc60-1e56-4142-b131-54fefa4eceac)](https://github.com/user-attachments/assets/44a8bc60-1e56-4142-b131-54fefa4eceac){: .image-popup}<br>

- 3축 회전을 표현하던 중, '특정' 각도의 움직임을 통하여<br>
  축들이 서로 '겹쳐'버리는 현상<br>

- 오일러 각은 '순서'대로 축을 회전시킨다<br>
  그렇기에 '특정 축'이 +- 90 도 기울어져 버린 상황에서<br>
  다른 축과 겹쳐버릴 수 있음<br>
  (위는 극단적인 예시)<br>

- 말그대로 '짐벌' (3차원 회전 장치)가<br>
  '락' (잠겼다)는 뜻의 현상이다<br>

일반적인 해결법은<br>
'쿼터니언'을 통해 4차원 회전을 이용하거나<br>
'회전'에 제한을 거는 상황을 통해 해당 현상을 예방한다<br>

### 그러면 FRotator 사용하면 위험한가?

다행히도 `AddActorLocalRotation` 같은<br> 
함수 내부에서 쿼터니언으로 '변환'하여 회전이 적용된다<br>

```cpp
void AActor::AddActorLocalRotation(FRotator DeltaRotation, bool bSweep, FHitResult* OutSweepHitResult, ETeleportType Teleport)
{
	if(RootComponent)
	{
		RootComponent->AddLocalRotation(DeltaRotation, bSweep, OutSweepHitResult, Teleport);
	}
	else if (OutSweepHitResult)
	{
		*OutSweepHitResult = FHitResult();
	}
}

///---
// 내부적으로 FQuat로 변환하여 사용
void USceneComponent::AddLocalRotation(FRotator DeltaRotation, bool bSweep, FHitResult* OutSweepHitResult, ETeleportType Teleport)
{
	const FQuat CurRelRotQuat = RelativeRotationCache.RotatorToQuat(GetRelativeRotation());
	const FQuat NewRelRotQuat = CurRelRotQuat * DeltaRotation.Quaternion();
	SetRelativeLocationAndRotation(GetRelativeLocation(), NewRelRotQuat, bSweep, OutSweepHitResult, Teleport);
}
```

- 그래도 너무 '큰 회전'을 사용한다던가<br>
  90도 같은 겹치기 쉬운 각도를 사용한다던가<br>
  '보간' 등을 할 필요가 있다면<br>
  FQuat를 사용하는 편이 훨씬 안전하다<br>

- FQuat에는 Normalize 같은 정규화 함수도 있기에<br>
  이러한 '누적' 회전에도 안전<br>

