---
title: "변환 4"
last_modified_at: "2025-08-10T16:30:00"
categories:
  - 수학
tags:
  - 애파인 변환
  - 이동 변환
  - 회전 변환
  - 스케일 변환
---

## 애파인 변환

<img width="551" height="544" alt="Image" src="https://github.com/user-attachments/assets/c21962d0-0ecc-4480-84dc-ca44a737c574" /><br>

애파인 변환이 바로<br>
선형 변환과 이동 변환을 '같이 쓸 수 있도록' 결합한 변환<br>

동차좌표를 사용하여 이동과 각각의 선형변환을 저장하여 한번에 처리<br>
(3차원 이동을 위하여 4차원의 동차좌표를 사용)<br>
(2차원은 3차원 동차좌표 사용)<br>

- 포인트는 마지막 요소가 1, 벡터는 마지막 요소가 0이기에<br>
  각 연산이 기하학적인 의미도 지니게 된다<br>
  (포인트(1) - 포인트(1) = 벡터(0))<br>

<img width="532" height="456" alt="Image" src="https://github.com/user-attachments/assets/6fb3d537-8019-4e2e-80c0-97be6fe17195" /><br>

애파인 변환 a(u)는<br>
선형 변환(u)에 이동 변환(b)를 더한 것과 같다고 표현 가능<br>

### 이동 변환

<img width="536" height="581" alt="Image" src="https://github.com/user-attachments/assets/c4880ad6-aca8-4645-b0e4-f719b626f4e3" /><br>

단위 행렬 T의<br>
마지막 번째 요소에 변화값을 넣어 사용한다<br>
(bx,by,bz,1 부분)<br>
(1은 point를 나타내지만)<br>

이전에 말하였듯<br>
'벡터'는 마지막 요소가 0 이기에<br>
'이동 변환'이 적용되지 않는다<br>

### 스케일링과 회전 행렬

<img width="530" height="518" alt="Image" src="https://github.com/user-attachments/assets/562e4203-3d20-452e-9507-74556a2d3e02" /><br>

스케일 변환은 단위 행렬 T의 요소에<br>
Scalar를 곱한 것과 같다<br>

회전 변환은 3x3 부분에 각 요소가 적용된다<br>