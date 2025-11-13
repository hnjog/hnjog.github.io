---
title: "Float Render Target"
date : "2025-11-13 12:00:00 +0900"
last_modified_at: "2025-11-13T12:00:00"
categories:
  - 그래픽스
  - DirectX
tags:
  - HDRI
  - Float Render Target
---

## PBR 적용 시 중요한 요소 중 하나

- 환경맵을 HDRI로 이용하는 것<br>
  - HDRI : High Dynamic Range Image<br>

- 조명을 HDRI로 이용하겠다는 뜻이며<br>
  IBL을 간접 조명(Ambient)으로 사용한다는 의미<br>

결국 렌더링 전체가 `HDR`을 사용해야 함<br>

- LDR은 0.0 ~ 1.0으로 색을 표현<br>
  - UNORM 0 ~ 255 역시 쉐이더에서 0.0 ~ 1.0 으로 변환하여 사용함<br>

## Float Render Target

[![Image](https://github.com/user-attachments/assets/dc5507ff-4bdd-4c26-b2cd-68cf933a2c36)](https://github.com/user-attachments/assets/dc5507ff-4bdd-4c26-b2cd-68cf933a2c36){: .image-popup}<br>

렌더링을 진행할 때<br>
원하는 장면(Scene)을 렌더링(Rasterizer)할 때<br>
픽셀 포맷이 Floating Point 여야 HDR로 렌더링이 가능<br>

- Float Render Target<br>
  : 더 넓은 밝기 범위를 담을 수 있는 고정밀도 렌더 타겟<br>

- 일반적인 8Bit로만 색을 표현하는 것은<br>
  Bloom, Exposure, Tone Mapping, Luminance 등을 계산할 때<br>
  데이터 손실이 발생<br>

- 상황에 따라 Render->FP Render Target 까지는 HDR 로 진행하나<br>
  ToneMapping 에 따라 LDR로 다시 바꾸어주는 경우도 존재<br>
  (모든 모니터와 기능이 HDR로 구성될 필요가 없을 수 있음)<br>
  - ToneMapping : HDR을 사람이 인식할 수 있는 LDR로 변환<br>
  - 밝기를 적절한 수준으로 조절하여 여러 보정과 색상에 대한 조절등을 실행<br>

[![Image](https://github.com/user-attachments/assets/d4d77ab6-1520-4e15-acf5-fc6ca88010df)](https://github.com/user-attachments/assets/d4d77ab6-1520-4e15-acf5-fc6ca88010df){: .image-popup}<br>

- 후처리 + 안티 앨리어싱 통합 파이프라인<br>

- MSAA Render Target : 3차원 물체 렌더링 결과<br>
  - MSAA는 하드웨어 가속을 받는게 좋음<br>
    하나의 픽셀에 여러 샘플을 가지는 형태<br>

- 위의 결과를 Resolve하여 한 픽셀에 하나의 결과를 가지는<br>
  FP Render Target 으로 변화<br>
  - 다만 Floating Point로 변화하려면 MSAA에서부터 FP 기반이여야 함<br>
    (동시에, 해당 단계에서 반드시 HDR이 아닐수도 있음)<br>
    (정확히는 Resolve의 Render Target의 포맷이 LDR이면 사라짐)<br>
  - 우리의 예제에선 이 타이밍에 Bloom 적용 (Post Process)<br>

- SDR/LDR Render Target<br>
  : 0~1 범위를 갖는 렌더타겟<br>
  - 다시 Post Process 사용 가능<br>
    
- UI 렌더링은 별도로 진행<br>
  (HDR이 필요하진 않으므로)<br>

요약하자면<br>
처음에 MSAA를 이용하여 HDR 렌더링을 진행하고<br>
이후로는 각 렌더타겟의 포맷팅에 맞춘다<br>

## 예제 - HDRI Pipeline

```cpp
void ExampleApp::Render() {
...
// MSAA로 Texture2DMS에 렌더링 된 결과를 Texture2D로 변환(Resolve)
m_context->ResolveSubresource(m_resolvedBuffer.Get(), 0,
                              m_floatBuffer.Get(), 0,
                              DXGI_FORMAT_R16G16B16A16_FLOAT);
}
```

MSAA는 픽셀 내부를 여러 샘플로 쪼개고<br>
경계선의 커버리지 비율을 구하여 계단 현상을 완화하는 방식<br>

- MSAA 용으로 렌더링된 텍스쳐들을 합쳐<br>
  하나의 Non Msaa 로 변환하는 과정<br>


```cpp
bool AppBase::InitDirect3D() {
...

  DXGI_SWAP_CHAIN_DESC sd;
  ZeroMemory(&sd, sizeof(sd));
  sd.BufferDesc.Width = m_screenWidth;
  sd.BufferDesc.Height = m_screenHeight;
  sd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
  sd.BufferCount = 2;
  sd.BufferDesc.RefreshRate.Numerator = 60;
  sd.BufferDesc.RefreshRate.Denominator = 1;
  sd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
  sd.OutputWindow = m_mainWindow;
  sd.Windowed = TRUE;
  sd.Flags =
      DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH; // allow full-screen switching
  // sd.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD; //ImGui 폰트가 두꺼워짐
  sd.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
  sd.SampleDesc.Count = 1; // _FLIP_은 MSAA 미지원
  sd.SampleDesc.Quality = 0;
...
}

```

- 포맷이 DXGI_FORMAT_R8G8B8A8_UNORM<br>
 : 플로팅 포인트가 아님<br>
  - 일반적인 모니터 등이 표시하는 포맷팅 방식<br>
  - 굳이 스왑 체인용으로 더 많은 용량의 포맷팅을 잡을 이유는 없음<br>
    -> 내부에서 HDR 렌더링에 톤매핑 까지 하여 화면에 뿌린다면 이런 방식이 정석적인 구조<br>
       (FP 포맷팅을 사용할 필요가 없다면 일반적인 포맷팅 사용을 권장)<br>

- 그렇기에 백버퍼용 Swapchain 부분에 MSAA 미지원 방식을 적용 가능<br>



```cpp
void AppBase::CreateBuffers() {
  ...
ThrowIfFailed(m_device->CheckMultisampleQualityLevels(
    DXGI_FORMAT_R16G16B16A16_FLOAT, 4, &m_numQualityLevels));

D3D11_TEXTURE2D_DESC desc;
backBuffer->GetDesc(&desc);
desc.MipLevels = desc.ArraySize = 1;
desc.BindFlags = D3D11_BIND_RENDER_TARGET | D3D11_BIND_SHADER_RESOURCE;
desc.Format = DXGI_FORMAT_R16G16B16A16_FLOAT;
desc.Usage = D3D11_USAGE_DEFAULT; // 스테이징 텍스춰로부터 복사 가능
desc.MiscFlags = 0;
desc.CPUAccessFlags = 0;
if (m_useMSAA && m_numQualityLevels) {
    desc.SampleDesc.Count = 4;
    desc.SampleDesc.Quality = m_numQualityLevels - 1;
} else {
    desc.SampleDesc.Count = 1;
    desc.SampleDesc.Quality = 0;
}

ThrowIfFailed(
    m_device->CreateTexture2D(&desc, NULL, m_floatBuffer.GetAddressOf()));
    ...
}
```

- MSAA 용 텍스쳐 버퍼 만들기<br>



