---
title: "변환"
last_modified_at: "2025-08-08T15:30:00"
categories:
  - 수학
tags:
  - 회전 변환
  - 이동 변환
---

## 변환
좌표나 벡터를 다른 좌표나 벡터로 바꾸는 것<br>

보통 위치, 방향, 크기 등을 바꾸는 연산을 총칭하며<br>
대개 행렬을 통해 표현<br>

회전, 이동, 반사 등등의 변환 뿐 아니라<br>
선형 변환, 아핀 변환 등 매우 많고 다양한 변환이 존재<br>

### 회전 변환
<img width="667" height="663" alt="Image" src="https://github.com/user-attachments/assets/163cbc51-1143-4122-ab8d-3214bff69343" /><br>

x,y 좌표에 대하여<br>
Theta 각도 만큼 회전시킨다는 것을<br>
특정한 '행렬'로 표현<br>

#### 분석

- 기본적으로 '길이'는 보존됨<br>

- 원점을 기준으로 반지름 1인 원을 그리고<br>
  그 원 위의 좌표는<br>
  정점에서 가리키는 각도 Theta에 의하여<br>
  (cos(theta),sin(theta))<br>
  (반지름이 1이므로 삼각함수를 이용해 x,y좌표를 구할 수 있음)<br>
  이를 각각 x,y로 표현<br>

- 이 때, 새로운 각도 Theta2를 통해<br>
  점을 회전시키게 되면<br>
  새로운 각도인<br>
  cos(theta + theta2), sin(theta + theta2)<br>
  이를 각각 x',y'로 표현<br>

- 이것을 삼각함수의 덧셈 정리를 통해 분리하게 되면<br>
  cos(theta + theta2) = cos(theta) * cos(theta2) - sin(theta) * sin(theta2)<br>
  sin(theta + theta2) = sin(theta) * cos(theta2) + cos(theta) * sin(theta2)<br>
  가 된다<br>

- 따라서 새로운 좌표는<br>
  x' = x * cos(theta2) - y * sin(Theta2)<br>
  y' = x * cos(theta2) + y * sin(theta2)<br>
  가 되며<br>
  이를 행렬 형태로 바꾸어 표현하면 위의 변환식이 된다<br>

#### 전치 행렬을 통해 row Vector로 바꾼 경우
<img width="674" height="565" alt="Image" src="https://github.com/user-attachments/assets/9039221b-9b07-492c-a143-5f3fad1a8218" /><br>

양 측면에 Transpose를 사용하여<br>
위처럼 변형할 수 있다<br>
(순서가 바뀜에 주의)<br>

(사용하는 API가 row , Column에 따라<br>
자유롭게 바꾸어 구현할 수 있음을 기억해두자)<br>

### 이동 변환

<img width="898" height="1032" alt="Image" src="https://github.com/user-attachments/assets/dec5c678-d4bf-4d59-abfd-7e81c66843dd" /><br>

2x2 행렬로는 이동을 표현할 수 없기에<br>
'동촤 좌표'를 사용하여 3x3 행렬로 확장하여 진행한다<br>

이동 수치값 bx,by 를<br>
행렬에 넣어 연산하기 위해 확장<br>

### 동차 좌표(Homogeneous Coordinates)
<img width="703" height="720" alt="Image" src="https://github.com/user-attachments/assets/77384f0f-eace-4957-811d-19cb456592c5" /><br>

좌표의 차원을 하나 늘려서 표현하는 방식<br>

상응하는 좌표를 확장한 좌표의 값으로 나누어 계산<br>
ex : 동차좌표 : (x,y,w) -> 실제 좌표 : (x/w,y/w) <br>

그래픽스에선 w에 1을 사용하여 단순하게 변환이 가능하다<br>

- w에 1을 넣는 의미?<br>
  : 1을 넣어 '위치'(포인트) 임을 표현한다<br>
  반대로 0인 경우는 '방향'(벡터)로 표현<br>
  => 포인트는 '이동 변환'을 받을 수 있지만<br>
     벡터는 '이동 변환'을 받을 수 없도록<br>
     (해당 위치에 0을 넣으면 x' = x, y' = y가 되므로)<br>


<img width="686" height="664" alt="Image" src="https://github.com/user-attachments/assets/a42722ac-b274-47ff-91ff-56d2fe252660" /><br>


이동을 표현하기 위하여 사용하는 것 또한 존재하나<br>
다른 변환인 '회전','스케일' 등을 같이 묶어 처리할 수 있기에<br>
연산의 효율을 올릴 수 있음<br>