---
title: "변환 5"
last_modified_at: "2025-08-12T16:30:00"
categories:
  - 수학
tags:
  - 변환의 구성
  - 좌표계 변환
---

## 기저 벡터의 변환

<img width="636" height="721" alt="Image" src="https://github.com/user-attachments/assets/cfb61585-79f7-4232-9f2a-fbd7a58be5be" /><br>

좌표계를 변환할 때,<br>
기존 벡터를 새로운 축을 기준으로 표현하는 방식<br>
(요점은 '축' 자체가 바뀐다는 점)<br>

- 기저 벡터는 좌표계의 '축(Axis)'를 표현하며<br>
  이러한 '축'의 표현을 변형함으로서<br>
  변환된 좌표를 얻을 수 있다<br>

각각의 축 자체가 이동했다고 볼 수 있음<br>
i,j,k들을 '변환'시킴(T(i) 등)<br>
+ b(이동)<br>

그림 3.8에서 사각형 내부의 점 p가<br>
사각형 자체의 '변환' 이후에도<br>
'사각형' 내부의 중점의 위치에 존재<br>
(각각의 좌표축인 i,j에 변형이 가해졌고<br>
 기존 좌표인 x,y값은 유지되어 만들어진 좌표값)<br>

<img width="680" height="698" alt="Image" src="https://github.com/user-attachments/assets/a4e3dd06-c3b0-45ad-995e-dd53c199f454" /><br>

박스를 담고 있는 좌표계 자체가<br>
특정한 벡터 b에 의하여 옮겨진 상황<br>
(Unreal 등의 게임 엔진에서 많이 보는 상황이다)<br>

## 변환의 구성

<img width="680" height="661" alt="Image" src="https://github.com/user-attachments/assets/434e4f4a-e4b5-4b3b-84af-050de5051e70" /><br>

Scale 변환 -> 회전 변환 -> 이동 변환<br>
의 순서대로 변환<br>

순서대로 변환을 하든<br>
변환들을 '모아' 이후 벡터에 곱해주든<br>
결과가 같음<br>

(변환의 순서가 중요하다)<br>
(a,b 변환의 결과가 달라지듯)<br>

- 반드시 s -> r -> t의 순서를 유지하란 뜻이 아님!<br>
  그저 특정 변환을 할 때, 그 순서를 유지하여<br>
  일관된 결과값을 얻으라는 뜻이다<br>

## 좌표계 변환

<img width="723" height="704" alt="Image" src="https://github.com/user-attachments/assets/def8e8d5-e8f8-4b9a-9c12-f00d010d7b68" /><br>

위의 기저 벡터 변환과 유사하지만 조금 관점이 다르다<br>
(그래도 거의 같은 맥락)<br>

- 요점은 '좌표계 변환'은 같은 벡터를 다른 좌표로 표현한다는 점<br>


| 구분          | 기저 벡터 변환                                             | 좌표계 변환                                |
| ----------- | ---------------------------------------------------- | ------------------------------------- |
| **핵심 개념**   | 축(기저 벡터) 자체가 바뀜                                      | 같은 벡터를 다른 좌표계 기준으로 재표현                |
| **변하는 것**   | 축의 방향/위치                                             | 좌표값(성분)                               |
| **안 변하는 것** | 좌표값(해당 기저 기준)                                        | 축(기저 벡터)                              |
| **게임 예시**   | 부모 Actor 회전 시 자식의 Local 좌표는 변하지 않지만, 월드에서의 위치/방향은 변함 | 월드 좌표를 로컬 좌표로 변환하거나, 로컬 좌표를 월드 좌표로 변환 |
| **그래픽스 예시** | 로컬 축(Forward/Right/Up)이 월드 기준에서 다른 방향을 가리키게 됨        | 월드 → 뷰 → 클립 → NDC → 스크린 좌표 변환         |


좌표의 변환은 여러모로 다양한 것을 내포하고 있다<br>
게임 엔진등에서 보이는 로컬 좌표계 등<br>
ex) 플레이어가 박스를 들고 10m 앞으로 가게 되면<br>
    박스의 world 좌표 역시 10m 앞으로 이동한다<br>
    (다만 플레이어의 기준에서 box의 좌표는 변함 없음)<br>
    (local 좌표)<br>

