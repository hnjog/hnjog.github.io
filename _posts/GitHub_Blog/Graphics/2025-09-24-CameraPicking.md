---
title: "Camera Picking"
date : "2025-09-24 12:00:00 +0900"
last_modified_at: "2025-09-24T12:00:00"
categories:
  - Direct X
  - Graphics
tags:
  - Camera Picking
---

## Camera Picking 에 대하여

기본적으로 다양한 게임과 같은 3D 렌더링에서<br>
마우스 좌클릭을 통해 '물체'를 선택하는 과정을<br>
`Picking` 이라 표현한다<br>

특정한 '마우스 좌표'가 '어느 부분'을 향하려는 지를 알려면<br>
Mouse의 NDC 좌표와 이전에 배운 '절두체'의 관계에 대하여 다시 복습해야 한다<br>

- Mouse 좌표는 이미 NDC로 변환된 좌표<br>
  (공간들 : Object -> World -> View -> Clip -> NDC)<br>
  (각각의 변환을 위해 Mode, View ,Projection 행렬변환 후, 원근 분할과 뷰포트 변환일 이루어진다)<br>
  (예제에서는 MouseNDC 좌표를 통해 Clip 시점의 좌표로 변환된 상태)<br>
  (화면 넓이/높이 를 통한 적용)<br>

[![Image](https://github.com/user-attachments/assets/b829827c-215a-4012-9fc4-0b5a4882100e)](https://github.com/user-attachments/assets/b829827c-215a-4012-9fc4-0b5a4882100e){: .image-popup}<br>

- 따라서 피킹은 절두체인 Projection 행렬을 '역'으로 적용하여<br>
  MouseNDC 좌표를 Clip 에서 View 행렬로 변환하면<br>
  해당 시점에서의 '마우스 클릭 방향'을 구현할 수 있다<br>
  (마우스 클릭의 절두체 최소 z값 -> 절두체 최대 z값 에 대한 Projection 행렬 역적용)<br>

- 이후 해당하는 '방향'이 특정한 물체와 '부딪히는지'를 검사<br>
  (예제에서는 Ray와 Intersects 함수를 이용)<br>

[![Image](https://github.com/user-attachments/assets/5a968def-72ae-4c4c-8831-7d6effbbf56e)](https://github.com/user-attachments/assets/5a968def-72ae-4c4c-8831-7d6effbbf56e){: .image-popup}<br>


### 예제 - RayTrace를 통해 클릭하여 만난 물체에 '구체' 생성하기

1. 충돌 여부 확인<br>

```cpp
Vector3 cursorNdcNear = Vector3(m_cursorNdcX, m_cursorNdcY, 0.0f);

// ViewFrustum에서 먼 면 위의 커서 위치 (z값 주의)
Vector3 cursorNdcFar = Vector3(m_cursorNdcX, m_cursorNdcY, 1.0f);

// NDC 커서 위치를 월드 좌표계로 역변환 해주는 행렬
Matrix inverseProjView = (viewRow * projRow).Invert();

// ViewFrustum 안에서 PickingRay의 방향 구하기
Vector3 projectNear = Vector3::Transform(cursorNdcNear,inverseProjView);
Vector3 projectFar = Vector3::Transform(cursorNdcFar, inverseProjView);

Vector3 dir = projectFar - projectNear;
dir.Normalize();

// 광선을 만들고 충돌 감지
SimpleMath::Ray curRay = SimpleMath::Ray(projectNear,dir);
float dist = 0.0f;
m_selected = curRay.Intersects(m_mainBoundingSphere,dist);
```

- 해당 변수 좌표는 화면의 Witdh, height 적용을 하지 않은 값<br>
  (Clip Space)<br>

- NDC 좌표를 기반으로 한 각 x,y 좌표를 통하여<br>
  Near(Z : 0), Far (Z : 1)<br>
  좌표를 만든다<br>

- 이후 각각의 좌표를 World 좌표로 변환하기 위하여<br>
  View, Projection 행렬을 역으로 적용하기 위해<br>
  Invent(역행렬) 적용한 행렬을 '적용'하여<br>
  두 좌표를 World 좌표로 변환<br>
  (이런 식으로 '역변환' 행렬을 사용하여 좌표 변환을<br>
   하는것은 다양한 렌더링 기술에 응용 가능)<br>

- 이후 끝좌표 - 시작 좌표 를 통해 Dir을 구한다<br>

- normalize한 뒤, Intersects를 해주어<br>
  충돌 여부를 검사<br>
  (부딪히면 dist가 ref로 반환된다)<br>

2. 구체 생성<br>

```cpp
if (m_selected) {
    m_cursorSphere.m_basicPixelConstantData.eyeWorld = eyeWorld;

    dist -= 0.05f;
    // 충돌 지점에 작은 구 그리기
    Matrix modelMat =
        Matrix::CreateTranslation(projectNear + dir * dist);

    Matrix invTransposeRow = modelMat;
    invTransposeRow.Translation(Vector3(0.0f));
    invTransposeRow = invTransposeRow.Invert().Transpose();
    m_cursorSphere.m_basicVertexConstantData.model =
        modelMat.Transpose();
    m_cursorSphere.m_basicVertexConstantData.invTranspose =
        invTransposeRow.Transpose();
    m_cursorSphere.m_basicVertexConstantData.view = viewRow.Transpose();
    m_cursorSphere.m_basicVertexConstantData.projection =
        projRow.Transpose();
    m_cursorSphere.UpdateConstantBuffers(m_device, m_context);
}
```

- 이후 클릭용 구체의 세팅을 해주면 완성!<br>

- 초기 위치는<br>
  우리가 'ray'를 시작하는 ProjectNear로 잡는다<br>
  (World 좌표로 변환한 마우스 클릭점)<br>

- 이후 반환받은 dist 를 곱한 dir 만큼<br>
  해당 위치에서 '이동'시킨 좌표가<br>
  '충돌 지점'임을 짐작 할 수 있음!<br>

### 결과

[![Image](https://github.com/user-attachments/assets/e843aa6d-940d-4525-981d-8561aa4e3456)](https://github.com/user-attachments/assets/e843aa6d-940d-4525-981d-8561aa4e3456){: .image-popup}<br>

클릭한 '원'의 표면에 구체가 잘 생기는 모습이다!<br>