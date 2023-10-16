---
title: "우선순위 큐"
last_modified_at: "2023-10-16T21:50:00"
categories:
  - 크래프톤 정글
tags:
  - 크래프톤 정글
  - 키워드
  - 자료구조
---

## 우선순위 큐
  ![P_Queue](https://user-images.githubusercontent.com/43630972/275509648-e083ff56-8049-4cd3-87f3-8d3ec59b7339.png){: width="70%" height="70%"}<br><br>
  [출처] : <https://velog.io/@holicme7/%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84-%ED%81%90Prioirity-Queue-mbk48cz764><br><br>

  일반적인 '큐' 자료구조(선입선출)와는 다르게,<br>
  '우선순위'가 높은 데이터를 먼저 선출하는 자료구조<br>

  'Tree' 자료 구조 중 하나인,<br>
  '힙 트리'를 이용하여 구현하는 방식이<br>
  삽입, 삭제 시 O(log n)의 시간복잡도를 지녀<br>
  일반적으로 해당 방식을 이용한다<br>

## 힙 트리
  ![P_Queue](https://user-images.githubusercontent.com/43630972/275518963-076d257f-04db-4e8b-87b6-857feb8d57aa.png){: width="70%" height="70%"}<br><br>

  (이미지의 경우 '최소 힙')<br>
  '완전 이진 트리'의 형태를 띄며,<br>


  - 데이터 삽입<br>
  ![P_Queue](https://user-images.githubusercontent.com/43630972/275521631-9cd4dce5-2861-4ccc-8091-21fe6657dbb9.png){: width="40%" height="40%"}<br><br>
  
  _최소 힙 기준_ <br>
  새로운 데이터가 삽입된 경우,<br>
  처음에는 가장 끝의 노드에 삽입되며,<br>
  이후 자신의 '부모 노드'와 비교(누가 더 작은지)하여<br>
  작은 쪽이 큰 쪽의 노드와 교환한다<br>
  부모 노드보다 작지 않으면, 교환을 멈추며<br>
  가장 작은값인 경우 root 노드가 바뀐다<br>

  - 데이터 삭제<br>
  ![P_Queue](https://user-images.githubusercontent.com/43630972/275521699-044e46fc-a7f3-4fe5-b36d-6b0af58d9953.png){: width="40%" height="40%"}<br><br>
  
  root 노드를 지우고,<br>
  가장 끝의 노드를 root 노드로 만든다<br>
  이후 위처럼 '조건'에 맞도록 교환 시켜주는 방식을 취함

## 우선순위 큐의 용도
  - 운영체제의 작업 스케줄링
    (Queue에 우선순위가 필요한 경우에 대한 응용)
  - 다익스트라 알고리즘의 성능 개선
  - ~~힙 정렬?~~
    (정확히는 '힙 트리'를 사용하는 개념이기에 포함시키지 않음)
