---
title: "구 그리기"
date : "2025-09-02 12:00:00 +0900"
last_modified_at: "2025-09-02T12:00:00"
categories:
  - Direct X
tags:
  - Sphere
---

## 구 그리기

<img width="587" height="551" alt="Image" src="https://github.com/user-attachments/assets/35314191-6829-450b-b125-1f8618c6c796" /><br>

이전에 그렸던 '실린더'와 매우 유사하다<br>
원통의 각 '그리드'에 해당하는 부분을<br>


### 힌트

<img width="723" height="679" alt="Image" src="https://github.com/user-attachments/assets/4e66b546-76f1-42d1-8147-8dee24eca2fb" /><br>

요점은 radius값을 잘 이용하는 것<br>
0~1일땐 '중앙'으로 값을 모아주고<br>
0.5에 가까울수록 radius 값을 증폭 시키는 식으로 구현이 가능<br>
x : radius * sin(a);<br>
y : radius * cos(a);<br>

이후 시작점을 2PIE / (가로축 개수) 로 돌려주면 된다<br>

- 각각의 y값이 고정된 '원'들 위에 x값이 바뀐 정점들이 생성<br>
  이후 '삼각형'을 그려주어 이어주면 '원'이 된다<br>

<img width="738" height="747" alt="Image" src="https://github.com/user-attachments/assets/4a9e2aac-fbfe-42cb-ac1b-3f2d7be53272" /><br>



## 예제 - 구 그리기

```
const float dTheta = -XM_2PI / float(numSlices);
const float dPhi = -XM_PI / float(numStacks);

MeshData meshData;

vector<Vertex> &vertices = meshData.vertices;

for (int j = 0; j <= numStacks; j++) {

    // 스택에 쌓일 수록 시작점을 x-y 평면에서 회전 시켜서 위로 올리는 구조
    Vector3 stackStartPoint =
        Vector3(radius * sin(dPhi * j),
                radius * cos(dPhi * j),
                0.0f); // 실린더

    for (int i = 0; i <= numSlices; i++) {
        Vertex v;

        // 시작점을 x-z 평면에서 회전시키면서 원을 만드는 구조
        v.position = Vector3::Transform(
            stackStartPoint, Matrix::CreateRotationY(dTheta * float(i)));

        v.normal = v.position; // 원점이 구의 중심
        v.normal.Normalize();
        v.texcoord =
            Vector2(float(i) / numSlices, float(j) / numStacks);

        vertices.push_back(v);
    }
}

vector<uint16_t> &indices = meshData.indices;

for (int j = 0; j < numStacks; j++) {

    const int offset = (numSlices + 1) * j;

    for (int i = 0; i < numSlices; i++) {

        indices.push_back(offset + i);
        indices.push_back(offset + i + numSlices + 1);
        indices.push_back(offset + i + 1 + numSlices + 1);

        indices.push_back(offset + i);
        indices.push_back(offset + i + 1 + numSlices + 1);
        indices.push_back(offset + i + 1);
    }
}
```

- 요점은 x는 sin을 통하여<br>
  (0,1 : 0 , 0.5 는 1)<br>
  y는 cos 를 통하여<br>
  (0 : -0.5, 0.5 : 0, 1 : 0.5)<br>
  '비율'을 맞추어 주는것<br>

- radius * sin(dPhi * j),<br>
  radius * cos(dPhi * j)<br>
  로 각각 x,y 좌표를 잡으면<br>
  원점으로 부터의 길이도 radius<br>

- nomral 계산은 자기 자신의 위치를 normalize<br>
  (자신의 위치 - 원점이나<br>
  원점이 (0,0,0)이기에)<br>

- 텍스쳐 코드는 아래에서 시작하기에 그대로 유지<br>