---
title: "Unreal TIL (0916)"
date : "2025-09-16 20:00:00 +0900"
last_modified_at: "2025-09-16T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Reflection
---

# 리플렉션 시스템이 필요한 이유?

## 1️⃣ **Blueprint (시각적 스크립팅) 이해**

- **Blueprint**는 언리얼 엔진에서 제공하는 시각적 스크립팅 도구로, 노드 (블록)를 연결하여 게임 로직을 작성<br>
- **장점**
    - Blueprint 그래프를 수정 후, 에디터에서 Play 버튼을 누르면 곧바로 결과를 확인 가능<br>
    - 프로그래밍에 익숙하지 않은 다른 작업자도 쉽게 접근 가능<br>
- **한계점**
    - 노드 수가 많아질수록 그래프가 복잡해짐(가독성과 유지보수 문제)<br>
    - C++과 비교했을 때, Blueprint는 오버 헤드 존재, 물리 연산이나 AI 같은 높은 성능이 필요한 시스템에 비적합<br>

## 2️⃣ **C++ (네이티브 코드 프로그래밍) 이해**

- **장점**
    - 엔진 코어까지 직접 수정 가능, 복잡하고 성능이 중요한 게임 로직을 빠르고 최적화된 방식으로 구현<br>
    - 표준 라이브러리와 외부 라이브러리를 자유롭게 사용 가능하여 대규모 프로젝트에 적합<br>
    - 포인터, 템플릿 같은 C++ 언어적 기능을 통해 메모리를 사용(정교한 로직)<br>
- **한계점**
    - C++ 코드를 수정하면 에디터를 재시작 or Live Coding을 재컴파일 (번거로움)<br>

## 3️⃣ Blueprint와 C++의 상호보완적 관계

- 실무에서는 **Blueprint와 C++**를 함께 사용하는 편, **하나만** 사용하는 것보다 각각의 장점을 취하는 **하이브리드 워크플로우**가 일반적<br>
    - **Blueprint 활용**: UI 제작, 간단한 이벤트 처리, 시각적 연출 등 **빠른 프로토타이핑**과 **직관적인 로직** 작성에 사용<br>
    - **C++ 활용**: 높은 성능이 필요한 게임플레이 로직이나 엔진 레벨의 확장, 복잡한 수학 연산 등에 사용<br>
- 이렇게 분업하면 개발 속도와 퍼포먼스를 모두 확보 가능<br>

## 4️⃣ **리플렉션 (Reflection)이 필요한 이유**

- 언리얼 엔진의 리플렉션 시스템은 **C++ 클래스의 변수 및 함수 정보를 엔진 내부의 메타데이터 형태로 저장**하고, 이를 **에디터나 블루프린트**에서 활용할 수 있게 만들어주는 것<br>
    - **C++ 클래스**에 있는 여러 멤버(변수, 함수 등)를 **“Reflectoin (반사)”**해, 에디터와 블루프린트에서 직접 설정, 호출이 가능하게 만듦<br>
    - 이 덕분에 프로그래머가 만든 **C++ 로직의 뼈대**를 디자이너나 다른 팀원들이 **에디터**에서 직관적으로 조정 가능해짐<br>
    - 매개변수를 코드에서만 변경하는 것이 아니라, **에디터**에서 바로 조정 (슬라이더나 숫자 입력)하여 **반복 테스트**를 빠르게 진행<br>
- 리플렉션 시스템을 제대로 이해하면, **개발 효율**과 **협업** 효과가 극대화<br>

