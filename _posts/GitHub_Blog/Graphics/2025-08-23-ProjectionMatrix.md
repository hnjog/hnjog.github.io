---
title: "Projection Matrix"
date : "2025-08-23 16:30:00 +0900"
last_modified_at: "2025-08-23T16:30:00"
categories:
  - Direct X
tags:
  - Projection
---

## 원근 투영(Perspective Projection)

<img width="566" height="471" alt="Image" src="https://github.com/user-attachments/assets/ef20cd01-54cb-4906-ae2e-9cc578b4c82a" /><br>

원근 투영은 '절두체' 안의 물체를 '화면'에 나타나도록<br>
(위의 Viewing frustum)
렌더링을 하는 방식이다<br>

far/near clip plane - 너무 멀리/가까이 있는 물체를 판별하기 위한 거리<br>


- 투영은 이전에 다루었다싶이<br>
  특정한 벡터에서 '요소'를 제거하는 수학적 기법이다<br>

- 그렇기에 Orthogonal Projection(직교 투영) 에서는<br>
  요소들의 Z 요소만 제거하여 2D 화면의 결과로 만든다<br>

<img width="642" height="486" alt="Image" src="https://github.com/user-attachments/assets/99aa34c2-dce2-4111-b274-fc4fc0fadf7a" /><br>

- '수평 시야각 a'와 측면 비율(aspect ratio)이 주어졌을 때,<br>
  '수직 시야각 b' 도출이 가능<br>
  (측면 비율은 일반적으론 세로/가로 이므로 1보다 큰 편)<br>
  (일반적으로 a가 fov로 설정된 값이 들어옴)<br>
  b를 사용하는 편은 적지만 '구할 수 있다' 정도로 알아 두자<br>

- Projection Window : 화면에 그려질 렌더링 결과 위치<br>

- n부터 f 까지의 절두체 안의 요소만 그리게 된다<br>

- 삼각형의 시작(보통 view point)부터<br>
  Projection Window 까지의 거리 d는<br>
  tan 함수를 이용하여 구할 수 있다<br>
  d = 1 / tan(a/2)<br>

## 원근 투영 행렬

<img width="486" height="547" alt="Image" src="https://github.com/user-attachments/assets/e87bc859-f328-4389-89f9-f0d308d7086c" /><br>

- x,y,z,1 : 프로젝션 하려는 위치 값 포인트(w값이 1)<br>
           (구체적으로는 '투영'을 적용할 각각의 월드 좌표들)<br>

- Homogeneous Divide(동차 좌표 나눗셈)를 통하여 마지막 w 값으로<br>
  전체 x,y,z 좌표에 해당하는 값들을 나누어 줌<br>
  이러면 'z' 즉, 깊이가 깊어질 수록<br>
  x,y 좌표가 더 '작아짐'<br>
  (원근감 생성)<br>

<img width="574" height="536" alt="Image" src="https://github.com/user-attachments/assets/134358f6-e179-427d-9d16-1acdc493c92d" /><br>

해당 결과값은<br>
x,y : -1 ~ 1<br>
z : 0 ~ 1(위 z식을 보면 z가 f에 가까우면 1, n에 가까우면 0이 나온다)<br>
인 NDC로 변환된다<br>

- NDC로 변환되는 것이 반드시 보장되는가?<br>
  : 정확히는 'NDC 좌표'로 '변환되는 녀석'들만 그려줌<br>
  (절두체는 그냥 시각적 이해를 돕기 위한 것!)<br>
  (행렬 연산 결과가 NDC 범위 내에 못들어오면 Culling이 된다)<br>

- 즉, VS의 결과인 Clip 공간이 NDC로 변환되는 그 과정에서<br>
  (이건 RS 이전)<br>
  GPU가<br>
  Cliping 을 체크(물체들의 x,y,z 범위 확인) 후<br>
  Homogeneous Divide를 실행하고<br>
  그 범위가 NDC 좌표에 들어가는 녀석들을 모아 RS에게 넘겨줌<br>

실제 코드 설정 부분<br>

```
// 뷰포트 설정
m_screenViewport.MinDepth = 0.0f;
m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering

// rasterizer State
...
rastDesc.DepthClipEnable = true; // <- zNear, zFar 확인에 필요
```

DepthClipEnable 가 설정되면<br>
Normalize Depth 값이<br>
뷰포트의 MinDepth ~ MaxDepth 범위를 벗어나면<br>
Culling 조건에 부합하여 빼버리게 한다<br>

- Triangle Clipping<br> 

<img width="584" height="557" alt="Image" src="https://github.com/user-attachments/assets/f04ca27d-fddf-495b-be61-e2b2c752d305" /><br>

GPU는 NDC 좌표를 벗어나는 삼각형들을 판단한다<br>

3번 케이스는 삼각형을 잘라내었더니 4각형이 되었기에<br> 
다시 쪼개 삼각형으로 만들어준다<br> 
(4번도 마찬가지로 쪼개서 3각형 3개로 만든다)<br> 

<img width="472" height="407" alt="Image" src="https://github.com/user-attachments/assets/007815d7-3b8b-4f1b-8cbe-8cb5f7cc07cd" /><br>

- 삼각형이 Near Plane을 뚫고 들어온다면?<br>
  그 부분 역시 '잘라'서 남은 부분을 사용한다<br>