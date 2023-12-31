---
title: "묵시적, 명시적 가용 리스트"
last_modified_at: "2023-11-13T19:30:00"
categories:
  - 크래프톤 정글
  - CS
tags:
  - 크래프톤 정글
  - CS
  - 동적 할당
  - 가용 리스트
---

## 동적 할당을 위한 가용 리스트
 함수를 호출하면 '스택'에 스택 프레임이 생성되고<br>
 지역변수를 선언하면 그것 역시 '스택'에 메모리를 잡는다<br>

 그러나, 프로그래밍 시 동적할당이 필요하다는 점은<br>
 이전 포스팅에도 설명했듯,<br>
 효율적이며, 반드시 필요한 상황이 존재한다<br>

 '동적 할당' 시,<br>
 힙 공간에 메모리가 할당된다<br>
 
 이러한 힙 공간은 할당(malloc)과<br>
 해제(free)를 통해 해당 메모리 블록을<br>
 '할당(allocate)' 과 '가용(free)' 상태로 볼 수 있다<br>

 malloc과 free가 번갈아져 일어나게 되면,<br>
 해당 공간은 메모리가 듬성듬성 할당되어 있듯<br>
 메모리 '단편화'가 일어나기 쉬운 상황이 된다<br>
 
 그렇기에 동적할당을 할 때, 유의할 점으로<br>
 '할당할 크기', '할당할 위치', '할당 위치 탐색'<br>
 , '할당 여부', '재할당 여부', '단편화 관리' 등이 존재한다<br>

 이러한 유의점들을 인지하고 효과적으로<br>
 메모리를 관리하기 위한 '구조'가 바로<br>
 '블록'이다

## 블록(block)
  ![stack](https://user-images.githubusercontent.com/43630972/282442474-dc66f46f-e6df-4afe-bbd6-6e5bb6d9511f.jpg){: width="50%" height="50%"}<br><br>

  왼쪽 블록은 '묵시적 가용 리스트'에서 사용하며,<br>
  오른쪽은 '명시적 가용리스트'에서 사용한다<br>

  - Header : 4Byte를 가지며, 블록의 시작점을 나타낸다<br>
    블록의 크기는 언제나 8Byte 이상(header + footer, 각 4바이트)이기에<br>
    3비트가 쓸모가 없어진다<br>
    그 중 마지막 비트는 '할당 여부'를 확인하는<br>
    비트로 사용한다<br>
    그렇기에 header에는 '블록의 크기'와 '할당 여부'를<br>
    확인할 수 있다.
  - Payload : 실제 정보들이 들어가는 부분<br>
    
  - Padding : 정렬을 위하여 일부 내부 단편화를 감수하고<br>
  일정 단위로 메모리를 반올림하여 메모리를 맞춰준다<br>

  - Footer : 4Byte를 가지며, 블록의 끝을 나타낸다<br>
    header와 동일한 정보를 가지며, 이는<br>
    '경계'의 역할을 겸한다<br>
    (다른 블록의 header에서 -8Byte로 <br>
    이전 블록의 footer에 접근이 가능하다)
  
  - Pred free ptr / Next free ptr : 각각<br>
  4Byte를 가지며,이들은 각각 '할당이 해제된 블록의 포인터'를 저장한다<br>
  이중 연결리스트의 형태로 할당 해제된<br>
  블록의 데이터를 저장한다

## 묵시적 가용 리스트 (Implicit Free List)
 '순차적으로 모든 블록'을 검사하는 선형적인 방법<br>
 
   ![stack](https://user-images.githubusercontent.com/43630972/282446850-e426f31c-e188-4378-ac50-cb3e26b43fcf.jpg){: width="50%" height="50%"}<br><br>

  시작 시, 프롤로그 헤더/푸터 와 에필로그 헤더<br>
  를 생성하는데, 이는 '가장자리'를 표현하기 위한<br>
  일종의 트릭인 점을 알아두자<br>

  할당 시,<br>
  '탐색법'에 따라 다르지만,<br>
  기본적으로는 인접한 '다음' 블록으로 넘어간다<br>

  이후 해당 블록의 '할당 여부'와 '블록 크기'를 체크하고<br>
  조건에 맞는다면 할당을 한다<br>

  할당 해제 시,<br>
  해제 주소를 받은 뒤,<br>
  앞 뒤, 블록의 할당 여부를 확인한다<br>
  인접한 블록이 '가용' 상태라면,<br>
  해당 블록과 통합(coalesce)한다<br>

## 명시적 가용 리스트 (Explicit Free List)
 할당이 해제되어 있는 블록들을<br>
 연결리스트 형태로 관리하여<br>
 할당이 해제된 블록들만 검사하는 방법<br>

 ![stack](https://user-images.githubusercontent.com/43630972/282448877-260e6e2c-6c8b-4042-8541-3737ed1ea887.jpg){: width="50%" height="50%"}<br><br>

 가장 최근에 해제된 블록을 리스트의 가장 앞에 추가하는 LIFO방식과<br>
 물리적 순서대로 리스트를 구성하는 ordered-list 방식이 존재한다<br>
 (LIFO의 경우, 연결리스트의 순서와<br>
 물리적인 순서와는 다를 수 있음)<br>

 프롤로그 헤더의 경우,<br>
 예외적으로 '할당'이 되었으나<br>
 '이전', '다음' 포인터를 포함하여 사용하기도 한다<br>

 그 외의 '할당된 블록'의 경우,<br>
 '이전' 포인터의 위치부터 payload로 사용한다<br>

 할당 시,<br>
 '다음' 포인터가 가리키는 위치로 이동한다<br>
 할당이 가능하다면 해당 블록을 제거한다<br>
 이후, 해당 포인터가 가리키는 각각의 위치에서<br>
 다음 블록의 '이전'에 현재 블록의 '이전'을,<br>
 이전 블록의 '다음'에 현재 블록의 '다음'을<br>
 넣어준다
 (연결 리스트의 제거와 비슷하다)<br>

 연결 리스트의 '끝'에 도달 시<br>
 '힙'을 확장하여 할당한다<br>

  할당 해제 시,<br>
  묵시적 연결 리스트와 비슷하나<br>
  통합 전에 해당 블록의 '이전', '다음' 포인터를<br>

  정리해줘야 한다(위의 방식)<br>

 ## 도움이 된 사이트
  [https://velog.io/@emplam27/CS-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%8F%99%EC%A0%81%ED%95%A0%EB%8B%B9-Implicit-Explicit-Segregated-list-Allocator#-%EB%A9%94%EB%AA%A8%EB%A6%AC-%ED%95%A0%EB%8B%B9-%ED%95%B4%EC%A0%9C%EA%B3%BC%EC%A0%95]