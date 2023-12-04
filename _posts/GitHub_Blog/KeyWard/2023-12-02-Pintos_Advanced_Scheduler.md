---
title: "핀토스 1주차 - Advanced_Scheduler"
last_modified_at: "2023-12-02T14:00:00"
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

## Advanced Scheduler
 Part 1 : Threads <br>

 1주차의 과제는 총 3개인데<br>
 - Alarm Clock
 - Priority Scheduling
 - Advanced Scheduler

 그 중 마지막인 Advanced Scheduler 에 관한 것이다<br>

 참고한 pdf 파일에서는<br>
 다수의 '큐'를 활용하는 방식 대신<br>
 하나의 '큐'를 활용하고,<br>
 MLFQ의 '스케쥴링' 방식을 차용하여 작업을 진행하였다<br>

 일반적인 MLFQ의 경우,<br>
 다음의 규칙에 따라 작업이 스케쥴링 된다<br>
 - 우선순위가 A > B 인 경우, A가 실행되고 B는 실행되지 않음
 - 우선순위가 A == B 인 경우, A 와 B 는 Round Robin 방식으로 실행된다
 - 작업이 시스템에 들어오면, 최상위 우선순위 큐에 배치된다
 - 작업에게 주어진 CPU 총 사용량을 모두 소진하였다면,<br>
      작업의 우선순위가 감소한다(한 단계 아래 큐로 이동)<br>
 - 일정 주기가 지난 후, 시스템 내의 모든 작업을 최상위 큐로 이동시킨다<br>

 이러한 규칙을 통해<br>
 '응답성'과 '기아 상태'의 예방을 고려하는 스케쥴링 방식이다<br>

 작업 큐 하나로 구현한 방식은<br>
 작업에 '우선순위'를 부여하되,<br>
 '현재 상황'에 맞도록 적절히 우선순위를 조정하는 것이다<br>
 (우선순위 기반의 스케쥴링 방식이다)<br>
 ~~(현재 구현한 것이 이름을 붙이자면 Single Level Feedback Queue 가 아닐까)~~<br>
 

 이러한 'Feedback'을 주기 위하여 고려되는 요소는<br>
 recent_cpu , load_average, nice 등이 존재한다<br>
 (우선순위를 시간에 따라 조정하기에 이전에 <br>
 구현한 Priority Donation 관련 부분은 Feedback 환경에선 무시되도록 한다)<br>

 priority : 우선순위<br>
 priority = PRI_MAX - (recent_cpu /4) - (nice * 2) <br>
 현재 자신의 'CPU' 사용량과 nice 수치에 따라 정해진다<br>

 nice : 스레드의 '양보 성향' 수치<br>
 ~~(설명으로는 CPU를 '양보'하기 때문에 'nice'라는 용어가 붙었다나...)~~<br>
 -20~20 의 수치를 가지며,<br>
 수치가 낮다면(음수), 높은 우선순위를 가지며,<br>
 수치가 높으면(양수), 낮은 우선순위를 가지게 된다<br>

 recent_cpu : 해당 스레드의 '최근' CPU 작업 소요 시간<br>
 타이머로 현재 진행중인 스레드의 recent_cpu를 증가시키며,<br>
 decay_factor (감쇠) 를 통해 매초마다 recent_cpu를 감소시킨다<br>
 또한, nice를 더하여 recent_cpu를 조정한다<br>

 recent_cpu = decay * recent_cpu + nice<br>
 
 decay : 감쇠<br>
 무거운 부하가 걸릴수록 감쇠는 1에 가까워지며,<br>
 가벼울수록 decay는 0에 가까워진다<br>
 (이는 recent_cpu에 영향을 주며<br>
 현재 시스템의 부하가 클수록, recent_cpu의 값이 커져<br>
 현재 스레드의 우선순위를 낮추어 스레드 교환을 유도한다)<br>

 decay = (2 * load_average) / (2 * load_average + 1)<br>

 load_average : 최근 1분동안 수행이 준비된 작업의 평균<br>
 (이는 지수 가중 이동 평균 (Exponetially Weighted Moving Average) 방식을 사용하여 계산됨)<br>

 EMA(t) = (1 - a) * EMA(t-1) + a * x(t)<br>
 x(t) : 현재 시점의 데이터 포인트<br>
 a : 스무딩 파라미터 (데이터에 부여되는 가중치를 제어하는 값이며 0~1 사이의 값)<br>

 load_average = (59/60) * load_average + (1/60) * ready_threads<br>

 load_average 의 ready_thread는<br>
 현재 '실행이 준비된' 프로세스를 의미한다<br>

 load_average는 ready_thread에 영향을 받되,<br>
 기존의 값을 유지하기에, 데이터의 변화에 따른 평균이 완화되어 작동한다<br>
 (대기하는 스레드가 많을수록 load_average는 점점 늘어나며,<br>
 대기하는 스레드가 없다면, load_average는 점점 줄어든다)<br>
 => 이로 인해 decay는 1에 가까워지며,<br>
    recent_cpu에 영향을 준다<br>
    (nice 수치가 음수가 아니라면 사실상 recent_cpu는 점점 늘어나게 된다)<br>

 매 틱 마다, recent_cpu는 1 증가하고,<br>
 1초가 지난 경우, load_avg를 재계산하며 동시에, 모든 스레드의 recent_cpu와<br>
 priority를 재계산한다<br>
 또한 매 4틱마다, 현재 cpu를 사용하는 스레드의 우선순위를 재계산한다<br>

 위에서 제시된 공식들을 보면<br>
 recent_cpu, load_average, decay factor 등은 소수가 필요한 실수이다<br>
 그러나,<br>
 핀토스에서는 커널에서 부동소수점 연산을 지원하지 않기에<br>
 임의로 소수점 연산을 직업 할 필요가 있다<br>

 따라서 int (32비트)를<br>
 1 / 17 / 14 비트로 쪼개어 각각<br>
 부호 / 정수 / 소수 부분으로 표현한다<br>

 (가장 좌측의 1 부호 비트의 경우,<br>
 기존 int에서도 부호 비트로 사용되고 있기에<br>
 연산 시, 의식해서 수정할 부분은 없었다)<br>

 다만, 여전히 선언하는 변수의 타입은 'int' 였으므로<br>
 typedef 등으로 int를 fp 등으로 바꾼다던가<br>
 변수명을 fp_ 혹은 real_ 등으로 선언하여<br>
 해당 변수명이 '정수'인지 '실수'인지 판단하는 것이 생각보다 중요하였다<br>

 'idle_thread'에 관한 예외처리가 필요한데<br>
 이 스레드는 'CPU'가 아무런 작업을 하지 않을 때 수행되는 스레드이다<br>
 따라서, 시스템의 '유휴 상태'를 표현하는 스레드이며,<br>
 CPU를 점유하지 않고 대기하는 역할만을 수행한다<br>

 이에 따라, idle thread는 <br>
 recent_cpu 나 priority에 대하여 고려되지 않으며,<br>
 load_average의 ready_thread의 개수에 포함되지 않는다<br>
 
 <br><br>
![Advance_Scheduel](https://github.com/hnjog/hnjog.github.io/assets/43630972/e7021f42-cf58-4cfa-ad72-26e92fd11bc1)

 [GitHub] : <https://github.com/hnjog/pintos-kaist/tree/hnjog>