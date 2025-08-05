---
title: "Post Process Material"
last_modified_at: "2025-08-05T16:00:00"
categories:
  - Unreal
tags:
  - Post Process
  - Material
---

## Post Process
'렌더링'이 끝난 후, 화면 전체에 적용되는 '후처리'효과로<br>
화면 단위의 '픽셀' 마다 개별적인 연산을 수행할 수도 있다<br>
(이 부분은 DirectX의 'Pixel Shader'와 유사)<br>

오늘은 화면 전체에 적용되는 머티리얼인<br>
Post Process에 Material을 적용한 방식이지만<br>
실제로는 더 다양한 효과들을 지니고 있어 다양한 연출이 가능하다<br>

| 범주             | 주요 기능                             | 설명             |
| -------------- | --------------------------------- | -------------- |
| **렌더링 후처리 효과** | Bloom, Exposure, DOF 등            | 렌더 결과를 다듬는 효과  |
| **화면 컬러/톤 조정** | Color Grading, LUT                | 전체적인 색조/분위기 조정 |
| **스타일 변경**     | Vignette, Grain, Filmic Tone      | 감성/영화 느낌 연출    |
| **특수 효과**      | Motion Blur, Chromatic Aberration | 속도감, 광학 효과     |
| **머티리얼 적용**    | Post Process Material             | 사용자 정의 효과 적용   |

## Post Process Material 적용하기

내가 원하는 효과는<br>
캐릭터(카메라)를 기준으로 일정 영역을 기준으로<br>
효과를 주는 것이다<br>

