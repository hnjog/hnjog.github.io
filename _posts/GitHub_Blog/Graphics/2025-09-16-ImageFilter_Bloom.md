---
title: "Image Filter & Bloom"
date : "2025-09-16 16:00:00 +0900"
last_modified_at: "2025-09-16T16:00:00"
categories:
  - Direct X
  - Graphics
tags:
  - Image Filter
  - Post Process
  - Bloom
---

## Image Filter

- 렌더 타겟으로 이미 그린 '결과'를 텍스쳐로 불러들여<br>
  화면 위에 '덮어씌우는' 방식으로 보정한다<br>
  (후처리 - Post Process)<br>

일반적인 방식<br>
1. 원래 장면을 RTV로 렌더링<br>
2. RTV->SRV 변환하기<br>
  - 렌더링 결과 텍스쳐를 SRV로 파이프라인에 바인딩<br>
3. 픽셀 쉐이더에서 해당 SRV를 샘플링하여 Bloor 와 같은 효과를 적용하기<br>
4. 결과를 다시 렌더 타깃(스왑 체인 백 버퍼 등)에 출력하고 화면 표시<br>

### 예시용 이미지 필터 코드 - Init 부분

```cpp
D3D11_TEXTURE2D_DESC txtDesc;
ZeroMemory(&txtDesc, sizeof(txtDesc));
txtDesc.Width = width;
txtDesc.Height = height;
txtDesc.MipLevels = txtDesc.ArraySize = 1;
txtDesc.Format = DXGI_FORMAT_R32G32B32A32_FLOAT; //  이미지 처리용도
txtDesc.SampleDesc.Count = 1;
txtDesc.Usage = D3D11_USAGE_DEFAULT;
// GPU가 내부적으로 렌더링 결과를 텍스쳐 메모리에
// 이미지로 저장을 한다 -> 이것을 스왑 체인이 백 버퍼와 프론트 버퍼를 스왑시켜 그리는 것
txtDesc.BindFlags = D3D11_BIND_SHADER_RESOURCE | 
                    D3D11_BIND_RENDER_TARGET |
                    D3D11_BIND_UNORDERED_ACCESS; // 없어도 동작? -> 픽셀 쉐이더에서 읽고 RT에서 쓸것이라면
txtDesc.MiscFlags = 0;
txtDesc.CPUAccessFlags = 0;

D3D11_RENDER_TARGET_VIEW_DESC viewDesc;
viewDesc.Format = txtDesc.Format;
viewDesc.ViewDimension = D3D11_RTV_DIMENSION_TEXTURE2D;
viewDesc.Texture2D.MipSlice = 0;

device->CreateTexture2D(&txtDesc, NULL, texture.GetAddressOf());
device->CreateRenderTargetView(texture.Get(), &viewDesc,
                               m_renderTargetView.GetAddressOf());
device->CreateShaderResourceView(texture.Get(), nullptr,
                                 m_shaderResourceView.GetAddressOf());
```

- 같은 텍스쳐를 RTV와 SRV로 모두 만들어줌<br>

- 컴퓨트 쉐이딩을 사용하는 방법도 존재<br>
  지금은 사용하지 않으나<br>
  이미지의 각 '픽셀'의 위치만 알면<br>
  배열처럼 각각의 픽셀의 값을 가져올 수 있음<br>

```cpp
m_pixelConstData.dx = 1.0f / width;
m_pixelConstData.dy = 1.0f / height;

D3D11Utils::CreateConstantBuffer(device, m_pixelConstData,
                                 m_mesh->pixelConstantBuffer);
```

(나중에 가우시안 블러 구현할때 사용할 예정)<br>

### SRV vs RTV

| 구분           | SRV (Shader Resource View)                        | RTV (Render Target View)                                |
| ------------ | ------------------------------------------------- | ------------------------------------------------------- |
| **주요 용도**    | **셰이더에서 읽기 전용** 리소스. (픽셀/컴퓨트 셰이더 등에서 텍스처 샘플링)     | **렌더링 출력(쓰기)** 대상으로 사용. (OM 단계에서 Color Buffer)          |
| **바인딩 슬롯**   | `PSSetShaderResources` 등 각 셰이더 스테이지의 SRV 슬롯에 바인딩. | `OMSetRenderTargets` 로 Output-Merger 단계에 바인딩.           |
| **읽기/쓰기 권한** | **읽기 전용** (DirectX11에서는 동시 읽기/쓰기가 불가. 쓰기하려면 UAV). | **쓰기 전용** (그 프레임 동안 읽기 금지, 다른 뷰로 읽을 땐 resolve/copy 필요). |
| **대표 사용 예**  | - 텍스처 맵핑                                          |                                                         |

