---
title: "Effect Factory"
date : "2025-09-02 19:00:00 +0900"
last_modified_at: "2025-09-02T19:00:00"
categories:
  - C++
tags:
  - C++
  - Factory
  - 구현
---

## RPG와 '효과'에 대한 구현 방식?

사실 '효과'는 무척 추상적으로 보이지만<br>
RPG 게임에서 '아이템'과 '스킬'등에 빠지기 힘든<br>
구현 방식이다<br>

ex)<br>
Item에서 '회복 포션' 이라는 소모 아이템이 존재<br>
Skill에서 '힐' 이라는 스킬이 존재<br>
- 두 요소에서 모두 '체력 회복'과 관련되어 있음<br>
  그런데 각각을 '별도'로 구현하는 것은<br>
  코드 중복의 요지가 있음<br>

- 그렇기에 Effect 라는 '효과'를 공통 부분으로 분리<br>
  아이템/스킬 이 이 효과를 '전달'하는 수단으로 설계<br>

### 클래스 구조 간략

- Effect <br>
  : 자신에게 전달된 결과만 담당<br>

- Item / Skill<br>
  : 트리거를 통하여 특정 타이밍에 Effect를 호출<br>
  (아이템 트리거 : 즉시 발동, 장착, 장착 해제 등)<br>
  (스킬 트리거 : 패시브, 액티브 등)<br>
  (상황에 따라 트리거를 합칠 수 있을듯)<br>

- Context<br>
  : Effect가 외부 상황은 모르지만<br>
    최소한의 처리를 위하여 전달받아야 할것들<br>
    보통<br>
    시전자, 효과 적용 대상, 소모값, 계수와 고정값,<br>
    효과 레벨 등<br>
    (그외에도 'Tag' 등을 벡터로 모아서 보내줄 수도 있을듯)<br>
    - Tag?<br>
      : '분류'를 위한 기능<br>
        (단순히 매핑할 Effect를 고르는 방식부터<br>
         Effect가 적용할 옵션의 구분 까지 다양)<br>
        (ex : 힐이라도, '즉시 힐', '도트 힐', '크리티컬 여부'<br>
        등의 기능을 태그로 차별화할 수 있음)<br>

## 구현 방식 1. Enum - Switch 방식

```
enum Effect
{
  EFF_HEAL,
  EFF_INCREASE_ATTACK,
  EFF_COUNT
}


void EffectApply(EffectContext ...)

Switch(context.Type)
{
 case EFF_HEAL:
 {
    // 여기서 직접 적용하거나
    // 관련 클래스를 호출
 }
 break;
 ...
}
```

- 장점<br>
  - 단순하고 빠른 구현 가능 : 초반 프로토타입으로 적합<br>
  - 직관적 흐름 : Switch 문 하나를 통해 구현하기에, 코드 이해가 쉽다(다만 Enum이 지나치게 늘면 감소하는 장점)<br>
  - 저비용 : 기본적으로 값 분기이기에 '오버헤드'가 적다<br>
  - Enum의 장점 : 저장이나 네트워킹 시 다루기 편함 (대신 버전관리는 주의)<br>

- 단점<br>
  - 확장성 낮음 : Effect 가 점점 늘어날수록 Switch가 '비대'해짐<br>
                (새로운 타입 추가시 Switch 수정이 반드시 필요)<br>
  - 결합도 증가 : Switch를 포함하는 함수에 대한 결합도가 점점 증가<br>

사실 단점이 아주 명확하고 피해야 할것처럼 보이지만<br>
소규모 프로젝트 등에서 고려될 정도로<br>
'빠른 구현'과 '직관적 이해' 는 큰 장점이기도 하다<br>
다만 프로젝트가 점점 커질수록 반드시 교체를 고려해야 하는<br>
구현 방식이라 생각한다<br>

## 구현 방식 2. String(ID) -> Factory Map 방식

일종의 예시 코드와 함께 이해<br>
기본적인 흐름은<br>

- IEffect 인터페이스와 새로운 Effect 클래스<br>
- 매크로를 통한 static Effect 용 팩토리를 EffectManager에게 등록시키는 패턴<br>
- EffectManager에서 Map을 들고 있고<br>
  Map<String,Factory 용 함수> 만 관리<br>
- 이후 사용시, It->Second()를 호출하여<br>
  객체 생성후 반환<br>

EffectManager<br>

```
EffectSpec : 이펙트에 대한 ID 나 소모값을 포함하는 구조체
EffectContext : 이펙트가 효과 실행을 위해 필요한 구조체

// 효과 인터페이스
struct IEffect {
    virtual ~IEffect() = default;
    virtual bool Apply(const EffectSpec&, EffectContext&) = 0;
};

// "팩토리" 시그니처: 호출 시 새 IEffect 인스턴스를 만들어 반환
using EffectFactory = std::function<std::unique_ptr<IEffect>()>;

class EffectManager {
public:
    static EffectManager& I() { static EffectManager inst; return inst; }

    // id와 팩토리를 레지스트리에 보관
    void Register(std::string id, EffectFactory f) {
        table_.emplace(std::move(id), std::move(f)); // 람다(콜러블)를 이동 저장
    }

    // id로 팩토리를 찾아 '호출' → 새 객체 생성
    std::unique_ptr<IEffect> Create(std::string const& id) const {
        auto it = table_.find(id);
        return (it == table_.end()) ? nullptr : it->second(); // ← ()로 팩토리 '호출'
    }

private:
    EffectManager() { table_.reserve(512); } // 예상 개수만큼 버킷 확보(선택)
    std::unordered_map<std::string, EffectFactory> table_;
};

```

