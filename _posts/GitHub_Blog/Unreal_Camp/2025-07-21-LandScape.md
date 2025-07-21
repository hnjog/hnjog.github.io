---
title: "LandScape, Foliage"
last_modified_at: "2025-07-21T16:30:00"
categories:
  - 언리얼 5
tags:
  - LandScape
  - Foliage
---

## LandScape
LandScape는 언리얼에서 넓은 지형을 생성하고, 편집하기 위한 특수한 액터 타입으로<br>
HeightMap 기반의 그리드형 타일 지형으로 구성된다<br>

### 랜드 스케이프의 구조

| 요소                      | 설명                          |
| ----------------------- | --------------------------- |
| **Heightmap**           | 회색조 이미지 기반의 높이 정보           |
| **Layer Paint**         | 지형 재질에 페인팅하는 머티리얼 레이어       |
| **Section / Component** | LOD 및 최적화를 위한 내부 분할 단위      |
| **LandscapeSpline**     | 도로/강/경로 등을 그릴 수 있는 스플라인 시스템 |
| **Collision**           | 높이에 따른 자동 충돌 생성 지원          |

### 랜드 스케이프의 기능들

| 기능                      | 설명                                       |
| ----------------------- | ---------------------------------------- |
| Sculpt                  | Raise/Lower, Flatten, Smooth 등 툴로 높낮이 조정 |
| Paint                   | Landscape Material에 정의된 Layer를 칠함        |
| Visibility Mask         | 특정 영역을 투명하게 처리 (예: 구멍 뚫기)                |
| Landscape Material      | Layer 기반 머티리얼 시스템 지원 (예: 풀, 바위, 눈 등 분기)  |
| Runtime Virtual Texture | 거리별 텍스처 표현 최적화                           |
| LOD / Streaming         | Level Streaming과 함께 대규모 월드 최적화 가능        |

전용 렌더링 파이프라인과 LandScape용 머테리얼이 존재<br>

Foliage 와의 구성으로 대규모 월드를 구성하는데 도움을 주는 기능이다<br>

---

<img width="3063" height="1307" alt="Image" src="https://github.com/user-attachments/assets/06053d92-f693-4347-bb01-56c4a37a967f" /><br>

스컬프팅, 스무드, 플래튼, 침식 등의 기능을 이용하여<br>
간략한 지형맵을 구성한 모습<br>

---

넓은 게임 월드를 구상할때 고려할만한 기술이며<br>
동시에 '넓은 게임 월드'를 구현할때 고려되기에 최적화를 뺴놓을 수 없음<br>

### 랜드 스케이프와 연관된 최적화 기술들

| 기능                                | 설명                                           |
| --------------------------------- | -------------------------------------------- |
| **LOD (Level of Detail)**         | 자동으로 거리 기반 LOD 적용 (섹션 단위로 분할)                |
| **World Partition**               | 월드를 자동 스트리밍 가능하게 분할                          |
| **Runtime Virtual Texture (RVT)** | 지형 텍스처를 미리 굽고, Foliage와 캐릭터 그림자를 통합 렌더       |
| **시야 절단(Culling)**                | 오클루전 및 Distance Cull로 보이지 않는 부분 제거           |
| **Heightmap Resolution 제어**       | 너무 정밀하면 오히려 LOD가 늦게 적용되어 성능 저하               |

---

## Foliage
LandScape가 '지형'을 생성하는 주요 기능이라면<br>
Foliage는 '정적인 물체'를 배치하는 주요 기능이다<br>
(ex : 건물, 차, 나무, 풀 등등)<br>

### 폴리지의 구조

| 요소                          | 설명                                        |
| --------------------------- | ----------------------------------------- |
| **Foliage Type Asset**      | Foliage 인스턴스의 설정 데이터 (스케일, 밀도, 정렬, 랜덤성 등) |
| **Foliage Tool**            | 브러시처럼 클릭-드래그로 다수 배치 가능                    |
| **Instanced Static Mesh**   | 성능을 위해 한 종류의 메시를 인스턴싱하여 렌더링               |
| **Cull Distance**           | 거리별 제거 설정으로 최적화                           |
| **Align to Surface Normal** | 지면 기울기에 따라 정렬 여부                          |


### 폴리지의 기능들

| 기능                 | 설명                         |
| ------------------ | -------------------------- |
| Paint Tool         | 특정 영역에 브러시처럼 자연물 그리기       |
| Erase Tool         | 선택적 제거                     |
| Select Tool        | 개별 선택 및 조정 (편집 모드)         |
| Foliage Type       | 메시/스케일/랜덤/밀도 등 조절 가능       |
| Collision          | 충돌 ON/OFF 가능 (선택적)         |
| Procedural Foliage | 자동 스폰 (나무 자라기, 번식 등 시뮬레이션) |

### 폴리지를 이용한 '배치'의 특징

| 장점                 | 단점                          |
| ------------------ | --------------------------- |
| ✅ GPU 인스턴싱으로 매우 빠름 | ❌ 개별 액터가 아님 (트리거, 타겟팅이 어려움) |
| ✅ 대규모 자연환경 구축에 최적  | ❌ 상호작용하려면 수동 처리 필요          |
| ✅ 에디터 상에서 빠른 배치    | ❌ 리플리케이션/네트워크 미지원           |

---

<img width="3067" height="1967" alt="Image" src="https://github.com/user-attachments/assets/8e861fa8-b316-4d58-9d4e-fa7810de80d0" /><br>

- 건물을 Actor 폴리지로 배치한 상황<br>

<img width="3067" height="1383" alt="Image" src="https://github.com/user-attachments/assets/f10fe65a-0b26-40cf-9d31-009a84e43caa" /><br>

- 이후 풀과 나무, 바위 등을 StaticMesh 폴리지로 배치하였음<br>

폴리지는 3D 환경의 '배치'용도로 자주 사용된다<br>
다만, 많은 양의 물건을 배치할 수 있기에 폴리지 역시 최적화와 연관이 깊음<br>

---

### 폴리지와 연관된 최적화 기능들

| 기능                              | 설명                                                         |
| ------------------------------- | ---------------------------------------------------------- |
| **Instanced Static Mesh (ISM)** | 동일 메시를 GPU 인스턴싱으로 매우 빠르게 렌더                                |
| **HISM (Hierarchical ISM)**     | 트리 구조로 거리별로 배치된 Foliage를 효율적으로 Cull                        |
| **Cull Distance 설정**            | 거리별로 자동으로 삭제/비표시 처리                                        |
| **LOD 모델 사용**                   | 나무 같은 건 가까이선 고해상도, 멀리선 저해상도 버전 표시                          |
| **Shadow 옵션 제한**                | Foliage는 종종 그림자를 끄거나, 간접광만 받도록 설정함                         |


- Nanite(나나이트)?<br>
  https://dev.epicgames.com/documentation/ko-kr/unreal-engine/nanite-virtualized-geometry-in-unreal-engine <br>
  UE5의 가상화 지오메트리 시스템으로, LandScape와 Foliage에 적용할 수 있는 새로운 기술<br>
  LOD의 자동처리와 매우 높은 수준의 퀄리티를 보장하지만 아직 개발중이며 지원하지 않는 부분이 많기에<br>
  Unreal의 버전이 더 올라갔을 때, 사용이 가능해보이며 현재로서는 키워드만 알아두자<br>
  - 초고해상도 메시를 실시간 렌더링하며, 자동으로 LOD까지 처리해주는 기술 정도로 인식하되<br>
    아직 개선이 필요한 기술임을 알아두자<br>