---
title: "핀토스 1주차 - Priority_Scheduling Donation"
last_modified_at: "2023-11-29T14:40:00"
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

 그 중 Donation에 관련된 문제를 해결하는 방식이다<br>
 Donation에 대한 구현 내용은<br>
 - priority donation<br>
 - nested donation<br>
 - multiple donation<br>

 이며, 이들은 'priority inversion' 상태를 해결하기 위하여 고안된 개념이다<br>
 (요약하자면, lock의 요청으로 인하여 '우선순위'가 높은 스레드가 <br>
 오히려 늦게 실행되거나, 기아상태가 되는 상태이다)<br>

 그렇기에, priority donation 개념이 등장하는데<br>
 이는, 현재 스레드(A)가 특정한 'lock'을 원하는 상황인데,<br>
 해당하는 lock을 가진 스레드(B)의 우선순위가 A보다 낮다면,<br>
 B의 우선순위를 일시적으로 A까지 올려주도록 한다<br>
 (이후, lock_release 등이 호출되면 B는 원래의 우선순위대로 돌아가고,<br>
 A가 필요한 락을 가지고 먼저 수행된다)<br>

 nested는 'lock A'가 필요한 쓰레드(t1)가<br>
 해당 락을 가진 쓰레드(t2)의 우선순위를 올려주려 갔는데<br>
 t2는 'lock B'를 필요로 하며, 그 락은 t3가 가지고 있는 상황이다<br>
 이 경우, t1의 우선순위가 높은 경우,<br>
 t1의 우선순위를 t2와 t3에도 부여한다<br>
 (t3가 락 B를 해제하고, t2가 락 A를 해제하고<br>
 이후 t1이 먼저 쓰레드를 종료하게 된다)<br>

 multiple도 상황은 비슷하다<br>
 다만 이 경우는<br>
 t1이 lock A, lock B를 모두 가지고 있는 경우<br>
 lock A를 t2가, lock B를 t3가 원하는 경우<br>
 t1은 둘 중 높은 priority를 donation 받는다<br>

 구현시 깜빡한 점<br>
 - 별도의 listelem을 사용하는 경우, list_entry() 의 3번째 인자는 해당 변수의 이름으로 해야한다
 - lock을 해제할 때, 먼저 '대기리스트'에서 '해당 락'을 요구하는<br>
   요소들을 제거한다<br>
 - set_priority로 '초기화할 priority'를 수정해야 한다<br>
 - NULL 체크는 좋으나, break를 할 위치를 잘 못 잡으면 들어오는것 조차 못한다<br>
 - lock이 필요한 경우는 결국 해당 lock 구조체의 wait 에서 기다리게 된다<br>
 - cond 에 집어넣는 세마포어는 사실상 mutex에 가깝다<br>
   (1개 요소의 삽입 후,추가적인 요소를 집어넣지 않으며, signal을 기다림)<br>

 다음 구현할 요소는 MLFQS 일 것 같다

 ![priority_SD](https://private-user-images.githubusercontent.com/43630972/286484997-36de32ca-e070-4605-9609-1418a1f2f7c9.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDEyMzU1MDAsIm5iZiI6MTcwMTIzNTIwMCwicGF0aCI6Ii80MzYzMDk3Mi8yODY0ODQ5OTctMzZkZTMyY2EtZTA3MC00NjA1LTk2MDktMTQxOGExZjJmN2M5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzExMjklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMTI5VDA1MjAwMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU3MDVkOWVkMGY3MGJmNTRiYWEyMzgyN2YyYzFlYjE0YjU1MTFlZTFmNTk2NTQ1M2I3OGM3ZjBhNjcwY2M3OWYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.AecubZupftiLjSVM8PDVHhpf-vx7CQ0dmrRSe90ovEY)

 [GitHub] : <https://github.com/hnjog/pintos-kaist/tree/hnjog>