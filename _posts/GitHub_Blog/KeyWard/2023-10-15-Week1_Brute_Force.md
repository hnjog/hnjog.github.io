---
title: "완전탐색_브루트포스"
last_modified_at: "2023-10-16T16:20:00"
categories:
  - 크래프톤 정글
tags:
  - 크래프톤 정글
  - 키워드
  - 탐색
---

## 탐색 알고리즘
  어떠한 '데이터 구조' 안에 존재하는 것을 구해오는 알고리즘<br>
  일반적인 '선형(linear)' 탐색 알고리즘의<br>
  <b>시간 복잡도</b>는 O(N) <br>
  '해시 맵'을 이용한 탐색 알고리즘의<br>
  <b>시간 복잡도</b>는 O(1) <br><br>
  ('해시'와 관련한 내용은 차후 포스팅을 따로 다룰 예정이다)<br>
  (다만 해시 맵을 사용한 '탐색'의 경우 추가적인 '공간'이 필요하다)

## 완전탐색 알고리즘
  '모든 경우의 수를 다 체크하여 정답을 찾는 방식'
  : 항상 '모든 경우의 수'를 판단하기에<br>
    <b>최소 O(n)</b>의 시간 복잡도를 가진다<br><br>

  해당 알고리즘에 포함되는 방식은 다음과 같다
  - 브루트 포스(Brute Force)
  - 순열(Permutation)
  - 재귀(Recursive)
  - 비트마스크
  - BFS,DFS (너비우선탐색, 깊이우선탐색) -> 이번 포스트에서는 다루지 않고 차후 다룰 예정

## 브루트 포스(Brute Force)
  보통 가장 직관적인 문제의 해결법
  : 넓게 보자면 '숫자'를 이용한 계산법 및 반복/조건 문을 이용한<br>
    순수한 문제해결법을 모두 표함할 수 있지만,<br>
    보통 알고리즘은 '효율성'(시간복잡도)가 우선시 되기에<br>
    그러한 면에서 '우선적'으로 고려되지는 않는 알고리즘이다<br><br>

  '일단 문제를 해결한다'라는 인식의 알고리즘으로,<br>
  문제에 따라서 이것보다 효율적인 알고리즘이 발견되지 않았을 수도 있다<br>
  (ex : 비밀번호 풀기(문자수^비번수), 외판원 문제(N!))<br><br>

## 순열(Permutation)
  임의의 수열이 있을때, 그것을 다른 순서로 연산하는 방법
  : 배열 데이터의 '나열'에 대한 모든 경우의 수를 나타내며<br>
    O(N!)의 시간복잡도를 가진다. <br><br>

    보통 순열에 원소를 하나씩 채워가는 방식을 사용한다. <br>

    
## 재귀
  자기 자신을 호출하여 문제를 처리하는 알고리즘 방식
  : '작은 해결'을 반복하여 '큰 문제'를 해결하는 방식이며,<br>
  DP(Dynamic Programming : 동적 계획)과 유사하다<br><br>
  다만 DP의 경우, '작은 해결'에 대한 결과를 저장해두어,
  차후에 값을 재이용하여 실행 속도를 빠르게 할 수 있음<br>


## 비트마스크
  비트마스크는 2진수를 사용하는 컴퓨터의 연산을 이용하는 방식으로,<br>
  나올 수 있는 모든 경우의 수는 '모든 비트를 포함' ~ '모든 비트가 포함되지 않음'<br>
  에 해당하게 된다.

  Bool 자료형과 비슷한 방식으로 사용할 수 있으나, 실질적으로<br>
  1 bit를 차지하기에 1개의 Byte에 8개의 비트마스크를 넣을 수 있다<br>
  (ex : 켜기 / 끄기 옵션)

