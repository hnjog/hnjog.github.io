---
title: "C 메모리 할당 및 해제 함수들"
last_modified_at: "2023-11-05T17:10:00"
categories:
  - 크래프톤 정글
tags:
  - 크래프톤 정글
  - 키워드
  - C
---

## 메모리 할당
  시스템에서 메모리를 '동적 할당'하는 경우,<br>
  OS가 virtual memory의 'heap' 영역에서<br>
  사용 가능한 메모리 블록을 찾아 할당하고<br>
  그 주솟값을 반환한다<br>

  이러한 'heap'영역에 대한 할당이 가능한 이유는,<br>
  OS가 가상 메모리 주소 공간을 분리하여,<br>
  각각의 프로세스가 메모리의 다른 공간을 사용하며,<br>
  어떤 메모리가 사용중인지를 추적하기 때문이다

## 메모리 할당 함수들
  - malloc(Memory Allocation)
    - 매개변수 : size_t size
    - 특징<br>
      : 크기가 size 바이트인 메모리 블록을 할당,<br>
      할당된 메모리는 초기화되지 않은 상태
    - 사용처<br>
      : 메모리를 동적할당할 때 사용

 - calloc(Contiguous Allocation)
    - 매개변수 : size_t num_elements(요소 수), size_t elements_size(요소 크기)<br>
    - 특징<br>
    : calloc 함수는 '요소 수' 만큼 '요소 크기' 바이트 요소를 갖는 메모리 블록을 할당,<br>
    해당 메모리들은 0으로 초기화됨
    - 사용처<br>
    : 배열이나 구조체 초기화에 유용하게 사용 가능<br>
    int* a = calloc(n,sizeof(int));<br>
    위 코드는 int 형 요소를 n개 가지는 배열을 생성하고 0으로 초기화하였다

  - realloc(Reallocate Memory)
    - 매개변수 : void* ptr, size_t new_size
    - 특징<br>
    : realloc 함수는 이미 할당된 메모리 블록(ptr이 가리키는 메모리)의 크기를<br>
    new_size로 변경하려고 시도한다<br>
    (이 때, 확장할 수 있다면 기존 메모리 블록의 크기가 늘어난다)<br>
    그렇지 않다면, 새로 메모리를 할당할 수 있는<br>
    공간을 찾은 후, 이전 데이터를 복사해준다<br>
    => 다만 실패시, 기존 데이터를 유지하고 null을 반환한다

    기존 데이터를 유지한 채로, 크기를 조절할 수 있다
    - 사용처<br>
    : 동적 배열의 크기를 동적으로 조절할 때 유용

## 메모리 해제
  메모리 해제를 통해,<br>
  해당 프로세서는 자신의 힙 공간에서 할당한<br>
  메모리 블록을 해제하여,<br>
  시스템의 메모리를 효율적으로 사용할 수 있도록 하며,<br>
  메모리 누수를 방지할 수 있다<br>

  - 메모리 누수?<br>
    : 동적으로 할당된 메모리를 해제하지 않고,<br>
      해당 메모리에 접근이 불가능한 상황<br>
      (해당 프로세스가 종료되면 OS가 회수하여 반환되지만,<br>
      프로세스가 종료되기 전까지는 불필요한 메모리 공간을 잡아먹어<br>
      시스템의 리소스 사용을 비효율적으로 만든다)

## 메모리 해제 함수
  - free(Free Memory)<br>
    - 매개변수 : void* ptr
    - 특징<br>
     : free 함수는 힙 메모리에서 동적으로 할당된 메모리 블록을 해제한다<br>
     메모리를 해제하고, 시스템에 반환하여, 다른 부분에서 사용할 수 있게 한다<br>
    - 사용처<br>
     : 동적으로 할당된 메모리를 더 이상 사용하지 않을 때 사용