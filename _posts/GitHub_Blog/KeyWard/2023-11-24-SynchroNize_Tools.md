---
title: "동기화 도구"
last_modified_at: "2023-11-24T10:20:00"
categories:
  - 크래프톤 정글
  - CS
  - OS
tags:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
  - 동기화
  - 뮤텍스
  - 세마포어
  - 경쟁 상태
  - 데드락
---

## 임계 영역(Critical Section)
 보통 다중 쓰레드 or 프로세스 환경에서<br>
 공유된 자원(변수 or 자료구조 등)에 접근하는 '코드 영역'을 의미한다<br>
 (공유된 자원과 임계 영역은 다른 개념)<br>
 (공유된 자원은 '데이터','메모리', 변수, 파일 등의 실제 '자원'임)<br>
 ('임계 영역'은 프로그램 내에서 '공유된 자원'에 '접근하는 부분'을 의미함)<br>
 (그렇기에 '코드'로 표현된다)<br>

 '상호 배제'(Mutual Exclusion)의 원칙이 지켜져야 하며,<br>
 그렇지 않으면 '경쟁 상태'가 발생하게 된다<br>

## 경쟁 상태(Race Condition)
 여러 쓰레드 or 프로세스가 '동시에'<br>
 '공유된 자원'에 접근하려 하여 발생하는 형태<br>
 (동시에 '임계 영역'을 실행하려 한다는 의미로도 해석 가능)<br>

 A 쓰레드가 값을 쓰고, '저장'하기 전에<br>
 B 쓰레드가 값을 '다시' 써버려서<br>
 A 쓰레드가 저장한 값이 '정상적'이지 않은 상황을 의미한다<br>

 이를 '비결정적'인 결과라 말하며,<br>
 (정의되지 않은 동작)<br>
 정상적인 시스템 동작을 보장하지 않기에 피해야 한다<br>
 (시스템의 '무결성'을 훼손한다)<br>

 (DB와 같이 데이터의 무결성이 보장되어야 하며,<br>
 다양한 클라이언트가 접근이 가능할 때도 사용되는 표현)<br>

## 뮤텍스(Mutex)
 상호 배제와 동기화를 위한 도구 중 하나<br>

 'lock' 과 'unlock'의 개념을 사용하여<br>
 '임계 영역'에 여러 스레드가 접근하는 것을 제한한다<br>
 (이를 통해, '경쟁 상태'를 예방할 수 있다)<br>
  
## 세마포어(Semaphore)
 뮤텍스와 마찬가지로 상호 배제와 동기화를 위한 도구 중 하나<br>

 프로세스 및 스레드의 '공유된 자원'에 접근을 제어하고 조정하기 위해 사용된다<br>

 P 연산과 V 연산을 통해 이를 시행한다<br>
 init value : 세마포어의 초기화된 용량
 P : 세마포어 값을 감소시키고 이 떄 값이 0(또는 음수)가 되면 대기 상태로 만든다<br>
 V : 세마포어 값을 증가시키고 P로 인해 대기가 된 스레드 중 하나를 실행 상태로 만듦<br>
 (해당 세마포어에서 P로 여러 스레드가 대기 중일 때,<br>
 V가 실행 상태로 만드는 스레드를 예측할 수는 없음)<br>
 (그렇기에 V가 특정한 스레드를 '깨울 것'이라 전제하는 코드는 짜지 않는 것이 좋음)<br>

## 데드락(Dead Lock)
 '교착 상태' 라고도 표현되며,<br>
 위의 '동기화' 도구를 잘못 사용한 경우 발생할 수 있는 문제<br>

 2개 이상의 프로세스나 스레드가 '서로의 작업이 끝나기를 대기'하는 상태를 뜻하며<br>
 말 그대로 무한히 대기하는 상황이다<br>

 데드락의 필요 조건 4가지는<br>
 1. 상호 배제(Mutual Exclusion) : 한 번에 하나의 프로세스나 스레드만이 공유 자원 사용 가능<br>
 2. 보유 및 대기 : 최소한 하나의 자원을 보유하고 있는 상태에서 다른 스레드의 작업을 대기<br>
 3. 비선점 : 서로의 자원을 '뺏을 수 없음'<br>
 4. 순환 대기 : 서로의 자원을 '필요로 하는' 상황<br>

 따라서 데드락을 '해제'하기 위해서는<br>
 위의 조건 중 하나를 제거하거나, 미리 예방하는 방법을 사용<br>
 (미리 예방 : 코드 좀 잘 짜라)<br>
 (뮤텍스나 세마포어가 서로 자원을 얻을 수 없도록 만들면 발생하므로 이 부분을 신경쓰면<br>
 예방 가능)<br>

 데드락의 대처 방식들<br>
  - 회피 : '은행원 알고리즘' 같이 '자원 할당'을 미리 검사하고<br>
           사전에 데드락의 발생을 막는 방식<br>
           (은행원 알고리즘은 '자원 할당'시 <br>
           '할당 가능 여부', '데드락 발생 여부', '시스템 안정성 여부'를 파악 후<br>
           자원을 할당, 제약 조건이 많기에 큰 규모의 시스템에서는 부적합하다)<br>
  - 탐지 및 회복 : 데드락의 발생을 감지하고, 복구한다<br>
           프로세스를 종료시키거나, 점유하는 자원을 다른 프로세스에 할당하는 등의<br>
           방식을 이용한다<br>
  - 무시 : 데드락을 허용하고 방치하는 상태(보통 상업적인 시스템인 경우 선택x)<br>
  

 라이브 락?<br>
 : 각각의 프로세스가 서로의 락을 탐할 때,<br>
   먼저 락을 가진 요소를 '해제'하고 새로운 자원을 취하는 것<br>
   (이 때, 서로 '해제'하고, 새로 받고 난 후,<br>
    또 서로의 락을 탐하기에 계속 반복된다)<br>


