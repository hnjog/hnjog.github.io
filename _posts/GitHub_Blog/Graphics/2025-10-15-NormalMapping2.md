---
title: "Normal Mapping 2"
date : "2025-10-15 12:00:00 +0900"
last_modified_at: "2025-10-15T12:00:00"
categories:
  - 그래픽스
  - DirectX
tags:
  - Normal Mapping
---


## AO(Ambient Occlusion, 환경 폐색)

빛이 닿지 않는 곳을 표현하기 위한 기술<br>
(`음영`, 그림자와 관련)<br>

- 실제 조명 계산을 진행하지 않고<br>
  기하 정보의 '굴곡' 이나 '틈새'의<br>
  어두운 음영을 미리 계산하는 방식<br>

## Tangent

- 표면의 한 점에서<br>
  U 방향(텍스쳐 X 축)을 가리키는 단위 벡터<br>
  (삼각함수의 Tan()이 아님)<br>

| 이름            | 의미        | 직교 관계   | 데이터 출처                   | 좌표계 기준                       |
| ------------- | --------- | ------- | ------------------------ | ---------------------------- |
| **Tangent**   | 텍스처 U축 방향 | 서로 직교 ⟂ | 메시의 Vertex 데이터           | 보통 *모델 로컬 공간* (Object Space) |
| **Bitangent** | 텍스처 V축 방향 | 서로 직교 ⟂ | 보통 Tangent × Normal 로 계산 | 로컬 공간                        |
| **Normal**    | 표면의 수직 방향 | 서로 직교 ⟂ | 메시의 Vertex 데이터           | 로컬 공간                        |

- 이들은 각각 TangentSpace를 정의하는 축인 동시에<br>
  모델 좌표계(Object Space)의 기준으로 표현됨<br>

그림으로 보면<br>

