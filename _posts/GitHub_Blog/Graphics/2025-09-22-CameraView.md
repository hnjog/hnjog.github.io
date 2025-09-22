---
title: "Camera View"
date : "2025-09-21 12:00:00 +0900"
last_modified_at: "2025-09-21T12:00:00"
categories:
  - Direct X
  - Graphics
tags:
  - Camera View
---

## Chap 3 시작
기본적으로는 [HongLav](https://www.honglab.ai/enrollments)의<br>
강의를 듣기에 일부 내용만 블로깅 할때 첨부하며<br>
개인의 공부를 위해서 블로깅을 남기려 한다<br>

- Chap 3 를 수강한 이후부터는<br>
  Unreal 쪽의 그래픽 엔진 부분을 보며<br>
  이해해 나가는 과정을 지니면 좋을 것이라 하였다<br>

- 또한, 개인적인 엔진을 제작해 보는것이<br>
  그래픽스 공부에 큰 도움이 된다고도 하였다<br>

일단 Chap3 를 수강한 이후로는<br>
언리얼 쪽의 엔진 코드를 보며 추가적인 공부를 해볼 예정이다<br>

## Camera View 에 대하여
이전까지는 거의 '고정'된 시점을 가지는 View 행렬을 사용했으나<br>
일반적인 게임 등에서 '카메라'는 자연스럽게 움직일 수 있으므로<br>
이러한 View 행렬에 대한 '변화'가 필요하다<br>

그렇기에 Camera 클래스가 별도로 필요하게 되며<br>
동시에, 'Camera'라는 '관찰자'가<br>
View 및 Projection 행렬을 반환하는 역할도 맞게 된다<br>

### 예제 - View 행렬과 Mouse Update 구현하기

```cpp
Matrix Camera::GetViewRow() {
    // 렌더링에 사용할 View 행렬을 만들어주는 부분
    // 이번 예제에서는 upDir이 Y로 고정되었다고 가정합니다.
    // 시점 변환은 가상 세계가 통째로 반대로 움직이는 것과 동일
    // m_pitch가 고개를 숙이는 회전이라서 -가 두번 붙어서 생략

    // TODO:
    return Matrix::CreateTranslation(-m_position) *
           Matrix::CreateRotationY(-m_yaw) * Matrix::CreateRotationX(m_pitch);
}

Vector3 Camera::GetEyePos() { return m_position; }

void Camera::UpdateMouse(float mouseNdcX, float mouseNdcY) {
    // 얼마나 회전할지 계산
    // https://en.wikipedia.org/wiki/Aircraft_principal_axes
    m_yaw = mouseNdcX * DirectX::XM_2PI;     // 좌우 360도
    m_pitch = mouseNdcY * DirectX::XM_PIDIV2; // 위 아래 90도

    // 이동할 때 기준이 되는 정면/오른쪽 방향 계산

    //TODO:
    m_viewDir = Vector3::Transform(Vector3(0.0f, 0.0f, 1.0f),
                                   Matrix::CreateRotationY(m_yaw));
    m_rightDir = m_upDir.Cross(m_viewDir);
}
```

이번 예제 자체는 Vector3와 Matrix를 잘 사용하면 충분히 풀 수 있는 부분이었다<br>

- `CreateTranslation`을 통해 '이동' 행렬을,<br>
  `CreateRotation`을 통해 '회전' 행렬을 구할 수 있다<br>
  (이러한 행렬의 '적용'은 `곱`하여 적용한다는 점을 잊지 말자)<br>

- `Vector3::Transform` 을 통하여<br>
  회전 등의 `변환`을 '적용'할 수 있음<br>

- `ViewDir`은 '정면'을 담당하는 방향이므로<br>
  'Vector3(0.0,0.0,1.0)' 이 시작<br>
  이후, 좌우 360도 만이 적용되도록 한다<br>

- `m_rightDir`은 '정면'을 담당하는 ViewDir을<br>
  m_upDir에 외적(Cross) 시켜 구할 수 있다<br>
  (이때, m_viewDir.Cross 를 통해 구하면 leftDir이 되므로 주의<br>
  외적은 순서를 바꾸면 값이 정 반대가 나옴)<br>

- `Yaw Pitch Roll`<br>
  : 각각 y,x,z 를 기준으로 '회전'하는 역할을 가지는 축<br>
   (오일러 각도를 기반으로 하기에 나중에는 쿼터니언을 사용하는 편)<br>
   (항공 표준 이라는 말로도 사용된다)<br>

### 각 축의 기준에 대하여

| 용어        | 기준 축 (DirectX / 항공 표준) | 설명                             |
| --------- | ---------------------- | ------------------------------ |
| **Yaw**   | **Y축** (수직 축, Up)      | 좌·우로 도는 회전 (고개를 좌우로 돌리는 느낌)    |
| **Pitch** | **X축** (우측 축, Right)   | 상·하로 드는 회전 (고개를 위/아래로 끄덕이는 느낌) |
| **Roll**  | **Z축** (앞뒤 축, Forward) | 몸을 비트는 회전 (고개를 기울이는 느낌)        |