다만, 기본적으로 둘다<br>
Texutre2D / Buffer 등의 리소스 객체인<br>
'뷰'(해석)이며 실제 메모리를 소유하진 않음<br>

추가로 알아둘 점<br>
- 파이프라인 Stage에서 GPU가 데이터를 읽거나 쓸 수 있도록 함<br>
- 같은 텍스쳐라도 여러 뷰로 동시 생성이 가능<br>

### 예제 - Filter 생성

```cpp
void ExampleApp::BuildFilters() {

    m_filters.clear();

    // shared_ptr 이기에 make_shared
    // wstring 요구하기에 L"" 넘겨줌
    auto copyFilter =
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                                 m_screenWidth, m_screenHeight);

    // 현재 우리가 AppBase 쪽에서
    // 스왑 체인 백버퍼를 RTV,SRV로 만들어두고,
    // 각각 m_renderTargetView, m_shaderResourceView로 저장해둠
    // 그렇기에 m_shaderResourceView 에 '렌더링'된 결과가 저장되기에
    // 이걸 필터에 넘겨주면 된다
    
    // 어차피 처음에 Initalize 부분에서 '기본 렌더 타겟' 자체는
    // 생성을 하고, 렌더타겟에 넣어두기에 Render 호출해도 안터짐
    copyFilter->SetShaderResources({this->m_shaderResourceView});
    m_filters.push_back(copyFilter);

    // 위쪽에서 출력된 결과에서 쉐이딩을 한 번 돌린 후,
    // 우리가 추가적으로 필터를 돌리고 렌더 타겟을 마지막으로 설정함
    // (읽기/쓰기 분리, 또한 나중에 추가 효과 등에 대한 적용을 위해)
    auto finalFilter =
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                                 m_screenWidth, m_screenHeight);

    finalFilter->SetShaderResources({m_filters.back()->m_shaderResourceView});
    // 다만 이거 호출하면 이전에 Default로 넣어둔 녀석은 사라짐
    finalFilter->SetRenderTargets({this->m_renderTargetView});
    m_filters.push_back(finalFilter);
}
```

- copy Filter를 shared_ptr로 관리하여 메모리 쪽 관리를 일임<br>

- 기본적으로 내부에서 RTV를 자체적으로 생성하여 사용중<br>
  (그래서 screen 데이터를 받는다)<br>
  (나중에 SetRTV 호출되면 그걸로 바뀌긴 하지만)<br>

- srv를 필터에 넘겨주고, 해당 적용한 결과 역시 srv에 담기게 됨<br>

- 나중에 최종 반환 받는 필터를 통하여 우리가 가지는 RTV에 쓰도록 함<br>
  (스왑 체인 백 버퍼를 통해 넘겨 받음)<br>
  - 읽기와 쓰기를 분리하기 위함(SRV로 '읽으면서' 동시에 RTV로 쓰려고 하면 안되기에)<br>
  - 나중에 여러 효과를 동시에 적용하기 위함<br>

## Post Process 효과 방식들

### Down/Up Sampling

