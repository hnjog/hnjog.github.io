---
title: "스파르타 - 첫 발표 및 후기"
date : "2025-09-15 16:00:00 +0900"
last_modified_at: "2025-09-15T16:00:00"
categories:
  - 포트폴리오
  - 발표
  - 후기
tags:
  - Canva
  - draw io
  - 발표
  - 후기
---

# 조별 과제 마무리 및 정리
첫 조별 과제가 끝났다<br>
일단 TextRPG 기반이였고<br>

C++, Git 사용에 익숙해지며<br>
언리얼에 들어가기전, 협업에 익숙해지기 위한 취지인 듯하다<br>

## UML 제작

[![Image](https://github.com/user-attachments/assets/b443889f-baf6-4762-b7f7-1987fb175db5)](https://github.com/user-attachments/assets/b443889f-baf6-4762-b7f7-1987fb175db5){: .image-popup}<br>

[1조 TextRPG Sparta UML](https://app.diagrams.net/#G1p9FV7qsxyYK5KyON9Qgbw5q-oTzeOcDd#%7B%22pageId%22%3A%22c4acf3e9-155e-7222-9cf6-157b1a14988f%22%7D){:target="_blank"}<br>

[![Image](https://github.com/user-attachments/assets/43b7e2db-a74f-4cce-8127-4bfc277848ea)](https://github.com/user-attachments/assets/43b7e2db-a74f-4cce-8127-4bfc277848ea){: .image-popup}<br>

[1조 TextRPG Sparta 데이터 관련](https://app.diagrams.net/#G1iOIti2LcJnMoZcF6xcrK-28HOZ-3vafa#%7B%22pageId%22%3A%22C5RBs43oDa-KdzZeNtuy%22%7D){:target="_blank"}<br>

이전 포스팅에도 남겼듯<br>
당장 프로젝트를 시작하는 시점에서의<br>
UML 제작은 확실히 효과가 좋았던것 같다<br>

팀적인 효과로는 다음과 같다<br>

- 공동의 목표에 대한 시각적 공유<br>
- 해야할 작업에 대한 인식과 분업화<br>

또한 개인적으로는 느낀 점은<br>

- 제작할 프로젝트의 구조에 대한 끊임없는 생각으로 인한 구조 이해도 증가<br>
- 시각적 가시성을 생각하는 센스의 상승<br>

## Git 프로젝트 및 전략

[![Image](https://github.com/user-attachments/assets/e702ef6f-4888-45ef-9ca8-18cf6c55592f)](https://github.com/user-attachments/assets/e702ef6f-4888-45ef-9ca8-18cf6c55592f){: .image-popup}<br>

[1조 TextRPG Sparta](https://github.com/hnjog/TextRPG_SPARTA/tree/main?tab=readme-ov-file){:target="_blank"}<br>

간단하지만 git flow 방식으로 진행하였다<br>

- main : 기본 기능, 도전 기능 완료에 따른 병합으로 '릴리즈'할 버전 관리<br>
- dev : 주요 개발 브랜치<br>
- Feature : 각각의 기능 개발 브랜치들, 병합 완료 후 삭제<br>

다만 아쉬운 점들도 존재한다<br>

- 병합에 대한 전략을 세워서 pull request를 이용해봤으면?<br>
- git Feature에 대한 네이밍 공유, 혹은 커밋 네이밍 통일?<br>
- 커밋 관련한 협업 규칙에 대하여?<br>
  (ex : 커밋 전에 빌드 확인 및 버그 없는지 간략 테스트 등)<br>

협업을 더 잘할 방법은 없었나 조금 아쉽기는 하다<br>

### 개인 작업
코드 부문<br>
- CSV-Json-Cpp 파싱을 통한 데이터를 이용한 프로그래밍을 만들어보려 함<br>
- DataManager<br>
- ItemData/ItemInstance/ItemManager<br>
- EffectManager 와 실제 Effect 들<br>

프로젝트 관련<br>
- UML 제작<br>
- 발표 ppt 작성<br>
- 발표<br>
(관련 내용은 각각 6~8시간 정도 소모한 듯하다)<br>

## 시연 영상

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/4_7S0zNTBI0?start=1&rel=0&modestbranding=1"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

- 영상은 같은 1조 팀장이신 '박태웅'님이 찍어주셨다<br>

## 발표 준비 및 발표

### 발표 ppt 제작
[![Image](https://github.com/user-attachments/assets/b3a4a702-fc70-4e75-aebc-172bcf303844)](https://github.com/user-attachments/assets/b3a4a702-fc70-4e75-aebc-172bcf303844){: .image-popup}<br>

[발표 ppt_Canva](https://www.canva.com/design/DAGyYLqBi8Q/cC_E3cmo-uTXfUGy_GJpMg/edit){:target="_blank"}<br>

[TextRPG_SPARTA_Team1pdf.pdf](https://github.com/user-attachments/files/22289055/TextRPG_SPARTA_Team1pdf.pdf)<br>

처음에는 내가 발표를 하게 될지 몰라서<br>
다소 무난한 느낌의 내용을 채워넣었다<br>

- canva를 사용하여 가벼운 애니메이션들이 포함된 디자인을 사용하였다<br>
- 일단 주어진 템플릿을 이용하긴 하였으나 각 페이지들을 어떻게 구성할지가 제일 난관이였다<br>
- '가시성'에 대한 많은 고민을 할 수 있는 기회였다<br>
  - 코드를 쓰는게 좋을까?<br>
    코드를 사용하면 가시성이 떨어질것 같은데...<br>
    이미지를 쓰자니 너무 큰것 같기도 하고?<br>
    뭐가 정답일까?<br>
  - UML이 너무 큰데...? 이걸 일일이 볼수는 없겠어<br>
    부분씩 자르는 방법을 고려할까? 아니야 남은 시간이 별로 없으니<br>
    아래에 담당 이름과 내용을 적고, 발표시에 동그라미 치는 쪽으로 가자<br>
  - 여기 뭔가 허전한데? 내용과 맞는 괜찮은 그래픽 같은건 없나?<br>
  - 하... 똑같은 레이아웃이 반복되면 너무 질릴것 같은데...<br>
    좀 통일성을 해치지 않으면서 다른 느낌이 나는건 없나?<br>

그리고 나중에 내가 발표를 하려 하니깐<br>
정작 시간이 너무 부족해서<br>
가능한 상황에서 발표 내용을 맞추려고 하였다<br>

### 발표 준비

기본적으로는 다음과 같은 과정을 거쳤다<br>
1. 일단 ppt를 읽어보면서 페이지 별로 '어떤 내용'을 설명하려 했는지를 확인<br>
2. 해당 내용을 기반으로 초안 작성<br>
3. 작성한 초안을 기반으로 ppt를 한 페이지씩 읽어보기!<br>
4. 읽어보면서, '적절하지 못한', '읽기 힘든', '발음하기 힘든' 부분을 수정<br>
5. 전체적으로 읽어보면서 자연스럽지 못한 부분 찾아보기<br>
6. 자연스럽지 않은 페이지 전환 내용에 대한 내용 추가 및 삭제<br>
7. 포트폴리오 모드로 전환하여 발표하면서 시간에 맞춰보기<br>
8. 너무 긴 문자, 혹은 듣기에 적절치 않은, 너무 자주 쓴 단어가 있는지 재확인<br>
9. 수정 후 계속 반복하며 포트폴리오의 '뉘앙스' 외우기<br>

가볍게 말하자면<br>
'대본 확인' , '수정', '연습' 의 반복이긴 하다<br>

### 총 발표 영상

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/EpCsKnZI5PU"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

01:53 ~ 03:28, 04:58 ~ 05:15 - 발표자 전환으로 인한 miss 발생 부분<br>

### 발표 후기

개인적으로는 다른 분들이 '코드' 위주 발표를 하기에<br>
방향을 잘못 잡았나? 싶었지만<br>
피드백에서 크게 지적이 없었기에 다행이었다<br>

또 하필이면 zep이 아니라 zoom에서 처음 공유하다 보니<br>
조금 당황스러운 부분이 있었다<br>

원래는 화면 공유 후, 마이크를 키려 했는데<br>
화면 공유를 키니까 원래의 화면이 사라져 버렸다...<br>

또 발표를 넘기는 과정에서 우물쭈물 되기도 해서<br>
불필요한 시간이 소모된 듯 하다<br>

다음에는 꼭 Zoom 으로 연습을 해본다던가 고려해야 겠다<br>
