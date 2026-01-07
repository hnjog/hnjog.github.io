---
title: "Context Switching(문맥 전환)"
date : "2026-01-07 14:30:00 +0900"
last_modified_at: "2026-01-07T14:30:00"
categories:
  - OS
tags:
  - OS
  - Context Switching
  - 문맥 전환
---

## Context Switching(문맥 전환)

[![Image](https://github.com/user-attachments/assets/b9cf70f4-c732-43ed-b35a-08faf5960ed2)](https://github.com/user-attachments/assets/b9cf70f4-c732-43ed-b35a-08faf5960ed2){: .image-popup}<br>

- 컨텍스트 스위칭은 CPU가 실행 중인 프로세스나 스레드의 상태(`Context`)를 저장하고<br>
  다음 작업의 상태를 불러와 실행을 재개하는 과정(`Switching`)<br>

- OS가 '멀티태스킹'을 진행하여<br>
  여러 프로세스를 동시에 처리하기 위하여 필요!<br>

- 프로세스는 PCB (Process Control Block)라는 것을 통해 정보를 저장<br>
  - 프로세스 식별자 (PID)<br>
  - 프로레스의 상태 (ex : New, Ready, Running, Waiting)<br>
  - 프로그램 카운터(PC) 같은 레지스터의 값들 (다음 실행할 명령어 주소)<br>
  - 메모리 관리 정보<br>
    - Page Table 등<br>
  - 그 외에도 스케줄링 정보, I/O 상태 등등<br>

- 스레드는 TCB (Thread Control Block)를 통해 정보 저장<br>
  - 스레드 식별자 (TID)<br>
  - 스레드 상태 (New,Running,Ready,Waiting)<br>
  - PC : 프로그램 카운터<br>
  - 스택 포인터 : 각 스레드가 '독립적인 스택 영역'을 가지므로<br>
  - PCB를 가리키는 포인터 : 공유 자원에 접근하기 위함<br>

- 이러한 PCB, TCB는 '커널 영역'에 존재함<br>
  - '보안'과 '안정성' 때문<br>

### 프로세스의 문맥 전환이 비싼 이유

[![Image](https://github.com/user-attachments/assets/4bf524b6-05b6-48c4-b5c1-7f23e4483614)](https://github.com/user-attachments/assets/4bf524b6-05b6-48c4-b5c1-7f23e4483614){: .image-popup}<br>

- 프로세스는 독립적인 '**가상 메모리 공간**'을 가짐<br>
  - 보호 + 보안<br>

- 그렇기에 이러한 '가상 메모리 공간'에서 '물리 메모리 공간'으로 실제 주소를 찾아가기 위해<br>
  **페이지 테이블**이라는 매핑된 테이블이 필요함<br>
  - 가상 메모리 공간의 일정 크기 : 페이지 <-> 물리 메모리 공간의 일정 크기 : 프레임<br>

- 이러한 '주소 변환' 연산을 빠르게 하기 위하여<br>
  TLB(Translation Lookaside Buffer) 캐시를 두어<br>
  '가상 주소' 와 '물리 주소'의 쌍을 저장(캐싱)<br>
  - TLB는 실제 물리 장치로서 존재하는 페이지 테이블용 캐시!<br>
  - PageTable은 DRAM에 존재하며 CPU는 <br>
    PTBR(Page Table Base Register) 레지스터 (or CR3 레지스터)를 통해 접근<br>
    (매 주소 확인마다 DRAM(메인 메모리)까지 내려가는 것은 느리므로!)<br>

- 문맥 전환 자체는 'PCB' 포인터를 변경하는 등 비교적 비싸지는 않지만<br>
  *TLB Flush(무효화) 및 L1/L2/L3 캐시가 재사용되기 힘듦!*<br>
  - TLB와 L1,L2,L3 자체에 '이전 프로세스'에서 사용한 데이터들로 가득 차있는데<br>
    이러한 데이터들은 '현재 프로세스'에게는 무의미한 데이터들<br>
  - 결국 다시 메인 메모리 or 디스크 에서 캐시를 채워나가기 시작해야 함<br>

### 그렇기에 스레드의 문맥 전환은 비교적 저렴하다

[![Image](https://github.com/user-attachments/assets/7d039d66-50f6-4ee9-a2a5-33fa94e5e3fb)](https://github.com/user-attachments/assets/7d039d66-50f6-4ee9-a2a5-33fa94e5e3fb){: .image-popup}<br>

- 여러 스레드들은 하나의 프로세스의 Code/Data/Heap 등을 공유!<br>

- 다만, 스레드들은 독자적인 Stack 영역을 가짐<br>
  - 스레드마다 '독립적인' 함수 호출을 가져야 하기 때문<br>
  - 각자 다른 코드를 실행시킴으로서 '병렬적인 처리'를 가능하게 함<br>

- 그렇기에 스레드의 '전환'은 페이지 테이블을 갈아끼울 필요가 없기에 가벼움!<br>
  - 기본적인 TCB 포인터 변경 같은 CPU 비용 지불<br>
  - 또한 추가적으로 Stack 영역에 대한 캐시가 무효화됨<br>
    (그래도 프로세스의 문맥 전환보단 훨씬 가볍다!)<br>


### TMI - CPU는 스레드만을 실행시키고, 프로세스엔 관심이 없다?

[![Image](https://github.com/user-attachments/assets/ebaa779f-c230-4161-b110-d22b0862b996)](https://github.com/user-attachments/assets/ebaa779f-c230-4161-b110-d22b0862b996){: .image-popup}<br>

- 현대의 OS에서 '프로세스'는 일종의 '컨테이너' 역할을 하며<br>
  스레드는 실제로 일을 처리하는 '일꾼' 역할을 함<br>

- 그렇기에 CPU는 '스레드'를 실행시켜 작업을 진행<br>
  (OS 역시 스케쥴링의 대상이 스레드!)<br>
  - 위 이미지를 정리하자면 다음과 같음<br>
    - OS 스케쥴러가 사용 가능한 CPU 코어에 스레드 할당<br>
    - 각 코어는 특정 시점에 하나의 스레드만 실행<br>
    - 프로세스 내의 여러 스레드가 존재시, 스레드가 여러 코어에서 동시에 실행 가능<br>

- CPU가 작업을 진행하기 위하여<br>
  TCB 내부의 PC를 통해 '명령어 주소'를 가져오고<br>
  PCB 내부의 Page Table Pointer 를 통해 **가상<->물리 메모리 매핑**만 파악<br>

### TMI 2 - OS의 스케쥴링과 인터럽트, 시스템콜?

문맥 전환은<br>
Timer 같은 '인터럽트'나<br>
I/O 대기 같은 '시스템콜'로 인하여 발생<br>

- 인터럽트?<br>
  : CPU에게 보내는 신호의 일종으로서<br>
    이를 통해 현재 작업을 중단하고, 관련 처리 로직을 실행<br>
    - Timer, 키보드 입력 같은 '하드웨어 인터럽트'와<br>
      Page Fault 같은 '소프트웨어 인터럽트'가 존재<br>

- 시스템 콜?<br>
  : 프로세스가 커널에게 요청하는 일종의 인터페이스(함수)<br>
    (정확히는 스레드가 요청하지만 OS의 입장에선 그 스레드 소속인 '프로세스'가 요청한 것으로 인식)<br>
    - 자원 요청 등의 일을 OS에게 전담<br>
      -> 보안, 안정성, 자원 관리에 대한 추상화를 제공<br>

[![Image](https://github.com/user-attachments/assets/f81de367-7328-4ac2-8791-fecb66059b83)](https://github.com/user-attachments/assets/f81de367-7328-4ac2-8791-fecb66059b83){: .image-popup}<br>

- Timer 인터럽트?<br>
  - OS의 스케쥴링을 위한 인터럽트<br>
    (현재 CPU를 독점하는 프로그램에게서 일정 시간마다 CPU 제어권을 OS가 가져옴)<br>
  - 실제 '하드웨어'적 장치(타이머 칩)를 통해 신호를 발생시키기에 '하드웨어 인터럽트'로 분류<br>

- OS는 이러한 Timer 인터럽트를 이용하여<br>
  Time Slicing이라는 '정책'을 실행 가능함<br>
  - 일정한 시간마다 Thread 들의 작업을 번갈아가면서 진행<br>
  
### TMI 3 - 커널?

[![Image](https://github.com/user-attachments/assets/a86bcfd8-658b-4cbe-a902-8dc36909c1ff)](https://github.com/user-attachments/assets/a86bcfd8-658b-4cbe-a902-8dc36909c1ff){: .image-popup}<br>

- OS도 '프로그램'!<br>
  - 그렇기에 '메모리'에 항상 상주하며 '핵심 기능'을 수행하는 부분을<br>
    '커널'이라 함<br>

- 커널의 역할<br>
  - 자원 관리 : CPU 스케쥴링, 메모리 할당/해제<br>
  - 하드웨어 제어 : 키보드,마우스 등<br>
  - 보안과 보호 : 프로세스 간 격리, 접근 권한 파악<br>

- 커널 모드? 유저모드?<br>
  - CPU가 보안을 위해 실행 권한을 나눠둔 것<br>
    - 커널 모드<br>
      : 모든 메모리 및 명령어 사용 가능<br>
        OS의 핵심 코드를 실행 가능<br>
        오류 발생시, 시스템이 전체적으로 다운<br>
    - 유저 모드<br>
      : 제한적인 접근만 허용<br>
        프로그래머가 제작한 프로그램 등을 실행<br>
        오류 발생 시, 해당 프로그램만 Kill<br>

- 커널의 위치<br>
  : 모든 프로세스의 가상 메모리 상단부에 위치함<br>

- 같은 가상 메모리에 존재해야 하는 이유?<br>
  - 같은 프로세스에 존재해야 페이지 테이블을 교체하지 않고 커널 기능 이용 가능<br>
    - 커널 접근시마다 페이지 테이블을 바꿔야 한다면 성능 하락<br>

- 이러한 커널의 위치(가상/물리)는<br>
  모든 프로세스가 같은 주소를 가지게 됨<br>
  (어차피 커널은 모든 프로세스가 공유하므로)<br>
  - PageTable을 효율적으로 만들기 위해서!<br>
    + TLB 등의 캐시 중 커널 접근에 대한 것은 무효화되지 않음<br>
    