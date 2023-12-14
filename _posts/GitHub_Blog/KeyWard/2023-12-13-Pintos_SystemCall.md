---
title: "핀토스 2주차 - Pintos_SystemCall"
last_modified_at: "2023-12-13T20:30:00"
categories:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
  - 핀토스
tags:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
---

## system call
 정말 정신없었던 1.5주 였던 것 같다<br>
 반에 코로나가 유행하였고,<br>
 개인적으로 컨디션이 좋지 않아<br>
 til도 잘 작성하지 못한 것 같다<br>
 
 '커널 공간'에서 진행되는 <br>
 '사용자 공간'에 대한 처리 방식<br>
 즉, '시스템 콜'에 대한 이해가 부쩍는 듯 하다<br>

 사용자 공간은 각각의 'process 공간'을 가지되,<br>
 커널 공간은 해당하는 프로세스의 '가상 주소 공간'에 포함되지 않는다<br>
 (여담으로 스레드의 위치(thread_current())는 커널 공간에 위치)<br>

 main 함수의 스택 프레임에 직접<br>
 argument 인자들을 세팅해주는 것은 아주 흥미로웠다<br>
 특히, 함수가 호출될 때 매개변수를 전달하는 방식을<br>
 직접 구현(스택 포인터를 늘려, 해당 위치에 값을 써준다)해봄으로서<br>
 OS의 소중함(?)을 느낄 수 있었던 것 같다<br>

 또한 '시스템 콜'을 통하여<br>
 userMode -> Curnel Mode 를 캐치하는 부분(system call hander),<br>
 역시 OS 내부의 커널 코드가 해준다는 부분을 직접 작성해보니<br>
 OS의 '추상화'에 대하여 좀 더 이해할 수 있었다<br>
 (실제로 이 내용을 공부하기 전까지, OS 역시 코드로 구현되었음을 알았지만<br>
  PintOS에서 시스템콜을 구현해보니, '이런 역할을 해주겠네'라는 추상화의<br>
  강력함을 느낄 수 있었다)<br>

![user_prog_test](https://github.com/hnjog/hnjog.github.io/assets/43630972/8c39047d-a9e1-467c-828b-ae7594773b84)


## Try and Error
![elem 및 semaphore 데이터 이상함](https://github.com/hnjog/hnjog.github.io/assets/43630972/c4d8a5b5-65c2-4e20-bde1-826cfe3b8683)

 해당 문제가 발생한 것은 wait에 fd table 생성 및 running file을 생성한 시점이다<br>
 (test case는 'args-single')<br>

 마치 데이터를 '밟고' 지나간 듯 한 현상이기에<br>
 문제 위치를 특정하기 어려웠다<br>

 처음에는 fork를 구현하면 해결될 문제라 생각하였는데,<br>
 이후, 같은 문제가 재발하기에 연관된 문제라 생각되지 않아<br>
 브랜치를 밀고 다시 조사하였다<br>

 놀랍게도 해당하는 원인은<br>
 'argument passing' 부분을 수정하자 해결되었는데<br>
 (정확히는 load가 호출되는 process_exec 부분)<br>

 hex_dump 함수로 찍었을 때, 전달되는 인터럽트 프레임 값은 동일하였기에<br>
 더욱 황당하였다<br>
 (심지어 기존 코드가 args 테스트 케이스를 통과하였기에<br>
 원인이라 생각하지 못하였다)<br>

```
// 기존 코드 결과
000000004747ffc0                          00 00 00 00 00 00 00 00 |        ........|
000000004747ffd0  ed ff 47 47 00 00 00 00-f9 ff 47 47 00 00 00 00 |..GG......GG....|
000000004747ffe0  00 00 00 00 00 00 00 00-00 00 00 00 00 61 72 67 |.............arg|
000000004747fff0  73 2d 73 69 6e 67 6c 65-00 6f 6e 65 61 72 67 00 |s-single.onearg.|

// 올바른 코드 결과
000000004747ffc0                          00 00 00 00 00 00 00 00 |        ........|
000000004747ffd0  ed ff 47 47 00 00 00 00-f9 ff 47 47 00 00 00 00 |..GG......GG....|
000000004747ffe0  00 00 00 00 00 00 00 00-00 00 00 00 00 61 72 67 |.............arg|
000000004747fff0  73 2d 73 69 6e 67 6c 65-00 6f 6e 65 61 72 67 00 |s-single.onearg.|
```

 직접적인 원인을 찾아내지는 못하였으나<br>
 추측되는 원인이 있다면<br>
 - process_exec() 내부에서 file_name 을 <br>
   palloc_free_page 해주기에<br>
   해당 palloc_free의 위치를 sucess == false 내부로 옮겨줌<br>

   ```
   //palloc_free_page (file_name);
   if (!success) {
	   palloc_free_page (file_name);
		return -1;
	}
   ```

   file_name이 argv와 같은 공간(주소)을 사용하기에,<br>
   올바르지 않은 인자값이 전달될 수 있음<br>

   (스택에 값을 '써주고' 해당 file_name을 free 해주는 것도<br>
   고려되나, 해당 문제와 직접적인 연관이 없어 보였기에 패스)<br>

 - 기존 코드가 load() 내부에서 변수와 stack을 이용하였기에<br>
   load 스택 프레임이 해제되며, 전달하는 argument에 영향을 주었음<br>
   (해당 해제된 스택 프레임의 위치에 데이터가 남아있을<br>
    수도 있고, 아닐수도 있다)

   위의 추측보다 더 신뢰성이 있는 추측<br>
   특히, 'make check'를 돌렸을 때,<br>
   종종 Fail과 pass가 번갈아가며 터지던 문제가 없어진 듯 하다<br>

   다만, 위 사진의 문제가 확정적으로 일어났고,<br>
   근본적으로 이것이 데이터에 어떠한 영향을 주는지<br>
   판단하기 어려웠다<br>

   현재 시점에서 '해결'은 하였지만,<br>
   구체적으로 argument passing이<br>
   'semaphore'를 사용한 wait 이후에 영향을 미치는지에 대하여<br>
   짐작할 수 없었다<br>