[![Image](https://github.com/user-attachments/assets/708a88ce-e002-4fe2-977e-3f3dd0afcb4b)](https://github.com/user-attachments/assets/708a88ce-e002-4fe2-977e-3f3dd0afcb4b){: .image-popup}<br>

```cpp
auto copyFilter =
    make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                             m_screenWidth / 32, m_screenHeight / 32); // 32로 나누어주어 마치 화질이 매우 떨어진 사진처럼 보인다

auto finalFilter =
    make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                             m_screenWidth, m_screenHeight); // 출력 시에는                  
```

- 후처리 적용용 필터는 사이즈를 32로 나누어 받은 후<br>
  그대로 출력<br>
  (첫 필터에서 '아주 작게 만든'곳에 uv 좌표들이 그대로 들어가게 된다)<br>
  (몇몇 세부 정보들을 잃어버림)<br>

- 이후 최종 출력 필터에서는 원래 해상도로 다시 바꾼다<br>
  (이것마저 /32 유지하면 화면이 아주 검게 나오거나 화면 일부에 출력)<br>
  (매우 작아진 '텍스쳐'를 다시 키움에 따라 하나의 '텍셀'이 블록화됨)<br>

### 블러(Blur) vs 블룸(Bloom)?

- Blur : 화면/텍스쳐의 '디테일'을 줄이는 효과<br>
- Bloom : 화면/텍스쳐의 '밝은 부분'만을 추출하고 '블러'하여, '가산 합성'하여 '빛 번짐'을 만듦<br>

Bloom 안에 Blur 가 들어가는 편<br>

#### 요약 표

| 항목       | Blur                   | Bloom                                          |
| -------- | ---------------------- | ---------------------------------------------- |
| 목적       | 전체 이미지 소프트닝, 노이즈/계단 완화 | 밝은 영역 주변의 광 번짐(렌즈/망막 산란 감성)                    |
| 적용 범위    | 보통 **전체 프레임**          | **밝기 임계값 이상**의 마스크에만                           |
| 핵심 단계    | 컨볼루션(가우시안/박스/양방향 등)    | **(1) 밝은영역 추출 → (2) 다중 스케일 블러 → (3) 원본에 가산합성** |
| 결과       | 전반적으로 흐릿               | 밝은 곳만 후광/헤일로, 나머지 디테일 유지                       |
| 파이프라인 위치 | 언제든(주로 후처리 중간)         | 보통 **HDR 톤매핑 이전**에 만들고 최종 합성                   |
| 대표 파라미터  | 커널 반경/표준편차, 패스 수       | 임계값/소프트니, 강도, 다중해상도 스케일, 업샘플 블렌드               |

### 가우시안 블러

```cpp
// 가우시안 블러 적용
for (int i = 0; i < 10; i++)
{
    auto &prevResource = m_filters.back()->m_shaderResourceView;
    m_filters.push_back(make_shared<ImageFilter>(
                            m_device, m_context, L"Sampling", L"BlurX",
                            m_screenWidth, m_screenHeight));
    m_filters.back()->SetShaderResources({prevResource});

    auto &prevResource2 = m_filters.back()->m_shaderResourceView;
    m_filters.push_back(
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"BlurY",
                                 m_screenWidth, m_screenHeight));
    m_filters.back()->SetShaderResources({prevResource2});
}
```

각 2개의 Pixel Shader에 각각의 좌표에 따라 구현<br>

```cpp
BlurX

static const float weights[5] = { 0.0545, 0.2442, 0.4026, 0.2442, 0.0545 };

float4 main(SamplingPixelShaderInput input) : SV_TARGET
{
    float4 color = float4(0.0, 0.0, 0.0, 1.0);
    
    int i;
    for (i = 0; i < 5;i++)
    {
        color += weights[i] * g_texture0.Sample(g_sampler, input.texcoord + float2(dx, 0.0) * float(i - 2));
    }
    
    return color;
}
BlurY
float4 main(SamplingPixelShaderInput input) : SV_TARGET
{
    float4 color = float4(0.0, 0.0, 0.0, 1.0);
    
    int i;
    for (i = 0; i < 5; i++)
    {
        color += weights[i] * g_texture0.Sample(g_sampler, input.texcoord + float2(0.0, dy) * float(i - 2));
    }
    
    return color;
}
```

- 두 PixelShader를 하나로 합칠수도 있음<br>
- 합이 1인 Weights를 통해 옆 픽셀들의 수치를 일부 가져와 현재 픽셀 값에 적용<br>
- -2 ~ 2 사이의 값을 이용하여 다른 픽셀의 위치값을 가져와 사용<br>

[![Image](https://github.com/user-attachments/assets/75d454ed-5518-417e-8b6a-00bec7f91283)](https://github.com/user-attachments/assets/75d454ed-5518-417e-8b6a-00bec7f91283){: .image-popup}<br>

- 전체적으로 살짝 부드러워진 모습<br>
- 동시에 약간은 '날카로운 표현'은 없어진 듯하다<br>

## Bloom 구현하기

지정된 평균 값 이하의 색을 제거한 후,<br>
'밝은 부분'만을 얻은 후, 이것을 원본에 합쳐주는 방식<br>

```cpp
auto finalFilter =
    make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                             m_screenWidth, m_screenHeight);

finalFilter->SetShaderResources({m_filters.back()->m_shaderResourceView});
// 다만 이거 호출하면 이전에 Default로 넣어둔 녀석은 사라짐
finalFilter->SetRenderTargets({this->m_renderTargetView});
finalFilter->m_pixelConstData.threshold = 0.0f;
finalFilter->m_pixelConstData.strength = 1.0f;
finalFilter->UpdateConstantBuffers(m_device, m_context);
m_filters.push_back(finalFilter);
```

일단 CopyFilter 쪽의 Sampling PS를 수정할 예정이니<br>

먼저 최종 필터 쪽에서는 threshold와 strength를 고정시켜<br>
Bloom 효과에 영향을 받지 않도록 수정한다<br>

그리고 Update 쪽에서 front 쪽에 집중하도록 수정<br>

```cpp
if (m_dirtyflag) {
    m_filters.front()->m_pixelConstData.threshold = m_threshold;
    m_filters.front()->m_pixelConstData.strength = m_strength;
    m_filters.front()->UpdateConstantBuffers(m_device,m_context);

    m_dirtyflag = 0;
}
```

이후 PixelShader 쪽에서 블룸 효과를 구현해준다<br>

```cpp
Texture2D g_texture0 : register(t0);
SamplerState g_sampler : register(s0);

cbuffer SamplingPixelConstantData : register(b0)
{
    float dx;
    float dy;
    float threshold;
    float strength;
    float4 options;
};

struct SamplingPixelShaderInput
{
    float4 position : SV_POSITION;
    float2 texcoord : TEXCOORD;
};

float4 main(SamplingPixelShaderInput input) : SV_TARGET
{
    float3 color = g_texture0.Sample(g_sampler, input.texcoord).xyz;
    float l = (color.x + color.y + color.z) / 3.0;
    
    return l > threshold ? float4(color * strength, 1.0f) :
    float4(0.0f, 0.0f, 0.0f, 1.0f);
}
```

[![Image](https://github.com/user-attachments/assets/95dd63c8-7ca3-4853-9ee2-2c742f91f87a)](https://github.com/user-attachments/assets/95dd63c8-7ca3-4853-9ee2-2c742f91f87a){: .image-popup}<br>

저 검은 부분은 수치가 낮으니 첫 필터에서 완전히 0.0으로 바꿔버려 발생한 것<br>

그 외에는 좀더 밝아진 느낌을 받을 수 있다<br>
(기존 밝은 부분 + 신규 strength 수치를 원본 이미지에 더함)<br>

### 더 은은한 효과를 표현할 순 없을까?

```cpp
auto copyFilter =
    make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                             m_screenWidth / 32, m_screenHeight / 32);

// 현재 우리가 AppBase 쪽에서
// 스왑 체인 백버퍼를 RTV,SRV로 만들어두고,
// 각각 m_renderTargetView, m_shaderResourceView로 저장해둠
// 그렇기에 m_shaderResourceView 에 '렌더링'된 결과가 저장되기에
// 이걸 필터에 넘겨주면 된다

// 어차피 처음에 Initalize 부분에서 '기본 렌더 타겟' 자체는
// 생성을 하고, 렌더타겟에 넣어두기에 Render 호출해도 안터짐
copyFilter->SetShaderResources({this->m_shaderResourceView});
m_filters.push_back(copyFilter);

// 가우시안 블러 적용
for (int i = 0; i < 25; i++)
{
    auto &prevResource = m_filters.back()->m_shaderResourceView;
    m_filters.push_back(make_shared<ImageFilter>(
                            m_device, m_context, L"Sampling", L"BlurX",
                            m_screenWidth / 32, m_screenHeight / 32));
    m_filters.back()->SetShaderResources({prevResource});

    auto &prevResource2 = m_filters.back()->m_shaderResourceView;
    m_filters.push_back(
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"BlurY",
                                 m_screenWidth / 32, m_screenHeight / 32));
    m_filters.back()->SetShaderResources({prevResource2});
}
```

이전에 배운 다운 샘플링을 각각의 필터에 적용해주면 된다<br>
다소 정밀적이지는 않지만, '낮은 해상도'에서 진행하기에<br>
Blur가 한번에 더 많이 진행함<br>
-> 그렇기에 Blurring 자체는 '낮은 해상도'에서 진행하는 편<br>

[![Image](https://github.com/user-attachments/assets/429c02de-4eaf-4a5e-b074-5924c91641b9)](https://github.com/user-attachments/assets/429c02de-4eaf-4a5e-b074-5924c91641b9){: .image-popup}<br>

- 완전 뿌옇게 변했다<br>

## 최종 Bloom 구현 결과

애초에 마지막 Pixel Shader를 별도로 구현해줘야 한다<br>

```cpp
CombinePixelShader.hlsl

Texture2D g_texture0 : register(t0);
Texture2D g_texture1 : register(t1);
SamplerState g_sampler : register(s0);

cbuffer SamplingPixelConstantData : register(b0)
{
    float dx;
    float dy;
    float threshold;
    float strength;
    float4 options;
};

struct SamplingPixelShaderInput
{
    float4 position : SV_POSITION;
    float2 texcoord : TEXCOORD;
};

float4 main(SamplingPixelShaderInput input) : SV_TARGET
{
    float3 Bloom = g_texture0.Sample(g_sampler, input.texcoord).xyz;
    Bloom *= strength;
    float3 color = g_texture1.Sample(g_sampler, input.texcoord).xyz + Bloom;
    return float4(color,1.0f);
}
```

- 0에 우리가 만든 'bloom'이 들어가게 되고<br>
  1에 원본이 들어갈 예정이다<br>

이후 Build 쪽에서 교체해주기<br>


```cpp
void ExampleApp::BuildFilters() {

    m_filters.clear();

    // shared_ptr 이기에 make_shared
    // wstring 요구하기에 L"" 넘겨줌
    auto copyFilter =
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling",
                                 m_screenWidth, m_screenHeight);

    // 현재 우리가 AppBase 쪽에서
    // 스왑 체인 백버퍼를 RTV,SRV로 만들어두고,
    // 각각 m_renderTargetView, m_shaderResourceView로 저장해둠
    // 그렇기에 m_shaderResourceView 에 '렌더링'된 결과가 저장되기에
    // 이걸 필터에 넘겨주면 된다
    
    // 어차피 처음에 Initalize 부분에서 '기본 렌더 타겟' 자체는
    // 생성을 하고, 렌더타겟에 넣어두기에 Render 호출해도 안터짐
    copyFilter->SetShaderResources({this->m_shaderResourceView});
    m_filters.push_back(copyFilter);

    auto downFilter =
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Sampling", m_screenWidth / m_down,
        m_screenHeight / m_down);

    downFilter->SetShaderResources({this->m_shaderResourceView});
    m_filters.push_back(downFilter);

    // 가우시안 블러 적용
    for (int i = 0; i < 5; i++)
    {
        auto &prevResource = m_filters.back()->m_shaderResourceView;
        m_filters.push_back(make_shared<ImageFilter>(
                                m_device, m_context, L"Sampling", L"BlurX",
                                m_screenWidth / m_down, m_screenHeight / m_down));
        m_filters.back()->SetShaderResources({prevResource});

        auto &prevResource2 = m_filters.back()->m_shaderResourceView;
        m_filters.push_back(
            make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"BlurY", m_screenWidth / m_down,
            m_screenHeight / m_down));
        m_filters.back()->SetShaderResources({prevResource2});
    }

    // 위쪽에서 출력된 결과에서 쉐이딩을 한 번 돌린 후,
    // 우리가 추가적으로 필터를 돌리고 렌더 타겟을 마지막으로 설정함
    // (읽기/쓰기 분리, 또한 나중에 추가 효과 등에 대한 적용을 위해)
    auto finalFilter =
        make_shared<ImageFilter>(m_device, m_context, L"Sampling", L"Combine",
                                 m_screenWidth, m_screenHeight);

    finalFilter->SetShaderResources(
        {m_filters.back()->m_shaderResourceView, m_filters.front()->m_shaderResourceView});
    // 다만 이거 호출하면 이전에 Default로 넣어둔 녀석은 사라짐
    finalFilter->SetRenderTargets({this->m_renderTargetView});
    m_filters.push_back(finalFilter);
}
```

- 원본용을 카피하기 위하여 맨 처음에 '카피'용 이미지 필터를 냄겨둔다<br>
  (m_filters.front()->m_shaderResourceView 대신<br>
  this->m_shaderResourceView 를 사용하면 사실상 '동일한 텍스쳐'를<br>
  읽고 쓰게 되니 주의 -> m_shaderResourceView를 읽은 후<br>
  m_renderTargetView로 '쓰려'하고 있음)<br>
  (이상한 결과가 발생할 가능성이 높음)<br>
  (copy용 이미지 필터에서 새로운 텍스쳐를 만들고 그를 가져오는게 안전함)<br>

- 이후 다운 샘플링을 해준다 (m_down : 16)<br>

- 마지막으로 Compute 쉐이더 쪽에서 합쳐주고 출력용 RTV에 값을 출력<br>

```cpp
if (m_dirtyflag) {
    m_filters[1]->m_pixelConstData.threshold = m_threshold; // down filter
    m_filters[1]->UpdateConstantBuffers(m_device, m_context);
    m_filters.back()->m_pixelConstData.strength = m_strength;
    m_filters.back()->UpdateConstantBuffers(m_device, m_context);

    m_dirtyflag = 0;
}
```

- Update 쪽의 filter 쪽도 바꿔준다<br>

[![Image](https://github.com/user-attachments/assets/091c5d2b-ee2e-47fa-b63e-34e0629a4cee)](https://github.com/user-attachments/assets/091c5d2b-ee2e-47fa-b63e-34e0629a4cee){: .image-popup}<br>

블룸을 적절히 사용해주니 때깔이 아주 예뻐진다<br>

### TMI - 앨리어싱을 줄이는 법<br>

[![Image](https://github.com/user-attachments/assets/77a969ae-a050-45f1-9a78-9ea06b71efca)](https://github.com/user-attachments/assets/77a969ae-a050-45f1-9a78-9ea06b71efca){: .image-popup}<br>

지금도 충분히 예쁘게 나오지만 Bloom 효과를 높이면<br>
종종 '앨리어싱'이 보이는 상황이 나온다<br>

[![Image](https://github.com/user-attachments/assets/eb78d721-66e3-4762-941b-7ffd17d84433)](https://github.com/user-attachments/assets/eb78d721-66e3-4762-941b-7ffd17d84433){: .image-popup}<br>

그렇기에 '해상도'를 낮출 때, '단계별로 낮추게 되면<br>
낮은 해상도에서 Smoothing을 하고 다시 높은 해상도에서<br>
해상도를 진행하면 된다<br>

```cpp
for (int down = 2; down <= m_down; down *= 2)
{
    auto downFilter = make_shared<ImageFilter>(
        m_device, m_context, L"Sampling", L"Sampling", m_screenWidth / down,
        m_screenHeight / down);

    if (down == 2)
    {
        downFilter->SetShaderResources({this->m_shaderResourceView});
    }
    else
    {
        downFilter->SetShaderResources(
            {m_filters.back()->m_shaderResourceView});
    }
    
    m_filters.push_back(downFilter);
}

for (int down = m_down; down >= 1; down /= 2)
{
    // 가우시안 블러 적용
    for (int i = 0; i < 5; i++) {
        auto &prevResource = m_filters.back()->m_shaderResourceView;
        m_filters.push_back(make_shared<ImageFilter>(
            m_device, m_context, L"Sampling", L"BlurX",
            m_screenWidth / down, m_screenHeight / down));
        m_filters.back()->SetShaderResources({prevResource});

        auto &prevResource2 = m_filters.back()->m_shaderResourceView;
        m_filters.push_back(make_shared<ImageFilter>(
            m_device, m_context, L"Sampling", L"BlurY",
            m_screenWidth / down, m_screenHeight / down));
        m_filters.back()->SetShaderResources({prevResource2});
    }

    if (down > 1) {
        auto upFilter = make_shared<ImageFilter>(
            m_device, m_context, L"Sampling", L"Sampling",
            m_screenWidth / down * 2, m_screenHeight / down * 2);

        upFilter->SetShaderResources({m_filters.back()->m_shaderResourceView});
        upFilter->m_pixelConstData.threshold = 0.0f;
        upFilter->UpdateConstantBuffers(m_device, m_context);
        m_filters.push_back(upFilter);
    }
}
```

다운 샘플링을 진행할 때 2배씩 낮아지며,<br>
동시에 블러를 진행할때는 2배씩 올라가며<br>
안티 앨리어싱을 적용<br>

- 결과<br>

[![Image](https://github.com/user-attachments/assets/6b9ffcb7-299b-40fb-a4e9-6f2e9787a4d9)](https://github.com/user-attachments/assets/6b9ffcb7-299b-40fb-a4e9-6f2e9787a4d9){: .image-popup}<br>

Stength를 높게 주더라도 앨리어싱이 잘 일어나지 않는다!<br>