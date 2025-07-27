---
title: "Backface Culling"
last_modified_at: "2025-07-27T14:30:00"
categories:
  - 그래픽스
tags:
  - 컬링
  - 벡터의 외적
  - 삼각형 내부 판정
  - 바리센트릭 좌표계
---

## 컬링(Culling)
3D 그래픽스에서 불필요한 '도형'이나 '픽셀'을 그리지 않는 최적화 기법<br>

- GPU가 계산하지 않아도 되는 정보를 걸러내어 성능을 향상<br>
- ex : 화면 밖의 오브젝트, 뒤집힌 삼각형, 다른 물체에 가려진 물체 등<br>

<img width="520" height="552" alt="Image" src="https://github.com/user-attachments/assets/99ae5dfe-b575-495a-b396-a7f0964f4cb8" /><br>

GPU 연산량을 감소시키며<br>
전처리를 통해 CPU/GPU의 최적화로 대규모 씬(오픈월드 등)에서<br>
필요한 최적화<br>

### 대표적인 컬링 종류 요약

| 컬링 종류                      | 설명                                  |
| -------------------------- | ----------------------------------- |
| **Backface Culling**       | 삼각형이 **카메라 반대 방향**을 향하면 제거          |
| **Frustum Culling**        | 카메라 시야(Frustum) 밖에 있는 물체 제거         |
| **Occlusion Culling**      | **다른 물체에 가려져서 안 보이는** 물체 제거         |
| **Distance / LOD Culling** | 너무 멀거나, 카메라에 영향이 적은 경우 제거 또는 LOD 처리 |

<img width="975" height="837" alt="Image" src="https://github.com/user-attachments/assets/e013206e-34b0-41c8-b36d-12653ec3ea40" /><br>

---

### Unreal에선?

| 컬링 방식             | 설명                                                    |
| ----------------- | ----------------------------------------------------- |
| Backface Culling  | **Mesh 설정**에서 가능 (`Two-Sided` 옵션 끄기)                  |
| Frustum Culling   | **월드의 시야 밖 객체는 자동 제거**                                |
| Occlusion Culling | GPU 기반 자동 처리 (HLOD, Precomputed Visibility 사용 가능)     |
| Distance Culling  | `Cull Distance Volume` 또는 `LODDistanceFactor` 설정으로 적용 |


## Backface Culling (후면 컬링)
삼각형의 앞면 / 뒷면을 판단하여<br>
'뒷면'인 경우 카메라에<br>
보이지 않으므로 대상에서 제외<br>

### 판별 기준?<br>

| 방식                         | 설명                           | 실제 사용 여부                        |
| -------------------------- | ---------------------------- | ------------------------------- |
| **법선 vs 카메라 방향 (ViewDir)** | normal과 view vector의 내적      | ❌ 일반적인 컬링에는 사용 안 함 (조명 계산에 사용됨) |
| **정점 순서 (Winding Order)**  | 정점 나열 순서를 기준으로 시계/반시계 여부로 판단 | ✅ GPU가 실제로 사용하는 방식              |

법선과 카메라 방향을 통한 방식은<br>
복잡하고 느리기에 후자가 선호된다<br>

<br>

후자를 통해 판별하는 예시와<br>
Vector 2D 외적을 통해 이를 적용하는 방식을<br> 
알아볼 예정이다<br>

---

### Edge Function
두 벡터의 '외적'을 통해<br>
'점'이 어떠한 방향에 존재하는지를 판단하는 기능<br>

<img width="712" height="708" alt="Image" src="https://github.com/user-attachments/assets/eaf0c904-4e7c-48f2-88d5-2e9c775ebdf9"/><br>

```
float EdgeFunction(const vec2 &v0, const vec2 &v1,
                                  const vec2 &point) {
    const vec2 a = v1 - v0;
    const vec2 b = point - v0;
    return a.x * b.y - a.y * b.x; // Cross
}
```

(이미 래스터화된 '스크린 좌표'이므로<br>
vec2 로 다룰 수 있음)<br>
(z값은 어차피 depth buffer에 존재)<br>

