---
title: "깃허브 블로그 댓글 기능 추가"
last_modified_at: "2025-08-18T14:00:00"
categories:
  - 깃허브 블로깅
tags:
  - giscus
---

## 댓글 기능의 추가와 giscus
블로그를 만든지 시간이 꽤 되었지만<br>
놀랍게도 댓글 기능을 만들지 않았었다<br>

원래는 공부용 및 TIL 용으로 사용하였으나<br>
생각해보니 없는게 이상하며<br>
내가 잘 정리한 내용에 대한 반응이나<br>
피드백을 받을 수 있다면 덧글 기능을 다는 것이 좋다고 생각하여<br>
기능을 추가하려 하였다<br>

그런데 Jekyll 기반에는 기본 댓글 엔진이 없음<br>
그렇기에 [giscus](https://giscus.app/ko) 라는 외부 플러그인을 사용하였다<br>


## 기본 설치 방법
먼저 giscus 내부의 '저장소'를 통해 연결해야 한다<br>

<img width="607" height="262" alt="Image" src="https://github.com/user-attachments/assets/09cec31d-b038-4027-8ae8-50abaf14041a" /><br>

붉은 사각형의 부분에서 알 수 있듯<br>
1. 만들려는 해당 저장소가 'public' 설정이 되어 있어야 하고<br>
2. [giscus 앱](https://github.com/apps/giscus)이 설치되어야 하며<br>
3. Discussions 기능이 활성화 되어 있어야 한다

<img width="689" height="50" alt="Image" src="https://github.com/user-attachments/assets/e5cc3590-fdb9-4e9f-afbc-799bd7a4b186" /><br>

<img width="695" height="56" alt="Image" src="https://github.com/user-attachments/assets/e58ae6c7-bce0-480e-a8b2-babb3ee4fc8e" /><br>

구체적인 위치는 Settings->Features->Discussions 에 존재한다<br>

해당 작업을 완료하게 되면<br>

<img width="403" height="93" alt="Image" src="https://github.com/user-attachments/assets/393bd5fa-a32d-47e2-9e4a-26b488e93019" /><br>

이런식으로 가능하다는 표시가 뜬다!<br>

## 추가 세팅
이제 조금 더 추가적인 세팅 몇가지를 해야 한다<br>

### giscus에 대한 Access 설정

설치를 완료한 경우<br>
[여기](https://github.com/apps/giscus)서 Install이 Configure 버튼으로 바뀐다<br>

이후 자신의 계정으로 들어간 이후<br>

<img width="970" height="664" alt="Image" src="https://github.com/user-attachments/assets/067107f6-9417-4471-a72e-3f40841e1e34" /><br>

모든 저장소 or 선택한 저장소에 대한<br>
접근 여부를 저장해주어야 한다<br>

### Discussion이 없다면 생성해주기

<img width="1243" height="602" alt="Image" src="https://github.com/user-attachments/assets/0f0b3398-e9d2-4334-a993-a0b18b1c30e3" /><br>

아무래도 혼자 쓰는 블로그다 보니 Discussion이<br>
하나도 생성되지 않아있었다<br>

그렇기에 General로 설정한 후 Comments라는 것으로<br>
새로 생성하였다<br>

### 페이지와 댓글창의 연결 옵션

<img width="737" height="667" alt="Image" src="https://github.com/user-attachments/assets/7c10ee87-a4b1-48ff-847e-317636d76261" /><br>

페이지 와 Discussions 연결을 세팅한다<br>
나는 아래쪽의 옵션은 체크 해제하고 진행하였다<br>

- 아마 CategoryName과 관련된 것이나<br>
  나는 General로 생성한 후, 이름을 'Comments'로 지었기 때문인지<br>
  자꾸 에러가 발생하여 댓글이 제대로 작성되지 않았다<br>
  (이후 해당 옵션을 체크해제한 후 정상적으로 동작하였다)<br>

### config.yml 세팅해주기

<img width="739" height="384" alt="Image" src="https://github.com/user-attachments/assets/ec8e53b5-0562-4658-94cd-9044728bdbe7" /><br>

여기까지 진행하였으면 저런식으로 Script를 제공해주는데<br>
빨간색 친  부분들을 config의 Comments 부분에<br>
채워넣어주면 된다!<br>

<img width="691" height="299" alt="Image" src="https://github.com/user-attachments/assets/71ba4cbd-636f-4a60-aea5-94c30b4b2a98" /><br>

## 결과
<img width="628" height="689" alt="Image" src="https://github.com/user-attachments/assets/68604420-56c7-4622-b719-c3a0750ea9da" /><br>

깔끔하게 포스팅에 댓글이 생겼다!<br>
(아마 Discussions 쪽에 페이지와 링크되는 새로운<br>
Discussions가 생성되고 거기에 쓰여지며,<br>
그 상황을 볼 수 있는 것 같다)<br>

혹시라도 나중에 git blog를 작성한다 던가<br>
giscus를 다시 사용하게 될 수도 있으니<br>
TIL로 남겨두어 정리하였다<br>