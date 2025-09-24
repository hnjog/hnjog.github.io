---
title: "Unreal/C++ 과제 7 - 도전 구현2 - 중력 및 낙하 구현"
date : "2025-09-24 20:00:00 +0900"
last_modified_at: "2025-09-24T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
---

## Unreal/C++ 과제 7(2) - 중력 및 낙하 구현

### 과제 소개 및 목표

- **인공 중력 구현**
    - **Tick 함수**를 통해 매 프레임 중력 가속도를 직접 계산<br>
    - 적절한 중력 상수 (예: -980 cm/s²)를 사용하여 낙하 속도를 구현<br>
    - LineTrace 또는 SweepTrace를 사용하여 지면 충돌을 감지<br>
    - 착지 시 Z축 속도를 0으로 초기화합니다.
- **에어컨트롤 구현 (공중 WASD 제어)**
    - 공중에서는 지상 이동속도의 30~50% 정도로 제한<br>
    - 지상/공중 상태에 따라 이동 로직을 구분하여 자연스러운 움직임을 구현<br>

## 구현 기능

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/uBF1pDQ7VpI"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

- 인공 중력 구현<br>
  - Tick + 임의의 중력 상수 사용<br>
  - LineTrace를 통하여 지면 충돌을 감지<br>

- 에어 컨트롤 및 궁중 이동 관성 구현<br>
  - JumpVelocity를 별도로 사용하여 기존 movement에 반영<br>

---

### 클래스 설명

- 추가 구현 클래스

```
TaskCharacter (APawn 상속)
- 기존 기능 + Jump 및 인공 중력 상수 추가
- LineTrace를 이용하여 바닥 도착 판정 추가

TaskPlayerController
- Jump Action 추가
```

## 코드 구현부

```cpp
void ATaskCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (MoveVec.Size() > 0)
	{
		double JumpScale = bJump ? 0.3 : 1.0;

		FVector TempMoveVec = MoveVec.GetSafeNormal() * MoveSpeed * DeltaTime * JumpScale;

		AddActorLocalOffset(TempMoveVec);
	}

	if (JumpVec.Size() > 0)
	{
		JumpVec.X += MoveVec.X * MoveSpeed * DeltaTime;
		JumpVec.Y += MoveVec.Y * MoveSpeed * DeltaTime;
		JumpVec.Z += GravityScale * DeltaTime;

		if (UWorld* World = GetWorld())
		{
			FVector Start = GetActorLocation();
			FVector End = GetActorLocation() + GetActorUpVector() * -TraceHeight;

			FHitResult HitResult;

			FCollisionQueryParams Params;
			Params.AddIgnoredActor(this);

			bool bLand = World->LineTraceSingleByChannel(
				HitResult, Start, End, ECollisionChannel::ECC_Visibility, Params);

			if (bLand)
			{
				SetActorLocation(Start + GetActorUpVector() * CapsuleComponent->GetScaledCapsuleHalfHeight() / 2);
				bJump = false;
				JumpVec = FVector::Zero();
			}
		}

		FVector TempJumpVec = JumpVec * DeltaTime;

		AddActorLocalOffset(TempJumpVec);
	}

	if (FMath::IsNearlyZero(RotateR.Yaw) == false)
	{
		AddActorLocalRotation(FRotator(0.0,RotateR.Yaw * RotateSpeed * DeltaTime,0.0));
	}

	if (FMath::IsNearlyZero(RotateR.Pitch) == false)
	{
		FRotator SpringRot = SpringArmComp->GetRelativeRotation() + FRotator(RotateR.Pitch * RotateSpeed * DeltaTime,0.0 , 0.0);
		SpringRot.Pitch = FMath::Clamp(SpringRot.Pitch, -40.0, 60.0);

		SpringArmComp->SetRelativeRotation(SpringRot);
	}

	MoveVec = FVector::Zero();
	RotateR = FRotator::ZeroRotator;
}

---

void ATaskCharacter::Jump(const FInputActionValue& value)
{
	if (value.Get<bool>() &&
		bJump == false)
	{
		bJump = true;
		JumpVec = FVector(MoveVec.X * MoveSpeed, MoveVec.Y * MoveSpeed, JumpSpeed);
	}
}

```

- Jump에서 JumpVec에 MoveVec 와 JumpSpeed 를 통해<br>
  점프 상태에서 적용할 관성용 벡터값 저장<br>

- bool 변수를 통하여 점프 중, 이동에 대한 속도 저하<br>
  (JumpScale)<br>

### 트러블 슈팅 1 - 점프 이후, 내려오지 않는다...?

