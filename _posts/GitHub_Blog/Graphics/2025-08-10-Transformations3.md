---
title: "변환 3"
last_modified_at: "2025-08-10T15:30:00"
categories:
  - 수학
tags:
  - 회전 변환
---

## 다시 보는 회전 변환
회전 변환 역시 '선형 변환'이며<br>
그렇기에 '행렬'로 표현이 가능<br>

먼저 3차원 공간에서의 '회전'은<br>
특정한 '축'을 기준으로 이루어진다<br>

그리고 그 각도만큼<br>
회전할지를 결정<br>

'축'과 '각도'를 먼저 생각해야 함!<br>

각각의 '회전'들은 모두<br>
'선형 변환'이기에 x,y,z 축에 대한 다양한 회전을<br>
하나의 '회전 행렬'로 압축하여 표현이 가능<br>
(연산 절약)<br>

처음 공부 시작할 때는 각각의 x,y,z 를 기준으로<br>
2차원 회전을 적용하여 반복함으로서 공부<br>
'오일러 회전(Euler Rotation)'이라 함<br>

- 오일러 회전?<br>
  : 3D 회전을 각각의 x,y,z 를 기반으로<br>
   2차원 회전을 시켜 그 각도를 구하는 방식<br>
   회전 순서가 중요하다<br>
   직관적이며, 쉽게 이해가 가능하나<br>
   '짐벌락'이 발생할 수 있기에 자유로운 회전이 어려움<br>
  
- 짐벌락(Gimbal Lock)?<br>
  : 오일러 회전 방식은 x,y,z 축을 회전시키는 방식이나<br>
    특정한 회전 각도에서 '두 축'이 겹쳐버리면<br>
    두 개의 축이 같이 회전해버리는 현상<br>
    (두 축이 겹쳐져 하나의 '평면'이 되어버리며<br>
    이를 구분할 수 없어 lock이라 부름)<br>
    사원수 방식이나 회전 각도 제한 등을 이용하여<br>
    문제를 예방하거나 방지하는 편이다<br>

<img width="687" height="431" alt="Image" src="https://github.com/user-attachments/assets/4dcffcd8-1723-4ca4-a38a-08bc5f7bef5a" /><br>
[출처](https://science.howstuffworks.com/gimbal1.htm%5D)<br>

나중에 가면 사원수 (Quaternion)을 이용하여<br>
3차원 회전을 진행<br>

- 2차원 회전은 '회전 축'을 위하여<br>
  결국 3차원의 개념을 도입할 수 밖에 없음<br>
  따라서 3차원 회전 역시 '회전 축'을 위해<br>
  4차원의 개념인 '사원수'를 도입<br>


### 3차원 회전의 유도 과정

<img width="560" height="548" alt="Image" src="https://github.com/user-attachments/assets/90de71d7-4f9a-4d88-ab7d-c978edacc4b9" /><br>

일단 회전 축은 벡터 n(단위 벡터)이며<br>
회전 각도는 theta 이다<br>

벡터 v를 theta 만큼 회전시켜<br>
Rn(v)를 구한다 가정<br>

v를 n에 직교투영 하여 새로운 지점(point)을 구할 수 있음<br>

proj n(v) : 벡터 n에 v를 직교 투영하여 얻은 결과<br>
이것을 n에 곱해주면<br>
v를 n에 대입한 위치를 구할 수 있음<br>
즉, 사진 상에서 보는 원의 '중점' 위치<br>

(말이 어려운데, 그냥 v에서 n축만큼의 요소들을<br>
추출한 결과라 생각하자)<br>

이후, v에 그 투영한 결과(원의 중점)를 빼줌으로서<br>
(a - b : b에서 a를 향해 바라보는 방향 벡터)<br>
중점 -> v 를 향하는 새로운 벡터(vt)를 구할 수 있다<br>
(t를 뒤집은 기호는 perpendicular 라 함)<br>

이후 v와 n을 외적하여<br>
n X v 벡터를 구한다<br>
(두 벡터에 '직교'하는 새로운 벡터)<br>

n x v 벡터는 vt와도 수직이며<br>
(vt는 v,n과 같은 평면 위에 존재한다)<br>
두 벡터의 길이는 동일하다<br>

- 외적의 길이?<br>
  : 외적의 길이 |v X n|은<br>
   |v| 와 |n| 그리고 두 각을 이루는 sin(a)의 곱과 같다<br>
  (이 때, n은 단위벡터라 길이가 1)<br>

- |vt| 역시, |v| * sin(a)이다<br>
  (삼각함수를 통해서 구할 수 있음)<br>
  (v와 n이 이루는 각도가 a이므로<br>
  sin(a) = |vt| / |v| 이다<br>
  따라서 |vt| = |v| * sin(a))<br>

=> 결과적으로 두 벡터의 길이가 같다<br>
두 벡터는 같은 원 위에 존재한다 말할 수 있음<br>

n x v 와 vt를 기준으로 2차원으로 보게 되는 경우<br>
우측의 2차원 원으로 볼 수 있음<br>

Rn(vt)는 theta 각도 만큼 회전시킨 vt이다<br>

이 때, Rn(vt)와 vt, v x n 의 길이는 모두 같음<br>

따라서, Rn(vt) = cos(theta) * vt + sin(theta) * n x v 가 성립<br>

1. 일단 cos(theta) * vt에 sin(theta) * n x v를 더하면<br>
   기하학적 의미의 '벡터의 덧셈'의 의미에서 해당 공식이 성립<br>

2. cos(theta) * vt는 rn(vt)에서 vt 방향으로 직교 투영 하여<br>
   구할 수 있는 값이며, 그 값은 cos(theta)를 곱하여<br>
   현재 vt의 위치에 0~1 사이의 값을 곱하여 얻는 위치<br>

3. sin(theta) * n x v 도 비슷하다<br>
   sin(theta) = cos(90 - theta)이고<br>
   rn(vt)가 n x v로 직교 투영할 때의 각도가 90 - theta<br>

그림으로 요약하자면,<br>

<img width="384" height="413" alt="Image" src="https://github.com/user-attachments/assets/b6cfc067-bd7d-4adc-83e2-6ad27525dd76" /><br>

최종적으로 Rn(v) = proj n(v) + Rn(vt)가 된다<br>
해당 수식을 풀면 다음과 닽다<br>

<img width="520" height="491" alt="Image" src="https://github.com/user-attachments/assets/f379a1f9-eb9e-4f39-b211-c06cd76d421a" /><br>
<img width="599" height="461" alt="Image" src="https://github.com/user-attachments/assets/166ec57d-b25c-4161-b582-3a3931b98fd3" /><br>

[로드리게스 회전 공식](https://en.wikipedia.org/wiki/Rodrigues%27_rotation_formula)<br>

### 회전 행렬의 역행렬

<img width="588" height="512" alt="Image" src="https://github.com/user-attachments/assets/43cabe1f-6f20-40b3-b9d7-6990aa80a27c" /><br>

회전 행렬은 역행렬을 직접 구하는 것 말고도<br>
전치 행렬을 곱해줌으로서 회전을 역변환 시켜<br>
원래 행렬로 되돌릴 수 있음<br>
(회전 행렬에 한해 '역행렬 = 전치 행렬'이 동일)<br>