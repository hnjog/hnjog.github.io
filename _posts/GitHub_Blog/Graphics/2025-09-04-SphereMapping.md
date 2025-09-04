---
title: "Sphere Mapping"
date : "2025-09-04 18:00:00 +0900"
last_modified_at: "2025-09-04T18:00:00"
categories:
  - Direct X
tags:
  - Subdivision
  - Texture Mapping
---

## Sphere Mapping

[![Image](https://github.com/user-attachments/assets/c0ca268c-4ec2-41c9-8176-b95a6d7b2ab1)](https://github.com/user-attachments/assets/c0ca268c-4ec2-41c9-8176-b95a6d7b2ab1){: .image-popup}<br>

부드럽게 잘 텍스쳐링 된 줄 알았으나<br>
이음새 부분에서 갑자기 이상하게 보인다<br>

- 텍스쳐는 '정사각형' 모습으로 존재하며<br>
  그것을 각 정점의 uv좌표에 맞게 그리는 방식<br>

- 현재의 subDivision은 모든 Vertex 하나 밖에 없는 Mesh에 적용 중<br>
  (텍스쳐 좌표를 vertex 좌표 기준으로 역산정하여 구하는 중이다)<br>

- atan2f 를 통해 텍스쳐 좌표를 역산중이기에<br>
  원본 텍스쳐의 0,1 이 겹쳐지는 부분에서 문제가 발생<br>
  (구체 모델링)<br>

### 문제의 원인?

[![Image](https://github.com/user-attachments/assets/6adbab64-129e-4878-9451-241b98b9e81d)](https://github.com/user-attachments/assets/6adbab64-129e-4878-9451-241b98b9e81d){: .image-popup}<br>

끝 부분의 Vertex 상태들<br>

- 위쪽은 '같은 위치' 이지만 서로 다른 텍스쳐 좌표를 가지기에<br>
  문제가 없음<br>

- 아래쪽이 '문제가 되는 케이스'<br>
  '같은 삼각형' 안에서<br>
  한 쪽의 텍스쳐 좌표는 (1.0,~)<br>
  다른 한쪽의 텍스쳐 좌표는 (0.0,~)<br>
  이러면 중간쪽의 텍스쳐 좌표가 '보간'이 되면서<br>
  난리가 난다...<br>
  (0~1 사이의 모든 것이 압축되어 있듯 표현이 된다...)<br>

### 해결책?

- 이전에 구현한 'Sphere' 방식처럼<br>
  '중복된 영역'이지만 Texture Coord를 '다르게' 가지는<br>
  Mesh를 사용(최선의 방식!)<br>
  (이전 예제에서 사용한 방식)<br>

- 텍스쳐 좌표를 'Vertex' 단위가 아니라<br>
  '픽셀 단위'로 적용하는 방식<br>
  (현재 사용할 방식)<br>

### 예제 코드

Common.hlsli<br>

```cpp
struct PixelShaderInput
{
    float4 posProj : SV_POSITION; // Screen position
    float3 posModel : POSITION0; // Model position
    float3 posWorld : POSITION1; // World position (조명 계산에 사용)
    float3 normalWorld : NORMAL;
    float2 texcoord : TEXCOORD;
    float3 color : COLOR; // Normal lines 쉐이더에서 사용
};
```

- PS에서 처리하도록 Model Position용 코드 추가<br>

VertexShader

```cpp
PixelShaderInput main(VertexShaderInput input)
{
    PixelShaderInput output;
    float4 pos = float4(input.posModel, 1.0f);
    
    output.posModel = pos.xyz; // model 좌표계 저장
    
    pos = mul(pos, model);
    
    output.posWorld = pos.xyz; // 월드 위치 따로 저장

    pos = mul(pos, view);
    pos = mul(pos, projection);

    output.posProj = pos;
    output.texcoord = input.texcoord;
    output.color = float3(0.0f, 0.0f, 0.0f); // 다른 쉐이더에서 사용
    
    float4 normal = float4(input.normalModel, 0.0f);
    output.normalWorld = mul(normal, invTranspose).xyz;
    output.normalWorld = normalize(output.normalWorld);

    return output;
}
```

- posModel 을 저장해둔다<br>

PixelShader

```cpp
float2 uv;
uv.x = atan2(input.posModel.z, input.posModel.x) / (3.14592 * 2.0) + 0.5f;
uv.y = acos(input.posModel.y / 1.5) / 3.14592;

return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, uv) : float4(color, 1.0);
```

- texCoord 대신 posModel 을 기반으로 한<br>
  uv 좌표를 사용한다<br>

### 결과

[![Image](https://github.com/user-attachments/assets/efe79b24-68dc-4a5c-b3c4-a04091372557)](https://github.com/user-attachments/assets/efe79b24-68dc-4a5c-b3c4-a04091372557){: .image-popup}<br>

텍스쳐링이 깔끔해졌다<br>
이런 식으로 '픽셀 쉐이더 쪽'에서 텍스쳐 좌표를 계산하는 방식도 존재<br>
(Sphere Mapping이 이 방식으로 이루어진다)<br>