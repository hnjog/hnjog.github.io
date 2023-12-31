---
title: "demand zero memory"
last_modified_at: "2023-11-13T20:30:00"
categories:
  - 크래프톤 정글
  - CS
tags:
  - 크래프톤 정글
  - CS
  - 메모리 관리
  - 운영 체제
---

## demand zero memory
 운영 체제에서 사용되는 메모리 관리 방식 중 하나이며<br>
 가상 메모리를 효율적으로 관리할 수 있는 방식이다<br>
 (구현 방식에 따라 사용되지 않기도 한다)<br>

 해당 전략의 요점은 <br>
 '정말로 필요할 때까지는 주지 않겠다'라는<br>
 개념이다<br>
 (메모리 할당을 '지연'시킨다)<br>
 (받은 쪽은 요구했는데 '0' 메모리를 받았으니<br>
 이런 작명이 되었을수도?)

 페이징 개념에 따라<br>
 'Page fault'가 발생한 경우,<br>
 운영 체제는 페이지 테이블을 업데이트 하지만,<br>
 해당 운영체제가 'demand zero memory' 전략을 사용하는 경우<br>
 실제로 '메인 메모리'(물리적 메모리)에<br>
 할당하지는 않는다<br>

 해당 페이지에 '실제로' 접근 하려 한다면,<br>
 운영체제는 이 때, 메모리를 할당하며,<br>
 이 때, 페이지를 초기화 시킨다<br>

 이러한 방식은 '필요 시'에만 메인 메모리에<br>
 할당하기에 효율적인 메모리 사용 방식을<br>
 지향할 수 있다<br>

 다만, '초기화'가 지연되기에 <br>
 발생하는 작업의 지연 및 오버헤드가 발생할 수도 있으니<br>
 해당 내용을 유의하는 것이 좋다<br>
 (그렇다고 OS가 반드시 이 전략을 사용하는 것도 아니다)