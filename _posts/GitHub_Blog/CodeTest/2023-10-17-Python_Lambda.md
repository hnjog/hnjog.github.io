---
title: "람다(Python)"
last_modified_at: "2023-10-17T22:00:00"
categories:
  - 파이썬
tags:
  - 파이썬
  - 람다식
---

## 람다 함수(Lambda Function)
  '간단한 함수'를 선언없이 사용하는 방식<br>

  ```
    lambda arguments: expression
  ```

  위처럼 lambda 키워드,<br>
  arguments(매개변수),<br>
  expression(표현)<br>
  으로만 구성된다

  람다식은 간단한 내용을 구현할 때,<br>
  '함수를 만들어야...?'하나 싶을때,<br>
  주로 사용하면 유용한 기능이다.<br><br>

  다만, '함수'의 이름이 없기에<br>
  '가독성'이 떨어질 수 있고,<br>
  이로 인해 '복잡한 로직'을 사용하는 경우라면<br>
  일반적인 함수 선언이 읽기에 더 좋다<br><br>

  또한, '함수'이기에 반복 호출에 따른<br>
  성능 저하를 염두해 두어야 하고<br>

  다른 코드에서도 사용한다면,<br>
  당연히 외부 함수로 추출하는 것이<br>
  코드 재사용성이 높다<br>

  그래도 간편하기에 sort 의 key 함수 등으로<br> 만들어 사용하기 편리하다<br>
  
  ```
    # 결과 문자열 배열을 람다식으로 정렬
    # 길이, 사전순서
    resultString.sort(key= lambda item : (len(item),item))
  ```