마치 '캐릭터'만이 SpotLight를 받는?<br>
그렇기에 괜찮은 효과를 검색해보고 직접 PostProcess에 적용해보았다<br>
[참고 효과](https://www.youtube.com/watch?v=9_39Wyo-uho)<br>

<img width="1873" height="1421" alt="Image" src="https://github.com/user-attachments/assets/0e76c5d4-0b4a-4b65-820e-7bb7dd3f29c0" /><br>

해당 효과는 Depth를 기반으로 하는 Outline 구현 효과이다<br>
(Depth-Based Outline 기법)<br>
<br>
화면 공간(Screen Space)에서 '깊이' 값을 사용해<br>
주변 '픽셀'과의 깊이 차이를 비교해 '경계선'을 감지하는 방식으로<br>
'평균 픽셀 깊이'를 Blur를 통해 구하고<br>
그 '깊이의 차이'가 큰 부분이 '테두리'라 판단<br>
<br>

- ScreenPosition : 화면상에서 현재 픽셀의 위치를 표시<br>
    (UV와 Pixel Position)<br>
- SceneTexture : 렌더링의 중간 결과들을 텍스쳐처럼 가져와 사용하는 노드<br>
  - SceneColor : 최종 화면 색상<br>
  - SceneDepth : 깊이 값 (0 : 가까움 ~ 1 : 먼 곳)<br>
  - WorldNormal : 월드의 노멀값 (RGB = XYZ)<br>
  - AmbientOcclusion : SSAO 값<br>
  - CustomDepth / Stencil : 특정한 오브젝트 마스크<br>

- SceneTexelSize : 현재 Viewport의 텍셀 크기 정보 (1 / 해상도)<br>

- Saturate : 0~1 으로 값을 클램핑(제한) 해줌<br>

<img width="2257" height="1159" alt="Image" src="https://github.com/user-attachments/assets/fe399c1d-ba6f-471d-85d1-7975f4ce4cb0" /><br>

해당 결과로만 포스트 프로세스를 적용한 경우<br>
다음과 같이 테두리를 관측할 수 있다<br>
(현재는 색상 값을 포함 x)<br>

## Process 2 특정 범위에만 적용하기
이제 PostProcessInput0 와 함께 적용하여<br>
색을 특정 부분에 입히려 한다<br>

- SceneTexture:PostProcess<br>
  : 현재 포스트 프로세스가 적용되기 전의 '렌더 타겟'(Scene Color)<br>
    따라서 이 부분과 '적절하게' lerp 하여 적용하면<br>
    테두리만 적용되지 않을까?<br>

<img width="2251" height="670" alt="Image" src="https://github.com/user-attachments/assets/0c5f00f3-d544-4fc7-9bf9-c3c69dc8abd7" /><br>

ScenePosition에서 각각 0.5를 뺀 후<br>
자기 자신을 내적, 이후에 1-x 를 통해<br>
(0과 1의 값은 작게, 0.5에 가까운 값은 보다 크게)<br>
값을 조정한 후<br>
1000을 곱하고 SceneDepth에 해당 값을 나누어 주었다<br>
(SceneDepth가 0~1 인데 무척 큰 값으로 나눔)<br>
이후 saturate 및 1-x를 통해<br>

그나마 '화면'에 가까운 요소들을<br>
PostProcess에 가까운 효과를 가지도록 적용<br>

<img width="2247" height="1157" alt="Image" src="https://github.com/user-attachments/assets/175918f1-6075-43b8-b045-80e8a705deb7" /><br>

결과물...이지만<br>
뭔가 복잡하고, 모양이 타원형이라<br>
맘에 들지 않는다<br>

## Process 3 화면 중앙을 기준으로 해보자

일단 내가 생각한대로 구현이 되지 않았기에<br>
뭔가 '비율'이 잘못되었는지 확인해 보려 하였다<br>

<img width="2365" height="1080" alt="Image" src="https://github.com/user-attachments/assets/c5048670-11e0-4642-ab55-672f8750d0bd" /><br>

따라서 '화면 비율'에 맞도록<br>
ViewSize.x / ViewSize.y를 각각의<br>
-0.5된 픽셀 좌표에 곱해주었다<br>

이후 진행은 그대로<br>

- ViewSize : 현재 뷰포트(or 렌더 타겟)의 픽셀 단위 해상도를 반환<br>

<img width="2249" height="1171" alt="Image" src="https://github.com/user-attachments/assets/f6815d77-8fab-4a9e-b4d8-8d553dd865dd" /><br>

...?<br>
사실상 Depth...는 거의 필요없고<br>
화면에서의 UV좌표를 기준으로 값이 정해진 듯하다<br>
(기존에는 화면에 가까운 값들이 그나마 높은 값을 유지할 수 있었으나<br>
0과 1의 값은 작게, 0.5에 가까운 값은 보다 크게의 영향이 더 커진것 같다)<br>

## Process 4 화면 중앙이 아니라 캐릭터를 중심으로 해보자
아무래도 '기준'을 잘못 잡은 듯 하다<br>
생각해보니 내가 원하는 것은 '화면 기준'이 아니라<br>
'캐릭터(카메라)' 기준이었다<br>

찾아보니<br>
Absolute World Position 이라는 노드에 Camera Position이라는<br>
내가 원하는 옵션이 존재하였다<br>

- CameraPosition<br>
  : 월드 공간에서 '현재 픽셀 위치 - 카메라 위치'를 반환하는 값<br>
  (카메라 기준으로 픽셀의 위치를 계산하는 방식)<br>

<img width="256" height="256" alt="Image" src="https://github.com/user-attachments/assets/d23f4a3d-3a49-4445-b244-d5f42deee9e8" /><br>

그리고 포토샵을 켜서 간단한 텍스쳐를 만들어<br>
이 텍스쳐를 기반으로 효과를 주려 한다<br>
(RGB중 하나를 바탕으로 Lerp의 Alpha를 적용 시킬 예정)<br>
(이렇게 텍스쳐를 통해 효과를 적용하는 방식을<br>
 '샘플러' 라고 한다)<br>
(좀더 정확히는 '텍스쳐'를 어떻게 읽을지를 정하는 방식)<br>

<img width="2993" height="1403" alt="Image" src="https://github.com/user-attachments/assets/96f27a50-64f0-4989-be41-8d13054d516c" /><br>

카메라 상대 월드 포지션을 다시<br>
Divide로 나누어주고(정확히는 맵 Size값을 권장)<br>
0.5를 더하여 Texture의 UV좌표로 삼는다<br>
(이 때, 샘플러 소스를 'Clamp'로 하여<br>
0~1 부터의 값 사이로 제한한다)<br>
(그렇지 않으면 wrap으로 Default되어 있는 경우<br>
의도치 않은 효과를 보였다)<br>

- wrap : UV 좌표가 0~1을 벗어난 경우,<br>
         좌표를 '반복'하여 텍스쳐를 적용하는 방식<br>
- clamp : UV 좌표가 0~1을 벗어난 경우<br>
          그 값에 가장 가까운 0,1 중 하나를 대신 적용하는 방식<br>

<img width="2242" height="1174" alt="Image" src="https://github.com/user-attachments/assets/9de69d28-e97d-40e2-b066-9c9ad70e674e" /><br>

원하는 대로 깔끔하게 나왔다<br>
아마 텍스쳐가 구석에서 그라데이션 된것이였다면<br>
그 구석을 기준으로 효과가 발생하지 않았을까?<br>
추측해본다<br>

## 번외 : process 3 번째에서 별도의 효과를 준다면?

사실 3번 효과가 버리기 좀 아까운 효과여서<br>
조금 더 다듬어 보았다<br>

<img width="2477" height="1059" alt="Image" src="https://github.com/user-attachments/assets/5db1d331-959b-45a3-900d-3bb8e81b90c6" /><br>

마치 화면 바깥만 검은색이기에<br>
Sine이나 Cos로 반복되는 효과를 주면<br>
'긴장감'이나 '공포'스러운 연출이 가능하지 않을까?<br>
싶어서 Time과 Cosine 노드를 이용하여<br>
화면 좌표에 변화를 주었다<br>

### 결과물

<iframe width="560" height="315"
    src="https://www.youtube.com/embed/XeUKC76KY3s"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
</iframe><br>

역동적이다!<br>