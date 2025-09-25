---
title: "쿼터니언과 회전 (이론)"
date : "2025-09-25 12:00:00 +0900"
last_modified_at: "2025-09-25T12:00:00"
categories:
  - 그래픽스
  - 수학
tags:
  - Quaternions
  - 쿼터니언
  - 사원수
  - 회전
---

## 3차원 공간에서의 회전

[![Image](https://github.com/user-attachments/assets/d63b004e-c95b-497a-9047-6c36765e5cf3)](https://github.com/user-attachments/assets/d63b004e-c95b-497a-9047-6c36765e5cf3){: .image-popup}<br>

- 2차원 공간에서의 회전은 1 자유도(DOF)<br>
  (시계 <-> 반시계 회전이며, 서로 반대됨)<br>

- 3차원 공간에서 회전은 3 자유도<br>
  YawPitchRoll<br>
  이런식으로 3차원 공간에서의 물체의 회전 상태를 표현하는 것을<br>
  '오일러 각도' 라 표현한다<br>
  
### 오일러 각도(Euler Angles)

[![Image](https://github.com/user-attachments/assets/2d25a4a2-3e19-458a-8921-dfa2bb1a0351)](https://github.com/user-attachments/assets/2d25a4a2-3e19-458a-8921-dfa2bb1a0351){: .image-popup}<br>

각각의 '축'에 대한 회전 행렬을<br>
'곱'하여 회전 행렬을 만든다<br>
(그렇기에 곱하는 순서가 정해져 있어<br>
축에 종속적이기도 하다)<br>

(이건 Column Major 이기에 오른쪽에서 왼쪽으로 적용됨<br>
Z -> Y -> X 순으로 회전 적용)<br>
(Roll -> Yaw -> Pitch)<br>


- 또한 '축' 자체가 다른 회전에 영향을 받음<br>

[![Image](https://github.com/user-attachments/assets/53beb52c-c16b-4141-9c9b-b8b749db7629)](https://github.com/user-attachments/assets/53beb52c-c16b-4141-9c9b-b8b749db7629){: .image-popup}<br>

이미 기울어진 Pitch에 대하여 다시 Roll의 회전까지 적용하기도 함<br>

자이로스코프를 통해 보면 이해가 비교적 쉬운편<br>
바깥의 원의 이동에 안쪽의 원이 영향을 받음<br>

그렇기에 아래와 같은 짐벌락 현상이 발생할 수 있음<br>

[![Image](https://github.com/user-attachments/assets/51bf783e-5f02-4900-8adf-b0c4e6336be3)](https://github.com/user-attachments/assets/51bf783e-5f02-4900-8adf-b0c4e6336be3){: .image-popup}<br>

왼쪽의 원에서 초록색 원이 90도 회전하며<br>
파란색 원과 합쳐진 상태이며<br>

이후, 파란색 원과 초록색 원을 '분리할 수 없는' 현상이 발생<br>
(두 축이 겹쳐버린 현상)<br>

[![Image](https://github.com/user-attachments/assets/46c9640b-8f5d-426b-9668-753b17c0de6b)](https://github.com/user-attachments/assets/46c9640b-8f5d-426b-9668-753b17c0de6b){: .image-popup}<br>

수식으로 보자면<br>
결과적으로 2개의 각도만이 회전 결과에 반영됨을 볼 수 있다<br>
(마지막 줄)<br>

- UpVector와 ViewVector(현재 정면 관찰)가 각각 존재할때<br>
  RightVector는 Cross로 구할 수 있다<br>

- 그런데 ViewVector를 회전시켜 하늘을 바라보게 하였을때<br>
  UpVector와 동일해지므로<br>
  이 때는 Cross를 통해 RightVector를 정의할 수 없음<br>
  (다소 극단적인 예시이긴 하다)<br>



그렇기에 오일러 벡터 외에도 회전을 표현할만한 방식이 필요해짐<br>

- 그래도 오일러 벡터는 '직관적'이며 이해하기 쉽다는 장점이 있어<br>
  에디터, Tool, UI 등으로 회전을 표현하는 방식으로<br>
  여전히 사용된다<br>

- 또한 '입력'에 대한 수치로도 사용됨<br>
  (Right 로 90도 회전 등)<br>

## 쿼터니언 (Quaternion, 사원수)의 개념

[![Image](https://github.com/user-attachments/assets/ab548e5b-a2a8-4b66-b3bf-3a240503528e)](https://github.com/user-attachments/assets/ab548e5b-a2a8-4b66-b3bf-3a240503528e){: .image-popup}<br>

3차원 공간의 회전을 위하여 1가지 숫자를 추가한<br>
4개의 숫자로 회전을 표현하는 방식<br> 
(숫자가 4개이기에 4원수)<br>

- 숫자가 4개라도 3자유도<br>
  (마지막 숫자인 w에 대한 제약 조건이 존재하기에)<br>

- 회전 행렬과 호환<br>
  : 회전을 쿼터니언으로 진행 후, 회전 행렬로 변환할 수 있음<br>

- 회전의 보간(Interpolation)에 유리함<br>


## 쿼터니언의 사용 방식

[![Image](https://github.com/user-attachments/assets/135745dd-3abf-4447-83ed-d3784586a8bc)](https://github.com/user-attachments/assets/135745dd-3abf-4447-83ed-d3784586a8bc){: .image-popup}<br>

벡터 V를 회전축 N에 대하여 T만큼 회전시키고 싶을때<br>

- 1. 쿼터니언 p = (v,0)<br>
- 2. 쿼터니언 q = (n sin(t/2),cos(t/2))<br>
- 3. 켤레 쿼터니언 q* = (-n sin(t/2),cos(t/2))<br>
- 4. 쿼터니언 p' = (v',0) = qpq* 로 회전<br>

- 켤레?<br>
  : 어떤 수에서 '특정 성분'의 부호를 바꿔 '짝'을 이루는 수<br>
  (ex : a + bi -> a - bi)<br>

-> 직접 구하기 보단 이미 구현된 라이브러리를 사용하는 편<br>

[![Image](https://github.com/user-attachments/assets/0df5d5b6-017c-4f3c-a598-7d736d3d56e2)](https://github.com/user-attachments/assets/0df5d5b6-017c-4f3c-a598-7d736d3d56e2){: .image-popup}<br>

또는 쿼터니언 q로부터 회전 행렬을 만들어 곱함<br>
회전 행렬을 쿼터니언으로 만들어서 사용할 수도 있음<br>

(메모리를 아끼기 위해 행렬이 아닌 쿼터니언으로 저장하는 방식도 존재)<br>

## 쿼터니언의 구성

[![Image](https://github.com/user-attachments/assets/751c3885-732f-46f4-af09-f5d8254e1c6b)](https://github.com/user-attachments/assets/751c3885-732f-46f4-af09-f5d8254e1c6b){: .image-popup}<br>

4개의 숫자 `x,y,z,w` 로 이루어짐<br>
(x,y,z 를 u로 표현하여 (u,v)로 표현하기도 함)<br>

여기서 i,j,k는 허수<br>
(허수는 제곱하여 -1 이 되는 실제로는 없는 수)<br>

- i^2 = ijk = -1<br>
- ij = k = -ji<br>
- jk = i = -kj<br>
- ki = j = -ik<br>

### 그래서 이런 허수와 회전이 무슨 관계?

[![Image](https://github.com/user-attachments/assets/e34962b5-bf23-49b7-8368-fc4fd8f14647)](https://github.com/user-attachments/assets/e34962b5-bf23-49b7-8368-fc4fd8f14647){: .image-popup}<br>

허수는 곱할때마다<br>
`i -> -1 -> -i -> 1` 로 순환하는 특성을 가짐<br>

실수와 허수의 좌표를 가지는 좌표계에서<br>
하나의 '회전'으로 볼 수 있음<br>

[![Image](https://github.com/user-attachments/assets/cbe4fcf6-4e37-471a-aa9a-0c17468c6605)](https://github.com/user-attachments/assets/cbe4fcf6-4e37-471a-aa9a-0c17468c6605){: .image-popup}<br>

특정한 좌표 p가<br>
실수 부분 2, 허수 부분 i 정도의 값을 가진다면<br>
p를 p = 2 + i 로 표현이 가능<br>
(이런식으로 `허수 + 실수`의 표현을 '**복소수**' 라고 함)
이때, i를 곱하게 되면<br>
그래프에서 90도 정도 회전하게 된<br>
q와 동일해짐!<br>
(q = 2i - 1)<br>

- i를 계속 곱하면 같은 원리로 p -> q -> r -> s -> p가 됨<br>
  허수를 곱해줌으로서 '복소수'가 회전하는 듯한 효과를 얻을 수 있음<br>


[![Image](https://github.com/user-attachments/assets/f80b7cee-4faa-41a9-9561-5fd0928e60cc)](https://github.com/user-attachments/assets/f80b7cee-4faa-41a9-9561-5fd0928e60cc){: .image-popup}<br>

그렇기에 쿼터니언은<br>
3개의 허수를 이용하여 회전을 표현한다<br>

각각의 허수 좌표 및 연산을 통하여<br>
쿼터니언 만의 독자적인 회전 표현이 가능해짐<br>

- i에 j를 곱하면 k, j에서 i를 곱하면 -k 등<br>

## 쿼터니언의 기본연산 방식

[![Image](https://github.com/user-attachments/assets/d64a369d-9673-4e7f-ba8c-6e62827a8e43)](https://github.com/user-attachments/assets/d64a369d-9673-4e7f-ba8c-6e62827a8e43){: .image-popup}<br>

- 4개의 수가 모두 같으면 같은 쿼터니언 판정<br>

- 덧셈, 뺄셈은 각각의 요소에서 진행<br>

[![Image](https://github.com/user-attachments/assets/bdb8ddd7-88be-45ee-8a7c-5ed075ec5f5f)](https://github.com/user-attachments/assets/bdb8ddd7-88be-45ee-8a7c-5ed075ec5f5f){: .image-popup}<br>

식이 많아보이지만 기본적으로는 항을 정렬하게 되면<br>
가장 아래쪽의 식으로 변환된다<br>
(다항식의 전개)<br>
('복소수'의 곱셈만 주의하면 충분히 전개할 수 있는 식)<br>

- dot과 Cross를 사용하여 더 깔끔한 요약이 가능<br>


[![Image](https://github.com/user-attachments/assets/5fde1f80-2a04-4d83-8d4f-01672072c6c7)](https://github.com/user-attachments/assets/5fde1f80-2a04-4d83-8d4f-01672072c6c7){: .image-popup}<br>

쿼터니언의 절댓값은<br>
4개의 요소를 제곱한후 더하고, 루트를 씌우면 된다<br>


[![Image](https://github.com/user-attachments/assets/be53801a-3575-424a-b745-b336ae430d5b)](https://github.com/user-attachments/assets/be53801a-3575-424a-b745-b336ae430d5b){: .image-popup}<br>

쿼터니언의 역수를 구하는 방식<br>

먼저 켤레(Conjugate)를 구하고<br>
(q = (u,w)라면 q* = (-u,w))<br>

이후<br>

qq* = |q| ^ 2 = |q*| ^ 2 이므로<br>
q^-1 = q* / (|q|^2) 로 정의<br>

|q| = 1 이라면<br>
q^-1 = q*<br>

- 역수?<br>
  : 곱했을때 1로 만드는 수<br>
  (일반적인 숫자는 1 / a 등으로 역수를 만들 수 있다)<br>

### 쿼터니언의 회전

[![Image](https://github.com/user-attachments/assets/9428b6a2-0a89-4895-9726-1586b1495045)](https://github.com/user-attachments/assets/9428b6a2-0a89-4895-9726-1586b1495045){: .image-popup}<br>

앞에서 본 것의 식을 푸는 과정<br>

- 이러한 식을 외우기 보다는 '쿼터니언'의 개념을 이해하고<br>
  일반적인 오일러 각도와 차이점이 있다는 점을 느낀다면 일단 충분!<br>
  (엔진 등을 구현하려면 이러한 부분을 구현해야 할지도?)<br>
  (Math 라이브러리 등으로 엔진의 일부분을 대체하는건가?)<br>
