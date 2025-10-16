---
title: "Height Map"
date : "2025-10-16 12:00:00 +0900"
last_modified_at: "2025-10-16T12:00:00"
categories:
  - 그래픽스
  - DirectX
tags:
  - Height Map
---


## Height Map?

[![Image](https://github.com/user-attachments/assets/507fb5c8-4d3a-4f69-b7c8-286ad71e2944)](https://github.com/user-attachments/assets/507fb5c8-4d3a-4f69-b7c8-286ad71e2944){: .image-popup}<br>

이전 강의에서 적용한 노멀 매핑을 사용하면<br>
울퉁불퉁한 질감을 가진 텍스쳐를 구현할 수 있다<br>

그런데 텍스쳐는 '`다양한 메시`'에 붙일 수 있어야 함<br>
ex) 울퉁불퉁한 메시, 평평한 메시<br>

그렇기에 위처럼<br>
깔끔한 '구체'이지만 표현되는 질감은 '패인'듯한 인상을 줄 수 있음<br>

그렇기에 Height Map을 통하여<br>
'질감' 에 대한 표현을 별도로 관리하고<br>
**하나의 텍스쳐**로 더 `많고 자연스러운 표현을 연출`<br>

- 텍스쳐와 메시는 각각 표현의 역할이 분리<br>
  - 메시 (Geometry) : 공간적, 실질적 형태, 비용 높음<br>
  - 텍스쳐 (Texture) : 색상, 질감의 세부 디테일, 비용 낮지만 한계가 존재<br>

- 효율적인 표현을 위해 등장한 것이<br>
  노멀 맵 / 높이 맵!<br>

- 노멀 맵<br>
  : 실제 메시의 형태를 바꾸지 않고<br>
    조명 연산을 변화시키는 트릭<br>
  - 다만 '실제로 매끈'해야 하는 메시에<br>
    울퉁불퉁한 노멀맵을 붙이면<br>
    울퉁불퉁한 반사는 진행되지만<br>
    윤곽은 매끈한 `'이상한 상태'`가 됨<br>
    -> '높이 맵'의 등장 이유<br>

- 높이 맵<br>
  : 픽셀의 깊이를 표현하려는 다양한 시도들<br>
    - Displacement Mapping : Vertex를 이동시켜 Geometry에 변형을 줌(다만 고비용)<br>
    - Parallex / Occusion Mapping : 픽셀 쉐이더 단계에서 텍스쳐 좌표를 깊이값으로 왜곡 (카메라 관점에서 입체감만 흉내, 저비용)<br>

그렇기에 PBR과 같은 사실적 표현 방식은 양 쪽을 복합적으로 고려<br> 
- HeightMap을 통해 픽셀의 깊이 표현<br>
- NormalMap을 통해 디테일한 조명 표현<br>

우리가 해보려는 것은 Vertex Shader를 통해<br>
변형을 하는 Displacement Mapping을 사용해보려 함<br>

### Height Map Texture

[![Image](https://github.com/user-attachments/assets/2d3b21e0-3a75-467f-ba6e-3a22707a21ad)](https://github.com/user-attachments/assets/2d3b21e0-3a75-467f-ba6e-3a22707a21ad){: .image-popup}<br>

하얄수록 '높아져야 하는' 강도를 표시<br>
어두운 부분은 '안쪽으로 들어가는' 부분을 표시<br>

- 이러한 효과에 강도를 줌으로서<br>
  불규칙해보이는 패턴 등을 표현할 때 효과적<br>

3D 렌더링 툴에서도<br>
Height Map을 만들어 주는 기능이 존재<br>

### AO(Ambient Occlusion, 환경 폐색)

[![Image](https://github.com/user-attachments/assets/2c297faa-1ec3-427a-9795-969b344f33fc)](https://github.com/user-attachments/assets/2c297faa-1ec3-427a-9795-969b344f33fc){: .image-popup}<br>

이전 TIL 에서도 간략하게 이야기 한 내용이지만<br>

- 복잡한 3D 물체를 그릴 때,<br>
  조명 계산이 쉽지 않은 부분이 존재<br>
  (빛이 닿지 않는)<br>
  이러한 부분에 대한 '그림자 연산'을 진행해 두는 것<br>

- 레이트레이싱이 아닌 Rasterize 기반의 렌더링에서<br>
  이러한 부분을 판단하기 힘듦<br>

- 그렇기에 어둡게 나오는 것이 자연스러운<br>
  구석진 부분 등에 '표시'를 해둔 것<br>
  

[![Image](https://github.com/user-attachments/assets/a6c582be-ec73-4048-a3e6-dd5871fa801d)](https://github.com/user-attachments/assets/a6c582be-ec73-4048-a3e6-dd5871fa801d){: .image-popup}<br>

AO Texture<br>

파여있는 부분에 대하여 미리 Texture Map에 미리 표기하고<br>
이를 Pixel Shader에서 이용하여 조명 효과를 효율적으로 표현<br>

## 예제 - Height Map을 적용해보기


[![Image](https://github.com/user-attachments/assets/98aa304b-43e0-4bf0-a721-0ec66e5111b4)](https://github.com/user-attachments/assets/98aa304b-43e0-4bf0-a721-0ec66e5111b4){: .image-popup}<br>

그림과 같은 효과를 적용하려면<br>
미리 말하였듯<br>
정점의 위치를 변화시킴으로서 그 효과를 극대화 시킬 수 있음<br>

```cpp
VertexShader.hlsl

PixelShaderInput main(VertexShaderInput input)
{
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
    
    if (useHeightMap)
    {
        // VertexShader에서는 SampleLevel 사용
        float3 heightMap = g_heightTexture.SampleLevel(g_sampler, input.texcoord, 0.0).xyz;
        heightMap = 2 * heightMap - 1.0;
        
        pos.xyz += output.normalWorld * heightMap * heightScale;
    }

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

- HeightMap의 요소들 역시<br>
  0~1로 되어있는 '강도'에 대한 수치 이기에<br>
  이들을 2* -1 해줌으로서 [0,1] -> [-1,1] 로 범위 변환<br>

- 그리고 이들은 `각각의 표면 방향(Normal)` 으로<br>
  그 수치만큼 나아가야 하므로<br>
  normalWorld에 곱해준다<br>

- 커스텀 여지를 heightScale을 통해 전달<br>
  (Constance Buffer)<br>

### 예제 정답

```cpp
    if (useHeightMap)
    {
        // VertexShader에서는 SampleLevel 사용
        float heightMap = g_heightTexture.SampleLevel(g_sampler, input.texcoord, 0.0).r;
        heightMap = 2 * heightMap - 1.0;
        
        pos.xyz += output.normalWorld * heightMap * heightScale;
    }
```

- HeightMap은 보통 R만 사용하며 상황에 따라 g,b 채널에 다른 값을 부여<br>

- 또한 이러한 Height Map의 효과는 Vertex가 어느정도 존재해야 볼 수 있다는 점을 유념<br>
  (바닥 같은 경우는 단순한 Square이기에, Vertex 변형의 의미가 거의 없음)<br>
  - 그렇기에 Height Map의 효과를 보려면 Vertex가 일정 수준은 있어야 함<br>
  - 다만 이것은 '해상도'에 따라 결정되기에<br>
    너무 높은 수준의 해상도를 가진 텍스쳐를 위하여<br>
    Vertex를 늘리는 것은 성능 소모가 과해질 수 있으니 유의할 것!!<br>

- Height Map 대신 Displacement 라는 용어를 사용하기도 함<br>
  (아까 위에도 말했듯 Vertex를 변환시키므로)<br>

- 뭔가 앨리어싱이 발생하는 것 같은데?
  - 이전에도 보았듯, 거리에 따른 LOD 레벨을 조정!<br>
    (다만 이건 Pixel Shader의 영역임)<br>

