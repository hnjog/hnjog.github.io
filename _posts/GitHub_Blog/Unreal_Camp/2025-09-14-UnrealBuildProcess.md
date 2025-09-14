---
title: "Unreal Build Process"
date : "2025-09-14 20:00:00 +0900"
last_modified_at: "2025-09-14T20:00:00"
categories:
  - Unreal
  - C++
tags:
  - Unreal Build Process
---

## Week6를 시작하며
Unreal의 파일 구조 등에 대한 자료가 매우 훌륭하기에<br>
해당 데이터들을 공부하며 TIL을 작성하려 한다<br>

(다만 공유가 금지인지는 잘 모르겠어서 그냥 내가<br>
보면서 정리한 내용이나 타이핑한 내용을 남기려 한다)<br>

## Unreal - Visual Studio C++ 솔루션 구조 이해

[![Image](https://github.com/user-attachments/assets/0567d782-8eef-4150-ac2a-8d5f45a8dd0e)](https://github.com/user-attachments/assets/0567d782-8eef-4150-ac2a-8d5f45a8dd0e){: .image-popup}<br>

Unreal 프로젝트를 만들고 VS를 열면 다음과 같은 프로젝트 구조가 보인다<br>

- Visual Studio는 프로젝트 관리를 용이하기 위해<br>
  디스크의 실제 폴더 구조와 별개로 '논리적 가상 폴더 구조'를 생성하여 표기<br>
  (실제 폴더 구조와 1:1 대응 되진 않음)<br>

### 솔루션 구조 내 주요 폴더

- `Engine`<br>
 : 언리얼 엔진 자체의 소스 코드, 리소스가 있는 곳<br>
   (Editor, Engine 쪽의 코어 코드가 존재)<br>

- `Game`<br>
 : 우리가 만든 프로젝트 코드가 들어가는 곳<br>
   (`Source, Config, .uproject` 파일등이 들어가며<br>
    일반적인 C++ 게임 로직을 작성하는 곳)<br>

- `Programs`<br>
  : 엔진 동작에 필요한 유틸리티 프로그램이나 서버 모듈이 포함된 곳<br>

- `Rules`<br>
  : 엔진과 게임 등 각 모듈의 '빌드 규칙'을 정의해 놓은 파일들이 있는 곳<br>
   (모듈 의존성, 플러그인 활성화 여부, 빌드 대상 제어)<br>

- `Visualizers`<br>
  : VS 디버거에서 언리얼 엔진의 복잡한 자료구조(ex : FVector,FString)를<br>
    읽기 쉽게 시각화할 수 있는 설정 파일이 있는 곳<br>
  
## 프로젝트 루트 폴더(`Games/프로젝트명`)

[![Image](https://github.com/user-attachments/assets/765c7226-6f6c-42a9-a343-0ded9c4d3455)](https://github.com/user-attachments/assets/765c7226-6f6c-42a9-a343-0ded9c4d3455){: .image-popup}<br>

`Game` 폴더 안을 열면, 만든 프로젝트 이름이 보임<br>

해당 폴더가 '프로젝트 루트'이며<br>
게임 개발에 필요한 리소스와 설정 파일이 모여 있다<br>

### 프로젝트 루트 폴더 주요 구성 요소

- `Config`<br>
 : `.ini` 파일을 이용하여 에디터와 게임의 초기 상태 지정<br>
   - `DefaultEditor.ini`: 에디터 환경 설정 (뷰포트, UI 등)<br>
   - `DefaultEngine.ini`: 엔진 전반 설정 (렌더링, 네트워킹 등)<br>
   - `DefaultGame.ini`: 게임플레이 관련 설정 (게임 모드, 플레이어 컨트롤러 클래스 등)<br>
   - `DefaultInput.ini`: 키보드·마우스·패드 등의 기본 입력 바인딩<br>

- `Source`<br>
 : 실제 C++ 소스 코드 존재<br>
 (`.cpp`, `.h`)<br>
 빌드 설정 관련 주요 파일도 포함<br>
  - **`프로젝트명.Build.cs`**: 해당 프로젝트에 필요한 **모듈, 라이브러리, 종속성** 등을 정의<br>
  - **`프로젝트명.Target.cs`**, **`프로젝트명Editor.Target.cs`**: 각각 게임 실행용, 에디터용 빌드 방식을 정의<br>

- `프로젝트명.uproject`<br>
  : 언리얼 에디터에서 이 파일을 열면 프로젝트를 직접 실행 가능<br>
    이 파일을 통해 '에디터'가 '어떤 콘텐츠와 설정을 불러올지' 판단함<br>

## Visual Studio C++ 빌드 설정 이해하기

### C++ 코드 빌드의 목적

- C++ 코드 수정 시, 이를 컴파일 + 링크 하여<br>
  동적 라이브러리(DLL)로 만들어야 함<br>

- 생성된 DLL이 언리얼 에디터 실행 시, 로드되어<br>
  수정된 게임 로직이 '에디터'와 '게임'에 반영<br>

### 빌드 구성 및 플랫폼 확인

[![Image](https://github.com/user-attachments/assets/34f2760c-4b8f-432d-951e-f3c90a15273a)](https://github.com/user-attachments/assets/34f2760c-4b8f-432d-951e-f3c90a15273a){: .image-popup}<br>

왼쪽이 빌드 구성<br>
오른쪽이 플랫폼<br>

- 빌드 구성(Configuration) 종류<br>
  - **DebugGame**<br>
      - **게임 로직**만 디버그 정보를 포함, 엔진은 최적화된 상태로 빌드<br>
      - 언리얼 에디터 없이, 독립된 실행 파일 환경에서 게임 로직을 디버깅할 때 사용<br>
        엔진 코드는 디버깅이 제한적이며, 게임 로직만 효과적으로 디버깅 가능<br>
  - **DebugGame Editor**<br>
      - 에디터 환경에서 게임 로직을 디버그하기 편한 설정<br>
      - 에디터 플레이 중에 **C++ 로직**을 추적하거나 브레이크포인트를 설정 가능<br>
  - **Development**<br>
      - **디버그 정보**를 최소화해 실행 속도를 높인 **개발용** 빌드<br>
      - **독립 실행 파일** 환경 테스트·개발 단계에서 주로 사용<br>
  - **Development Editor**<br>
      - 에디터에서도 개발·테스트를 원활히 할 수 있도록 구성된 빌드 모드<br>
      - **Live Coding** 사용 시나리오와 궁합이 좋음<br>
        (초/중 급자 권장)<br>
  - **Shipping**<br>
      - 최종 사용자에게 배포할 때 사용하는 **릴리스 빌드**<br>
      - 디버그 정보를 제거하고, **성능 최적화**가 극대화<br>

- 플랫폼(Platform) 설정<br>
 - 기본적으로 Win64 선택<br>
 - 모바일/콘솔 등으로 빌드시 해당 플랫폼용 SDK를 추가로 설치해야 함<br>

## Visual Studio에서 C++ 빌드하기

### 전체 솔루션 빌드 방식

[![Image](https://github.com/user-attachments/assets/1c8e41d7-a4e7-4fd7-9b86-3c8be691dd82)](https://github.com/user-attachments/assets/1c8e41d7-a4e7-4fd7-9b86-3c8be691dd82){: .image-popup}<br>

- Visual Studio 메뉴 **`Build → Build Solution`** (단축키: `Ctrl + Shift + B`)<br>
- 엔진, 유틸리티, 게임 등 **모든 모듈**을 통째로 빌드<br>
- 첫 빌드나 엔진 소스를 수정했을 때, 또는 엔진 전체 파일이 필요한 경우에 사용<br>
- 프로젝트 규모가 크면 시간이 오래 걸림<br>

### 부분 빌드 방식

[![Image](https://github.com/user-attachments/assets/b46f7313-2dc3-4e64-8908-2f8d65f4d817)](https://github.com/user-attachments/assets/b46f7313-2dc3-4e64-8908-2f8d65f4d817){: .image-popup}<br>

- **Solution Explorer**에서 **프로젝트 (예:** `SpartaProject`**)를 우클릭** → **`Build`** 선택<br>
- 엔진이나 다른 모듈을 제외하고, **게임 프로젝트 코드만** 빠르게 빌드<br>
- 일반적으로 C++ 로직만 수정했다면 **이 방법**을 쓰는 것이 효율적<br>

### 빌드 전후 체크 포인트

- 빌드 시작전<br>
  - 언리얼 에디터를 종료하고 빌드하는 것이 안전<br>
  - 에디터 실행 중이면 수정된 DLL을 교체하지 못해 '빌드 에러' 발생 가능<br>

- 빌드 진행<br>
  - VisualStudio 하단의 Output 창에서 빌드 메시지를 모니터링 하기<br>
  - 첫 빌드는 '엔진 모듈'까지 새로 컴파일하여 시간이 걸림<br>
  - 이후는 변경된 소스만 컴파일하여 시간 절약<br>
  - 경고/에러 는 Error List 창에서 원인 확인 가능<br>
    - 우측 상단의 Build Only 를 선택하여 '빌드 오류' 메시지를 체크 가능<br>

[![Image](https://github.com/user-attachments/assets/5d366ea1-b126-486a-aa14-4dfd7d1fd351)](https://github.com/user-attachments/assets/5d366ea1-b126-486a-aa14-4dfd7d1fd351){: .image-popup}<br>


- 빌드 결과<br>
  - Build Complited 메시지면 정상 빌드된 것<br>

[![Image](https://github.com/user-attachments/assets/fb932405-5b07-4880-b145-129d9315961d)](https://github.com/user-attachments/assets/fb932405-5b07-4880-b145-129d9315961d){: .image-popup}<br>

- 빌드 후, 프로젝트 폴더의 Binaries/Win64 폴더 내에 UnrealEditor-프로젝트명.dll 등이 새로 생성<br>

[![Image](https://github.com/user-attachments/assets/0cf5744d-3122-4ec7-bd92-c9c53a8254ee)](https://github.com/user-attachments/assets/0cf5744d-3122-4ec7-bd92-c9c53a8254ee){: .image-popup}<br>

이제 에디터 실행시, 새로운 DLL을 로드하기에<br>
수정된 로직 등이 적용됨<br>

### 빌드 후 실행 대상 설정

[![Image](https://github.com/user-attachments/assets/a0712c1b-f6da-40db-bce2-061be1cad99d)](https://github.com/user-attachments/assets/a0712c1b-f6da-40db-bce2-061be1cad99d){: .image-popup}<br>

- Games 폴더 내의 프로젝트를 우클릭하고, 시작 프로젝트로 설정하기<br>
  (디버깅 시, 실행될 기본 프로젝트로 설정됨)<br>

- 이후, F5 (Start Debugging) 또는 Debug → Start Debugging을 선택하여<br>
  UE5 에디터의 실행도 가능<br>

- 종료를 하고 싶다면 Editor 상에서 종료하거나<br>
  VS에서` Shift + F5`로 종료 가능<br>

## 언리얼 에디터에서 Live Coding 실행하기

### 기존 빌드 방식과 Live Coding

기존에는 에디터와의 연결을 종료하고<br>
코드를 수정한 후, 빌드<br>
이후 다시 에디터를 실행하여 결과를 확인해야 하였음<br>

라이브 코딩은 에디터를 '종료'하지 않고<br>
C++ 코드를 수정하고 Live Coding으로 변경 사항만 컴파일하여<br>
즉시 에디터에 반영!<br>

- 일일이 에디터를 켜고 끄는 번거로움이 줄어듦<br>

### Live Coding 제약사항

- 간단한 코드변경은 가능, 즉시 반영<br>
  (함수 내부 로직, 변수값 변경, 로그 출력 등)

- 적용이 안되는 경우?<br>
  - UCLASS, USTRUCT, UENUM 매크로의 추가·삭제·수정 (리플렉션 관련)<br>
  - 새로운 C++ 클래스 (.h/.cpp) 생성<br>
  - 엔진 소스 코드 (Engine 폴더 내 코드) 변경<br>
  - 함수 시그니처 (인자, 반환값)나 클래스 상속 구조 변경<br>

적용이 안되는 경우들에 대해서는<br>
기존처럼 에디터를 껐다 켜야 함<br>

### Live Coding 기능 활성화

[![Image](https://github.com/user-attachments/assets/8572fcf7-e3a6-4d6f-ac05-d21c284ca2f4)](https://github.com/user-attachments/assets/8572fcf7-e3a6-4d6f-ac05-d21c284ca2f4){: .image-popup}<br>

- 에디터 하단의 Live Coding 아이콘 (벽돌 모양) 옆 세로 점 3개(… 메뉴)에서 Enable Live Coding을 체크<br>

### Live Coding 사용 방법

[![Image](https://github.com/user-attachments/assets/3c7621ee-960d-4737-ab29-a8ec50c40593)](https://github.com/user-attachments/assets/3c7621ee-960d-4737-ab29-a8ec50c40593){: .image-popup}<br>

- Visual Studio에서 코드를 수정 후, **`Ctrl + Shift + S`** (또는 상단 메뉴에서 `File → Save All`)로 모든 파일을 저장<br>
- **Live Coding 아이콘**을 클릭하거나, 단축키 `Ctrl + Alt + F11`을 눌러 **Live Coding 빌드**를 시작<br>
- 컴파일이 완료되면 Live Coding 창에 “**Finished**” 메시지가 뜨고, **에디터가 즉시 수정된 로직**을 반영<br>

## 빌드 문제 복구

### 변경 사항 미반영?

C++ 코드를 수정하고 빌드 완료 했음에도 에디터에서 반영이 안되는 경우<br>
빌드 캐시, 프로젝트 설정, 경로 문제 등이 원인이 될 수 있음<br>

- **컴파일 대상 누락**: Visual Studio 혹은 엔진이 수정된 소스를 인식 못 해 빌드 대상에서 누락<br>
- **DLL 교체 문제**: 에디터가 실행 중이거나 DLL 파일이 다른 프로세스에 의해 사용 중인 경우, 새로 빌드된 DLL 파일이 교체되지 않는 문제<br>
- **캐시 문제**: 이전 빌드 결과물이 남아 새 빌드 결과를 덮어씌우지 못함<br>
- **파일 경로 문제**: 헤더 파일 경로나 플러그인 설정이 잘못되어 컴파일에 포함되지 않음<br>

### 단계별 해결 가이드

- 에디터와 VS 종료하기<br>
  : 실행 중인 경우 프로세스가 빌드 파일을 잠글 수 있으므로 다시 시작<br>

- **프로젝트 폴더에서 `Intermediate`, `DerivedDataCache`, `Saved` 폴더 삭제**
  - 이 폴더들은 **빌드 캐시** 및 **임시 데이터**를 담고 있음<br>
  - 삭제 후 다시 빌드하면 새로 생성되어, 잘못된 캐시로 인한 오류를 해결 가능<br>

- **`.uproject` 파일을 우클릭 → “Generate Visual Studio project files”**
    - 솔루션(.sln) 파일과 프로젝트 설정을 재생성하여, 누락되었거나 꼬인 구성을 복구<br>

- **Visual Studio에서 클린 빌드 수행**
    - 새로 생성된 솔루션 (.sln 파일)을 Visual Studio에서 열고 다음을 수행<br>
        - `Build → Clean Solution`으로 기존 빌드 산출물을 정리<br>
        - `Build → Build Solution`으로 프로젝트 재빌드<br>
    - 에러가 난다면 **Output 창**과 **Error List**를 확인해 원인을 파악하고 수정<br>

- **언리얼 에디터 다시 실행**
    - 에디터를 다시 실행해 **코드 변경 사항**이 반영되었는지 확인<br>
    - 프로젝트 내 **`Saved/Logs`** 폴더에 있는 로그 파일을 열어 구체적인 에러 메시지를 확인, 엔진 버전 및 프로젝트 설정을 점검<br>

