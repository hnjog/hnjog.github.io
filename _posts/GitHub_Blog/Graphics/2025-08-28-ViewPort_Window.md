---
title: "GUI 및 Window 변환에 따른 ViewPort 크기 바꾸기"
date : "2025-08-28 18:30:00 +0900"
last_modified_at: "2025-08-28T18:30:00"
categories:
  - Direct X
tags:
  - Viewport
---

## GUI 크기 변환에 따른 ViewPort 크기 바꾸기 1

```
D3D11_VIEWPORT m_screenViewport;

// Set the viewport
ZeroMemory(&m_screenViewport, sizeof(D3D11_VIEWPORT));
m_screenViewport.TopLeftX = 0;
m_screenViewport.TopLeftY = 0;
m_screenViewport.Width = float(m_screenWidth);
m_screenViewport.Height = float(m_screenHeight);
// m_screenViewport.Width = static_cast<float>(m_screenHeight);
m_screenViewport.MinDepth = 0.0f;
m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering

m_context->RSSetViewports(1, &m_screenViewport);
```

TopLeftX와 TopLeftY,<br>
Width, Height를 수정함으로서 <br>
Viewport의 크기를 조절 가능하다<br>

ex) 화면의 오른쪽 절반에만 출력하기<br>

```
// Set the viewport
ZeroMemory(&m_screenViewport, sizeof(D3D11_VIEWPORT));
m_screenViewport.TopLeftX = m_screenWidth / 2;
m_screenViewport.TopLeftY = 0;
m_screenViewport.Width = float(m_screenWidth / 2);
m_screenViewport.Height = float(m_screenHeight);
// m_screenViewport.Width = static_cast<float>(m_screenHeight);
m_screenViewport.MinDepth = 0.0f;
m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering

m_context->RSSetViewports(1, &m_screenViewport);
```

<img width="2225" height="1661" alt="Image" src="https://github.com/user-attachments/assets/078b4e4d-757d-43ec-babe-faf556a7af14" /><br>

물론 이 경우 aspect ratio는 맞지 않기에 다소 어색한 편<br>
그렇기에 AspectRatio도 Width의 비율에 맞게 수정해주면<br>
보다 자연스러워 진다<br>

```
float AppBase::GetAspectRatio() const {
    return float(m_screenWidth / 2) / m_screenHeight;
}
```

<img width="2229" height="1719" alt="Image" src="https://github.com/user-attachments/assets/5056fd37-feb8-41f5-9db3-0a9f7ac1659b" /><br>

### ImGui 유틸 함수들

- ImGui 창의 위치와 크기를 얻는 방법<br>

```
ImVec2 pos =ImGui::GetWindowPos(); // 위치
ImVec2 size =ImGui::GetWindowSize(); // 크기
```

- ImGui 창의 위치와 크기를 넣는 방법<br>

```
// loop 에 넣어 사실상 반쯤 고정시킬순 있음
ImGui::SetWindowPos(ImVec2(0,0));
ImGui::SetWindowSize(ImVec2(400, 400));
```

## GUI 크기 변환에 따른 ViewPort 크기 바꾸기 2 - 동적으로 변환

먼저 Init 시점에선 어차피 ImGui와 연동할 수 없으므로<br>
다시 원래대로 초기화한다<br>

이후 새로 그릴때부터 GUI의 위치와 크기를 고려해 ViewPort 조정<br>

먼저 이전의 ViewPort 설정 부분을 하나의 함수로 설정하도록 뺀 후<br>

```
void SetViewport() {
    // Set the viewport
    ZeroMemory(&m_screenViewport, sizeof(D3D11_VIEWPORT));
    m_screenViewport.TopLeftX = 0;
    m_screenViewport.TopLeftY = 0;
    m_screenViewport.Width = float(m_screenWidth);
    m_screenViewport.Height = float(m_screenHeight);
    // m_screenViewport.Width = static_cast<float>(m_screenHeight);
    m_screenViewport.MinDepth = 0.0f;
    m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering

    m_context->RSSetViewports(1, &m_screenViewport);
}
```

GUI width 관련한 변수를 생성한 후<br>
다음과 같이 수정한다<br>

```
void SetViewport() {
    // Set the viewport
    ZeroMemory(&m_screenViewport, sizeof(D3D11_VIEWPORT));
    m_screenViewport.TopLeftX = m_guiWidth;
    m_screenViewport.TopLeftY = 0;
    m_screenViewport.Width = float(m_screenWidth - m_guiWidth);
    m_screenViewport.Height = float(m_screenHeight);
    // m_screenViewport.Width = static_cast<float>(m_screenHeight);
    m_screenViewport.MinDepth = 0.0f;
    m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering

    m_context->RSSetViewports(1, &m_screenViewport);
}

float AppBase::GetAspectRatio() const {
    return float(m_screenWidth - m_guiWidth) / m_screenHeight;
}

UpdateGUI()
{
  while()
  {
    ...
    
    m_guiWidth = ImGui::GetWindowSize().x;
  }
}

```

이후 마지막으로 이렇게 변경한 m_guiWidth가 적용된<br>
ViewPort를 다시 RSSetViewport() 하도록 SetViewport()를 호출해준다!<br>

