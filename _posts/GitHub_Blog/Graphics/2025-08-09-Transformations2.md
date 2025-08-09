---
title: "변환 2"
last_modified_at: "2025-08-09T15:30:00"
categories:
  - 수학
tags:
  - 선형 변환
  - 스케일 변환
---

## 선형 변환

<img width="575" height="679" alt="Image" src="https://github.com/user-attachments/assets/820a5fd5-c5d4-474d-bc8f-ffc0b2a2ed3d" /><br>

특정한 변환 T() 에 대하여<br>
다음 두 가지 성질을 만족하는 경우,<br>
T를 선형 변환이라 한다<br>

1. 덧셈 값 보존<br>
  : T(u + v) = T(u) + T(v)<br>

2. 스칼라 곱 보존<br>
  : T(cU) = c * T(U)<br>

-> 두 조건을 합쳐서도 가능<br>
   T(au + bv) = a * T(u) + b * T(v)<br>

'원점'을 보존한 상태로<br>
다양한 연산을 한 번에 진행하더라도<br>
문제가 없다고 보장 가능<br>

(회전은 선형 변환임)<br>

ex) 선형 보간인 예시<br>

<img width="634" height="596" alt="Image" src="https://github.com/user-attachments/assets/daac7dff-1544-4f9d-ba62-a27826bbaeea" /><br>


ex) 선형 보간이 아닌 예시<br>

<img width="636" height="631" alt="Image" src="https://github.com/user-attachments/assets/d57c25d4-7519-4811-8ff9-4a4035e742cd" /><br>

- 이동 변환은 선형 보간이 아님<br>

<img width="640" height="657" alt="Image" src="https://github.com/user-attachments/assets/76ca49e8-4125-47c9-9bad-0bd251487f78" /><br>

x 값에 영향을 주는 이동 변환에<br>
스칼라 곱을 적용한 경우<br>
두 값이 다르다!<br>

따라서 이동과 회전 변환을 '선형 변환'만 가지고<br>
같이 다룰순 없음<br>

### 선형 변환의 행렬 표현

<img width="700" height="701" alt="Image" src="https://github.com/user-attachments/assets/e98b7e78-03ed-472f-872e-979e463180c1" /><br>

- basis<br>
 : '좌표축'(Axis)과 같이 다른 벡터들을 표현할 때<br>
   사용되는 벡터를 표현<br>

- 선형 결합을 표현하는 방식이<br>
  scalar * 벡터 이므로<br>
  위와 같은 정리 가능<br>

## 스케일 변환

<img width="680" height="711" alt="Image" src="https://github.com/user-attachments/assets/757510d0-b4fc-4954-a9e3-ae5241e920ef" /><br>

좌표를 비율에 맞게 곱하거나 나누는 변환<br>
'선형 변환'이기에 회전 변환과 같이 사용 가능<br>

예시)<br>

<img width="736" height="689" alt="Image" src="https://github.com/user-attachments/assets/c50ab4a0-afcb-4ec7-8376-cf1a82b56918" /><br>

