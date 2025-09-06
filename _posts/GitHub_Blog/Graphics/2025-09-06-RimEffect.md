---
title: "Rim Lighting"
date : "2025-09-06 16:00:00 +0900"
last_modified_at: "2025-09-06T16:00:00"
categories:
  - Direct X
tags:
  - Rim Lighting
---

## Rim Lighting

[![Image](https://github.com/user-attachments/assets/5ecede9c-7f04-4411-a082-146abe9f90f7)](https://github.com/user-attachments/assets/5ecede9c-7f04-4411-a082-146abe9f90f7){: .image-popup}<br>

캐릭터나 '오브젝트'의 가장자리에 '밝은 윤곽선'을 넣어주는 조명 기법<br>

- 빛이 뒤/측면 에 올때의 피사체의 외곽선을<br>
  은은하게 강조<br>
  ('역광'이라고도 표현)<br>


### 구현 1

기본적으로 Pixel 쉐이더에서 진행한다<br>

```cpp
#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

Texture2D g_texture0 : register(t0);
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

    // Rim Lighting
    // OpenGL SuperBible 7th Edition, Ch13. Rendering Techniques
    
    // '테두리'의 위치 확인 필요
    // 화면 방향 벡터 기준으로 normal이 90도에 가까운 녀석들 (dot)
    // - (1 - dot) 를 통해 테두리에 가까운 녀석들을 판별
    // - 해당 부분을 pow 하여 '더 날카롭게' 테두리에 있는 녀석들만 선정 가능하다
    // - 이후 rim color와 strength를 적용
    color += ((rimColor * pow(1 - dot(toEye, input.normalWorld),rimPower)) * rimStrength);
    
    // Smoothstep
    // https://thebookofshaders.com/glossary/?search=smoothstep

    return useTexture ? float4(color, 1.0) * g_texture0.Sample(g_sampler, input.texcoord) : float4(color, 1.0);
}
```

- toEye : 화면을 향하는 벡터<br>
- input.normalWorld : 현재 normal 벡터<br>

- 테두리 구하는 법?<br>
  : 화면을 향하는 벡터와 'normal'을 내적(dot)하여<br>
    그 값이 0에 가까우면 '직교'하므로<br>
    0에 가까울수록 '테두리'에 가깝다고 판정 가능<br>

- 이후 해당 값에 1- 를 적용하여<br>
  테두리에 '가까운' 녀석들을 골라냄<br>

- Pow를 통하여 '테두리'에 더 가까운 부분만을 적용할수도 있음<br>
  (앞쪽에 abs를 넣어줄수도 있음)<br>

- 마지막으로 color와 strength를 적용<br>

사실 unreal 쪽의 fresnel 효과 구현과 매우 유사하여<br>
쉽게 구현을 할 수 있었다<br>

### 결과
[![Image](https://github.com/user-attachments/assets/2471d5c2-4796-437e-a56e-0d0909f7b3fa)](https://github.com/user-attachments/assets/2471d5c2-4796-437e-a56e-0d0909f7b3fa){: .image-popup}<br>

- 색이 잘 적용되는 모습<br>

[![Image](https://github.com/user-attachments/assets/a221f99c-60b0-469e-a824-14dd4663e770)](https://github.com/user-attachments/assets/a221f99c-60b0-469e-a824-14dd4663e770){: .image-popup}<br>

- pow를 높이면 테두리에 더 가까운 녀석들만 빛나게 된다<br>

## SmoothStep 적용해보기

``` cpp
float rim = 1.0 - dot(toEye, input.normalWorld);

if(useSmoothstep)
    rim = smoothstep(0.0, 1.0, rim);

color += ((rimColor * pow(rim, rimPower)) * rimStrength);
```

rim 값은 dot의 결과이기에<br>
0~1 사이의 값<br>
smoothstep 을 통하여 그 사이 값을 좀 더 '부드럽게' 보간할 수 있음<br>

- smoothstep을 켜지 않는 것<br>

[![Image](https://github.com/user-attachments/assets/b323ac02-bcb7-4a4b-a313-6c7c7af80b5a)](https://github.com/user-attachments/assets/b323ac02-bcb7-4a4b-a313-6c7c7af80b5a){: .image-popup}<br>

- smoothstep을 킨 것<br>

[![Image](https://github.com/user-attachments/assets/c0daee0d-070d-4700-92eb-e0f561a67fe6)](https://github.com/user-attachments/assets/c0daee0d-070d-4700-92eb-e0f561a67fe6){: .image-popup}<br>

여담으로 중앙의 하얀색 부분은<br>
specular 이다<br>