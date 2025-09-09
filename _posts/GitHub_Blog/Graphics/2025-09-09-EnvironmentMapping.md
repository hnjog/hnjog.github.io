---
title: "Environment Mapping"
date : "2025-09-09 14:00:00 +0900"
last_modified_at: "2025-09-09T14:00:00"
categories:
  - Direct X
tags:
  - Environment Mapping
  - Reflection Mapping
---

## Environment Mapping

[![Image](https://github.com/user-attachments/assets/cf04ffe0-0744-463a-966c-7d22418330b0)](https://github.com/user-attachments/assets/cf04ffe0-0744-463a-966c-7d22418330b0){: .image-popup}<br>

- 주어지는 '환경' 요소의 색을 반사하여<br>
  물체에 코팅하는 매핑 방식<br>

- 환경 요소(큐브맵 같은 '스카이 박스' 등)를 이용하는 기법<br>

- 반사 벡터를 통해 그 값을 가져온다<br>


## 구현 법?

[![Image](https://github.com/user-attachments/assets/3c95e82f-0ba5-4b01-8083-e9c3032a3a8b)](https://github.com/user-attachments/assets/3c95e82f-0ba5-4b01-8083-e9c3032a3a8b){: .image-popup}<br>

- 시점과 상관없이 '표면'에 수직인 방향으로부터<br>
  '큐브맵'에 '색'을 가져오는 것<br>

- 시각 벡터를 기반으로 반사시킨<br>
  '반사 벡터'에 부딪힌 '큐브맵'의 색 가져오기<br>
  (v = i - 2 * n * dot(i,n))<br>
  (v : 반사 벡터, i : 입사벡터, n : 법선 벡터)<br>

### 구현 시 주의사항

- '시각 벡터'에 영향을 받기에<br>
  관찰자 시점에 따라 구체에 보이는 모습이 달라져야 한다<br>

- Model Rotation과 Model Translation에 영향을 받지 않음<br>
  (회전, 이동)<br>

- 반사 벡터를 잘 구현할 것!<br>
  (이게 적용되냐 아니냐의 시각적 효과 차이가 큰 편)<br>

## 내 구현 - Environment Mapping 구현 1

```cpp
example.cpp - Render()
{
... 위쪽에서 큐브맵 렌더링

// 버텍스/인덱스 버퍼 설정
for (const auto &mesh : m_meshes) {
    m_context->VSSetConstantBuffers(
        0, 1, mesh->vertexConstantBuffer.GetAddressOf());

    // TODO: 물체 렌더링할 때 큐브맵도 같이 사용
    ID3D11ShaderResourceView *resViews[2] = {
        mesh->textureResourceView.Get(),
        m_cubeMapping.cubemapResourceView.Get()};
    m_context->PSSetShaderResources(0, 2, resViews);

    m_context->PSSetConstantBuffers(
        0, 1, mesh->pixelConstantBuffer.GetAddressOf());

    m_context->IASetInputLayout(m_basicInputLayout.Get());
    m_context->IASetVertexBuffers(0, 1, mesh->vertexBuffer.GetAddressOf(),
                                  &stride, &offset);
    m_context->IASetIndexBuffer(mesh->indexBuffer.Get(),
                                DXGI_FORMAT_R32_UINT, 0);
    m_context->IASetPrimitiveTopology(
        D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    m_context->DrawIndexed(mesh->m_indexCount, 0, 0);
}
}
```

큐브맵 에서 사용하는 SRV를 PixelShader에 전송<br>

그리고 Pixel Shader에서 해당 큐브맵을 이용하여 매핑을 해준다<br>

```cpp
// PixelShader.hlsl

#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
// TODO:
TextureCube g_textureCube1 : register(t1);

SamplerState g_sampler : register(s0);

cbuffer BasicPixelConstantBuffer : register(b0)
{
    float3 eyeWorld;
    bool useTexture;
    Material material;
    Light light[MAX_LIGHTS];
    float3 rimColor;
    float rimPower;
    float rimStrength;
    bool useSmoothstep;
};

float4 main(PixelShaderInput input) : SV_TARGET
{
    float3 toEye = normalize(eyeWorld - input.posWorld);

    float3 color = float3(0.0, 0.0, 0.0);
    
    int i = 0;
    
    // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-for
    // https://forum.unity.com/threads/what-are-unroll-and-loop-when-to-use-them.1283096/
    
    [unroll] // warning X3557: loop only executes for 1 iteration(s), forcing loop to unroll
    for (i = 0; i < NUM_DIR_LIGHTS; ++i)
    {
        color += ComputeDirectionalLight(light[i], material, input.normalWorld, toEye);
    }
    
    [unroll]
    for (i = NUM_DIR_LIGHTS; i < NUM_DIR_LIGHTS + NUM_POINT_LIGHTS; ++i)
    {
        color += ComputePointLight(light[i], material, input.posWorld, input.normalWorld, toEye);
    }
    
    [unroll]
    for (i = NUM_DIR_LIGHTS + NUM_POINT_LIGHTS; i < NUM_DIR_LIGHTS + NUM_POINT_LIGHTS + NUM_SPOT_LIGHTS; ++i)
    {
        color += ComputeSpotLight(light[i], material, input.posWorld, input.normalWorld, toEye);
    }

    // Normal에 해당하는 큐브맵 좌표값 사용
    color = g_textureCube1.Sample(g_sampler, input.normalWorld.xyz).xyz;
    
    // reflect(광선이 들어오는 방향, 노멀 벡터)
    // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-reflect
    
    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}

```

- color = g_textureCube1.Sample(g_sampler, input.normalWorld.xyz).xyz; 부분을 통하여<br>
  값을 '대입'해준다<br>
  (color += 해주니 광원에 영향을 너무 많이 받아 하얗게 나온다)<br>

여기까지 구현하면 사진처럼 나오게 된다<br>

[![Image](https://github.com/user-attachments/assets/4b25028c-fe52-40a3-b01f-80b299f9aaac)](https://github.com/user-attachments/assets/4b25028c-fe52-40a3-b01f-80b299f9aaac){: .image-popup}<br>

## 내 구현 - Environment Mapping 구현 2 (반사 벡터 적용)

이제 위의 공식을 이용하여 반사벡터 부분을 구현하자<br>

```cpp
#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
// TODO:
TextureCube g_textureCube1 : register(t1);

SamplerState g_sampler : register(s0);

cbuffer BasicPixelConstantBuffer : register(b0)
{
    float3 eyeWorld;
    bool useTexture;
    Material material;
    Light light[MAX_LIGHTS];
    float3 rimColor;
    float rimPower;
    float rimStrength;
    bool useSmoothstep;
};

float4 main(PixelShaderInput input) : SV_TARGET
{
    float3 toEye = normalize(eyeWorld - input.posWorld);

    float3 color = float3(0.0, 0.0, 0.0);
    
    int i = 0;
    
    // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-for
    // https://forum.unity.com/threads/what-are-unroll-and-loop-when-to-use-them.1283096/
    
    [unroll] // warning X3557: loop only executes for 1 iteration(s), forcing loop to unroll
    for (i = 0; i < NUM_DIR_LIGHTS; ++i)
    {
        color += ComputeDirectionalLight(light[i], material, input.normalWorld, toEye);
    }
    
    [unroll]
    for (i = NUM_DIR_LIGHTS; i < NUM_DIR_LIGHTS + NUM_POINT_LIGHTS; ++i)
    {
        color += ComputePointLight(light[i], material, input.posWorld, input.normalWorld, toEye);
    }
    
    [unroll]
    for (i = NUM_DIR_LIGHTS + NUM_POINT_LIGHTS; i < NUM_DIR_LIGHTS + NUM_POINT_LIGHTS + NUM_SPOT_LIGHTS; ++i)
    {
        color += ComputeSpotLight(light[i], material, input.posWorld, input.normalWorld, toEye);
    }

    // Normal에 해당하는 큐브맵 좌표값 사용
    //color = g_textureCube1.Sample(g_sampler, input.normalWorld.xyz).xyz;
    
    // reflect(광선이 들어오는 방향, 노멀 벡터)
    // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-reflect
    color = g_textureCube1.Sample(g_sampler, reflect(-toEye, input.normalWorld)).xyz;
    
    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}

```

- reflect 라는 쉐이더 내부에서 제공하는 함수가 이미 존재!<br>
  (해당 함수 내부에서 주어진 공식을 내온다)<br>
  이걸 가져다 사용하면 된다!<br>

- color = g_textureCube1.Sample(g_sampler, reflect(-toEye, input.normalWorld)).xyz;<br>
  : 해당 부분에서 '입사 벡터'는<br>
  눈 쪽에서 '현재 위치'를 향해 바라보는 것이므로<br>
  (input.posWorld - eyeWorld)<br>
  마침 toEye가 딱 -를 곱한 것이니<br>
  -를 곱하여 이용한다<br>

구현 결과<br>

[![Image](https://github.com/user-attachments/assets/e7b56b87-9dd6-4f8c-86c6-15cd2a3b11a9)](https://github.com/user-attachments/assets/e7b56b87-9dd6-4f8c-86c6-15cd2a3b11a9){: .image-popup}<br>


- 일단 예제 부분에서도 color를 바로 return 하는 방식을 취하는 편<br>

### TMI 모델을 바꾼다면...?

[![Image](https://github.com/user-attachments/assets/821e4267-c9cb-4cea-a73a-06823b61c14b)](https://github.com/user-attachments/assets/821e4267-c9cb-4cea-a73a-06823b61c14b){: .image-popup}<br>

- 마치 젤다가 '동상'처럼 보인다<br>

- 반사와 같은 요소를 잘 조절하면 '재질'을 표현할 수 있는 것을 잊지 말자!<br>