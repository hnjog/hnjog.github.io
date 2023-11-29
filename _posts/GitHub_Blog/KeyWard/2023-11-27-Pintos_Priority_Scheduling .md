---
title: "핀토스 1주차 - Priority_Scheduling"
last_modified_at: "2023-11-27T11:40:00"
categories:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
  - 핀토스
tags:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
---

## Priority Scheduling
 Part 1 : Threads <br>

 1주차의 과제는 총 3개인데<br>
 - Alarm Clock
 - Priority Scheduling
 - Advanced Scheduler

 이며,<br>
 현재 진행하는 과제는 Priority Scheduling이다<br>

 Priority Scheduling은<br>
 해당 thread에 '우선순위'를 부여하여,<br>
 현재 작업 중인 스레드보다 새로 들어온 스레드의 우선순위가<br>
 더 높은 경우, 현재의 스레드를 ready_list에 밀어넣고<br>
 새로운 스레드를 작업하는 방식이다<br>

 말 그대로, '우선순위'가 더 높은 작업을 먼저 처리하는 방식이다<br>

 구현 방식은 thread의 priority 필드를<br>
 실제로 사용하게 만들며,<br>
 이에 따라 ready_list에 삽입될 때, 우선순위가 높은 것이<br>
 앞에 오도록 하는 '내림차순' 방식을 이용하였다<br>

 이에 따라,<br>
 thread_create, thread_yield, thread_unblock,<br>
 thread_set_priority 함수를 수정하였다<br>

 또한, list의 compare 용 함수인 <br>
 cmp_priority 함수를 만들어<br>
 첫번째 인자가 두번째 인자보다 높은 경우 true를 반환하도록 하였다<br>
 (내림차순을 위함)<br>

 구현시 깜빡한 점<br>
 - set_priority 함수에서 처음에는<br>
   ready_list를 sort하는 방식이었는데<br>
   생각해보니 thread_current의 값을 수정하는데<br>
   ready_list만 sort하는 방식은 별로 좋지 못한것 같았다<br>
   이에 따라 바뀐 값과 list의 첫번째 값(내림차순이기에 이녀석이 제일 큼)<br>
   과 비교하여 그에 따라 수정<br>
   (애초에 insert 할 때, orderer 를 호출하고 있었기에<br>
   굳이 sort를 호출할 필요는 없었다)<br>
 - 자꾸 에러가 나기에 확인했더니,<br>
   cmp 함수에서 a->priority 와 b 자체를 비교하고 있었다...<br>
   b->priority로 수정 (항상 오타 조심...)<br>
 - 특정 test case를 통과하지 못하여 확인하였는데,<br>
   list_front 같은 함수는 list_empty인 상황을 따로 검사하지 않았다<br>
   (null return이 아니라 그냥 assert 해버렸다)<br>
   해당 함수를 호출 시, list_empty를 따로 호출해서 예외처리를 하였다<br>

  다만 priority 부분 중, 현재는<br>
  thread의 ready_list 부분만 처리한 것이며,<br>
  동기화 도구에 따른 문제가 아직 남아있는 상황이다<br>

![Priority_Scheduel](https://github.com/hnjog/hnjog.github.io/assets/43630972/4d59cfa4-ef4c-4389-bb8e-5c9f396bc825)

![Priority_Scheduel_sema1](https://github.com/hnjog/hnjog.github.io/assets/43630972/20e54e5c-ad86-4af8-b4b2-22c4867e9cbe)


 [GitHub] : <https://github.com/hnjog/pintos-kaist/tree/hnjog>