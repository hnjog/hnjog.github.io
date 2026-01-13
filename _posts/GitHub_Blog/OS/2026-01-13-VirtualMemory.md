---
title: "Virtual Memory"
date : "2026-01-13 14:30:00 +0900"
last_modified_at: "2026-01-13T14:30:00"
categories:
  - OS
tags:
  - OS
  - Virtual Memory
  - Page Table
  - Page Table Entry
  - Paging
  - Page Fault
---

## Virtual Memory(가상 메모리)

[![Image](https://github.com/user-attachments/assets/dcc202b5-6672-46df-91ee-d262aa792f13)](https://github.com/user-attachments/assets/dcc202b5-6672-46df-91ee-d262aa792f13){: .image-popup}<br>

메인 메모리인 램과 보조 기억 장치의 조합으로<br>
실제 메인 메모리보다 더 큰 메모리가 있는 것처럼 표현하는 기술<br>

- 각 프로세스의 가상 메모리 공간은 개념상 '연속된 공간'에 존재하지만<br>
  실제로는 물리 메모리(Ram)에 산재하여 존재하거나<br>
  보조 기억 장치에 존재<br>
  (HDD,SSD)<br>

- 이러한 방식을 통해 OS는 다음과 같은 역할 수행 가능<br>
  - 메인 메모리 보다 더 큰 파일의 실행<br>
  - 다른 프로세스와 같이 실행하는 '멀티태스킹'<br>
  - 각 프로세스의 영역을 격리 가능(보호와 고립)<br>

### 페이징 기법

이런식으로 '프로세스'를 조각내어 일정 크기만큼 저장하는 방식을<br>
**페이징 기법**이라 함<br>

- 이런식으로 메인 메모리를 꽉 채워서 사용하기에<br>
  '외부 단편화' 문제를 해결하는 데 유용함<br>
  - 외부 단편화?<br>
    : 메모리 내의 여유 공간이 '작은 조각'으로 나누어져 있어서<br>
      '총 여유 메모리'는 충분하지만, 한번에 할당할 수 있는 양은 부족함<br>
  - 내부 단편화?<br>
    : 할당된 메모리 공간이 실제로 사용 중인 데이터 보다 큰 상태<br>
      '사용되지 않는 할당 메모리 공간'이 있어, 낭비됨<br>
  - 단편화의 해결법<br>
    - 압축 : 메모리 공간을 재배치 하여, 분산된 공간을 하나로 합침<br>
    - 통합 : 단편화로 인해 분산된 메모리를 '인접'한 것끼리 통합<br>

- 이런식으로 프로세스를 조각내기에<br>
  *필요할 때만 가져오는 것*(Demanding Page)이 가져함<br>
  - 전체 로딩할 필요가 없으므로 필요할때만 로딩<br>
  - 실제 코드만 실행할 때, Page Fault를 발생시켜<br>
    필요한 데이터만 메인 메모리에 올림<br>
    (Lazy Loading)<br>
    -> 메모리 사용 효율 극대화!<br>

- 세그멘테이션(Segmentation) 기법?<br>
  : 꼭 필요한 만큼의 데이터를 할당받기에 '내부 단편화'를 해결했지만<br>
    일정한 크기가 아닌 상태로 할당과 해제를 반복하기에<br>
    외부 단편화가 발생함<br>
    - 가상 메모리 할당에 주로 사용<br>

*두 기법의 비교표*<br>

[![Image](https://github.com/user-attachments/assets/a86c0c3e-64ed-4639-a803-cf44649cf314)](https://github.com/user-attachments/assets/a86c0c3e-64ed-4639-a803-cf44649cf314){: .image-popup}<br>

### 주소 변환 과정과 데이터 로딩

[![Image](https://github.com/user-attachments/assets/3bf497b4-abcd-4125-8b77-f4fae4f750f6)](https://github.com/user-attachments/assets/3bf497b4-abcd-4125-8b77-f4fae4f750f6){: .image-popup}<br>

이런식으로 '외부 단편화'를 예방하기 위해<br>
각 프로세스는 실제 메인 메모리에 쪼개져 있거나<br>
보조 기억 장치에서 로딩하지 않은 상황<br>

이러한 경우에도 CPU가 문제없이 프로세스들을 동작시키기 위해<br>
사용하는 개념들이 존재!<br>

- MMU(Memory Management Unit)<br>
  : 가상 주소 <-> 물리 주소 변환을 담당하는 하드웨어 장치<br>
    - 내부에 TLB(Translation Lookaside buffer)라는 캐시를 두어<br>
      최근 변환한 주소를 저장하여, 캐시 히트시 빠른 처리를 가능하게 함<br>
    - 일일이 PageTable을 참고하는 것은<br>
      매번 메인 메모리까지 내려갔다 와야 하기에 꽤나 느린 작업이기에<br>
      이러한 캐시를 두어 성능을 향상 시킴<br>

- Page Table<br>
  : 메인 메모리에 존재하는, '프로세스'마다 있는 '가상 주소'와 '물리 주소'의<br>
    매핑 정보를 담고 있는 테이블<br>
    - 가상 주소의 '페이지 번호'를 물리 메모리의 '프레임 번호'로 연결<br>
      - 페이지 : 가상 메모리 공간을 나눈 고정된 크기의 블록<br>
      - 프레임 : 물리 메모리(Ram)을 나눈 고정된 크기 블록<br>
        - 일반적으로 두 블록의 크기는 같음<br>
        - 약 4kb 정도를 가짐<br>
          - 너무 크기가 작으면 PageTable의 크기가 늘어남<br>
          - 너무 크기가 크면 '내부 단편화' 문제가 발생 가능<br>

- Page Table Entry<br>
  : 페이지 테이블을 구성하는 각 행<br>
   - 구성 요소?<br>
     - 유효 비트 : 해당 페이지가 메인 메모리에 존재하는지<br>
     - 프레임 번호 : 메인 메모리의 어디에 존재하는지를 알려줌<br>
     - 참조 비트 : CPU가 읽거나 쓴지를 파악하는 비트<br>
     - 수정 비트 : 내용이 수정되었는지를 파악(단순 읽는 용도인지 저장할 필요가 있는지 파악)<br>
       등이 존재<br>

- Page Fault<br>
  : CPU가(정확히는 MMU가) 액세스하려하는 페이지가 '메인 메모리'에 없는 상황 (인터럽트!)<br>
    - 인터럽트?<br>
      : CPU를 호출하는 신호 -> CPU는 현재 작업을 중단하고 관련된 처리 로직 실행<br>
        - Timer , 키보드 입력 : 하드웨어 인터럽트<br>
        - Page Fault : 소프트웨어 인터럽트<br>
    - Page Fault가 자주 발생 시<br>
      CPU가 페이지 교체를 자주해야 하여 성능 저하<br>
      ('스레싱' 현상)

- Page Fault 발생 후의 일들<br>
  - MMU가 인터럽트 발생 후, OS가 제어권을 얻음(Context-Switching 발생)<br>
  - 보조 기억 장치를 탐색해, 해당 페이지 찾음<br>
  - 메인 메모리를 탐색하여 '빈 물리 메모리 프레임'을 찾아봄<br>
    - 있으면, 그 곳에 데이터를 올림<br>
    - 없다면, 페이지 교체 알고리즘을 통해 '교체'할 영역을 선정하고<br>
      그곳과 교체(Swap)<br>
  - 이후 PageTable 갱신하고 제어권을 다시 프로세스에게 돌려줌<br>
    (이후 명령 재개)<br>

- 페이지 교체 알고리즘의 종류?<br>
  : 보통 LRU를 사용하는 편<br>
    다만 실제로는 매번 참조 시간을 기록&체크 하는 방식이<br>
    오버헤드가 큰 편이기에 'Clock' 알고리즘 같은 유사 알고리즘을 사용<br>



### 이미지 출처

[가상 메모리 관련](https://www.geeksforgeeks.org/operating-systems/virtual-memory-in-operating-system/)<br>

[페이징,세그먼트 관련](https://bytebytego.com/guides/what-are-the-differences-between-paging-and-segmentation/)<br>

[주소 변환 과정 관련](https://ebrary.net/206233/computer_science/virtual_memory)<br>