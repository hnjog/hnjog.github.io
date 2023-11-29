---
title: "핀토스 1주차 - Alarm Clock"
last_modified_at: "2023-11-25T16:20:00"
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

## Alarm Clock
 Part 1 : Threads <br>

 1주차의 과제는 총 3개인데<br>
 - Alarm Clock
 - Priority Scheduling
 - Advanced Scheduler

 이며,<br>
 현재 진행하는 과제는 Alarm Clock이다<br>

 Alarm은 '호출한 프로세스를 정해진 시간 후에 다시 시작하는 커널 내부 함수' 이다<br>

 처음에는 'Busy waiting' 방식으로 구현되어 있고<br>
 첫번째 과제는 이를 'Sleep/Wake Up' 방식으로 다시 구현하는 것이다<br>

 - Busy Waiting<br>
  : Thread 가 CPU를 점유하면서 대기하고 있는 상태<br>
    CPU 자원이 낭비되며, 소모 전력이 불필요하게 낭비된다<br>

 - Sleep / Wake Up<br>
  Sleep : 일시적으로 활동을 중단하고 CPU를 쉬게 한다<br>
  Wake Up : Sleep() 상태에서 깨어나 작동하는 상태<br>

 기존 구현 방식은<br>
 timer Sleep() 함수에서<br>
 while()을 통해, 지속적으로 스레드를<br>
 문맥교환 해주고 있다<br>

 첫 시도<br>
 : 현재 실행 중인 Thread를 그냥 Wait 상태로 만들고<br>
   일정 시간이 지난 이후 다시 RUNNING으로 바꾸어 주기<br>
  
 당연하겠지만,<br>
 존재하는 스레드 중 하나는 Running이며,<br>
 그에 따른 수 많은 Assert 에 걸려 장렬히 산화하였다<br>

 그에 따라<br>
 각종 pdf 및 같은 정글러들끼리의 아이디어 교환 등을 통해<br>
 '새로운 리스트' 인 Sleep List 를 선언하고 그에 따른 제어를 하는 방향으로<br>
 시도하였다<br>

 해당 시도 방식 중 깜빡한 실수 및 인식한 점<br>
 - 인터럽트를 제한해야 하는 경우에 대한 예외처리를 하지 않았던 점<br>
 - list remove 혹은 list push_back 을 호출한후 '다음' 노드로 넘어갔던 점<br>
   (이중 연결 리스트 기준이며, 디버깅으로 보니 이상한 곳을 가리킨다는 것을 뒤늦게 알았다)<br>
 - timer 의 tick 전역변수는 하드웨어 인터럽트로 인해 증가한다(Timer Interrupt)

![Alarm_Pass](https://github.com/hnjog/hnjog.github.io/assets/43630972/a3f6723b-a65a-485f-98f0-721a6885e4a4)

 [GitHub] : <https://github.com/hnjog/pintos-kaist/tree/hnjog>