[이전에 정리한 Reflection 관련 블로깅(참고)](https://hnjog.github.io/unreal/UE_GC/){:target="_blank"}<br>

# C++ 클래스에서 보는 리플렉션

## 예시 코드

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Item.generated.h" // 반드시 마지막에 위치해야 합니다.

UCLASS()
class SPARTAPROJECT_API AItem : public AActor
{
		GENERATED_BODY()
	
public:	
		AItem();
	
protected:
		USceneComponent* SceneRoot;
		UStaticMeshComponent* StaticMeshComp;
	
		float RotationSpeed;
	
		virtual void BeginPlay() override;
		virtual void Tick(float DeltaTime) override;
};
```

- **`#include "Item.generated.h"`**<br>
    - 언리얼 엔진이 자동 생성하는 헤더 파일로, 클래스의 리플렉션 및 엔진 통합에 필요한 코드가 내부에 존재<br>
    - 반드시 **헤더 파일의 가장 마지막 `#include`** 구문 아래에 위치해야 함! (다른 `#include`보다 아래에 오지 않으면 빌드 에러 발생 가능)<br>
- **`UCLASS()`**<br>
    - 해당 클래스를 언리얼 엔진의 리플렉션 시스템에 **등록**한다는 의미<br>
    - 이 매크로가 있어야만 블루프린트 등 에디터 차원에서 이 클래스를 인식하고 사용 가능함<br>
- **`GENERATED_BODY()`**<br>
    - 언리얼의 코드 생성 도구가 사용하는 코드를 삽입하는 역할<br>
    - 클래스 내부에 필요한 리플렉션 정보를 자동으로 생성<br>

## **`UCLASS()`** 매크로의 지정자?

- 이러한 매크로들은 각각 `리플렉션 시스템`에 등록하면서<br>
  몇가지 옵션(지정자)를 설정 가능<br>

- 기본 동작<br>
  - `UCLASS()` 자체만 사용하는 경우, BP에서 상속 **가능**하고, 변수로 **참조** 가능하게 등록<br>

- 주요 옵션들<br>
  - `Blueprintable`<br>
    - 블루프린트에서 **상속** 가능하도록<br>
  - `NotBlueprintable`<br>
      - 블루프린트에서 이 클래스를 **상속**할 수 없도록<br>
  - `BlueprintType`<br>
      - 블루프린트에서 **변수나 참조**로 사용할 수 있게 함<br>
      - 이 옵션만 존재 시, 상속은 **허용되지 않고** 참조만 가능<br>
  - 필요에 따라 이 지정자들을 조합해 클래스가 어떻게 블루프린트와 상호작용해야 할지 명시<br>


# C++ 변수에서 보는 리플렉션


## `UPROPERTY()` **매크로의 주요 지정자**

- `UPROPERTY()`에는 여러 지정자를 작성해, 에디터에서의 표시 여부나 Blueprint 접근성, 읽기/쓰기 권한 등을 자세하게 설정 가능<br>
1. **편집 가능 범위 지정자**<br>
    - `VisibleAnywhere`: 읽기 전용으로 표시되며, 수정은 불가능<br>
    - `EditAnywhere`: 클래스 기본값, 인스턴스 모두에서 수정 가능<br>
    - `EditDefaultsOnly`: 클래스 기본값에서만 수정 가능<br>
    - `EditInstanceOnly`: 인스턴스에서만 수정 가능<br>
2. **Blueprint 접근성 지정자**<br>
    - `BlueprintReadWrite`: Blueprint 그래프에서 **Getter/Setter**로 값을 읽거나 쓸 수 있음<br>
    - `BlueprintReadOnly`: Blueprint 그래프에서 **Getter** 핀만 노출되어, 읽기만 가능<br>
3. **Category 지정자**<br>
    - Details 패널에서 이 변수는 “Rotation” 범주(폴더) 아래에 표시<br>
    - 여러 변수를 비슷한 카테고리에 묶으면, 세부 정보 패널에서 깔끔하게 정리<br>
4. **메타 옵션 지정자**<br>
    - `meta=(ClampMin="0.0")`: 에디터에서 변수 입력 시 **최소값**을 제한<br>
    - `meta=(AllowPrivateAccess="true")`: 해당 멤버가 `private`로 선언되어 있어도, 에디터나 Blueprint에서 접근 허용<br>
- **만약 UPROPERTY()만 있고, 추가 지정자를 하나도 주지 않는다면?**
    - 엔진 리플렉션 시스템에는 등록, 에디터나 Blueprint에 **노출**되지는 않음<br>
    - 등록만 되어 있어도 **가비지 컬렉션(메모리 관리)과** **직렬화(세이브/로드)** 같은 엔진 내부 기능이 작동 가능<br>

## 예시 코드

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Item.generated.h"

UCLASS()
class SPARTAPROJECT_API AItem : public AActor
{
		GENERATED_BODY()
	
public:	
		AItem();
		
protected:
		// Root Scene Component, 에디터에서 볼 수만 있고 수정 불가
		UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Item|Components")
		USceneComponent* SceneRoot;	
		// Static Mesh, 에디터와 Blueprint에서 수정 가능
		UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Item|Components")
		UStaticMeshComponent* StaticMeshComp;
	
		// 회전 속도, 클래스 기본값만 수정 가능
		UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Item|Properties")
		float RotationSpeed;
	
		virtual void BeginPlay() override;
		virtual void Tick(float DeltaTime) override;
};
```


# C++ 함수에서 보는 리플렉션


## `UFUNCTION()` **매크로의 주요 지정자**
- **Blueprint 관련 지정자**<br>
    - `BlueprintCallable`<br>
        - Blueprint 이벤트 그래프(노드)에서 **호출(Execute)** 가능<br>
    - `BlueprintPure`<br>
        - **Getter** 역할만 수행 (Exec 핀 없이 Return Value만 노출)<br>
    - `BlueprintImplementableEvent`<br>
        - 함수의 선언만 C++에 있고, **구현은 블루프린트**에서 하도록, C++ 코드에서는 함수 이름만 정의, 실제 동작은 Blueprint Event Graph 안에서 이벤트 노드처럼 구현<br>
- **만약 `UFUNCTION()`에 지정자를 하나도 쓰지 않았다면?**<br>
    - `UPROPERTY()`와 마찬가지로, 함수가 언리얼 리플렉션에 등록, **특별히 Blueprint에 노출**되지는 않음<br>

## 예시코드

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Item.generated.h"

UCLASS()
class SPARTAPROJECT_API AItem : public AActor
{
		GENERATED_BODY()
	
public:	
		AItem();
	
protected:
		UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Item|Components")
		USceneComponent* SceneRoot;
	
		UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item|Components")
		UStaticMeshComponent* StaticMeshComp;
	
		UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Item|Properties")
		float RotationSpeed;
	
		virtual void BeginPlay() override;
		virtual void Tick(float DeltaTime) override;
		
		// 함수를 블루프린트에서 호출 가능하도록 설정
	  UFUNCTION(BlueprintCallable, Category="Item|Actions")
	  void ResetActorPosition();
		
		// 블루프린트에서 값만 반환하도록 설정
    UFUNCTION(BlueprintPure, Category = "Item|Properties")
    float GetRotationSpeed() const;

		// C++에서 호출되지만 구현은 블루프린트에서 수행
    UFUNCTION(BlueprintImplementableEvent, Category = "Item|Event")
    void OnItemPickedUp();
};
```
