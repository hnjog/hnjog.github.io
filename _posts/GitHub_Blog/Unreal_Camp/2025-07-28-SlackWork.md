---
title: "Slack 과제 TIL 1"
last_modified_at: "2025-07-28T18:00:00"
categories:
  - Unreal
tags:
  - 과제
  - 내일배움캠프
---

## 갑작스런 과제
예비캠프의 마지막 주,<br>
오늘도 열심히 진도를 나가볼까 하는 그때...!<br>


<img width="1371" height="479" alt="Image" src="https://github.com/user-attachments/assets/76faddd8-9eaf-4346-b215-b9f22e2d4789" /><br>


느슨해진 캠프에 긴장감을 더해주는 매니저님의 조별 과제...!<br>

그나마 다행인것은 Git 등을 이용하여 같은 맵을 만드는 것이 아닌<br>
각자 과제를 진행한 후 공유하는 것이었다<br>

## 과제 시작
이런 과제는 '에셋 찾기'로 결정된다고 봐도 무방...!<br>

따라서 괜찮은 에셋을 찾아볼까 하던 중<br>

<img width="607" height="352" alt="Image" src="https://github.com/user-attachments/assets/3880b42e-7646-4956-94df-ebc2df53f72b" /><br>

언리얼 2주차 학습 과정 폴더에 이미 괜찮은 캐릭터 에셋이 있었다!<br>

따라서 이것과 어울릴만한 요소는 같은 '로우폴리' 계열이라고 생각하였고<br>

<img width="1588" height="802" alt="Image" src="https://github.com/user-attachments/assets/cf9310b0-8e8c-43f3-b244-45cd9eb21102" /><br>

다행히 괜찮은 에셋을 발견할 수 있었다<br>
[링크는 이쪽](https://www.fab.com/ko/listings/4673cb5f-4879-4871-8324-2bc1f1ff16a8)<br>

## 이전에 배웠던 것의 복습...
이전에 배웠던 캐릭터 BP와 Input Action, Mapping Context를 다시 만들고<br>
이미 존재하는 애니메이션을 가볍게 수정하여 ABP를 완성하였다<br>

<img width="3831" height="1799" alt="Image" src="https://github.com/user-attachments/assets/61f1b29a-4ca8-4afc-8e5e-59b0efe2a38e" /><br>

이후 landscape를 완성!<br>

<img width="2439" height="1173" alt="Image" src="https://github.com/user-attachments/assets/2a58a814-e6f7-46a7-ab82-f11a25bdc749" /><br>

황량한 허허벌판에 고독한 선인장 몬스터 한마리...<br>

## 인터랙티브 액터 작업 중
플레이어가 해당하는 '인터랙티브 액터'를 건들 수 있도록<br>
관련된 액터에는 '블루프린트 인터페이스'를 넣어주었다<br>

그 후, Player에 변수로 Actor를 추가한 후<br>
해당 Actor가 그 인터페이스를 상속받았는지를 추가!<br>

<img width="2047" height="1481" alt="Image" src="https://github.com/user-attachments/assets/e79be83d-dcd3-4c21-a674-16342bc5fec2" /><br>

Setter를 사용할 때,is valid와 함께 체크해주면<br>
더 안전하게 사용할 수 있다<br>

이후 인터랙티브한 물체에 플레이어가 가까이 가면<br>
Custom Depth Buffer를 이용하여 테두리를 주려 했는데...<br>
(이건 챗 GPT의 조언이었다)<br>

<img width="2434" height="1162" alt="Image" src="https://github.com/user-attachments/assets/d1e10f56-8015-4593-96aa-c30a34d621eb" /><br>

overlap 이벤트가 작동을 안한다...???<br>
일단 관련 Collision Preset을 체크<br>
Door 은 DynamicAllOverlap이며,<br>
Player는 Pawn이였다<br>
작동을 안하는게 이상한지라 혹시 몰라 껏다 켰더니<br>
다행히 이벤트가 잘 동작하였다<br>
~~(껏다 키는것은 범용적이고 유용한 해결책이다)~~<br>

그런데 여전~히 테두리가 나타나지 않았다<br>
이 이상의 세팅을 할 바에 그냥 머테리얼을 하나 만드는 것이 좋다고 생각되었기에<br>

이전 강의에서 배운 Fresnel을 이용하여<br>
가볍게 바깥쪽이 빛나는 머테리얼을 작성해주었다<br>

<img width="2969" height="1491" alt="Image" src="https://github.com/user-attachments/assets/79a27611-2df3-4914-8f25-f719602d395c" /><br>

이후 문쪽에 살짝 큰 StaticMesh를 추가하고<br>
해당 머테리얼을 바른 후, 숨겨두었다(visible = false)<br>

## 마무리

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/5EBv1oWAd1I"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe>


box Collision의 범위에 들어가면<br>
그것이 player인지 체크하고<br>
맞다면 staticmesh를 보이게 함으로서 빛나는 연출을 주었다<br>

아마 내일은 문을 열거나 닫는 작업을 할 예정일듯 하다<br>