---
title: "벡터에 대한 추가 내용"
last_modified_at: "2025-07-24T16:30:00"
categories:
  - 수학
tags:
  - 벡터 내적
  - 벡터 외적
---

## 벡터
이미 이전에 3D 관련으로 정의한 적이 있으나 좀 더 심층적으로 알아보려 한다<br>

### 벡터의 길이와 단위벡터
벡터에는 '크기'와 '방향'이 존재한다<br>
(크기를 벡터의 '길이'라고도 표현한다)<br>

이때 크기가 1인 벡터를 '단위벡터'(unit vector)라 하고<br>
임의의 벡터를 '크기'가 1인 벡터로 만드는 것을 '정규화'(normalize)라 한다<br>

벡터의 길이는<br>
l = sqrt(v.x * v.x + v.y * v.y + v.z * v.z) 로 정의하며<br>
단위벡터는<br>
uV = (v.x / l, v.y / l, v.z / l) 로 정의된다<br>

- 단위벡터를 사용하는 이유?<br>
  : 벡터의 '방향'만이 필요한 경우를 위해 사용<br>
    (ex : 회전, 법선벡터의 표현)<br>
    단위벡터가 아니라도 계산 자체는 가능하나<br>
    불필요한 '크기'로 인하여 값이 커질 수 있음<br>
    (일반적으로 단위벡터에 필요한 만큼의 '크기'를 곱하여 사용한다)<br>
    ('적의 방향'을 단위벡터로 구하고, '사정거리'를 곱하여 공격 가능한지 판별한다던가)<br>

---

### 두 점으로 정의한 벡터
점 p 에서 점 q로부터의 벡터 v는 다음과 같이 표시<br>

<img width="611" height="399" alt="Image" src="https://github.com/user-attachments/assets/fdb859a4-9481-483e-94d5-ac42ac59d170" /><br>

---

### 벡터 연산
두 벡터 p 와 q, 그리고 상수 c 가 존재할 때,<br>
다음과 같은 벡터 연산을 정의할 수 있으<br>

1. 덧셈<br>
  p + q = (p.x + q.x,p.y + q.y,p.z + q.z)<br>
  (뺄셈은 사실상 덧셈과 유사)<br>

2. 스칼라 곱셈<br>
  c * p = (p.x * c,p.y * c,p.z * c)<br>
  
3. 내적(dot product)<br>
  p * q = p.x * q.x + p.y * q.y + p.z * q.z = len(p) * len(q) * cos(theta)<br>
  (이 때, theta는 두 벡터의 각도)<br>

4. 외적(cross product)<br>
  p x q = (p.y * q.z - p.z * q.y, p.z * q.x - p.x * q.z, p.x * q.y - p.y * q.x)<br>


### 벡터의 내적
내적은 '두 벡터'사이의 각도를 판별하는데 자주 쓰이는 중요한 값<br>
아래의 cos 그래프를 보면 알 수 있듯이<br>

cos(theta)는 0~90, 270~360 사이에서 양의 값을 갖는다<br>
이런 방식을 이용해 '카메라 방향' 좌표와<br>
물체의 면에 직교하는 '법선벡터'(normal)을<br>
내적하여 그 값이 양수인 경우에만 화면에 표시하는<br>
은면제거(backface-culling)에 사용 가능하다<br>
(이전에 언리얼 머티리얼에 나온 fresnel도 내적을 이용)<br>

<img width="1039" height="510" alt="Image" src="https://github.com/user-attachments/assets/07f23014-77ed-42d9-8973-ee0b50c9e2b4" /><br>

내적은 3D 그래픽 전반에 걸쳐 많이 사용되는데<br>
특히 좌표 변환과 '조명 계산'에 자주 사용된다<br>

조명을 계산할 때, 광원과 법선 벡터의 내적만으로도<br>
괜찮은 조명 효과를 줄 수 있기 때문<br>

<img width="680" height="408" alt="Image" src="https://github.com/user-attachments/assets/6e86f242-c21a-480f-bbe5-925396261896" /><br>

이미지는 간단한 조명모델인 '램버트 조명모델'<br>
(광원의 방향과 법선 벡터간의 각도 차이를 내적하여<br>
 해당 '면'에 빛이 얼마나 '잘 받을 수 있는지'를<br>
 간략히 계산할 수 있음)<br>

---

### 벡터의 외적
외적 연산의 결과 벡터는<br>
두 벡터와 '직교'하는 벡터가 나온다는 점도 기억해두자<br>
(직교 : 90도 만나는)<br>

<img width="871" height="601" alt="Image" src="https://github.com/user-attachments/assets/54356fbf-5ac8-4b61-baa9-f39dec4b58a5" /> <br>

3차원 공간에서 3개의 점이나 2개의 벡터에 의해 '면'이 생성되는데<br>
이 때, 2개의 벡터를 '외적'함으로서 해당 면의 '법선벡터'를<br>
구할 수 있음<br>
(다만 그림과 같이 연산 순서가 반대인 경우, 방향이 정반대가 된다)<br>

---

