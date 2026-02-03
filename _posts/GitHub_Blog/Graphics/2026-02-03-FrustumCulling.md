---
title: "절두체 컬링(Frustum Culling)"
date : "2026-02-03 14:00:00 +0900"
last_modified_at: "2026-02-03T14:00:00"
categories:
  - 그래픽스
tags:
  - 컬링
---

## 절두체 컬링(Frustum Culling)

> 카메라의 시야 볼륨(Frustum) 밖 오브젝트를 **드로우 제출(렌더링) 이전에 제외**하여, 불필요한 렌더링 비용을 줄이는 최적화 기법

[![Image](https://github.com/user-attachments/assets/8ca29efc-86b9-4e77-9cc9-64c8e77afdc3)](https://github.com/user-attachments/assets/8ca29efc-86b9-4e77-9cc9-64c8e77afdc3){: .image-popup}<br>

- **정의**<br>
  - 카메라의 시야를 나타내는 **절두체(Frustum)** 내부에 있는 오브젝트만 렌더링<br>
  - 절두체 밖 오브젝트는 **렌더링 단계(드로우콜 제출) 이전에 제외**<br>
    - 렌더링 파이프라인 단게에서 수행되는 다른 컬링 개념과는 다소 다른 편<br>
      - 클리핑(Cliping) : 제출된 도형이 화면 밖으로 나갔을때, 화면 경계에 맞게 '잘라냄'(RS)<br>
      - 오클루젼(Occlusion)<br>
        : 절두체 내부에 존재하여도 '다른 물체'에 가려져 카메라에 안보이는 오브젝트를 제외<br>
          -> 다만 이는 실제로 모든 물체를 '그려봐야' 알기에 매우 무거움!<br>
          (OM)<br>

- **왜 필요한가?**<br>
  - 화면에 보이지 않는 오브젝트를 렌더 파이프라인에 태우면 비용이 발생함<br>
    - CPU: DrawCall 기록/상태 변경/커맨드 버퍼 생성 등 제출 오버헤드<br>
    - GPU: Vertex/Primitive 처리, Rasterization, Pixel shading, 메모리 대역폭 사용<br>
  - 따라서 “가벼운 컬링 계산”으로 “무거운 렌더링 작업”을 처리!<br>

## 절두체(Frustum)란?

- **형태**<br>
  - 원근(Perspective) 투영: 가까운 면(Near)이 작고 먼 면(Far)이 큰 **절두체(Truncated pyramid)**<br>
  - 직교(Orthographic) 투영: **상자(Box)** 형태<br>

- **구성**<br>
  - 절두체는 다음 6개의 평면으로 정의<br>
    - `Left / Right / Top / Bottom / Near / Far`<br>

## 절두체를 구하는 방법

### 1) 기하학적 구성: 8개 코너 → 6개 평면

- **입력**<br>
  - 카메라 위치 `C`<br>
  - 카메라 축 `R( right ), U( up ), F( forward )`<br>
  - 투영 파라미터 `FOV`, `Aspect`, `Near`, `Far`<br>

- **아이디어**<br>
  - Near/Far 평면의 중심을 구한 뒤,<br>
  - `R`, `U` 방향으로 너비/높이만큼 이동해 **Near 4개 + Far 4개 = 8개 코너**를 만듦<br>

[![Image](https://github.com/user-attachments/assets/3b190a96-f196-4765-8f37-d04210c1a69c)](https://github.com/user-attachments/assets/3b190a96-f196-4765-8f37-d04210c1a69c){: .image-popup}<br>

- **평면의 법선(외적)**<br>
  - 코너 3점 `A, B, C`가 있을 때 평면 법선은 다음처럼 표현 가능<br>
    - `n = normalize((B - A) × (C - A))`<br>
  - 법선 방향은 **절두체 내부를 향하도록** 점 순서(와인딩) 또는 부호를 맞춤<br>

### 2) 행렬 기반: View-Projection(VP)에서 6개 평면 추출

- **View 행렬**<br>
  - 월드 좌표를 카메라 좌표로 변환<br>
  - “카메라를 움직이는 대신, 월드를 카메라 반대로 이동”시키는 역변환 개념<br>

- **Projection 행렬**<br>
  - `FOV / Aspect / Near / Far`를 이용해 원근(또는 직교) 투영을 수행<br>
  - 클립 공간(Clip Space)으로 사상하기 위한 변환<br>

- **ViewProjection**<br>
  - `VP = Projection * View`<br>
  - 클립 공간 경계 조건(예: `-w ≤ x ≤ w`, `-w ≤ y ≤ w`, z 범위)을<br>
    `clip = VP * world`에 대입하면 월드 공간의 **6개 평면(Frustum planes)** 을 얻을 수 있음<br>

## 평면 방정식과 내적 판정

- **평면 방정식**<br>
  - 평면은 보통 아래 형태로 표현<br>
    - `n · p + d >= 0`  → 내부(inside)<br>
      - `n`: 평면의 법선(normal)<br>
      - `d`: 상수항<br>
      - `p`: 검사할 점(위치)<br>

[![Image](https://github.com/user-attachments/assets/4422fa46-b007-4c13-ac75-82d5b779f905)](https://github.com/user-attachments/assets/4422fa46-b007-4c13-ac75-82d5b779f905){: .image-popup}<br>

- **왜 내적으로 판정할 수 있나?**<br>
  - `n · p + d`의 결과(+/0/-)가 점 `p`가 평면 기준으로 안/밖에 있는지를 판단<br>
    - 양수: 평면 안쪽<br>
    - 0: 평면 위<br>
    - 음수: 평면 바깥쪽<br>
  - 절두체는 6개 평면의 교집합이므로, **모든 평면에 대해 조건을 만족하면 내부**이다.<br>

[![Image](https://github.com/user-attachments/assets/ec564df3-4ba0-467f-9f09-bea79e2ff86a)](https://github.com/user-attachments/assets/ec564df3-4ba0-467f-9f09-bea79e2ff86a){: .image-popup}<br>

- **중요한 규약**<br>
  - `n · p + d >= 0`를 내부 조건으로 쓰려면,<br>
    평면의 법선 `n`이 **절두체 내부를 향하도록** 맞춰져 있어야 한다.<br>

  - 또한 거리 기반 판정을 정확히 하려면,<br>
    평면을 `||n|| = 1`이 되도록 **정규화**해두는 것이 안전하다.<br>

## 실제 컬링은 바운딩 볼륨으로 수행한다

> 메시의 모든 정점을 검사하는 대신, 오브젝트의 바운딩 볼륨(Bounding Volume)을 검사해 비용을 줄인다.

### 1) Bounding Sphere

[![Image](https://github.com/user-attachments/assets/8d744b3b-818e-460c-b157-be982fc68363)](https://github.com/user-attachments/assets/8d744b3b-818e-460c-b157-be982fc68363){: .image-popup}<br>

- **아이디어**<br>
  - 구의 중심과 반지름만으로 빠르게 판정 가능.<br>

- **판정**<br>
  - `dist = n · center + d`<br>
  - `dist < -radius` 이면 구 전체가 평면 바깥 → 컬링 가능<br>
  - 그 외에는 내부 또는 교차(완전 컬링 불가)<br>

- **특징**<br>
  - 매우 빠르지만, 형태에 비해 느슨할 수 있다.<br>

### 2) Bounding Box (AABB/OBB)

- **아이디어**<br>
  - 박스로 오브젝트를 감싸고 평면과의 관계를 검사한다.<br>
  - AABB는 축 정렬이라 계산이 비교적 쉽고,<br>
  - OBB는 회전까지 포함해 더 타이트하지만 계산이 더 무겁다.<br>

- **개념**<br>
  - 평면 법선 방향으로 “가장 불리한 점(지원점)”을 이용하면<br>
    모든 코너를 검사하지 않고도 “완전 외부” 판정이 가능하다.<br>


## TMI - CPU 절두체 컬링의 한계와 GPU-driven 컬링

- **CPU 절두체 컬링이 비효율적일 수 있는 경우**<br>
  - **컬링 검사 자체가 너무 많을 때**<br>
    - 오브젝트 수가 매우 많으면(AABB/Sphere 테스트가 가볍더라도) 전체 검사 비용이 CPU에서 누적될 수 있음<br>
  - **컬링 후에도 살아남는 오브젝트가 너무 많을 때**<br>
    - 최종적으로 제출해야 할 렌더링 후보가 많으면 그만큼 **DrawCall 기록/상태 변경/커맨드 생성 비용**이 증가<br>
    - 즉, 병목이 “GPU 셰이딩”이 아니라 “CPU 제출” 쪽에서 생길 수 있음<br>

- **그래서 등장한 흐름: GPU-driven 컬링**<br>
  - CPU는 “엄청 많은 개별 DrawCall” 대신, 비교적 적은 수의 큰 작업을 제출하고<br>
  - GPU가 내부에서 **가시성(컬링) + LOD + 렌더링 단위 선택**을 수행하는 방식<br>
  - 대표적인 형태<br>
    - **Indirect Draw / ExecuteIndirect**<br>
      - GPU가 “그릴 목록(인자 버퍼)”을 만들고, 그 버퍼를 기반으로 그리기<br>
    - **Mesh Shader / Task Shader 기반**<br>
      - 기존 VS/GS 중심 파이프라인보다 “GPU 쪽에서 프리미티브 생성/컬링/증폭”을 유연하게 수행 가능<br>
    - **UE5 Nanite 개념**<br>
      - 렌더링 단위를 더 잘게(클러스터) 쪼개고,<br>
      - GPU가 화면 기여도에 따라 **클러스터 단위 가시성/LOD**를 결정해<br>
      - CPU의 개별 메시/폴리곤 단위 제출 부담을 줄이는 방향(“그릴 단위의 가상화/자동 선택”)<br>