- std::function<>는 '호출 가능한 것'을 담아서 래핑하는 용도<br>
  따라서 std::function<std::unique_ptr<IEffect>()> 를 통해<br>
  IEffect 타입을 만들어주는 팩토리 함수를 등록할 예정<br>

- Register를 통하여 '람다' 함수를 move를 통해 이동시켜 보관<br>
  (map)<br>

- Create()에서 일회성 Effect를 Unique_ptr로 반환<br>

- 필요시 총 'Effect' 개수 등을 다른 파싱용 매니저에서 가져와 reserve를 해줄 수 있음<br>

등록용 템플릿용 코드와 매크로 함수<br>

```
// EffectRegister.h
template<class T>
void RegisterEffectType() {
  EffectManager::I().Register(T::Id(), [](){ return std::make_unique<T>(); });
}

// 자가 등록(정적 객체) — 또는 모듈 Init 에서 RegisterEffectType<T>()를 호출해도 됨
template<class T>
struct EffectAutoRegistrar {
  EffectAutoRegistrar(){ RegisterEffectType<T>(); }
};

// 편의 매크로
#define REGISTER_EFFECT_TYPE(T) \
  static EffectAutoRegistrar<T> _auto_registrar_##T;

```

- REGISTER_EFFECT_TYPE 에 실제 구체화된 클래스 타입을 넣음으로서<br>
  EffectManager에 현재 IEffect의 ID와 임시용 람다 함수를 Move<br>

- 자기 등록형 static 구조체를 전역 변수로 사용하지만 비교적 가벼운편<br>
  (오버헤드를 더욱 줄이고 싶다면 function 대신 함수 포인터 이용을 고려 가능)<br>


실제 Effect 코드 예시<br>

```
// Heal.h

#pragma once
#include "EffectCore.h"

struct HealEffect : IEffect {
    static const char* Id() { return "Heal"; } // 이 부분의 고정 string?
    bool Apply(const EffectSpec& s, EffectContext& ctx) override;
};

--- 

// Heal.cpp
#include "Heal.h"
#include <algorithm>

bool HealEffect::Apply(const EffectSpec& s, EffectContext& ctx) {
    // payload나 기본값을 해석하고, HP 회복 로직 수행
    // 조건에 따라 false 등을 반환
    return true;
}


// 이 한 줄로 “Heal” → HealEffect 팩토리 등록
REGISTER_EFFECT_TYPE(HealEffect)
```

- Heal Effect는 매크로 함수를 통해 자기 자신을<br>
  EffectManager에 등록할 수 있음<br>
  (EffectManager는 딱히 수정할 필요가 없다!)<br>

- Apply를 통해 본인의 효과만을 적용<br>
  
Effect 사용 예시<br>

```
// 실행부
bool RunEffect(const EffectSpec& spec, EffectContext& ctx) {
    if (auto e = EffectRegistry::I().Create(spec.id)) {
        return e->Apply(spec, ctx); // 생성된 객체로 실제 효과 적용
    }
    return false; // 등록 안 된 효과
}

```

- 실제 사용시 Create를 통해 만들어진 Unique_ptr을 받고<br>
  그것을 통해 IEffect->Apply를 적용한다<br>
  (다형성)<br>

- 키값이 잘못된 경우는 실패처리<br>

- Unique_ptr 이기에 스코프를 벗어날시 메모리 해제<br>

### 정리

- 장점<br>
  - 확장성 우수 : 새 Effect 추가 시, 기존 코드의 수정이 없음<br>
  - 낮은 결합도 : 각 클래스가 독립적이기에 테스트에 용이<br>

- 단점<br>
  - 문자열 오타 등으로 인한 오류 발생 가능성 존재 <br>
  : "Heeel" 같은 오류 발생 가능(이를 컴파일 시간에 발견하기 어려움)<br>
  - 룩업 비용 : map 사용에 따른 해시/비교 오버헤드 가능성(다만 보통은 미미한 편)<br>

가능하면 도전해보고 싶은 구조이다<br>
특히, Enum은 구현하기 쉽지만 잠재적으로는<br>
비대화될 가능성이 크기에 확장성이 높은 이 방식으로 도전을 해보고 싶은 마음이 큰 편<br>

## Factory 패턴

마지막으로 Factory 패턴에 대하여 조금 더 정리를 하자<br>

- '생성'과 관련된 디자인 패턴<br>
- 'New'와 관련된 코드를 '캡슐화'하여<br>
  사용자는 '구체적인 클래스' 이름을 몰라도<br>
  객체를 얻을 수 있음<br>
- 어떤 객체를 생성할지 '팩토리 클래스/메서드'가 결정<br>

예시 코드<br>

```
class Effect {
public: virtual void Apply() = 0;
};

class HealEffect : public Effect { ... };
class DamageEffect : public Effect { ... };

class EffectFactory {
public:
    static std::unique_ptr<Effect> Create(const std::string& type) {
        if (type == "Heal")   return std::make_unique<HealEffect>();
        if (type == "Damage") return std::make_unique<DamageEffect>();
        return nullptr;
    }
};
```

이 부분은 사실상 Switch에 가까운 방식이지만<br>
'분기'를 통해 사용할 객체를 return 한다<br>

전통적으로 Factory 패턴의 사용 이유는<br>

- 객체 생성의 책임을 '분리'<br>
- 확장의 용이(호출하는 측면의 코드는 변함이 없는 편)<br>
  (+ Map/ Register 방식은 Switch도 없다)<br>
- 유연한 생성 정책으로 '풀링/싱글톤/캐싱' 등의<br>
  다양한 로직 커스텀 가능<br>

등이 존재한다<br>