[![Image](https://github.com/user-attachments/assets/39c87c6c-d0d3-429e-9597-cafb35a76eb2)](https://github.com/user-attachments/assets/39c87c6c-d0d3-429e-9597-cafb35a76eb2){: .image-popup}<br>

(U와 V는 좌표계와 모델링에 따라 달라질 수 있으니<br>
위의 내용은 개념 이해의 도움 측면으로만 이해하자)<br>

DX 기준은 이렇다<br>
(다만 World 좌표에선 y,z 값이 뒤집힘)<br>

[![Image](https://github.com/user-attachments/assets/8b0d1a9c-7c72-4f0e-866d-4c31dee1d5f5)](https://github.com/user-attachments/assets/8b0d1a9c-7c72-4f0e-866d-4c31dee1d5f5){: .image-popup}<br>

- 이러한 Tangent 데이터는 Mesh (Vertex) 데이터의 일부로 포함<br>
  - NormalMap 은 픽셀 단위의 '표면 방향'을 RGB로 저장한 텍스쳐임<br>
    (픽셀의 Normal이 TangentSpace 기준으로 어느 방향인가만 저장함)<br>
  - 그렇기에 예제에도 Vertex 쪽에서 관리하고<br>
    VS에서 PS로 전달<br>

- 그렇기에 보통은 Modeling 소프트 웨어에서 만들어진다<br>
  (디자이너가 제작)<br>
  (다만 포함되지 않는 경우는 우리가 계산해서 사용해야 함)<br>

### Normal 텍스쳐 내부 데이터

| 채널        | 의미                | 해석               |
| --------- | ----------------- | ---------------- |
| **R (X)** | Tangent 축 방향 성분   | U방향으로 얼마나 기울었는가  |
| **G (Y)** | Bitangent 축 방향 성분 | V방향으로 얼마나 기울었는가  |
| **B (Z)** | Normal 축 방향 성분    | 표면에서 얼마나 위로 향했는가 |

- 즉, 각각의 수치들이<br>
  Tangent Space의 X,Y,Z 축 기준으로 얼마나 '기울었는지'<br>
  즉 `투영`되어 있는지에 대한 정보만을 저장<br>
  (dot(direction, tangent), dot(direction, bitangent), dot(direction, normal))<br>

- 그리고 나중에 월드 좌표 변환된 Tangent Space에<br>
  이 수치들을 곱해주어 World 좌표에서의 Normal 을 얻을 수 있는 것!<br>

### Tangent Space

[![Image](https://github.com/user-attachments/assets/ee8011bd-d83f-4dd9-86c6-07e489b057db)](https://github.com/user-attachments/assets/ee8011bd-d83f-4dd9-86c6-07e489b057db){: .image-popup}<br>

픽셀의 한 면에서<br>
*접선(Tangent), 비접선(Bitangent), 법선(Normal)*로 만든 **로컬 3축 좌표계**<br>

이것을 묶어 3x3 행렬로 만들걸 줄여서 `TBN 행렬`이라 부른다<br>

[![Image](https://github.com/user-attachments/assets/48b6d9f0-ca82-46cd-a462-d9ccf803a106)](https://github.com/user-attachments/assets/48b6d9f0-ca82-46cd-a462-d9ccf803a106){: .image-popup}<br>

변환 방식<br>

[참고 블로깅 : 선형변환의 행렬 표현 부분](https://hnjog.github.io/%EC%88%98%ED%95%99/Transformations2/)<br>

```cpp
float3 normalTex = texNormal.Sample(sampler, uv).xyz * 2.0 - 1.0; // [-1,1] 변환
float3x3 TBN = float3x3(Tangent, Bitangent, Normal);
float3 normalWorld = normalize(mul(normalTex, TBN));
```

- Tangent, Bitangent, Normal 들은 Vertex 쪽에서<br>
  건네 받는 편<br>
  (이들은 VS에서 월드 변환이 '된 상태'로 PS로 건네짐)<br>

- 결과로 얻는 normalWorld 가<br>
  Nomral Texture 픽셀의 '로컬 방향'을<br>
  World 좌표로 변환한 벡터 이다<br>
  (요약하자면 `World 기준으로 보는 한 픽셀의 Normal 벡터`)<br>

### 요약
- Normal Map 은 Tangent Space에서 '상대적인 방향 수치' 만을 저장<br>
- TBN은 Vertex 쪽에서 가지고 있으며 World 좌표 변환까지 한 후 PS로 건네줌<br>
- PS에서 실제 픽셀의 Normal 방향을 구하고 '조명 계산' 등에 사용<br>

```mathematica
[Tangent Space]
        ↑ Normal(U x V)
        |
        |         World Normal (Z - 좌표변환 완료)
        |        /
        |       /
        |------→ Tangent (U)
       /
      /
 Bitangent (V)

```

---

## 예제 - Normal Map을 이용한 노멀 매핑


[![Image](https://github.com/user-attachments/assets/cb207c13-4246-4a38-a179-b580d3d83d2f)](https://github.com/user-attachments/assets/cb207c13-4246-4a38-a179-b580d3d83d2f){: .image-popup}<br>

이전에 작성한 구 위치에 NormalMap을 이용하도록 수정하는 예제<br>

```cpp
PixelShader.hlsl

 if (useNormalMap) // NormalWorld를 교체
 {
     // UNORM_ 으로 건네지기에 [0.0 ~ 1.0] 범위를 가짐
     float3 normalTex = g_normalTexture.SampleLevel(g_sampler, input.texcoord, 0.0).rgb;
     normalTex = 2.0 * normalTex - 1.0; // 범위 조절 [-1.0, 1.0] - 일종의 약속
     
     float3 BieTan = cross(input.normalWorld, input.tangentWorld);
     
     float3x3 TBN = float3x3(input.tangentWorld, BieTan, input.normalWorld);
     
     normalWorld = normalize(mul(normalTex, TBN));
 }
```

위의 공식을 적용하여 쉽게 만들 수 있다!<br>

### 예제 정답

```cpp
float3 N = input.normalWorld;
float3 T = normalize(input.tangentWorld - dot(input.tangentWorld, N) * N);
float3 B = cross(N,T);

float3x3 TBN = float3x3(T, B, N);
```

다른건 거의 비슷한데 Tangent 구하는 부분이 달라졌다<br>

- T에서 `N 성분을 빼내` (N에 수직)<br>
  '투영'한 뒤 정규화<br>

- `T와 N이 완벽한 직교라는 보장이 없기`에<br>
  이 쪽이 더 정확한 처리임<br>

- TBN은 직교 정규 여야 정확히 동작<br>

### Vertex Shader의 좌표 변환 과정

```cpp

PixelShaderInput main(VertexShaderInput input)
{
    // 모델(Model) 행렬은 모델 자신의 원점에서 
    // 월드 좌표계에서의 위치로 변환을 시켜줍니다.
    // 모델 좌표계의 위치 -> [모델 행렬 곱하기] -> 월드 좌표계의 위치
    // -> [뷰 행렬 곱하기] -> 뷰 좌표계의 위치 -> [프로젝션 행렬 곱하기]
    // -> 스크린 좌표계의 위치
    
    // 뷰 좌표계는 NDC이기 때문에 월드 좌표를 이용해서 조명 계산
    
    PixelShaderInput output;
    
    // Normal 벡터 먼저 변환 (Height Mapping)
    float4 normal = float4(input.normalModel, 0.0f);
    output.normalWorld = mul(normal, invTranspose).xyz;
    output.normalWorld = normalize(output.normalWorld);
    
    // Tangent 벡터는 modelWorld로 변환
    float4 tangentWorld = float4(input.tangentModel, 0.0f);
    tangentWorld = mul(tangentWorld, modelWorld);

    float4 pos = float4(input.posModel, 1.0f);
    pos = mul(pos, modelWorld);
    
    ...

    output.posWorld = pos.xyz; // 월드 위치 따로 저장

    pos = mul(pos, view);
    pos = mul(pos, projection);

    output.posProj = pos;
    output.texcoord = input.texcoord;
    output.tangentWorld = tangentWorld.xyz;

    output.color = float3(0.0f, 0.0f, 0.0f);

    return output;
}
```

- normalWorld는 '평면의 법선'이기에<br>
  변환 방식이 다소 다름<br>
  - Normal은 평면의 '수직'이라는 조건을 보존해야 함<br>
  - 평면의 방정식 : n T x = c (n : 노멀, x : 점 , c : 평면)<br>
    (T는 전치, n을 전치한 행렬)<br>

대략적인 정리<br>

```
x_world = M_modelToWorld * x_model (Model -> World 변환)

n_worldᵀ * x_world = n_modelᵀ * x_model (법선도 다음을 만족해야 함)

(x_world를 푼 경우)
n_worldᵀ * M * x_model = n_modelᵀ * x_model

(모든 모델에 공통적으로 적용되어야 하므로 정리)
n_worldᵀ * M = n_modelᵀ

양변에 전치를 취하면
Mᵀ * n_world = n_model

최종정리
n_world = (M⁻¹)ᵀ * n_model
```

- 탄젠트는 Model To World 로 좌표 변환 진행<br>
