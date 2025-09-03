---
title: "SubDivision"
date : "2025-09-03 12:00:00 +0900"
last_modified_at: "2025-09-03T12:00:00"
categories:
  - Direct X
tags:
  - SubDivision
---

## SubDivision?

메시(폴리곤)을 더 '잘게' 쪼개서 부드러운 곡면을 만드는 기법<br>

- 원래 '정점'과 '면'을 기준으로 새로운 버텍스를 계산<br>
  (중심에서 선을 이어서 삼각형 1개에서 삼각형 4개가 만들어짐)<br>
- Smoothing 과정을 통해 '곡선'에 가까운 형태를 만든다<br>

일반적으로 모델링 단계에서 자주 사용됨<br>
(ex : 블렌더 /마야)<br>

저폴리곤 -> 고폴리곤<br>
실시간 처리보단 '사전 처리'에 이용<br>
(모델링 초기화 등)<br>

과도하게 사용하거나 원본 텍스쳐가 저폴리곤 일시 텍스쳐 좌표가<br>
다소 깨질 수는 있음<br>

- GPU에서도 사용할 수 있음!<br>
  (저폴리곤으로 CPU에서 다루고, GPU에서 사용하여<br>
   폴리곤을 올릴 수 있음)<br>
  (그래도 Tessellation의 성능이 좋기에<br>
  보통 테셀레이션을 사용하는 편)<br>

### Tessellation과 비교?

테셀레이션은 GPU에서 실시간으로 삼각형을 더 잘게 나누어<br>
해상도를 높이는 기법이다<br>

Tessellation Shader : DX<br>
UE/Unity에선 Displacement Mapping 이라 함<br>

- 입력된 폴리곤을 '하드웨어'가 분할<br>
- 분할된 버텍스 위치를 Hull/Domain Shader에서 재조정 가능<br>

실시간 LOD 에 적용 가능<br>
(GPU 연산량이 많이 요구)<br>


## SubDivision 예제

<img width="2225" height="1713" alt="Image" src="https://github.com/user-attachments/assets/1b0baa60-8deb-40b3-805c-d1449d5e0071" /><br>

Sphere 을 그리려 하나<br>
매우 낮은 count가 주어진 경우<br>
SubDivision을 이용하여<br>
폴리곤 수를 높이는 예제이다<br>

<img width="694" height="579" alt="Image" src="https://github.com/user-attachments/assets/10bbeb80-aa68-4cf1-b584-5502f2f537d6" /><br>

각각의 새로운 vertex는<br>
기존 삼각형 좌표들 사이의 '평균'을 내서 구할 수 있음<br>

- 3,3 구체에 1번 적용<br>

<img width="2215" height="1703" alt="Image" src="https://github.com/user-attachments/assets/7d9d5eb0-f3d2-484f-9ea7-de790b23c750" /><br>

- 3,3 구체에 2번 적용<br>

<img width="2229" height="1727" alt="Image" src="https://github.com/user-attachments/assets/ec70d685-53d2-4f76-9321-8d6972244e98" /><br>

적용할때마다 점점 폴리곤 수가<br>
1 -> 4 -> 16... 이런 식으로<br>
기하급수적으로 늘어난다<br>

- 12,12 구체에 1번 적용<br>

<img width="2225" height="1721" alt="Image" src="https://github.com/user-attachments/assets/6e653058-c82d-45ca-8a99-635da72c88c8" /><br>

상당히 부드러워 보이는 구체이며<br>
텍스쳐 쪽의 문제도 보이지 않는다<br>

### 예제 코드

