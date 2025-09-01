---
title: "실린더 그리기"
date : "2025-09-01 18:00:00 +0900"
last_modified_at: "2025-09-01T18:00:00"
categories:
  - Direct X
tags:
  - Cylinder
  - Normal
---

## 실린더 (원통) 그리기

<img width="2229" height="1727" alt="Image" src="https://github.com/user-attachments/assets/85d5b388-fe5f-460d-ba00-f5ba70633f11" /><br>

각도와 height<br>
아래와 위쪽 radius가 주어지고<br>

그에 따른 원통을 구현하는 것<br>
2D 원 자체는 이전에 구현하였으니<br>
저 옆 부분을 구현하는 것이 목표이다<br>

## 실린더 Vertex 구하기

```
const float dTheta = -XM_2PI / float(sliceCount);

MeshData meshData;

vector<Vertex> &vertices = meshData.vertices;

// 옆면의 바닥 버텍스들 (인덱스 0 이상 sliceCount 미만)
for (int i = 0; i <= sliceCount; i++) {
    // TODO: 작성 (텍스춰 좌표계, 버텍스 노멀 필요)

    Vertex v;
    v.position =
        Vector3::Transform(Vector3(bottomRadius, -0.5f * height, 0.0f),
                           Matrix::CreateRotationY(dTheta * float(i)));

    
    // Vector3(0.0f, -0.5f * height, 0.0f) : 원점 역할
    // 현재 위치 - 원점(radius 중앙) 을 통해 nomral을 구함
    v.normal = v.position - Vector3(0.0f, -0.5f * height, 0.0f);
    v.normal.Normalize();
    v.texcoord = Vector2(float(i) / sliceCount, 1.0f);

    vertices.push_back(v);
}

// 옆면의 맨 위 버텍스들 (인덱스 sliceCount 이상 2 * sliceCount 미만)
for (int i = 0; i <= sliceCount; i++) {
    Vertex v;
    v.position =
        Vector3::Transform(Vector3(bottomRadius, 0.5f * height, 0.0f),
                           Matrix::CreateRotationY(dTheta * float(i)));

    // Vector3(0.0f, 0.5f * height, 0.0f) : 원점 역할
    // 현재 위치 - 원점(radius 중앙) 을 통해 nomral을 구함
    v.normal = v.position - Vector3(0.0f, 0.5f * height, 0.0f);
    v.normal.Normalize();
    v.texcoord = Vector2(float(i) / sliceCount, 0.0f);

    vertices.push_back(v);
}
```

참고로 저 Transform 부분은<br>
아래와 같이 구현할 수도 있다<br>

- Normal 계산 부분<br>
  : '원점'에 해당하는 Vector3(0.0f, 0.5f * height, 0.0f) 를 통해<br>
    '현재 좌표 - 원점'으로 'Nomral'을 구하는 방식<br>

- Texture 좌표<br>
  : float(i)를 slicecount로 나누는 방식<br>
  (이전 grid와 유사)<br>

```
v.position = Vector3(0.0f, -0.5f * height, 0.0f);
float angle = -dTheta * float(i); (위에 보면 Theta에 -가 붙어 있음)
v.position.x = bottomRadius * cos(angle);
v.position.z = bottomRadius * sin(angle);
```

- 요점은 2 * Pie 를 '개수'로 나눈 후<br>
  그 각도를 * i 로 각자의 위치에 cos 와 sin으로 뿌려주는 것<br>
  (너무 외울 필요는 없다)<br>

## 실린더 Index 구하기

```
vector<uint16_t> &indices = meshData.indices;

for (int i = 0; i < sliceCount; i++) {
    // TODO: 삼각형 두 개 씩
    indices.push_back(i);
    indices.push_back(i + sliceCount + 1);
    indices.push_back(i + 1 + sliceCount + 1);

    indices.push_back(i);
    indices.push_back(i + 1 + sliceCount + 1);
    indices.push_back(i + 1);
}
```

이것도 이전에 배운 grid의 인덱스 구하는 1차 높이 부분과 매우 유사하다<br>

