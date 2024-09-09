---
title: "Extension Method"
last_modified_at: "2024-09-09T15:30:00"
categories:
  - C#
tags:
  - Extension
---

## Extension Method(확장 메서드)
 기존 클래스에 '새로운 메서드'를 추가하는 문법이나,<br>
 기존 클래스를 수정하지 않기에 각종 유틸리티 메서드 를 작성할때<br>
 특히 유용하다<br>

 - 조건<br>
   1. 정적 클래스 내에 정의
   2. 정적 메서드로 선언
   3. 확장 메서드의 첫 매개변수는 this 키워드를 사용하여 Type을 지정

 ```
 // Unity

 public static class Extension
{
    public static T GetOrAddComponent<T>(this GameObject go) where T : UnityEngine.Component
    {
        return Util.GetOrAddComponent<T>(go);
    }

    public static void BindEvent(this GameObject go, Action<PointerEventData> action = null, Define.EUIEvent type = Define.EUIEvent.Click)
    {
        // BindEvent : static
        UI_Base.BindEvent(go, action, type);
    }

    public static bool IsValid(this GameObject go)
    {
        return go != null && go.activeSelf;
    }

    public static void DestroyChilds(this GameObject go)
    {
        foreach (Transform child in go.transform)
            Managers.Resource.Destroy(child.gameObject);
    }
    ...
}
 ```
  위 클래스의 확장 메서드 사용 시,<br>
  별도로 this 타입 쪽에서 Extension. 으로 사용할 필요가 없기에<br>
  Utility 용 클래스를 하나 만들어 사용하기 편리하다<br>
  
  '기존 클래스'를 수정하지 않아도 된다는 점은 곧<br>
  내부를 수정할 수 없는 '외부 라이브러리'의 클래스 등을 사용할 때도 용이하다<br>

  다만 사용 시 주의할 점으로는<br>
  1. 이미 해당 클래스에 확장 메서드와 동일한 이름이 있는 경우,<br>
     해당 클래스가 호출됨
 2. '해당 클래스'의 메서드 처럼 보이므로, 디버깅 시 혼동을 줄 수 있음<br>
 3. 타입을 잘못 지정했을 경우, '런타임'에 문제를 발생<br>

 이와 같은 부분은 주의하여 사용해야 한다<br>