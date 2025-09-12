---
title: "Image Based Lighting"
date : "2025-09-12 12:00:00 +0900"
last_modified_at: "2025-09-12T12:00:00"
categories:
  - Direct X
tags:
  - Image Based Lighting
---

## Image Based Lighting

[![Image](https://github.com/user-attachments/assets/e2ceeccd-cc4b-42f6-a5a2-54346c9a0e77)](https://github.com/user-attachments/assets/e2ceeccd-cc4b-42f6-a5a2-54346c9a0e77){: .image-popup}<br>

이미지 기반 라이팅은 '실제 환경에서 촬연한 이미지'를 이용하여<br>
3D 씬의 조명을 구현하는 방식이다<br>

환경 전체의 '빛 정보'를 반구 or 구 형태로 샘플링 하여<br>
오브젝트의 반사,굴절, 확산 등을 사실적으로 표현<br>

- 환경맵을 사용<br>
  (보통 파노라마 같은 360도 사진을 이용하여<br>
   구형 or 큐브맵으로 사용)<br>
  (동시에 이것이 전방위 '광원'의 역할을 한다)<br>

- Diffuse IBL<br>
  : 환경맵을 적분하여 부드럽게 퍼지는 간접 조명 계산용<br>

- Specular IBL<br>
  : 환경 맵을 러프니스를 이용하여 필터링<br>
  (언뜻 봤을때 이미지에 가까움)<br>

### 큐브맵? 환경맵?

- 큐브맵<br>
  : 환경이나 방향 데이터를 저장하기 위한 6장의 정사각형 텍스쳐이다<br>

- 환경맵<br>
  : 특정한 '장면'을 둘러싼 '주변 환경'의 조명/색상 정보를 담은 텍스쳐<br>
    (특정 점에서 주변 세계의 모습을 묘사)<br>
    (반사,굴절 IBL 등에서 사용)<br>
    구형 맵/ 큐브맵/ 파노라마 등의 형식을 포함<br>
    (환경을 표현하는 전체적인 개념을 뜻함)<br>

| 용어  | 범위/역할                | 예시 또는 형식                       |
| --- | -------------------- | ------------------------------ |
| 환경맵 | “주변 환경”을 표현하는 **개념** | 큐브맵, equirectangular 텍스처, 파노라마 |
| 큐브맵 | 환경맵을 구현하는 **특정 포맷**  | 6장의 ±X,±Y,±Z 텍스처               |


#### 샘플링 개념 복습

'연속된 공간, 이미지, 환경'을 픽셀 or 텍스쳐 좌표의 이산 값으로 읽어오는 과정<br>
(이산 값 : 딱 떨어지는, 셀수 있는 정수값)<br>


### PBR(Physically Based Rendering)과의 관계?

| 구분        | IBL의 역할                                         | PBR과의 연계                            |
| --------- | ----------------------------------------------- | ----------------------------------- |
| **재질 정의** | 금속/비금속(Metalness), 러프니스(Roughness), 알베도(Albedo) | PBR 쉐이더의 입력 값으로 사용됨                 |
| **조명**    | HDR 환경맵 기반의 전역 조명 제공                            | PBR BRDF 모델(GGX 등)로 광원-재질 상호작용 계산   |
| **정확도**   | 다중 샘플링과 적분으로 현실적인 반사 구현                         | Fresnel, 에너지 보존, 마이크로패싯 모델 등과 함께 사용 |
| **최적화**   | Pre-filtered Environment Map과 LUT로 성능 확보        | 실시간 렌더링(UE5, Unity HDRP)에서 표준       |


<br>

앞서 배운 phong 의 조명 타입은 3가지<br>
directional, spot, point 를 각각 배웠다<br>

그런데 현실의 조명들은 아주 복잡한 편...<br>
위의 조명들을 잘 조합하더라도 현실과 같은 조명 효과를 내기는<br>
매우 어려운 일이다<br>

- 컴퓨터 그래픽스는 '텍스쳐'를 점점 더 '효율적'으로 사용하는 방향으로 진화<br>
  - 먼 배경을 만드는 '큐브맵' 같은 기술<br>
  - 그러한 배경을 물체에 적용하는 '환경맵'과 같은 기술<br>
    - 그렇다면 조명도 이런식으로 사용하면 '현실'과 비슷하게 사용할 수 있지 않을까?<br>


### 원칙적인 계산법

[![Image](https://github.com/user-attachments/assets/083e0285-bafa-4430-b352-112634fadd7a)](https://github.com/user-attachments/assets/083e0285-bafa-4430-b352-112634fadd7a){: .image-popup}<br>

특정한 한 점에 대한 빛의 계산은<br>
그림처럼 환경맵의 여러 부분에서 '조명 정보'를 얻어와<br>
계산해야 한다<br>

- 특정한 점에서 여러 방향으로 벡터를 쏘아<br>
  만나는 텍스쳐에 대한 샘플링 정보를 가지고 오고<br>
  그 평균을 구하면 된다<br>

- 다만 이러한 방식은 '게임' 같은 실시간 렌더링이 필요한 경우에는<br>
  아주 느린 연산이 되어 못하게 된다<br>
  - 그렇기에 1~2번의 샘플링 정도만 진행해야 함<br>
  - 텍스쳐링이 효율적인 이유를 다시 생각해보자<br>
  - 자세하게 만들어진 사진을 텍스쳐로 만들고 GPU에 넣어두고 텍스쳐링을 적용하면<br>
  - 픽셀 쉐이더에서 이미지를 샘플링하여, 색을 아주 빠르게 결정 가능<br>

따라서 우리는 '텍스쳐링'을 잘 활용하여<br>
게임과 같은 실시간 렌더링에서도 좋은 그래픽을 만들 수 있어야 함!<br>

### 그렇다면 IBL을 어떻게 구현하지?

제한 조건은 '한 번만' 텍스쳐링 하는 것!<br>
따라서 '미리' 선처리된 '광원 텍스쳐'를 준비하고<br>
해당 텍스쳐를 샘플링 함으로서 IBL을 구현<br>

위에서 설명한<br>
- Diffuse IBL<br>
  : 난반사 모델링<br>

- Specular IBL<br>
  : 금속/거울의 하이라이트<br>

이 나뉘어져 있는 이유이다<br>

실제로 diffuse 자체는 난반사가 되었기에<br>
반사가 잘 되지 않는, 벽 등에 대한 질감을 표현할 수 있고<br>
Specular는 반사가 잘 되었기에 거울/금속 등의 표현에 적절<br>

- 이러한 IBL 데이터들은 전부 바르는 텍스쳐(albedo)가 아닌<br>
  '빛'을 표현하는 '환경광' 데이터라 생각하자<br>

- DX에서 SRV 형태로 GPU에 올라가고<br>
  픽셀 쉐이더 쪽에서 Sample() 을 통해 데이터를 샘플링<br>

- 보통 Roughness,metalic,intensity 같은 파라미터를 통하여<br>
  계산<br>

```
L IBL​ = kd ​ ⋅LdiffuseIBL​(N) + ks ​⋅ LspecularIBL​(R,α)

kd : diffuse/albedo 성분 (일반적으로 1 - metalic)
ks : specular 부분 (보통 metalic + fresnel term)
LdiffuseIBL : DiffuseIBL로서 보통 Irradiance map(블러된 환경맵) 샘플링 값
              (Normal 을 집어넣어 계산)
LspecularIBL​ : SpecularIBL로서 보통 (Prefiltered Environment Map + BRDF LUT)을 이용한 샘플링 값 
              (R : 반사벡터 , a : 러프니스와 같은 파라미터)
```

#### 람버트 조명(Lambertian Shading)

표면이 완벽한 난반사(diffuse)를 한다는 가정한 조명 모델<br>

```
L diffuse ​= kd ​ ⋅ I ⋅ max(0,N⋅L)

L : 광원 방향
N : 법선 벡터(표면)
I : 광원의 세기
Kd : 확산 반사 계수 (diffuse reflectance)
```


## 예제 - 이미지 기반 라이팅

```cpp
Cubemapping.h

#pragma once

#include <wrl.h>

#include "GeometryGenerator.h"
#include "Material.h"
#include "Vertex.h"

namespace hlab {

using Microsoft::WRL::ComPtr;

struct CubeMapping {

    std::shared_ptr<Mesh> cubeMesh;

    ComPtr<ID3D11ShaderResourceView> diffuseResView;  // Add
    ComPtr<ID3D11ShaderResourceView> specularResView; // Add

    ComPtr<ID3D11VertexShader> vertexShader;
    ComPtr<ID3D11PixelShader> pixelShader;
    ComPtr<ID3D11InputLayout> inputLayout;
};
} // namespace hlab
```

큐브매핑에서 SRV를 2개 추가<br>
각각 '큐브맵 텍스쳐'로 받을 예정이며<br>

위에서 말한<br>
DiffuseIBL, SpecularIBL로 사용될 것들이다<br>
(.dds 파일들)<br>

- 기본적으로 원본 이미지들을 수정한 것들<br>
  (디자이너의 영역에 가깝기는 함)<br>
  (Blur 같은 이미지 처리 기술을 활용할 순 있음)<br>

```cpp

ExampleApp.cpp

Render()
{
  ...
  // 큐브매핑
  m_context->IASetInputLayout(m_cubeMapping.inputLayout.Get());
  m_context->IASetVertexBuffers(
      0, 1, m_cubeMapping.cubeMesh->vertexBuffer.GetAddressOf(), &stride,
      &offset);
  m_context->IASetIndexBuffer(m_cubeMapping.cubeMesh->indexBuffer.Get(),
                              DXGI_FORMAT_R32_UINT, 0);
  m_context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

  m_context->VSSetShader(m_cubeMapping.vertexShader.Get(), 0, 0);
  m_context->VSSetConstantBuffers(
      0, 1, m_cubeMapping.cubeMesh->vertexConstantBuffer.GetAddressOf());
  ID3D11ShaderResourceView *views[2] = {m_cubeMapping.diffuseResView.Get(),
                                        m_cubeMapping.specularResView.Get()};
  m_context->PSSetShaderResources(0, 2, views);
  m_context->PSSetShader(m_cubeMapping.pixelShader.Get(), 0, 0);
  m_context->PSSetSamplers(0, 1, m_samplerState.GetAddressOf());

  m_context->DrawIndexed(m_cubeMapping.cubeMesh->m_indexCount, 0, 0);

  // 버텍스/인덱스 버퍼 설정
  for (const auto &mesh : m_meshes) {
      m_context->VSSetConstantBuffers(
          0, 1, mesh->vertexConstantBuffer.GetAddressOf());

      // 물체 렌더링할 때 큐브맵도 같이 사용
      ID3D11ShaderResourceView *resViews[3] = {
          mesh->textureResourceView.Get(), m_cubeMapping.diffuseResView.Get(),
          m_cubeMapping.specularResView.Get()};
      m_context->PSSetShaderResources(0, 3, resViews);

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

- views를 통하여<br>
  큐브맵 쪽에 SRV를 픽셀 쉐이더로 보내준다<br>

- 또한 해당 환경맵 데이터를 '광원'으로 활용할 것이기에<br>
  일반 물체들의 srv에도 추가해준다<br><br>


```cpp
cubeMappingPixelShader.hlsl

#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

TextureCube g_diffuseCube : register(t0);
TextureCube g_specularCube : register(t1);
SamplerState g_sampler : register(s0);

float4 main(PixelShaderInput input) : SV_TARGET
{
    // 주의: 텍스춰 좌표가 float3 입니다.
    // 주의: error X4532: texlod not supported on this target -> 쉐이더 모델(버전) 5_X로 올리기
    return g_specularCube.Sample(g_sampler, input.posWorld.xyz);
}
```

큐브맵 쪽에서는 딱히 무엇을 하지는 않는다<br>
다만, 기본적으로 이미지와 비슷한 역할의 Specular를 이용하여<br>
큐브맵을 그려준다<br>

- diffuse로 바꾸면 Blur가 심하게 처리된듯한 뿌연 느낌만이 남음<br>

### 예제 구현 1

```cpp

BasicPixelShader.hlsl

#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
TextureCube g_diffuseCube : register(t1);
TextureCube g_specularCube : register(t2);
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

    // 쉽게 이해할 수 있는 간단한 구현입니다.
    // IBL과 다른 쉐이딩 기법(예: 퐁 쉐이딩)을 같이 사용할 수도 있습니다.
    // 참고: https://www.shadertoy.com/view/lscBW4
    
    color = g_diffuseCube.Sample(g_sampler, input.normalWorld);
    color += g_specularCube.Sample(g_sampler, reflect(-toEye, input.normalWorld));

    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}
```

- color에 기존의 라이팅 부분을 초기화 시키고<br>
  diffuse와 specular 부분을 적용<br>
  (specular 부분은 이전의 환경맵 방식을 적용)<br>


### 결과

[![Image](https://github.com/user-attachments/assets/e63233c2-9e2b-46a8-a2bd-135928efc03e)](https://github.com/user-attachments/assets/e63233c2-9e2b-46a8-a2bd-135928efc03e){: .image-popup}<br>

diffuse 와 specular IBL 텍스쳐들이 적용된 모습<br>
다만 GUI 쪽의<br>
Diffuse와 Specular, Shiniess 부분은 적용되지 않았음<br>

## 예제 2 - 각 요소들이 적용되게 하기

일단 내가 구현한 방식

```cpp
BasicPixelShader.hlsl

#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
TextureCube g_diffuseCube : register(t1);
TextureCube g_specularCube : register(t2);
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

    // 쉽게 이해할 수 있는 간단한 구현입니다.
    // IBL과 다른 쉐이딩 기법(예: 퐁 쉐이딩)을 같이 사용할 수도 있습니다.
    // 참고: https://www.shadertoy.com/view/lscBW4
    
    color = g_diffuseCube.Sample(g_sampler, input.normalWorld).xyz * material.diffuse;
    color += g_specularCube.Sample(g_sampler, reflect(-toEye, input.normalWorld)).xyz * material.specular;
    color *= material.shininess;

    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}

```

- 각 요소의 diffuse와 specular를 곱해주었고<br>
  최종적으로 shiniess를 마지막에 곱해주었다<br>

- 다만 지금 생각해보니 shininess는 specular 쪽에서 처리하는게 맞지 않나 싶기도 하다<br>

### 결과

[![Image](https://github.com/user-attachments/assets/e587c999-ffda-476c-9ca9-63456221ae9b)](https://github.com/user-attachments/assets/e587c999-ffda-476c-9ca9-63456221ae9b){: .image-popup}<br>

- diffuse를 높일 경우는 전체적으로 밝아지는 느낌이 강하다<br>
  뭔가 조명을 세게 받는 느낌이 됨<br>

- Specular를 높일수록 주변 환경 색을 잘 받기에<br>
  매끈매끈한 느낌이 강해졌다<br>

- Shininess가 낮으면 다소 검어지고<br>
  높아질수록 전체색이 흰색에 가까워진다<br>

텍스쳐를 입힌 버젼<br>

[![Image](https://github.com/user-attachments/assets/87b9b9f3-8b07-4827-85ce-36877a816668)](https://github.com/user-attachments/assets/87b9b9f3-8b07-4827-85ce-36877a816668){: .image-popup}<br>

- specular를 높였더니 뭔가 실제 존재하는 잘닦인 지구본 같은 느낌이 되었다!<br>

실제 PBR 기법에서는 '거칠기'(Roughness)를 사용하여<br>
텍스쳐를 다르게 샘플링한다 함<br>
(거칠기가 높을수록 빛을 잘 반사하지 않으며 - 나무, 벽 느낌<br>
낮으면 빛을 잘 반사한다 - 거울,금속 등)<br>

### 예제 코드 방식

```cpp

float4 diffuse = g_diffuseCube.Sample(g_sampler, input.normalWorld);
diffuse.xyz *= material.diffuse;

float4 specular = g_specularCube.Sample(g_sampler, reflect(-toEye, input.normalWorld));

specular *= pow((specular.x + specular.y + specular.z) / 3.0, material.shininess);
specular.xyz *= material.specular;

return diffuse + specular; // 텍스쳐 x

```

- shininess 수치가 올라갈수록 '빛나는 부분'을 제외하면 역으로 어두워진다!<br>
- specular의 빛 수치들의 평균을 낸 후, shininess 만큼 pow (제곱) 해준다<br>

[![Image](https://github.com/user-attachments/assets/f4d885f6-946a-4de2-a47b-1ed2347ce7bf)](https://github.com/user-attachments/assets/f4d885f6-946a-4de2-a47b-1ed2347ce7bf){: .image-popup}<br>

### 완성된 지구본

```cpp
#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
TextureCube g_diffuseCube : register(t1);
TextureCube g_specularCube : register(t2);
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

    // 쉽게 이해할 수 있는 간단한 구현입니다.
    // IBL과 다른 쉐이딩 기법(예: 퐁 쉐이딩)을 같이 사용할 수도 있습니다.
    // 참고: https://www.shadertoy.com/view/lscBW4
    
    float4 diffuse = g_diffuseCube.Sample(g_sampler, input.normalWorld);
    diffuse.xyz *= material.diffuse;
    
    float4 specular = g_specularCube.Sample(g_sampler, reflect(-toEye, input.normalWorld));
    
    specular *= pow((specular.x + specular.y + specular.z) / 3.0, material.shininess);
    specular.xyz *= material.specular;
    
    color = diffuse.xyz + specular.xyz;

    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}
```

 그냥 마지막에 texture 부분을 적용하도록 하였다<br>

[![Image](https://github.com/user-attachments/assets/d23a8758-c592-4841-a316-b84a2d9a9f3b)](https://github.com/user-attachments/assets/d23a8758-c592-4841-a316-b84a2d9a9f3b){: .image-popup}<br>

- shininess를 잘 활용하니까 매우 잘닦인 지구본 같이 완성되었다<br>
- '빛이 나는 부분'이 더 강조되는 역할을 해준다<br>


### tmi - 젤다로 모델 변경 시

1. diffuse만 적용시키고 specular는 0에 가까운 경우<br>

[![Image](https://github.com/user-attachments/assets/0b0a2b35-04ff-4d4c-84e0-3543306d8987)](https://github.com/user-attachments/assets/0b0a2b35-04ff-4d4c-84e0-3543306d8987){: .image-popup}<br>

'광원'을 '환경맵'인 Diffuse IBL에서 갖고 오기에<br>
은은한 광원을 제공하기에 상당히 자연스러운 광원이 구현되어 보인다<br>

2. specular 값을 높인 경우<br>

[![Image](https://github.com/user-attachments/assets/ae812229-52bf-4aa2-9db2-98b272fb976c)](https://github.com/user-attachments/assets/ae812229-52bf-4aa2-9db2-98b272fb976c){: .image-popup}<br>

specular가 주변 환경의 색을 읽어오기에<br>
뭔가 '실물 동상' 같은 느낌의 젤다가 완성되었다<br>