요점?<br>
위에서도 말하였듯<br>
기저 벡터 변환은 기존의 '축'이 변환되어 좌표가 새로이 표현되는 것이고<br>
좌표계 변환은 '새로운' 축을 기준으로 좌표를 재해석 하는 것이다<br>

### 벡터 사례

<img width="735" height="668" alt="Image" src="https://github.com/user-attachments/assets/a98a3b8f-e04d-4f16-9f29-9bfc7577ce2c" /><br>

pa(x,y) 가 pb(x',y')로 변하는 상황<br>
u,v는 a 기준의 좌표축<br>

그렇기에 a에서는 새로운 X,Y 축을 알면<br>
반대로 b에서는 기존의 u,v를 알면<br>
서로 좌표계 변환이 가능하다<br>
(pa <-> pb)<br>

### 포인트 사례

<img width="652" height="636" alt="Image" src="https://github.com/user-attachments/assets/4cd63e57-1d6e-458a-aa5a-250a6db18e55" /><br>

포인트는 움직이지 않는 위치이기에<br>
pa 와 pb는 결국 같은 위치이지만<br>
좌표계 A와 좌표계 B의 축에서 각각 표현한 요소이기에<br>
A의 원점 - B의 원점 의 벡터인 Q가 필요하다<br>
(위 공식은 정확히는 B 좌표계의 기준의 설명)<br>

### 예시

<img width="446" height="685" alt="Image" src="https://github.com/user-attachments/assets/98111256-cba6-499e-af79-5ef4ce0925be" /><br>

두 좌표계 사이의 거리가 위쪽으로 10이라면<br>
초록색 점 p를<br>
pa에선 (1,1)<br>
pb에선 (1,11)로 표현이 가능<br>

- 좌표계 변환은 하나의 축에서 '다른 좌표계'에 겹쳐놓는 방식으로<br>
  이해하여도 된다<br>
  그렇기에 b 좌표계를 위쪽으로 10 옮겨 pb를 계산한 것<br>

### 행렬로 표현

<img width="613" height="713" alt="Image" src="https://github.com/user-attachments/assets/ab8e1dab-fd13-4231-8ec4-bdcd9aa472b4" /><br>

좌표계 변환도 행렬로 표현이 가능<br>


### 좌표계 변환 행렬의 결합 법칙

<img width="627" height="668" alt="Image" src="https://github.com/user-attachments/assets/4d09539d-5e8f-44a0-9814-8733f5506b5b" /><br>

3개의 좌표계가 존재할 때<br>
결합 법칙을 통하여<br>
값을 구할 수도 있음<br>

### 좌표계 변환 행렬의 역행렬

<img width="694" height="712" alt="Image" src="https://github.com/user-attachments/assets/0248f512-baf7-4b64-b399-cb556398562d" />

좌표계 변환의 역행렬을 구하여 이전 좌표계를 구할수도 있다<br>
(아니면 '역변환'을 구하여 역행렬과 같은 효과를 내던가)<br>

2D 좌표계에서 클릭한 좌표가 <br>
3D 월드에서 어디즈음 해당하는지를 알고 싶은 경우에<br>
주로 사용<br>
(Screen 기준의 line trace 라든가)<br>
(흔한 fps 게임의 사격 위치 구하기)<br>

### 변환 행렬 vs 좌표계 변환 행렬

<img width="701" height="662" alt="Image" src="https://github.com/user-attachments/assets/46afe766-2af0-48a6-b657-e7f6ee3e98db" /><br>

모든 변환 행렬을 처음부터 계산하는 방식도 있으나<br>
각각의 좌표계를 통해 계산하는 방식도 존재<br>

전자의 예시는 World 좌표에서 애니메이션 중인 로봇의 손가락의 움직임을 보는 것이고<br>
(하나의 World 좌표를 통해 전체적인 좌표 변환)<br>
후자는 World -> 로봇 -> 로봇 팔 -> 로봇 손 -> 손가락 을 보는 방식<br>
(각각의 좌표계를 통해 변환)<br>

-> 기본적인 결과는 동일<br>

<img width="755" height="665" alt="Image" src="https://github.com/user-attachments/assets/469d7873-c129-4263-9479-8cfecac65dcb" /><br>

다만 상황에 따라서 골라 쓸 수 있다는 점을 알아두자<br>
(World 좌표로만 표현하는게 이득일 수도 있고<br>
각각의 좌표계를 가지고 있는것이 이득일 수도 있으니)<br>
