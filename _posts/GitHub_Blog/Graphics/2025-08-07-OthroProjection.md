---
title: "벡터에 대한 추가 내용 2"
last_modified_at: "2025-08-07T18:30:00"
categories:
  - 수학
tags:
  - 내적
  - 직교 투영
  - 직교화
  - 외적
---

## 📚 목차

- [내적](#내적)
- [직교 투영](#직교-투영)
- [직교화](#직교화)
- [외적](#외적)

## 벡터 아마 마지막 정리?
이전에 내적과 외적을 다소 가볍게 다루었고<br>
하다 보니 Orthogonal Projection(직교 투영)에<br>
대하여 정리할 필요가 있어서 결국 또 포스팅을 남기려 한다<br>

## 내적

<img width="557" height="571" alt="Image" src="https://github.com/user-attachments/assets/23292dc0-ec1d-45ad-aea7-0348f296c2a2" /><br>

두 벡터의 요소를 각각 곱하여 더하는 방식<br>
-> 결과가 스칼라가 나오기에 스칼라곱이라고도 부른다<br>

각 벡터가 '단위 벡터'인 경우,<br>
Cos(theta)값만이 나오기에<br>
여기에 ArcCos를 적용하면 <br>
그 각도를 구할 수 있음<br>
(보통 radian 기준이기에 Radian to Degree 등의<br>
연산을 해야하긴 한다)<br>

게임으로 따지자면,<br>
dot의 결과가 > 0 이면 그 각도가 90 이하이므로<br>
'같은 방향'을 바라보는 판정 등으로 쓰기도 한다<br>
(완전히 같다면 1에 가까움)<br>

반대로 dot의 결과가 < 0 이면<br>
서로 '마주보고' 있을 가능성이 크므로<br>
AI 등에서 Player를 발견하였다는 걸로 처리가 가능하다<br>

아니면 그냥 ArcCos를 쓴 후<br>
그 값이 0 ~ 90 이거나 270 ~ 360 일 때<br>
dot > 0 과 비슷하다고 볼 수 있음<br>

- 내적은 이런식으로 '두 벡터'가 얼마나 같은 방향을 하고 있나<br>
  라는 '기하학적' 의미도 내포하고 있음<br>
  정확히 같은 방향일때, cos(theta) = 1로서 가장 큰 값이 되기에<br>

## 직교 투영
<img width="490" height="583" alt="Image" src="https://github.com/user-attachments/assets/8fdb4877-ef11-4fce-836d-4b7e7cfb236e" /><br>

### 의미

직교 투영(Orthogonal Projection)은<br>
하나의 벡터를 '다른 벡터' 위에 <br>
'수직으로 내린 그림자(투영)'을 구하는 것이다<br>

그 외의 기하학적 의미로는<br>
A 벡터는 B 벡터를 '얼마나 닮았는가' 혹은<br>
A 벡터는 B 벡터의 요소를 '얼마나 가지고 있는가'<br>
등의 의미로 해석한다<br>

### 수식 분석
벡터 v에서 벡터 X 방향으로 직교 투영할때<br>
그 결과를 p라 한다<br>
그렇다면 X의 단위벡터 n^에 대하여<br>
p = k * n^라 표현이 가능<br>
(p와 n^는 같은 벡터이며 그 길이만 차이가 존재)<br>
(그렇기에 길이를 k로 잡아 수식을 정의)<br>

이 때, k를 v와 n^의 내적으로 표현이 가능하다<br>
(벡터 v가 n^ 방향으로 얼마나 '닮았는지'에 대한 표현)<br>

따라서 k = dot(v,n^) * n^ 가 되며<br>
이를 풀면 k = |v| * cos(theta)<br>
(n^은 단위벡터이므로 길이 1)<br>
(v의 길이와 cos(Theta)값을 알면 구할 수 있음)<br>

최종적으로 Proj(v) = dot(v,n^) * n^가 된다<br>

또한 k가 음수여도 직교 투영이 가능<br>
(즉, theta가 90도 이상인 경우)<br>

### 응용
특정한 고차원 좌표를<br>
더 '낮은 차원'의 좌표로 변환(투영)한다던가<br>
단순화 시킬때 많이 이용된다<br>

ex) 그래픽 파이프라인에서<br>
3차원의 3D 공간을 2차원 Screen 좌표로 바꾸게 될때<br>
Z를 잘라내고 XY만 유지하게 될때도<br>
'투영'하게 된다<br>

그외에도 특정 벡터의 '요소'를 파악하여<br>
그 요소를 증폭하거나, 제거할 때도 사용 가능<br>

ex) 기울어진 바닥에서 '미끄러질 때'<br>
'움직임 벡터'에서 바닥의 'normal'으로 투영하고<br>
해당 방향에 속하는 '움직임 벡터'의 부분을 제거하면<br>
바닥에 '착' 붙어서 이동하는 움직임 벡터가 된다<br>

### 일반적으로 사용되는 수식 ver

```
Proj_B(A) = dot(A, normalize(B)) * normalize(B)
```

## 직교화
<img width="549" height="597" alt="Image" src="https://github.com/user-attachments/assets/d088899d-29ed-44ab-bc3e-477477795092" /><br>

### 의미
특정한 벡터 집합을 '서로 직교하는 벡터 집합'으로 변환하는 과정<br>
(대표적인 것이 Gram-Schmidt Orthogonalization)<br>

참고로 '변환'이란 위의 투영을 통하여<br>
'특정' 벡터를 '기준 벡터'의 값으로 투영시키고 제거한다<br>
(특정 벡터를 아래의 '직교 집합'으로 만드는 것)<br>

특정 벡터 A,B,C가 존재할 때<br>
A와 B, C를 각각 직교(90도, 내적 = 0) 시키게<br>
만드는 것<br>
(그냥 축 하나를 기준으로 x,y,z축으로 잡고<br>
나머지 벡터들을 y,z 축으로 만드는 거랑 비슷)<br>

ex) 2차원에서 보았을 때<br>
하나의 벡터 w0을 기준으로 잡고<br>
나머지 w1에서 w0의 '속성'(방향)을 제거<br>
(v1에서 w0로 '직교 투영'한 값을 빼주어<br>
w1을 구한다)<br>

- 하나의 축을 기준으로<br>
  나머지 벡터는 '그 축'과 '닮은 방향'을 제거해<br>
  새로운 '축'으로 쓴다고 생각하자<br>

### 직교 집합 (Orthogonal Set)
서로 직교하는 벡터들의 집합<br>
즉, 각각의 요소들을 아무거나 하나씩 잡고<br>
내적을 하더라도 그 값이 항상 0인 집합이다<br>

- 여기에 모든 벡터가 단위 벡터면<br>
  Orthonormal set (정규 직교 집합)이라 한다<br>

위의 예시로 결국 만들어진 {w0,w1}이<br>
직교 집합이다<br>

### 그람 슈미츠
<img width="502" height="560" alt="Image" src="https://github.com/user-attachments/assets/edbb16ff-cad0-40d6-9af6-2a5a53d72292" /><br>

직교화하는 하나의 방법<br>

하나를 '축'으로<br>
하나씩 직교화 시켜 나가는 방식<br>

![Image](https://github.com/user-attachments/assets/46cb04a1-ba69-45ec-9749-f3a5e1ea275d)<br>

[출처 : 위키피디아](http://ko.wikipedia.org/wiki/%EA%B7%B8%EB%9E%8C-%EC%8A%88%EB%AF%B8%ED%8A%B8_%EA%B3%BC%EC%A0%95)<br>

gif 예시를 통해 보자면<br>
u1을 기준으로 시작하여<br>
u2에서 u1의 요소를 제거<br>
이후 u3에서 u1과 u2의 요소를 제거하여<br>
직교 집합을 얻는다<br>
이후, normalize하여 정규 직교 집합을 만듦<br>

## 외적
<img width="560" height="543" alt="Image" src="https://github.com/user-attachments/assets/181e6066-919e-4d11-ac79-33b07608a718" /><br>

두 개의 3차원 벡터 u,v를 외적하면<br>
두 벡터에 '직교'하는<br>
새로운 벡터 w를 얻는 곱셈 방식<br>

u X v = w라 표현하며<br>
벡터를 결과로 얻기에 벡터곱 이라고도 부른다<br>
(u와 v는 같은 방향이거나<br>
완전히 다른 방향만 아니면 된다)<br>
(이 때는 하나의 축 자체가 전부 직교하기 때문)<br>

참고로 u X v = w 일 때<br>
v X u = -w 가 되므로 순서에 주의해야 한다<br>

normal 벡터를 구하거나<br>
3D 회전에서 회전축을 구할때 응용이 가능하다<br>

### 2차원 벡터의 외적
<img width="550" height="472" alt="Image" src="https://github.com/user-attachments/assets/d3a9d528-8719-4b18-89a4-1b44cc7688eb" /><br>

2차원 상의 벡터 u에 대하여<br>
'수직'인 벡터 v를 구하는 공식<br>

2D 상의 '반사' 등을 표현할 때 사용 가능<br>
(ccw/cw 등의 판별을 통해 앞/뒷면 즉,<br>
backspace culling 등에서 응용 가능)<br>

그 외에도 2D를 기준으로 90도 회전한 방향을 구할 때 유용<br>
(미니맵이라던가)<br>

### 그람 슈미츠와 외적

<img width="572" height="416" alt="Image" src="https://github.com/user-attachments/assets/950a015b-1ae7-49e5-ba50-7c5f4e2d1c6a" /><br>

외적을 사용하여 3차원 그람 슈미츠 공식을 표현한 것<br>
v0 = w0을 축으로<br>
w0 X v1을 외적하여 w2를 구한 뒤<br>
다시 w0 과 w2를 외적하여 w1을 간단히 구해낼 수 있다<br>

결과적으로 직교화된 w0,w1,w2를 얻는다<br>