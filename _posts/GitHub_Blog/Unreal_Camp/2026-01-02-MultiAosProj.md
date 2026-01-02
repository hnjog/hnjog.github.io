---
title: "Paragonia (멀티플레이 과제) 팀 프로젝트 회고"
date : "2026-01-02 16:00:00 +0900"
last_modified_at: "2026-01-02T16:00:00"
categories:
  - 포트폴리오
tags:
  - 후기
---

# 조별 과제 마무리 및 정리

멀티/AOS 장르의 팀 프로젝트였다<br>

## 기획 관련

[![Image](https://github.com/user-attachments/assets/65e79534-9b7b-4921-b6e0-69b36ba161b2)](https://github.com/user-attachments/assets/65e79534-9b7b-4921-b6e0-69b36ba161b2){: .image-popup}<br>

파라곤 에셋을 쓰는 김에 `Paragon` 모작을 만들어보기로 한 후,<br>
그대로 진행했던 것 같다<br>

- Charcter + Skill<br>
- UI<br>
- Wave & Minion<br>
- Nexus 와 게임모드<br>

전반적으로 여러 주제를 학습할 수 있는 좋은 기회였다<br>
실제 제작하면서 여러가지를 배울 수 있었다<br>

- GAS 적용 (캐릭터 및 미니언 쪽의 스킬, 전투 시스템)<br>
- State Tree 적용 (Minion 로직 - 게임 플레이 태그를 이용하여 GAS 연동을 엔진단에서 진행 가능)<br>
- GameMode, GameState 등의 '서버/클라 구분 로직' (게임 모드가 반드시 서버에 있고, 그 외에는 서버단에만 실행할 로직을 구분)<br>
- Replicated 관련 동기화 문제 (당연히 '화면'에서 보이는 것은 '클라이언트'이며, 서버의 상태와는 다름 -> 로깅을 습관화 할 것)<br>


## UML , 흐름도 관련

[![Image](https://github.com/user-attachments/assets/635d5120-9425-449d-8b66-daaba0e8994f)](https://github.com/user-attachments/assets/635d5120-9425-449d-8b66-daaba0e8994f){: .image-popup}<br>

[15조 Paragonia UML](https://www.figma.com/board/LLBvmzyivLrHa4Uw0BR3Sz/CH4_Team_Proj?node-id=115-1247&p=f&t=nFIqd62eqvRfbWXY-0){:target="_blank"}<br>

이전 프로젝트와 비슷하게<br>
Figma를 사용하였으나 이번에는<br>
프로젝트 자체가 이전보다 커진 느낌이 들기에<br>
개인적으로 작업하는 부분들 위주로 나누어 작성하는 느낌이 강하였다<br>

## Git 프로젝트

[15조 Paragonia](https://github.com/NbcampUnreal/5th_6th-Team15-CH4-Project){:target="_blank"}<br>

이번에는 '이민구' 님이<br>
PR을 맡아주시면서 Feature -> Dev 방식으로 진행하였다<br>

## 개인 작업

### 코드 부문<br>
- Title & Lobby Level<br>
  - Open IP 기반 접속 서브 시스템 제작<br>
  - Lobby의 GameMode,GameState를 제작하여<br>
    게임 준비 등에 대한 로직 처리<br>

- Game Level<br>
  - Wave<br>
    - Data Table + Data Asset을 통한 데이터 주도 프로그래밍<br>
      (Table : 미니언 스탯 과 클래스 지정, Asset : 소환한 Npc 태그와 양 등을 지정)<br>
    - 미니언을 생성하고, GameplayEffect를 이용한 스탯 초기화 방식 사용<br>
  - Minion<br>
    - 작성 타이밍에 이미 GAS에 대한 팀원 분들의 기초 작업이 끝나있기에<br>
      코드를 읽은 후, 미니언에 맡게 필요한 부분만 가져다 사용하였다<br>
      - Target Actor를 생성하는 부분을 제외하여 작성<br>
        (미니언의 숫자가 많기도 하고, 미니언이 '여러 명'을 때리지는 않는다고 판단)<br>
    - State Tree를 이용하여 Minion 로직을 작성하였다<br>
      - GameplayTag와 미니언 자체 스탯을 이용한 Task를 통해<br>
        모든 미니언이 같은 로직이지만 다른 세부 동작을 동작하도록 작성할 수 있었다<br>
      - Tick 기반의 BT보다 '많아질 수 있는' 미니언 작성 로직에 적합하다 판단하여 채택<br>

원래는 OSS 적용까지 하려 하였으나<br>
자꾸 에픽게임즈의 '적절하지 않은 제품' 이라는 페이지가 반복되어<br>
추가 키를 만들어야 하는 상황에서 중단되었다<br>

### 프로젝트 관련<br>
이번에는 '전반적'으로 주도하기 보다는 의견을 내는 쪽에 가까웠기에<br>
모두 합심하여 같이 제작한 느낌이 강하게 되었다<br>

- 영상, PPT 제작 (영상 편집 자체는 하였지만 소스는 팀원들이 제공)<br>
- 일정관리<br>
- Figma 와 같은 UML 계열<br>

## 시연 영상

<iframe width="560" height="315" 
  src="https://www.youtube.com/embed/BJ4nFquMYAQ" 
  title="YouTube video player" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
  allowfullscreen>
</iframe><br>

## 발표 ppt 제작
[![Image](https://github.com/user-attachments/assets/eb0ed4a4-b357-4e15-87ae-30a59fa7f6b0)](https://github.com/user-attachments/assets/eb0ed4a4-b357-4e15-87ae-30a59fa7f6b0){: .image-popup}<br>

[발표 ppt_Canva](https://www.canva.com/design/DAG82-oi7hE/D_Guomj-ic5uEhzllVxLOA/edit?ui=eyJBIjp7fX0){:target="_blank"}<br>

[Paragonia.pdf](https://github.com/user-attachments/files/24403701/PARAGONIA.pdf){:target="_blank"}<br>

항상 작업하면서 만나는 어려움이나 작업 결과를<br>
영상이나 GIF로 미리미리 만들어두어야 하는데<br>
쉽지 않음을 느낀다<br>

## Team Notion

[15조의 Team Notion](https://www.notion.so/teamsparta/15-2a82dc3ef51480109f74f9f9bed4f4cc){:target="_blank"}<br>

- 이번에는 마지막까지 작업에 집중하였기에<br>
  노션 작성이 꽤 어려웠다<br>

## 여담

State Tree 관련한 삽질에 대하여 많이 고민하였다<br>
그래도 팀 작업 중 TIL을 종종 작성해놓긴 하였다<br>

[Wave & Minion 구조 작업](https://hnjog.github.io/unreal/c++/%EB%82%B4%EC%9D%BC%EB%B0%B0%EC%9B%80%EC%BA%A0%ED%94%84/WaveSpawnerStruct/){:target="_blank"}<br>

[Minion GAS 작업](https://hnjog.github.io/unreal/c++/%EB%82%B4%EC%9D%BC%EB%B0%B0%EC%9B%80%EC%BA%A0%ED%94%84/Minion_GAS/){:target="_blank"}<br>

[StateTree 관련 TIL](https://hnjog.github.io/unreal/c++/Minion_StateTree_TIL/){:target="_blank"}<br>

[Minion 불사 버그 관련 TIL](https://hnjog.github.io/unreal/c++/Minion_TroubleShoot/){:target="_blank"}<br>