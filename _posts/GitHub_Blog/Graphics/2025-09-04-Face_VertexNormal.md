---
title: "Normal Vector 와 렌더링 결과"
date : "2025-09-04 12:00:00 +0900"
last_modified_at: "2025-09-04T12:00:00"
categories:
  - Direct X
tags:
  - Normal
---

## Normal 벡터와 렌더링 결과

[![Image](https://github.com/user-attachments/assets/560ec96a-e145-4e8c-99ab-6b0addad02b5)](https://github.com/user-attachments/assets/560ec96a-e145-4e8c-99ab-6b0addad02b5){: .image-popup}<br>

이전에 배운 SubDivision에서<br>
'정점'의 위치가 같으면<br>
Normal은 같은 벡터를 사용<br>
-> 원형 이기에 '부드러움'을 표현하기 위함<br>
(SubDivision과 직접적인 연관은 딱히 없지만)<br>

그렇지만 '사각형'의 예제 등에서는<br>
'같은 위치'라도<br>
그 사각형이 속해있는 위치에 따라<br>
Normald을 다르게 주었다<br>
-> 사각형 이기에 '끊어짐'을 표현<br>

### 예제 - 구체의 정점 노멀을 각각의 표면이 향하는 곳(Face Normal)으로 변경

[![Image](https://github.com/user-attachments/assets/ac99733f-844f-4dd9-ad70-f884582c1bb8)](https://github.com/user-attachments/assets/ac99733f-844f-4dd9-ad70-f884582c1bb8){: .image-popup}<br>

```
auto UpdateFaceNormal = [](Vertex &v0, Vertex &v1, Vertex &v2) {

    // v0, v1, v2로 이루어진 삼각형의 faceNormal 계산
    auto faceNormal = v0.normal + v1.normal + v2.normal;
    faceNormal.Normalize();

    v0.normal = faceNormal;
    v1.normal = faceNormal;
    v2.normal = faceNormal;
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
    v3.position = (v0.position + v2.position) * 0.5f;
    v3.texcoord = (v0.texcoord + v2.texcoord) * 0.5f;
    ProjectVertex(v3);

    Vertex v4;
    v4.position = (v0.position + v1.position) * 0.5f;
    v4.texcoord = (v0.texcoord + v1.texcoord) * 0.5f;
    ProjectVertex(v4);

    Vertex v5;
    v5.position = (v1.position + v2.position) * 0.5f;
    v5.texcoord = (v1.texcoord + v2.texcoord) * 0.5f;
    ProjectVertex(v5);

    UpdateFaceNormal(v4, v1, v5);
    UpdateFaceNormal(v0, v4, v3);
    UpdateFaceNormal(v3, v4, v5);
    UpdateFaceNormal(v3, v5, v2);

    newMesh.vertices.push_back(v4);
    newMesh.vertices.push_back(v1);
    newMesh.vertices.push_back(v5);

    newMesh.vertices.push_back(v0);
    newMesh.vertices.push_back(v4);
    newMesh.vertices.push_back(v3);

    newMesh.vertices.push_back(v3);
    newMesh.vertices.push_back(v4);
    newMesh.vertices.push_back(v5);

    newMesh.vertices.push_back(v3);
    newMesh.vertices.push_back(v5);
    newMesh.vertices.push_back(v2);

    for (uint16_t j = 0; j < 12; j++) {
        newMesh.indices.push_back(j + count);
    }
    count += 12;
}
```

각각의 정점의 '노멀 방향'을 해당 '표면'이 향하는 부분으로 수정<br>
(SubDivision 코드 사용)<br>

### 예제 2 - 코드 수정시 결과가 다르다?

[![Image](https://github.com/user-attachments/assets/8f966015-8532-43be-b5e8-8e6fd959fe3a)](https://github.com/user-attachments/assets/8f966015-8532-43be-b5e8-8e6fd959fe3a){: .image-popup}<br>

```
UpdateFaceNormal(v4, v1, v5);

newMesh.vertices.push_back(v4);
newMesh.vertices.push_back(v1);
newMesh.vertices.push_back(v5);

UpdateFaceNormal(v0, v4, v3);

newMesh.vertices.push_back(v0);
newMesh.vertices.push_back(v4);
newMesh.vertices.push_back(v3);

UpdateFaceNormal(v3, v4, v5);

newMesh.vertices.push_back(v3);
newMesh.vertices.push_back(v4);
newMesh.vertices.push_back(v5);

UpdateFaceNormal(v3, v5, v2);

newMesh.vertices.push_back(v3);
newMesh.vertices.push_back(v5);
newMesh.vertices.push_back(v2);
```

예제의 FaceNormal 호출 시점을 변경하면 이미지가 조금 다르게 보인다<br>

왜 그럴까?<br>

```
auto UpdateFaceNormal = [](Vertex &v0, Vertex &v1, Vertex &v2) {

    // v0, v1, v2로 이루어진 삼각형의 faceNormal 계산
    auto faceNormal = v0.normal + v1.normal + v2.normal;
    faceNormal.Normalize();

    v0.normal = faceNormal;
    v1.normal = faceNormal;
    v2.normal = faceNormal;
};
```

해당 부분에서 v0,v1,v2 에 해당하는 faceNormal을<br>
'바꿔주고 있기 때문'이다<br>

그렇기에 '처음'에 전부 호출하는 방식은 '각각' 전체를 기준으로<br>
FaceNomral이 처리된 것이고<br>

현재 방식은<br>
중간 중간 FaceNormal이 적용된 것이 들어가고<br>
그 이후에 들어가는 부분과 그 이전 같은 위치의 Normal이 '달라지기' 때문이다<br>