```
MeshData GeometryGenerator::SubdivideToSphere(const float radius,
                                              MeshData meshData) {

    using namespace DirectX;
    using DirectX::SimpleMath::Matrix;
    using DirectX::SimpleMath::Vector3;

    // 원점이 중심이라고 가정
    // 입력 받은 구 모델의 반지름 조절
    for (auto &v : meshData.vertices) {
        v.position = v.normal * radius;
    }

    // 구의 표면으로 옮기고 노멀 계산
    auto ProjectVertex = [&](Vertex &v) {
        v.normal = v.position;
        v.normal.Normalize();
        v.position = v.normal * radius;

        // 주의: 텍스춰가 이음매에서 깨집니다.
        // atan vs atan2
        // https://stackoverflow.com/questions/283406/what-is-the-difference-between-atan-and-atan2-in-c

         /*const float theta = atan2f(v.position.z, v.position.x);
         const float phi = acosf(v.position.y / radius);
         v.texcoord.x = theta / XM_2PI;
         v.texcoord.y = phi / XM_PI;*/
    };

    // 버텍스가 중복되는 구조로 구현
    MeshData newMesh;
    uint16_t count = 0;
    for (size_t i = 0; i < meshData.indices.size(); i += 3) {
        size_t i0 = meshData.indices[i];
        size_t i1 = meshData.indices[i + 1];
        size_t i2 = meshData.indices[i + 2];

        Vertex v0 = meshData.vertices[i0];
        Vertex v1 = meshData.vertices[i1];
        Vertex v2 = meshData.vertices[i2];

        Vertex v3;
        // 위치와 텍스춰 좌표 결정
        v3.position = (v0.position + v2.position) / 2;
        v3.normal = (v0.normal + v2.normal) / 2;
        v3.texcoord = (v0.texcoord + v2.texcoord) / 2;

        Vertex v4;
        // 위치와 텍스춰 좌표 결정
        v4.position = (v0.position + v1.position) / 2;
        v4.normal = (v0.normal + v1.normal) / 2;
        v4.texcoord = (v0.texcoord + v1.texcoord) / 2;

        Vertex v5;
        // 위치와 텍스춰 좌표 결정
        v5.position = (v1.position + v2.position) / 2;
        v5.normal = (v1.normal + v2.normal) / 2;
        v5.texcoord = (v1.texcoord + v2.texcoord) / 2;

        ProjectVertex(v3);
        ProjectVertex(v4);
        ProjectVertex(v5);

        // 모든 버텍스 새로 추가
        newMesh.vertices.push_back(v0);
        newMesh.vertices.push_back(v4);
        newMesh.vertices.push_back(v3);
        
        newMesh.vertices.push_back(v4);
        newMesh.vertices.push_back(v1);
        newMesh.vertices.push_back(v5);

        newMesh.vertices.push_back(v4);
        newMesh.vertices.push_back(v5);
        newMesh.vertices.push_back(v3);

        newMesh.vertices.push_back(v3);
        newMesh.vertices.push_back(v5);
        newMesh.vertices.push_back(v2);


        // 인덱스 업데이트
         for (uint16_t j = 0; j < 12; j++) {
             newMesh.indices.push_back(j + count);
         }
         count += 12;
    }

    return newMesh;
}
```

- 각각의 정점이 삼각형의 '위'에 있는 존재이기에<br>
  0.5에 해당하는 선형보간을 해준다<br>

- 버텍스들을 전부 하나씩 추가해준다<br>
  (인덱스는 나란히 더해주는 편)<br>

- SubDivision에서는 '새로운 정점' 자체가<br>
  기존에 없던 Vertex이며, 경계 등의 이유로<br>
  vertex를 재활용하지 않고 '삼각형'을 만들어 구현<br>
  (Index Buffer 재활용률이 낮다)<br>
  - '같은 좌표'이지만,<br>
    '텍스쳐'의 경계 (UV Seam)<br>
     노멀 불연속<br>
     등의 이유로 '다른 Vertex' 취급하는 것이 안정적<br>

- 이 부분은 '구'임을 감안하고 작성된 SubDivision 알고리즘이다<br>
  '모든 물체'에 알맞게 적용되는 것이 아니기에<br>
  사용되는 SubDivision 알고리즘에 따라 다른 결과가 발생할 수 있음<br>
  (KOBBELT , Catmull clack, butterfly 등등)<br>