[![Image](https://github.com/user-attachments/assets/f8ac1e43-3084-4c51-a783-70aec99947ea)](https://github.com/user-attachments/assets/f8ac1e43-3084-4c51-a783-70aec99947ea){: .image-popup}<br>

점프 이후 '상승'만 하고<br>
캐릭터가 내려오지를 않는 상황<br>

```cpp
JumpVec.Z -= GravityScale * DeltaTime;
```

코드를 보았을때는 '중력 상수' 만큼 빼주고 있는데 왜이러지? 싶었다<br>

그런데 알고보니...<br>

```cpp
GravityScale = -980.0f;
```

값을 음수로... 잡아두고 있었다<br>

```cpp
JumpVec.Z += GravityScale * DeltaTime;
```

이렇게 수정하니 적당히 올라간 후<br>
땅으로 떨어지게 되었다<br>

### 트러블 슈팅 2 - 점프 순간의 관성이 유지되지 않는다?

[![Image](https://github.com/user-attachments/assets/f9a58402-cb11-4135-99b0-82dd48625b35)](https://github.com/user-attachments/assets/f9a58402-cb11-4135-99b0-82dd48625b35){: .image-popup}<br>

궁중에서 손을 '놓'으니<br>
그대로 '툭'하고 떨어진다!<br>

문제점이 무엇인가 확인하니<br>
사실상 JumpVec가 '방향'을 저장하는 역할 밖에 없었으며<br>
그것도 매우 미약하기에 Jump의 '관성'적인 역할은 거의 없었다<br>

```cpp
// TICK
if (JumpVec.Size() > 0)
{
	FVector TempJumpVec = JumpVec.GetSafeNormal() * MoveSpeed * DeltaTime;
	AddActorLocalOffset(TempJumpVec);

	if (UWorld* World = GetWorld())
	{
		FVector Start = GetActorLocation();
		FVector End = GetActorLocation() + GetActorUpVector() * -TraceHeight;

		FHitResult HitResult;

		FCollisionQueryParams Params;
		Params.AddIgnoredActor(this);

		bool bLand = World->LineTraceSingleByChannel(
			HitResult, Start, End, ECollisionChannel::ECC_Visibility, Params);

		if (bLand)
		{
			SetActorLocation(Start + GetActorUpVector() * CapsuleComponent->GetScaledCapsuleHalfHeight() / 2);
			bJump = false;
			JumpVec = FVector::Zero();
		}
	}

}

---

void ATaskCharacter::Jump(const FInputActionValue& value)
{
	if (value.Get<bool>() &&
		bJump == false)
	{
		bJump = true;
		JumpVec = FVector(MoveVec.X, MoveVec.Y, JumpSpeed);
	}
}
```

- 문제는 Normalize를 통해<br>
  JumpVec 자체가 '방향'처럼 쓰인다는 점...<br>

내가 원하는 건 점프 시점에서<br>
그 점프의 관성을 유지하는 것이기에<br>

Normalize를 제거하고<br>
대신 미리 Jump 시점에서 그 '관성'을 이용하기로 하였다<br>

```cpp
void ATaskCharacter::Jump(const FInputActionValue& value)
{
	if (value.Get<bool>() &&
		bJump == false)
	{
		bJump = true;
		JumpVec = FVector(MoveVec.X * MoveSpeed, MoveVec.Y * MoveSpeed, JumpSpeed);
	}
}
```

다만 이렇게 되니<br>
궁중 '이동' 중에서<br>
'손'을 놓는 경우<br>

다시 JumpVec가 저장한 방향대로 가기에<br>
역시 어색한 면이 존재하였다<br>

그렇기에<br>
JumpVec가 MoveVec에 영향을 받도록 수정하여<br>
관성과 '움직임' 모두를 챙기려고 하였다<br>

```cpp
if (JumpVec.Size() > 0)
{
	JumpVec.X += MoveVec.X * MoveSpeed * DeltaTime;
	JumpVec.Y += MoveVec.Y * MoveSpeed * DeltaTime;
	JumpVec.Z += GravityScale * DeltaTime;

	if (UWorld* World = GetWorld())
	{
		FVector Start = GetActorLocation();
		FVector End = GetActorLocation() + GetActorUpVector() * -TraceHeight;

		FHitResult HitResult;

		FCollisionQueryParams Params;
		Params.AddIgnoredActor(this);

		bool bLand = World->LineTraceSingleByChannel(
			HitResult, Start, End, ECollisionChannel::ECC_Visibility, Params);

		if (bLand)
		{
			SetActorLocation(Start + GetActorUpVector() * CapsuleComponent->GetScaledCapsuleHalfHeight() / 2);
			bJump = false;
			JumpVec = FVector::Zero();
		}
	}

	FVector TempJumpVec = JumpVec * DeltaTime;

	AddActorLocalOffset(TempJumpVec);
}
```

이제는 영상처럼 잘 동작한다!<br>