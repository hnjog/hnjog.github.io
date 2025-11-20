---
title: "Property Replication"
date : "2025-11-20 20:00:00 +0900"
last_modified_at: "2025-11-20T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal
  - C++
  - Server
  - Property Replication
  - Replication
---

## Replication

[![Image](https://github.com/user-attachments/assets/39b699be-b3f4-4d04-82a6-81b5a3a164f2)](https://github.com/user-attachments/assets/39b699be-b3f4-4d04-82a6-81b5a3a164f2){: .image-popup}<br>

**생성된 액터의 정보를 '네트워크' 내의 '다른 클라이언트'에 복제하는 작업을 뜻함**<br>

- 서버 - 클라이언트 모델에선<br>
  서버에서 클라이언트로 복제되는 것이 기본 규칙<br>

- 권한을 가진(Authority) 서버가 진짜 결정을 내리고<br>
  클라이언트로 필요한 값을 '복사'<br>
  - Replicates = true; 설정된 Actor만 네트워크에 실리며<br>
    Component는 Actor의 옵션에 종속됨<br>

- 이러한 Replication에는 2가지 방식이 존재<br>
  - RPC(Remote Procedure Call) : [지난 포스팅](https://hnjog.github.io/unreal/c++/RPC/)에 다룬 것<br>
  - Property Replication : 이번 포스팅에 다룰 것<br>

## Property Replication

액터의 모든 속성의 '변경 상태'를 '모든 클라'에 복제하는 것은 비효율적<br>
따라서 **`서버`가 원하는 속성만을 `클라이언트`들에 복제하는 것**을<br>
`Property Replication` 이라 함<br>

- UPROPERTY(Replicated)<br>
  : 서버가 변수 값을 바꾸면 '클라이언트'에게 자동 동기화<br>
    클라이언트에 별도의 Callback은 없음<br>
    (값만 동기화하는 방식임)<br>

- 보통 AI State, 카운팅용 변수 등에 사용<br>

```cpp
UPROPERTY(Replicated)
int32 Health;
```

- UPROPERTY(ReplicatedUsing = OnRep_XXX)<br>
  : 클라 동기화 + 클라 쪽에서 OnRep 콜백 실행하기<br>
    (Health 값이 바뀌면 클라쪽에서 OnRep_ 실행)<br>    

- OnRep 함수의 네이밍은 임의로 지정 가능하며<br>
  UPROPERTY에서 지정한 함수 이름과 같으면 됨<br>

- 서버에서 값 변경이 일어난 후, 1번 호출<br>

- 변경에 따라 '실행할 동작'이 존재할 때 사용<br>
  - UI 업데이트<br>
  - Mesh 변경<br>
  - 이펙트 변경<br>
  - 애니메이션의 파라미터 동기화 등등<br>

```cpp
UPROPERTY(ReplicatedUsing = OnRep_Health)
int32 Health;

UFUNCTION()
void OnRep_Health();
```
