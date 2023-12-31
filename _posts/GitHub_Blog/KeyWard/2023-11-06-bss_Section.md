---
title: "bss 영역"
last_modified_at: "2023-11-06T10:30:00"
categories:
  - 크래프톤 정글
tags:
  - 크래프톤 정글
  - bss
  - 목적파일
---

## BSS(Block Started by Symbol)
  bss 영역은 초기화되지 않았거나 0 혹은 NULL로 초기화된<br>
  전역 및 정적 변수를 저장하는데 사용하는 영역이다<br>
  
  '목적파일'에서는 공간을 차지하지 않는 특징이 있다<br>
  또한 프로그램 실행 시, 위의 변수들을 0으로 초기화시키고<br>
  해당 메모리 영역을 관리하는 영역이다

  반대로 이외의 값으로 초기화된 경우는,<br>
  data 영역에서 관리한다<br>
  ~~(당연하지만 중간에 값이 바뀐다고 영역이 바뀌지는 않음)~~

  - '목적파일'에서 공간을 차지하지 않는다?<br>
    : bss 영역에 들어오는 변수들은 '초기화'되지 않았거나,<br>
      0 혹은 NULL로 초기화된 변수들이므로<br>
      이들에 대하여 따로 목적파일에서 초기화 데이터를 저장할 필요가 없다<br>
      이들은 메타데이터를 통해 bss 영역에서 초기화됨을 알 수 있고,<br>
      이 방식이 메모리가 더 절약된다

  - bss 영역의 사용 이유?<br>
    -  메모리 공간 절약<br>
      : 초기화되지 않은 변수들을 bss 영역에 저장하고<br>
        프로그램 실행시, 메모리를 할당하기에,<br>
        '초기화 값'을 저장하지 않는 만큼 메모리가 절약된다
    - 목적 파일의 크기 절약<br>
      : 목적파일에 초기화 데이터를 저장할 필요가 없으며,<br>
        이를 통해 목적파일의 크기를 줄일 수 있음
    - 효율적인 초기화<br>
      : bss 영역의 변수들은 0으로 초기화되기에<br>
        초기화 데이터를 로드할 필요가 없으며,<br>
        이에 따라 메모리가 절약된다<br>

    프로그램이 크거나, 복잡해질 수록<br>
    목적파일과 메모리의 효율적인 관리에 도움이된다
