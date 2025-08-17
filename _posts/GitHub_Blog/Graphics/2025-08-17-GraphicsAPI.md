---
title: "Direct X"
last_modified_at: "2025-08-17T16:30:00"
categories:
  - Graphics Rendering Pipeline
tags:
  - Direct X
---

## Graphics Rendering Pipeline

<img width="693" height="579" alt="Image" src="https://github.com/user-attachments/assets/a98fdefc-8b38-4cf4-aa0d-5746d956d192" /><br>

현재 배우고 있는 Direct X는 <br>
그래픽 API 중 하나이다<br>

### Graphics API
응용 프로그램 (ex : 게임, 3D 툴 등)과<br>
하드웨어 사이를 이어주는 프로그래밍 인터페이스<br>

개발자가 이들을 이용하여<br>
'삼각형 그리기', '텍스쳐 적용' 등의 명령을<br>
GPU에게 내릴 수 있게 됨<br>
(추상화)<br>

- API(Application Programming Interface)<br>
 : 앱(어플레이케이션, 게임 등)을 프로그래밍할 때 사용하는<br>
   인터페이스(완충제, 창구 등) 를 총칭한다<br>
   (개발 도구의 일종)<br>
   (다른 개념 들 사이를 이어줄때 많이 이용)<br>
   -> 이 예시에서는 GPU를 이용할 때<br>
   각각의 그래픽 카드나 드라이버에 따라<br>
   코드를 고칠 수 없으니 Graphics API를 이용한다<br>
   (OpenGL, WebGL, Vulkan 등도 이와 같은 역할을 해준다)<br>
   (Vulkan : 차세데 API - 안드로이드 쪽의 렌더링할거면 공부!)<br>
   (Metal : 아이폰 쪽 렌더링 할거면 공부!)<br>
   (DirectX : Windows나 XBox 같은 Microsoft 쪽 계열의 공부!)<br>
   (dx 11 : OpenGL에 대응, dx 12 : Vulkan, Metal에 대응)<br>

#### Direct X
<img width="701" height="491" alt="Image" src="https://github.com/user-attachments/assets/3d842115-17e1-42a4-88e9-736676195fcb" /><br>

우리가 DirectX 의 여러 기능 중<br>
가장 많이 사용할 것은 Direct 3D!<br>
(DirectX 중 3D 그래픽스 API 를 담당)<br>

대부분 그래픽스 자료 등에서<br>
Direct X를 표현할 때<br>
Direct 3D를 말하는 경우가 많으니 인지는 해둘 것<br>

---

### Graphics Driver (GPU Compiler 포함)
DirectX 같은 그래픽 API 에서 내려온 명령을<br>
GPU가 이해할 수 있도록 '저수준 언어'으로 변환<br>

- GPU Compiler<br>
  : HLSL(High-Level Shading Language) 같은 셰이더 코드를<br>
   GPU의 ISA(Instruction Set Architecture)로 변환<br>
   (ISA : 명령어 집합 구조)<br>
   (즉, 기계어 명령들로 변환하여 실행시킴)<br>

---

### GPU (그래픽 처리 장치)
주어진 명령어를 실제로 실행하여 픽셀을 계산<br>

- Command Processor: CPU에서 내려온 명령들을 스케줄링<br>
- Shader Cores (SM, CU 등): 정점/픽셀/컴퓨트 셰이더 실행<br>
- Rasterizer: 삼각형을 픽셀 단위로 잘라냄<br>
- ROP (Render Output Units): 최종 색상/깊이 값을 계산하고 프레임 버퍼에 저장<br>

병렬 처리가 가능하기에 한 번에 수천 개 이상의 스레드 실행 가능<br>

보통 교재 등에서 나오는 그래픽 파이프라인 등은<br>
'GPU' 내부 단계를 표현 하는 것<br>

ex)<br>
- Input Assembler – 정점(Vertex) 데이터 입력<br>
- Vertex Shader – 각 정점 변환 (월드/뷰/투영 행렬)<br>
- Tessellation / Geometry Shader – (선택적) 기하 분할, 도형 생성<br>
- Rasterizer – 삼각형을 픽셀 단위로 쪼갬<br>
- Pixel(Fragment) Shader – 각 픽셀 색상 계산 (조명, 텍스처 샘플링)<br>
- Output Merger (ROP) – 깊이/스텐실/블렌딩 처리 → 프레임 버퍼 기록<br>

등이 존재<br>

(크게 IA -> VS -> RS -> PS -> OM 이며<br>
세부 단계(ex : 테셀레이션,GS 등)가 중간에 더 존재)<br>

---

### Buffers(프레임 버퍼 / 백 버퍼)

GPU가 계산한 최종 픽셀 색을 담아두는 메모리 영역<br>

ex)<br>
- Color Buffer: 최종 RGBA 색상 값<br>
- Depth Buffer: 픽셀 깊이 값(Z-buffer)<br>
- Stencil Buffer: 마스킹용 값<br>

#### 더블 버퍼링 / 트리플 버퍼링?
화면에 출력 중인 버퍼(프런트 버퍼)와<br>
GPU가 새로 그리는 버퍼(백 버퍼)를 교체하면서<br>
깜빡임이나 티어링 방지<br>

#### 동기화?
V-Sync를 켜면 디스플레이 주사율(예: 60Hz)과 맞춰 버퍼 스왑<br>

---

### Display(모니터 등 출력장치)
GPU가 제공한 프레임 버퍼의 내용을<br>
주기적으로 스캔하여 화면에 출력<br>

- 주사율?<br>
  : 60Hz → 초당 60번 화면을 새로 그림 (1프레임 = 16.67ms)<br>
    모니터가 “언제” 프레임을 받아서 보여줄지 결정<br>

---