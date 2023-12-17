---
title: "Multi-level Page Table"
last_modified_at: "2023-12-17T16:30:00"
categories:
  - 크래프톤 정글
  - CS
  - OS
tags:
  - 크래프톤 정글
  - CS
  - OS
  - Multi-level Page Table
  - PintOS
---

## 다중 레벨 페이지 테이블이 필요한 이유!
 Page Table은 기본적으로 메인 메모리에 존재한다<br>
 이는 'DRAM'에 존재한다는 것인데....<br>
 문제는 Page의 개수가 상~당히 많다는 점이다<br>

 4KB(2 ^ 12개의 바이트)가 하나의 페이지 크기라 가정한다면,<br>
 32비트 운영체제에서 가질 수 있는 가장 주소 공간의 영역은 2의 32승이다.<br>
 (4GB)<br>

 32비트 운영체제에선 2^(32 - 12) = 2^20 에 해당하는 페이지의 개수가 필요하며<br>
 Page Table은 이 페이지들에 대한 정보를 가져야 한다<br>
 PTE는 4byte(2^2 개의 바이트)에서 8byte(2^3개의 바이트) 정도의 크기를 가지기에<br>
 하나의 프로세스당 Page Table은 2^(20 + 2~3)의 바이트를 가져야 하며<br>
 이는 약 4MB ~8MB를 뜻한다<br>
 (따라서 프로세스가 많아질수록 그만큼 PT가 먹는 공간도 많아진다)<br>

 32비트도 32비트이지만<br>
 진짜 문제는 64비트 운영체제 이다<br>
 이들의 가상 주소 공간은 이론상 2^64 (약 16ExaByte)이며,<br>
 고로 페이지의 개수는 2^52 라는 것이며,<br>
 Page Table의 크기는 2^54 (약 16 테라바이트)를 뜻한다<br>
 프로세스 하나마다 16테라바이트의 PT를 구성하는 것은 불가능하기에<br>

 가상 메모리 관리 기법이 사용되는 이유이다!<br>
 요는 '사용하지 않는 주소 공간'은 Page Table에 포함시키지 않는 것이다<br>

## MLPT의 구성 개념
![mlpt](https://github.com/hnjog/hnjog.github.io/assets/43630972/71e870bc-e191-4ba4-81b3-705b4acfdfe3)
[출처] : <https://velog.io/@junttang/OS-2.6-MV-6-Multi-Level-Page-Tables>

 일반적인 '선형' 페이지 테이블과는 다르게<br>
 MLPT는 '페이지 디렉터리' 자료구조를 사용하여<br>
 '페이지 테이블'의 할당 여부와 그 위치를 파악할 수 있다<br>
 (위 예시에서, MLPT는 사용되지 않는 중앙 공간은 할당하지 않고 있다)<br>

 Page Directory를 구성하는 요소를 Page Directory Entry(PDE)라 하며<br>
 이 구성은 PTE와 유사하다<br>
 '유효 비트' 와 'PFN'으로 구성되며<br>
 Page Frame Number(PFN) 를 통하여 Page Table을 가리킬 수 있다<br>

 PDE가 유효하다면,(즉, PDE의 '유효비트'가 )<br>
 해당 Page Table 내부의 최소 '하나' 이상의 PTE가 유효하다는 것을 의미한다<br>

 MLPT 방식의 장단점을 간략히 소개하자면<br>
 - 장점<br>
   - 사용된 주소 공간의 크기에 비례하여 PT 공간이 할당<br>
     (사용되는 주소의 공간만 할당)<br>
   - 페이지 크기로 분할하기에 메모리 관리가 용이함<br>
     (유연한 공간 할당)<br>
 - 단점<br>
   - 특정 상황에서 추가적인 비용 발생<br>
     : 대표적으로 TLB 미스 가 있는데<br>
       주소 변환을 위해 2번의 메모리 로드가 발생한다<br>
       (각각 페이지 디렉토리의 접근과 PTE 접근)<br>
       (선형에서는 PTE만 접근하면 되기에, 나름의 Trade-Off라 생각)<br>

## PintOS에서의 MLPT
![pml4](https://github.com/hnjog/hnjog.github.io/assets/43630972/337c3556-5609-4d7e-b2e9-8931d2a42f97)
[출처] : https://velog.io/@rivolt0421/Pintos-3.-VIRTUAL-MEMORY-1

 pintOS에서는 총 4단계의 페이지 테이블 구성을 통하여<br>
 64bit 컴퓨터의 가상 주소가 동작한다<br>

```
 PML4(Page Map level 4) -> PDP(Page Directory pointer)
  -> PD(Page directory) -> PT(Page Table)
```

 이러한 환경에서 '가상 주소'가 어떻게 표기되는지 확인하자면<br>


```
63          48 47            39 38            30 29            21 20         12 11         0
+-------------+----------------+----------------+----------------+-------------+------------+
| Sign Extend |    Page-Map    | Page-Directory | Page-directory |  Page-Table |  Physical  |
|             | Level-4 Offset |    Pointer     |     Offset     |   Offset    |   Offset   |
+-------------+----------------+----------------+----------------+-------------+------------+
              |                |                |                |             |            |
              +------- 9 ------+------- 9 ------+------- 9 ------+----- 9 -----+---- 12 ----+
                                          Virtual Address
```

 64비트 핀토스 과정 기준의 '가상 주소'의 표현이다<br>

 각각에 대하여 설명하자면<br>

 - Sign Extend (48~63 bits)<br>
 : 현재의 48비트에서 확장하여 64비트로 맞춘 것 (가상 주소의 '부호' 비트를 늘린 것)<br>

 - Page-Map Level-4 Offset (39~47 bits)<br>
 : 페이지 맵 레벨 4(PML4) 내부의 PML4 엔트리에 액세스하는 데 사용되는 PML4의 인덱스<br>
   PML4의 몇 번째 엔트리를 참조하는지 결정<br>

 - Page-Directory Pointer (30~38 bits)<br>
 : 페이지 디렉토리를 가리키는 주소를 나타냄.<br>

 - Page-directory Offset (21~29 bits)<br>
 : 페이지 디렉토리의 몇 번째 엔트리를 참조해야 하는지를 결정

 - Page-table Offset (12~20 bits)<br>
 : 페이지 테이블의 몇 번째 엔트리를 참조해야 하는지를 결정

 - Physical Offset (0~11 bits)<br>
 : 페이지의 시작에서부터 실제 바이트까지의 거리를 나타냄<br>
   (물리 주소이기에 '프레임'이라 하여도 상관없음)<br>

 위에서 본 대로<br>
 각각의 PML4, PDP, PD, PT를 타고 들어가서,<br>
 '물리 페이지(프레임)'의 번호를 구한 후,<br>
 가장 뒤의 physical Offset과 결합하여<br>
 물리 주소를 구할 수 있다<br>

 지금까지 본 것이<br>
 CPU -> MMU(내부 TLB) -> 메인 메모리의 Page Table -> disk 중<br>
 메인 메모리의 Page Table에 해당하는 내용임을 다시 상기하자<br>
 