2차원 벡터 간의 외적을 통하여 나오는 값은<br>
두 벡터 a,b를 통해 만든 평행사변형의 '넓이'와 같음<br>
따라서 그 넓이가 '양수'라면<br>
v1-v0 과 p - vo 이 이루는 벡터는 '양의 부호'를 가진다고 판단 가능<br>
(넓이가 0이라면 p가 사실상 두 선분위에 있다는 말이며<br>
넓이가 -라면 p의 위치는 '음의 부호'측에 있다 판별 가능)<br>

해당 결과를 result로 표현하면,<br>
- result > 0 : P가 v0->v1 의 왼쪽(반시계의 방향 : CCW)<br>
- result < 0 : P가 v0->v1 의 오른쪽(시계 방향 : CW)<br>
- result == 0 : P가 v0->v1 선분 위에 있음<br>

이러한 기능을 통해<br>
- 삼각형 내부 판정 (Barycentric Coordinate)<br>
- 바리센트릭 좌표계(Barycentric Coordinate)
- 면의 방향성 판정 (Backface Culling)<br>

등이 가능해진다<br>

---

### 삼각형 내부 판정법?

삼각형을 이루는 3개의 점을 v0,v1,v2 라 하고<br>
판별하고 싶은 픽셀을 p라 지정한 경우<br>
다음과 같은 방식으로 판정이 가능하다<br>

```
if (Edge(v1, v2, p) >= 0 &&
    Edge(v2, v0, p) >= 0 &&
    Edge(v0, v1, p) >= 0)
{
    // p는 삼각형 내부
}
```

---

### 바리센트릭 좌표계?
전체 삼각형에서 특정한 점 p가<br>
각 꼭지점에서 '얼마나' 가까운지를 보간하는 좌표<br>

```
alpha = Edge(v1, v2, p) 
beta = Edge(v2, v0, p)
gamma = Edge(v0, v1, p)
A = Edge(v0, v1, v2)

alpha /= A;
beta  /= A;
gamma /= A;

```

픽셀 값을 일일이 지정하지 않아도<br>
삼각형을 이루는 정점에 대한 정보를 통하여<br>
color, depth,uv 등의 <br>
상대적인 정보를 구하기 쉬워진다<br>

예시)<br>

```
const float area = alpha + beta + gamma; // A를 이렇게 구할 수 도 있음(한번더 함수를 안써도 됨)
const vec3 color =
    (alpha * c0 + beta * c1 + gamma * c2) / area;
const vec2 uv =
    (alpha * uv0 + beta * uv1 + gamma * uv2) / area;

const float depth = (alpha * this->vertexBuffer[i0].z +
                     beta * this->vertexBuffer[i1].z +
                     gamma * this->vertexBuffer[i2].z) /
                    area;
```

---

### 면의 방향성 판정?
그래픽스에서는 삼각형의 정점의 '순서'가 '반시계방향'(CCW)인 경우를 '앞면'으로 판단<br>
(정점 순서 기반, Winding Order)<br>

따라서 v0,v1,v2 삼각형의 '주어진 정점 순서'를 통해<br>
해당 면이 '앞면'인지를 판단하고 그릴지 말지를 정할 수 있다<br>

```
const float areaCheck = EdgeFunction(v0,v1, v2);
// 뒷면 컬링 중이면 이 삼각형을 그리지 않음
if (this->cullBackface && areaCheck < 0.0f)
    return;
```

v2가 v0,v1 벡터의 '왼쪽'에 존재(result > 0)하는지 확인하여<br>
'앞면'인지 확인하고 아니라면 그대로 return하여 면을 그리지 않음<br>


## 정리
1. Backface Culling은 카메라에서 볼 수 없는 '삼각형 뒷면'을 그리지 않는 최적화 기법이다<br>

2. '앞/뒷면'을 판별하는데에는 '정점의 나열 순서'를 보고<br>
   시계/반시계 중 '설정된 값'을 통해 파악한다<br> 

3. Edge Function을 통하여<br>
   하나의 '점'이 특정 벡터의 '왼쪽/오른쪽'인지를 판별하고<br>
   이를 통해 '정점'의 순서를 판별할 수 있다<br>


- 컬링을 꺼야할때?<br>
  : '양면 재질', '단면 렌더링', '얇거나 투명한 구조물' 등에는<br>
    컬링을 꺼야 의도된 재질을 연출할 수 있다<br>