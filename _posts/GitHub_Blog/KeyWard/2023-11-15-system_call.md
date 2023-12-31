---
title: "system call"
last_modified_at: "2023-11-15T10:30:00"
categories:
  - 크래프톤 정글
  - CS
tags:
  - 크래프톤 정글
  - CS
  - 운영 체제
---

## 운영 체제의 모드
 - User mode<br>
  일반적인 프로그램 실행 <br>
  응용프로그램은 시스템 자원에 직접 접근 불가<br>
  제한된 명령어 집합 실행 가능<br>
  메모리 일부 영역에 접근 가능<br>
  <br>
  제한된 운영체제 서비스에 접근이 가능<br>
  => 응용 프로그램 간의 상호 간섭의<br> 최소화를 통해 시스템의 안정성 유지<br>
  => 사용자 모드에서 발생한 오류가,<br>
  커널 모드에서 실행 중인 OS에 영향을 주지 않도록 함<br>
  <br>
  인터럽트(interrupt) 발생, 혹은<br>
  시스템 콜(system call) 호출 시 커널 모드로 전환<br>
  (User -> Kernel)
  <br><br>
 - Kernel mode<br>
 프로그램의 현재 CPU 상태를 저장함<br>
 시스템 자원에 대한 완전한 엑세스<br>
 (하드웨어, 메모리, 입출력 장치 등)<br>
 모든 명령어 실행 가능<br>
 <br>
 시스템 콜을 통해 User mode에서 요청된 서비스 처리<br>
 <br>
 시스템의 안전 및 보안을 담당<br>
 <br>
 CPU에서 커널 코드가 실행<br>
 처리 완료 시, 중단되었던 프로그램의 CPU 상태 복원<br>
 이후 통제권을 프로그램에 반환<br>
 (Kernel -> User)<br>

 - 커널(kernel)<br>
  : 운영체제의 핵심, 시스템 전반을 관리/감독하는 역할<br>
    하드웨어와 관련된 작업을 직접 수행<br>
    (운영 체제 시스템의 일부)<br>
    (커널 모드는 '운영체제'의 '실행 레벨'을 나타냄)

## 인터럽트 (Interrupt)
 시스템에서 발생한 다양한 종류의 이벤트 혹은<br>
 그런 이벤트를 알리는 메커니즘

 인터럽트의 종류<br>
 - 전원(Power)에 문제가 발생
 - I/O 작업 완료
 - Timer 관련
 - 0으로 나눔             (Trap 이라고도 함- 프로그램 레벨에서 발생 가능)
 - 잘못된 메모리 공간 접근 (Trap 이라고도 함 - 프로그램 레벨에서 발생 가능)
 - ...등등

 위의 예시를 크게 2종류로 나누는데,<br>
 1. 하드웨어 인터럽트<br>
    : 주로 외부 이벤트(I/O,Timer 등)에 의해 발생<br>
      수행 중인 작업 중단 후, 인터럽트 처리 실행
 2. 소프트웨어 인터럽트(트랩)<br>
    : 소프트웨어의 명령에 의해 발생하는 인터럽트<br>
      예외 사항, 혹은 시스템 콜과 관련있음

 인터럽트 발생 시, 즉각적으로<br>
 인터럽트 처리를 위해 커널 코드를 커널 모드에서 실행<br>

## 시스템 콜(System Call)
 프로그램이 OS 커널이 제공하는 서비스를<br>
 이용하고 싶을 때, 시스템 콜을 통하여 실행<br>

 시스템 콜의 종류<br>
 - 프로세스/스레드 관련
 - 파일 I/O 관련
 - 소켓 관련
 - 장치(device) 관련
 - 프로세스 통신 관련

 시스템 콜 발생 시,<br>
 (일반적으로는 어셈블리 명령어인 'int', 'syscall' 등)<br>
 커널 모드로 전환되며, 요청된 작업을 수행한다<br>
 이후, 결과나 필요 데이터를 반환한다

 - 트랩(Trap)<br>
   : 예외 상황, 특정 이벤트 발생 시, 발생하는 '인터럽트'의 일종<br>
   주로 2가지 목적으로 사용되는데<br><br>
   '시스템 콜'을 발생시켜 운영체제에게 특별한 작업을<br>
   요청하거나<br><br>
   '예외'를 발생시켜, 예외 상황에 대한<br>
   운영체제의 처리를 실행하게 한다<br>
   (ex : 종료 or 예외 처리 루틴)
 
## 프로그래밍과 시스템 콜
 하드웨어 혹은 시스템 관련 기능은<br>
 어떤 프로그램이라도 시스템 콜을 통하여만 이용 가능하다<br>

 보통 직접 OS 시스템 콜을 사용하진 않음<br>
 그러나 파일 I/O, 네트워크, 프로세스/스레드 기능을<br>
 프로그래밍 언어가<br> 시스템 콜을 포장(wrapping)하여 제공한다<br>

## 시스템 콜의 사용 이유
 일단 기본적으론 '사용자'가 직접적으로<br>
 시스템 자원(하드웨어,메모리,파일 등)에 접근하지 못하게<br>
 함으로서 보안과 안정성을 유지하기 위한 것이 전제이다<br>

 '응용 프로그램'이 필요 시, '운영 체제'에게 필요한 작업을<br>
 '요청'하는 것이 '시스템 콜'의 개념이다<br>

 이 방식으로, '보안과 안정성'이 보장되고,<br>
 사용자 입장에서는 '추상화'와 '편의성'을 얻을 수 있다<br>
 (운영체제에게 시스템 콜을 하는것으로 원하는 작업 수행)<br>

 