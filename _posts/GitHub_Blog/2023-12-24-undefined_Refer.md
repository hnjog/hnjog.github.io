---
title: "undefined reference 와 static 키워드"
last_modified_at: "2023-12-24T21:20:00"
categories:
  - 크래프톤 정글
  - C
tags:
  - 크래프톤 정글
  - C
  - undefined reference
  - static
---

## 발견한 문제
![unrefer](https://github.com/hnjog/hnjog.github.io/assets/43630972/74c9ba2a-47d2-450a-86dd-2da967993de9)

해당 상황은 mmap 시스템 콜을 구현하는 중<br>
load_segment의 코드와 유사하여 해당 부분을 사용하던 중<br>
발생하였다<br>

처음에는 #ifdef 와 관련한 문제로 추측하였다<br>
(file.c 내부에서 do_mmap 함수를 #ifdef VM 일때만 호출할 가능성도 없지 않으니)<br>

그렇기에 lazy_load_segment가 호출되는 부분을<br>
이렇게 감싸주었다 <br>

```
do_mmap(){

#ifdef VM
... (lazy_load_segment)
#end if
}
```

그런데도 여전히 문제가 고쳐지지 않았다<br>

원인이 뭘까... 하던 중 lazy_load_segment 관련하여 뜬 warning 중<br>

```
../../include/userprog/process.h:21:13: warning: ‘lazy_load_segment’ declared ‘static’ but never defined [-Wunused-function]
 static bool lazy_load_segment (struct page *page, void *aux);
```

이러한 error 를 접할 수 있었고<br>
처음에는 'define'이 분명 되어있는데 무슨소리지... 하면서 넘어갔었으나<br>
다시 보니 이 함수가 'static' 키워드로 선언되어 있었다<br>

```
static bool lazy_load_segment (struct page *page, void *aux)
```

C에서의 static 키워드의 특징에 대하여<br>
잘 알지 못하여 이런 일이 발생하였던 것 같다<br>
~~(C++에서는 안 그랬는데!)~~

## static 키워드?
 추가적으로 static 키워드에 대하여 조금 더 알아보려 한다<br>

 static 키워드의 용법은 크게 2가지로<br>
 1. 다른 파일(즉, 외부)에서 이 static 변수/함수 에 접근하지 못하게 막는 것<br>
 2. static을 선언한 지역 변수의 값이 계속 유지되도록 한다<br>
    (이 경우, 사실상 '전역 변수'의 개념에 가까우나 해당 지역변수가<br>
     선언된 '함수'에서만 접근이 가능하다)<br>

 위의 예시는 1 에 해당하는 사례로서<br>
 static으로 선언된 함수를 '다른 파일'에서 접근하려 해서<br>
 링커 오류가 발생한 상황임을 알 수 있었다<br>

 그렇다면 왜 이런 키워드를 사용할까?<br>
 
 이는 일종의 '모듈화'의 개념으로 설명할 수 있다<br>
 C에서는 'private' 라는 접근 제어자가 없기에<br>
 다른 외부 파일에서 접근이 허용된다<br>

 - extern 키워드?<br>
   : 외부 파일에 존재하는 전역 변수나 함수, 구조체 등을 가져다 사용하는 용도의 키워드<br>
   (조금 더 명확히 말하자면, extern 키워드를 사용하지 않으면<br>
    컴파일러는 '현재 파일'에서 우선적으로 식별자를 찾고,<br>
    그래도 없는 경우, #include로 연결된 외부 파일을 검색한다)<br>
    (반대로 extern 키워드를 '명확히' 사용한 경우는<br>
    '정의'가 외부 파일에 있다는 것을 컴파일러가 인지할 수 있음)<br>

 그렇기에,<br>
 static 키워드를 통해<br>
 내가 의도하지 않은 '전역 변수'의 값이 변하는 일을 막거나<br>
 사용되는 'Scope'를 제한할 수 있다<br>
 
 static 키워드를 사용하여 '이름 충돌' 문제를 방지할 수 있다는 점도 알아두자<br>

 - TMI. 이름 충돌을 방지하는 방식<br>
   static 키워드 : 각 파일들이 '같은 이름'의 변수를 사용할 수 있도록 Scope를 제한 가능<br>
   header에 명시하기 : 이는 이전에 코치님이 말씀해주신 방식 중 하나인데<br>
   header 파일에 '공개될 정보'를 적어 다른 작업자에게 공유하는 역할을 할 수 있다<br>
   Coding Convention : 비슷한 역할을 하더라도, 그 '타입'에 따라서<br>
   iCount, UiCount 등으로 이름을 바꾸어 충돌을 예방하는 코딩 규칙을 짤수 있다<br>
   => C에서 namespace 키워드가 존재하지 않기에 위 방식을 생각하며<br>
      이름 충돌에 조금 더 신경쓰는 것이 좋다<br>
