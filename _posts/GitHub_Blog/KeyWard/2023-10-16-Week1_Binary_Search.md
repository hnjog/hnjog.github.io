---
title: "이분(이진)탐색"
last_modified_at: "2023-10-16T16:00:00"
categories:
  - 크래프톤 정글
tags:
  - 크래프톤 정글
  - 키워드
  - 탐색
---

## 정렬된 데이터
  : 데이터의 종류에 따라서 <br>
  더욱 특화된 알고리즘을 사용하여 빠르게 처리할 수 있다<br>
  따라서 <b>'정렬할 수 있는 데이터'</b>의 경우,<br>
  그에 특화된 <b>'이진탐색'</b>이라는 알고리즘을 이용하여<br>
  더 효율적으로 데이터를 탐색할 수 있다<br><br>

  그렇다고, '데이터가 들어올 때 마다'<br>
  배열을 정렬하는 것은 '배보다 배꼽이 더 클 수 있다'는 점을 명심하자!<br>
  정렬 알고리즘이 이진 탐색 알고리즘보다 시간 복잡도가 높기 때문!!<br><br>

## 이진탐색
  '이미 정렬된 데이터 배열'에서 '어떠한 값'의 위치를 찾을 때,<br>
  사용하는 알고리즘이다<br>

  ![bsSearch](https://user-images.githubusercontent.com/43630972/275428138-82c3443f-bde0-4575-85f2-e78b0f384805.png){: width="50%" height="50%"}<br><br>
  [출처] : <https://woodforest.tistory.com/32><br><br>

  정렬되어 있는 데이터 구조에서 '탐색 범위'를<br>
  절반씩 좁혀나가며 데이터를 탐색한다<br><br>

  <b>시간 복잡도</b> : O(log N) (log : log2)<br>

  '분할 정복' 알고리즘의 하나!<br>
  (모든 영역을 방문하지는 않음!)<br>
  '재귀'로 쉽게 작성이 가능<br><br>
  
  - 예시코드 (Python)
  ```
    # 오름차순으로 정렬되어 있다고 가정하며, 찾았을시 인덱스를 반환
    def bsSearchRecur(nums : list, left : int, right : int, value : int)->int:
      # 찾기 실패!!
      if left > right:
          return -1
      
      mid = (left + right) // 2
      if list[mid] == value:  # 찾았다!
          return mid
      elif list[mid] < value: # 중간값이 value보다 작네??
          return bsSearchRecur(nums,mid + 1, right,value)
      
      # 중간값이 value 보다 크다!
      return bsSearchRecur(nums,left, mid - 1,value)
  ```
  
## 이진 탐색 과 선형 탐색
  선형 탐색의 시간 복잡도는 O(N)이다<br>
  이진 탐색의 시간 복잡도는 O(log n)이지만,<br> 정렬 알고리즘의 시간 복잡도를 고려해야 한다<br><br>

  ex : 병합 정렬을 사용한다고 한다면 <br>
  정렬 후, 이진 탐색하면 시간 복잡도는<br>
  O(log n) + O(n log n) -> O(n log n)이 됨<br><br>

  그러면 이진 탐색 말고 그냥 선형 탐색 쓰는게 좋지 않나??
  : 데이터가 들어오는 빈도는 적지만, <br>'탐색'을 사용해야 할 일이 많은 경우<br>
    이진 탐색이 더욱 효율적!!


## tmi) 정수론
  꽤 방대한 양을 다루며,<br>
  좋은 포스팅 글을 찾았기에 잠시 링크를 남겨둔다<br>
  [생새우초밥집]<https://freshrimpsushi.github.io/categories/%EC%A0%95%EC%88%98%EB%A1%A0/>