```
void ExampleApp::Update(){
  ...
  m_aspect = AppBase::GetAspectRatio(); // <- GUI에서 조절
  ...(Projection)
}

void ExampleApp::Render() {
  ...
  SetViewport();
}
```

### 결과

<img width="2229" height="1711" alt="Image" src="https://github.com/user-attachments/assets/1f0f4c55-bb77-4bba-8d07-d346beeb0d4e" /><br>

<img width="2223" height="1717" alt="Image" src="https://github.com/user-attachments/assets/548dd3ad-3632-4858-96b9-a3af9de65b84" /><br>

GUI 넓이에 따라 ViewPort가 동적으로 수정된다<br>

### SetViewport() 값이 같은데 바꿀 필요가 없지 않나?

그렇기에 함수에 지역 정적 변수를 사용하여<br>
같은 경우는 따로 RSSetViewports를 호출하지 않도록 할 수 있음<br>
(사소(매우)한 팁)<br>

```
void SetViewport(){
  static int Previous = m_guiWidth;

  if (Previous == m_guiWidth)
      return;

  ...
}
```

## Window 크기 변환에 따른 Viewport 바꾸기

Windows 창 크기 변환 시<br>
WM_SIZE 메시지가 발생<br>
(처음 실행하거나, 사이즈 변경시 발생하는 메시지)<br>

```
LRESULT AppBase::MsgProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {

    if (ImGui_ImplWin32_WndProcHandler(hwnd, msg, wParam, lParam))
        return true;

    switch (msg) {
    case WM_SIZE:
        // Reset and resize swapchain
        // std::cout << (UINT)LOWORD(lParam) << " " << (UINT)HIWORD(lParam)
        //          << std::endl;

        if (m_swapChain) { // 처음 실행이 아닌지 확인 (Swapchain 존재 확인)

            m_screenWidth = int(LOWORD(lParam));
            m_screenHeight = int(HIWORD(lParam));
            m_guiWidth = 0;

            m_renderTargetView.Reset();
            m_swapChain->ResizeBuffers(0, // 현재 개수 유지
                                       (UINT)LOWORD(lParam), // 해상도 변경
                                       (UINT)HIWORD(lParam),
                                       DXGI_FORMAT_UNKNOWN, // 현재 포맷 유지
                                       0);
            CreateRenderTargetView();
            CreateDepthBuffer();
            SetViewport();
        }
      ...
    }
```

따라서 그 중 '첫 실행'을 제외한 경우<br>
RTV와 Depth/Stencil 버퍼를 재 생성(화면 크기를 따름)<br>
이후 Viewport를 다시 세팅해준다<br>

```
bool AppBase::CreateRenderTargetView() {

    ComPtr<ID3D11Texture2D> backBuffer;
    m_swapChain->GetBuffer(0, IID_PPV_ARGS(backBuffer.GetAddressOf()));
    if (backBuffer) {
        m_device->CreateRenderTargetView(backBuffer.Get(), NULL,
                                         m_renderTargetView.GetAddressOf());
    } else {
        std::cout << "CreateRenderTargetView() failed." << std::endl;
        return false;
    }

    return true;
}

bool AppBase::CreateDepthBuffer() {
    D3D11_TEXTURE2D_DESC depthStencilBufferDesc;
    depthStencilBufferDesc.Width = m_screenWidth;
    depthStencilBufferDesc.Height = m_screenHeight;
    depthStencilBufferDesc.MipLevels = 1;
    depthStencilBufferDesc.ArraySize = 1;
    depthStencilBufferDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
    if (numQualityLevels > 0) {
        depthStencilBufferDesc.SampleDesc.Count = 4; // how many multisamples
        depthStencilBufferDesc.SampleDesc.Quality = numQualityLevels - 1;
    } else {
        depthStencilBufferDesc.SampleDesc.Count = 1; // how many multisamples
        depthStencilBufferDesc.SampleDesc.Quality = 0;
    }
    depthStencilBufferDesc.Usage = D3D11_USAGE_DEFAULT;
    depthStencilBufferDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
    depthStencilBufferDesc.CPUAccessFlags = 0;
    depthStencilBufferDesc.MiscFlags = 0;

    if (FAILED(m_device->CreateTexture2D(
            &depthStencilBufferDesc, 0, m_depthStencilBuffer.GetAddressOf()))) {
        std::cout << "CreateTexture2D() failed." << std::endl;
    }
    if (FAILED(m_device->CreateDepthStencilView(m_depthStencilBuffer.Get(), 0,
                                                &m_depthStencilView))) {
        std::cout << "CreateDepthStencilView() failed." << std::endl;
    }
    return true;
}
```

### 결과

<img width="3833" height="2025" alt="Image" src="https://github.com/user-attachments/assets/7722fcd8-e36d-440d-b236-0c20304ac343" /><br>
<img width="3829" height="2077" alt="Image" src="https://github.com/user-attachments/assets/eb9fb504-2a0a-4b77-b7fd-f0cfb4dfba57" /><br>
<img width="3819" height="2077" alt="Image" src="https://github.com/user-attachments/assets/e97805d9-2452-4691-ae8f-9306ab6e17a9" /><br>

Window의 크기에 따라 ViewPort가 동적으로 수정된다<br>