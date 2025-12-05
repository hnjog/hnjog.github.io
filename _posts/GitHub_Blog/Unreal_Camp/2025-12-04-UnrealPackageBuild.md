---
title: "Unreal Package Build"
date : "2025-12-04 20:00:00 +0900"
last_modified_at: "2025-12-04T20:00:00"
categories:
  - Unreal
  - Package Build
tags:
  - Unreal
  - Unreal Package Build
---

## Unreal Package Build

언리얼의 타겟 구조<br>

| 타입          | 용도                                          | 예시 실행 파일 이름                               |
| ----------- | ------------------------------------------- | ----------------------------------------- |
| **Game**    | 일반 게임 실행 파일 (클라 + 서버 코드 다 포함). Listen 서버 가능 | `TagNChase.exe`                           |
| **Client**  | **클라이언트 전용**. 서버 코드 완전히 제거                  | `TagNChaseClient.exe`                     |
| **Server**  | **Dedicated 서버 전용**. 렌더링/사운드/클라 코드 거의 없음    | `TagNChaseServer.exe`                     |
| **Editor**  | 에디터 실행 파일                                   | `UnrealEditor.exe`, `TagNChaseEditor.exe` |
| **Program** | 커맨드라인 유틸 등 (UAT 같은 툴)                       | `UnrealPak.exe` 등                         |

- 그렇기에 `.Target.cs`파일을 각기 정의해 주어야 함<br>
  - 일반적으로 우리가 언리얼 프로젝트를 만들면 Editor.Target.cs 만 존재함<br>


- 에픽 런쳐에서 받는 엔진은 기본적으로 Server/Client 용 바이너리를 빼고 빌드한 방식<br>
  - Listen 서버나 StandAlone 같은 게임을 만들때는 괜찮음<br>

- 다만 실제로 GameServer + Client 로 프로그램을 구분하려 한다면<br>
  언리얼 소스코드를 이용해야 함<br>
  - 정확히는 '엔진 소스'를 받아서 '빌드한 엔진'이 필요<br>
    (WithServer = true / WithClient = true 옵션 설정)<br>
  - 그렇기에 실제로는 C++ 엔진을 받은 후, 버전에 맞게 빌드하는 방식<br>

### 필요한 과정들

- 언리얼 버전에 맞게 엔진 설치 및 bat 실행 등에 대한 과정을 거쳐<br>
  엔진을 설치 (해당 방식을 너무 자세히 설명하진 않을 예정)<br>
  - [Unreal Git](https://github.com/EpicGames/UnrealEngine)<br>
  - 설치 후, 언리얼 엔진 빌드하기<br>


- Server / Client 각각의 Target.cs 를 생성하기<br>

- 에디터의 Project Settings 에서 `List Of maps to include in a package build`에<br>
  사용할 맵들 세팅하기<br>

- 이후 에디터에서 Tool > Project Launcher 를 통해 빌드하기<br>
  - ex) Client 라면<br>
    - Show Advanced 클릭<br>
    - variant -> WindowsClient 지정<br>
    - Config -> Development 지정<br>
    - Data Build -> By the book 지정<br>
    - 패키징 버튼 클릭<br>

- 빌드 실패시 AppData 쪽의 UnrealEngine 쪽에서 원인을 찾을 수 있음<br>

- 빌드 성공시 `프로젝트 폴더 > Saved 폴더 > StagedBuilds 폴더` 내부에 결과물이 생성됨<br>
