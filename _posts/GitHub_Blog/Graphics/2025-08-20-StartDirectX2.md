---
title: "Direct X InitDirect3D"
last_modified_at: "2025-08-20T16:30:00"
categories:
  - Direct X
tags:
  - Direct 3D
---

## 들어가기 앞서
내용 자체는 그렇게 어렵지 않았으나<br>
코드 + 설명을 하다보니 포스팅 크기가 너무 길어지는 듯 하여<br>
분할하여 올릴 예정이다<br>

## AppBase::InitDirect3D()
Direct3D와 관련된 요소들을 생성한다<br>

각 요소들에 대한 간략한 개념(구조체/클래스)<br>

#### Device (ID3D11Device)<br>

- GPU 리소스(버퍼, 텍스쳐, 쉐이더 등)을 생성하는 '팩토리' 역할<br>
- GPU와 직접 연결된 논리적인 장치(디바이스)<br>
- 리소스를 만드는 위주의 역할 (draw 등은 context에서)<br>
  (CreateBuffer, CreateTexture2D 등의 리소스 생성 함수)<br>

#### Context (ID3D11DeviceContext)

- 파이프라인 상태를 설정하며, 실제로 draw/ dispatch 명령을 내린다<br>
  (IASetVertexBuffers,VSSetShader,RSSetState,DrawIndexed 등)<br>
- Device가 만든 리소스를 파이프라인에 바인딩하여 GPU에서 실행시킴<br>
- 일반적으로는 Immediate Context(실시간 실행)을 사용<br>
  멀티 스레드 환경에선 Deferred Context(명령 리스트에 기록 후 실행 타이밍에 실행)도 고려 가능<br>

#### SwapChain (IDXGISwapChain)

- 프레임 버퍼(Back Buffer) 와 화면 출력(Window Surface) 사이를 연결하는 인터페이스<br>
  (버퍼 관리 + 화면 출력 + Window 와의 호환)
- 더블 / 트리플 버퍼링 지원<br>
- 렌더링 완료된 이미지를 Present() 호출로 실제 윈도우에 출력<br>

  - OM 단계 완료까지가 GPU 파이프 라인의 마무리이며<br>
    이것이 RenderTargetView에 저장 (Back Buffer)<br>
  - 그러나 화면에는 아직 보이지 않는다<br>
    (Back Buffer 데이터는 GPU에 존재)<br>
  - SwapChain->Present() 호출을 통하여 OS/드라이버가<br>
    BackBuffer와 FrontBuffer를 '교체'한 후 출력<br>

#### RenderTargetView(ID3D11RenderTargetView)

- GPU가 렌더링 결과를 기록할 수 있도록 만드는 '뷰'(View)<br>
- SwapChain의 BackBuffer나 Texture를 렌더링 대상으로 사용할 때 필요<br>
- RTV를 통해 OMSetRenderTargets로 출력 타겟 지정<br>
  (BackBuffer -> RTV -> OMSetRenderTargets()로 <br>
   파이프라인의 Output Merge 단계에 설정)<br>

#### ViewPort (D3D11_VIEWPORT)

- 렌더링 결과를 어느 화면 영역(ViewPort)에 그릴지 결정<br>
- NDC(-1 ~ 1 좌표)를 실제 픽셀 좌표로 변환<br>
- 윈도우 크기와 동일하게 설정하는 편이지만<br>
  일부만 사용도 가능(Split-screen 등)<br>

#### RasterizerState (ID3D11RasterizerState)

- RS(래스터화) 단계의 동작 정의<br>
- 삼각형이 픽셀로 변환되는 과정에서 어떻게 처리할지 결정<br>
- Culling Mode(앞/뒷 면 제거), Fill Mode, 깊이 관련 등<br>
- Device -> CreateRasterizerState() , Context -> RSSetState()를<br>
  통하여 생성 및 설정<br>
  (세부 설정은 D3D11_RASTERIZER_DESC 를 통해 설정한다)<br>

### 요소들의 간략한 관계도

```
(ID3D11Device) ← 리소스 생성
        │
        ▼
(ID3D11DeviceContext) ← 리소스 바인딩 + Draw 호출
        │
        ├─ (ID3D11RasterizerState, Viewport) 등 파이프라인 상태 설정
        ├─ (ID3D11RenderTargetView) 에 렌더링 출력
        ▼
(IDXGISwapChain) ← BackBuffer 관리 → Present() → 화면 출력
```


## AppBase::InitDirect3D() 1. Device, Context 생성 부분

```
bool AppBase::InitDirect3D() {
    // 이 예제는 Intel 내장 그래픽스 칩으로 실행을 확인하였습니다.
    // (LG 그램, 17Z90n, Intel Iris Plus Graphics)
    // 만약 그래픽스 카드 호환성 문제로 D3D11CreateDevice()가 실패하는 경우에는
    // D3D_DRIVER_TYPE_HARDWARE 대신 D3D_DRIVER_TYPE_WARP 사용해보세요
    // const D3D_DRIVER_TYPE driverType = D3D_DRIVER_TYPE_WARP;
    const D3D_DRIVER_TYPE driverType = D3D_DRIVER_TYPE_HARDWARE;
    
    // 여기서 생성하는 것들
    // m_device, - 장치
    // m_context, - 문맥
    // m_swapChain, - 스왑체인
    // m_renderTargetView, - 렌더 대상
    // m_screenViewport, - 스크린의 어떤 위치에 그릴 지
    // m_rasterizerSate - 래스터화 관련 옵션? 상태?

    // m_device, m_context 를 먼저 생성

    UINT createDeviceFlags = 0;
#if defined(DEBUG) || defined(_DEBUG)
    createDeviceFlags |= D3D11_CREATE_DEVICE_DEBUG;
#endif

    // 지역변수로 먼저 생성 후, 문제 없으면 멤버 변수로
    ComPtr<ID3D11Device> device;
    ComPtr<ID3D11DeviceContext> context;

    const D3D_FEATURE_LEVEL featureLevels[2] = {
        D3D_FEATURE_LEVEL_11_0, // 더 높은 버전이 먼저 오도록 설정
        D3D_FEATURE_LEVEL_9_3};
    D3D_FEATURE_LEVEL featureLevel;

    if (FAILED(D3D11CreateDevice(
            nullptr,                  // Specify nullptr to use the default adapter. (DXGI : 여러 버전의 D3D가 공통적으로 사용할 수 있게 디스플레이에 대한 저수준 제어를 묶어 놓은 것 + (D2D + D3D)에서도 사용)
            driverType,               // Create a device using the hardware graphics driver. (드라이버 타입 - 하드웨어)
            0,                        // Should be 0 unless the driver is D3D_DRIVER_TYPE_SOFTWARE. (소프트라면 이쪽에서 NUll이 아닌 값을 지정)
            createDeviceFlags,        // Set debug and Direct2D compatibility flags. (옵션 지정 - 디버깅 등)
            featureLevels,            // List of feature levels this app can support. (버전)
            ARRAYSIZE(featureLevels), // Size of the list above. (배열 사이즈)
            D3D11_SDK_VERSION, // Always set this to D3D11_SDK_VERSION for Microsoft Store apps.
            &device,           // Returns the Direct3D device created. (결과물 : Device를 리턴)
            &featureLevel,     // Returns feature level of device created. 
            &context           // Returns the device immediate context. (결과물 : Context 리턴)
            ))) {
        cout << "D3D11CreateDevice() failed." << endl;
        return false;
    }

    /* 참고: 오류가 있을 경우 예외 발생 방법

    // MS 예제
    inline void ThrowIfFailed(HRESULT hr)
    {
        if (FAILED(hr))
        {
            // Set a breakpoint on this line to catch Win32 API errors.
            throw Platform::Exception::CreateException(hr);
        }
    }

    // Luna DX12 교재
    #ifndef ThrowIfFailed
    #define ThrowIfFailed(x)                                              \
    {                                                                     \
        HRESULT hr__ = (x);                                               \
        std::wstring wfn = AnsiToWString(__FILE__);                       \
        if(FAILED(hr__)) { throw DxException(hr__, L#x, wfn, __LINE__); } \
    }
    #endif
    */

    if (featureLevel != D3D_FEATURE_LEVEL_11_0) {
        cout << "D3D Feature Level 11 unsupported." << endl;
        return false;
    }

  ...

}
```

먼저 Device를 생성하는 부분을 보자<br>

- ComPtr 를 통해 생성하여<br>
  실패하는 경우를 감안하여 바로 변수에 생성하지 않고<br>
  안전한 스마트 포인터를 통해 설정<br>

- 사용자의 DX 버전을 고려한 featureLevels<br>
  실패하는 경우, 더 낮은 버전으로 시도하는 처리를<br>
  할 수 있음<br>
  (테스트용이기에 별도의 실패처리는 하지 않음)<br>

- D3D11CreateDevice<br>
  : HResult를 반환하기에 FAILED 매크로를 통해 성공여부 검사하고<br>
    설정된 결과값의 device,featureLevel,context를 반환(Out)<br>
    ([이런식](https://learn.microsoft.com/ko-kr/windows/win32/api/d3d11/nf-d3d11-d3d11createdevice){:target="_blank" rel="noopener noreferrer"}의<br>
    공식 문서가 존재하며 파라미터 등을 설명해주는 페이지가 존재한다)<br>

- DXGI <br>
 : 여러 버전의 D3D가 공통적으로 사용할 수 있게<br>
   디스플레이에 대한 저수준 제어를 묶어 놓은 것<br>
   + (D2D + D3D)에서도 사용<br>


## AppBase::InitDirect3D() 2. SwapChain 생성 부분

```
bool AppBase::InitDirect3D() {
    ... Device, Context 생성 부분

    // 4X MSAA 지원하는지 확인
    UINT numQualityLevels;
    device->CheckMultisampleQualityLevels(DXGI_FORMAT_R8G8B8A8_UNORM, 4, &numQualityLevels);
    if (numQualityLevels <= 0) {
        cout << "MSAA not supported." << endl;
    }

    // numQualityLevels = 0; // MSAA를 강제로 끄기

    // 옵션 서술용 구조체 (_DESC) - 이걸로 CREATE 해줌
    DXGI_SWAP_CHAIN_DESC sd;
    ZeroMemory(&sd, sizeof(sd)); // 옵션이 많아서 0으로 초기화
    sd.BufferDesc.Width = m_screenWidth;               // set the back buffer width
    sd.BufferDesc.Height = m_screenHeight;             // set the back buffer height
    sd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM; // use 32-bit color (색 표현 포맷 32비트(RGBA 각 8비트 사용) - UNORM : Unsigned Normalized Integer 0~1 로 만들어준다(255로 나눔))
    sd.BufferCount = 2;                                // Double-buffering
    sd.BufferDesc.RefreshRate.Numerator = 60;
    sd.BufferDesc.RefreshRate.Denominator = 1; // 위와 합쳐서 초당 몇프레임 결정
    sd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;  // how swap chain is to be used
    sd.OutputWindow = m_mainWindow;                    // the window to be used (출력할 윈도우)
    sd.Windowed = TRUE;                                // windowed/full-screen mode (true : 전체화면 아님)
    sd.Flags = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH; // allow full-screen switching (전체화면과 왔다갔다 허용)
    sd.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
    if (numQualityLevels > 0) {
        sd.SampleDesc.Count = 4; // how many multisamples
        sd.SampleDesc.Quality = numQualityLevels - 1;
    } else {
        sd.SampleDesc.Count = 1; // how many multisamples
        sd.SampleDesc.Quality = 0;
    }

    if (FAILED(device.As(&m_device))) {
        cout << "device.AS() failed." << endl;
        return false;
    }

    if (FAILED(context.As(&m_context))) {
        cout << "context.As() failed." << endl;
        return false;
    }

    // 참고: IDXGIFactory를 이용한 CreateSwapChain()
    /*
    ComPtr<IDXGIDevice3> dxgiDevice;
    m_device.As(&dxgiDevice);

    ComPtr<IDXGIAdapter> dxgiAdapter;
    dxgiDevice->GetAdapter(&dxgiAdapter);

    ComPtr<IDXGIFactory> dxgiFactory;
    dxgiAdapter->GetParent(IID_PPV_ARGS(&dxgiFactory));

    ComPtr<IDXGISwapChain> swapChain;
    dxgiFactory->CreateSwapChain(m_device.Get(), &sd, &swapChain);

    swapChain.As(&m_swapChain);
    */

    // 참고: IDXGIFactory4를 이용한 CreateSwapChainForHwnd()
    /*
    ComPtr<IDXGIFactory4> dxgiFactory;
    dxgiAdapter->GetParent(IID_PPV_ARGS(&dxgiFactory));

    DXGI_SWAP_CHAIN_DESC1 swapChainDesc = {0};
    swapChainDesc.Width = lround(m_screenWidth); // Match the size of the window.
    swapChainDesc.Height = lround(m_screenHeight);
    swapChainDesc.Format = DXGI_FORMAT_B8G8R8A8_UNORM; // This is the most common swap chain format.
    swapChainDesc.Stereo = false;
    swapChainDesc.SampleDesc.Count = 1; // Don't use multi-sampling.
    swapChainDesc.SampleDesc.Quality = 0;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.BufferCount = 2; // Use double-buffering to minimize latency.
    swapChainDesc.SwapEffect =
        DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL; // All Microsoft Store apps must use this SwapEffect.
    swapChainDesc.Flags = 0;
    swapChainDesc.Scaling = DXGI_SCALING_NONE;
    swapChainDesc.AlphaMode = DXGI_ALPHA_MODE_IGNORE;

    ComPtr<IDXGISwapChain1> swapChain;
    dxgiFactory->CreateSwapChainForHwnd(m_device.Get(), m_mainWindow, &swapChainDesc, nullptr,
    nullptr, swapChain.GetAddressOf());
    */

    if (FAILED(D3D11CreateDeviceAndSwapChain(0, // Default adapter
                                            driverType,
                                            0, // No software device
                                            createDeviceFlags, featureLevels, 1, D3D11_SDK_VERSION,
                                            &sd, &m_swapChain, &m_device, &featureLevel,
                                            &m_context))) {
        cout << "D3D11CreateDeviceAndSwapChain() failed." << endl;
        return false;
    }
}
```

MSAA 지원 여부의 확인, 이후 SwapChain 생성 부분<br>

- Anti-Aliasing<br>
  : 도형을 그릴때 나타나는 '계단 현상'을<br>
    제거하기 위한 기법<br>
    Blur를 이용하여 구현하는 방식이나 (FXAA,SMAA,TAA 등이 존재)<br>
    Sampling 을 이용하여 구현 (SSAA, MSAA 등)<br>

  - SSAA(Super-Sample Anti-Aliasing)?<br>
    : 더 큰 해상도로 렌더링을 한 후<br>
      다시 축소하는 방식<br>
      (고품질이지만 성능 비쌈)<br>

- MSAA(MultiSample Anti-Aliasing)<br>
  : 안티 앨리어싱 기법 중 하나로서<br>
  픽셀 내부를 '여러 샘플 지점'으로 나눠서 검사<br>
  각각의 샘플마다 피사체의 '내부/외부'를 검사하고<br>
  그 결과를 평균 내서 픽셀 색상 결정<br>
  (경계를 부드럽게 하나, 내부의 '계단현상'은 못잡는 경우 존재)<br>

- _DESC 구조체<br>
  : '옵션 서술용 구조체'이며<br>
    Direct X에서 무언가를 생성할 때<br>
    데이터나 옵션을 세팅할 때 사용하는 일종의 규약같은 구조체<br>
    (이 규약에 맞게 만들어 주세요 라는 의미의 구조체다)<br>

- ZeroMemory : memcpy()를 이용한 0으로 싹미는 매크로<br>
  (보통 _DESC는 옵션이 매우 많기에 제로 메모리로 싹 밀고 시작한다)<br>

- DXGI_SWAP_CHAIN_DESC<br>
  : 사용할 버퍼의 해상도 지정,<br>
    색 표현에 따른 포맷팅,<br>
    버퍼 갯수와 초당 프레임 갯수,<br>
    버퍼 사용처(DXGI의 플래그의 일종, 상황에 맞게 설정하자),<br>
    출력할 윈도우(이전에 CreateWindow()로 m_mainWindow의 핸들을 받아놓았음),<br>
    전체화면 / 창 모드 설정 등이 존재<br>
    (더 자세한 내용은 공식문서 등을 찾아보자)<br>

- 포맷팅 방식(DXGI_FORMAT_R8G8B8A8_UNORM)<br>
  : 사용하는 4Byte를 '어떻게 나누어 사용할지'를 미리 알린다<br>
  RGBA를 각각 1Byte로 사용(_R8G8B8A8)<br>
  또한 그 값들을 0.0 ~ 1.0 부동소수 값으로 표현 (UNORM - Unsigned Normalized)<br>
  (필요에 따라 더 큰 Byte 사용 가능)<br>

- D3D11CreateDeviceAndSwapChain 을 통해 SwapChain 생성<br>
  참고로 이 함수는 Device와 Context도 다시 만듦<br>
  (CreateSwapChain()등 같이 다시 안 만드는 함수도 존재한다)<br>
  (두 주석들 참고)<br>
  (-> 초기화 부분이라서 딜레이를 어느정도는 용납하는 듯 보인다)<br>

### SwapChain 추가 내용
DXGI에 구현되어 있음<br>

- Present() 호출 시의 예시<br>

<img width="690" height="459" alt="Image" src="https://github.com/user-attachments/assets/7c8ed293-4503-4320-995b-9f9fc7b07803" /><br>

Front Buffer와 BackBuffer를 교체(Swap)하는 지시를 내린다<br>
<br>

- Tripple Buffer 사용 시의 Swap Chain<br>
<img width="842" height="689" alt="Image" src="https://github.com/user-attachments/assets/3b0b4dee-0b18-417f-a405-54ea2cd47c88" /><br>

이러한 SwapChain을 사용함으로서<br>
버퍼를 1개 사용할 때 나타나는<br>
'깜빡임' 현상을 없앨 수 있음<br>
(부드럽게 화면이 이어지도록)<br>

ScreenBuffer(FrontBuffer)가 출력되는 동안<br>
그래픽스 API가 BackBuffer에 그림을 그린다<br>

ScreenBuffer가 Output(실제 모니터)에 데이터를 전송하는 동안<br>
(+ 모니터가 화면에 출력하는 시간)<br>

미리 BackBuffers에 그림을 그려놓고<br>
순식간에 Screen Buffer와 '바꿔치기' 함으로서<br>
'그려지는 빈 순간'을 없애기 위함<br>

<br>


- 버퍼를 2개 사용하는 경우도 비슷하다<br>
<img width="503" height="545" alt="Image" src="https://github.com/user-attachments/assets/b8a31edc-1632-4e1c-9bbd-3df46a07c124" /><br>

Back Buffer와 Front Buffer를 서로 바꿔주며 반복<br>
(Page Flipping 이라는 용어로도 사용됨)<br>

- 메모리를 복사하는게 아니라<br>
  버퍼를 가리키는 '포인터'를 교체함으로서<br>
  빠른 swap이 가능하다<br>


## AppBase::InitDirect3D() 3. RenderTargetView 생성 부분

```
bool AppBase::InitDirect3D() {
  ... SwapChain 생성 부분

  // CreateRenderTarget
  ID3D11Texture2D *pBackBuffer; // 컬러값을 2차원 배열로 저장된 메모리 공간 (이미지와 버퍼 등 다양한 곳에서 사용)
  m_swapChain->GetBuffer(0, IID_PPV_ARGS(&pBackBuffer));
  if (pBackBuffer) {
      m_device->CreateRenderTargetView(pBackBuffer, NULL, &m_renderTargetView);
      pBackBuffer->Release();
  } else {
      cout << "CreateRenderTargetView() failed." << endl;
      return false;
  }

}
```

위에서 만든 SwapChain의 버퍼를 가져다<br>
RenderTargetView를 만듦<br>

- RTV는 버퍼의 메모리를 가져다 쓰는 방식은 아님<br>
  메모리의 주체는 Buffer 이며<br>
  RTV는 버퍼의 메모리를 소유하지 않고,<br>
  그 버퍼를 '렌더링용 타겟'으로 해석하는 View 객체이다<br>
  (포인터용 객체라 생각해도 무방하다)<br>
  (아니면 이 버퍼를 'RTV'로 보겠다는 캐스팅 방식이나)<br>

- RTV 사실상 화살표 처럼 '어느 버퍼'에 그릴지를 가리킨다<br>
  그렇기에 OM 스테이지 이후<br>
  RTV를 바인딩하고, 그 출력이 RTV가 가리킨<br>
  버퍼에 기록된다<br>
  (이후 present 호출 시, 이 버퍼와 front buffer로<br>
  교체 -> 버퍼 포인터 교환(flip))<br>

- 백버퍼가 아니여도 RTV로 지정한 곳(ID3D11Texture2D 등)에도 지정할 수 있음<br>
  (포스트 프로세싱, 그림자 등 다양한 곳에 응용 가능)<br>

- 요점은 'OM 출력(그래픽 파이프라인 결과)를 기록할 수 있는 창구'라 인식하자<br>

- ID3D11Texture2D<br>
  : Color 값 등이 2차원 배열로 저장된 텍스쳐 처럼 보이나<br>
  실제로는 '2차원 배열 형태의 <br>
  GPU 메모리 리소스를 추상화한 범용 컨테이너'<br>
  라는 인식으로 사용된다<br>
  (Color, Depth, Stencil 버퍼 뿐 아니라<br>
  렌더 타겟, 쉐이더의 결과 등 매우 다양한 곳에 사용)<br>

## AppBase::InitDirect3D() 4. Viewport 설정 부분

```
bool AppBase::InitDirect3D() {
  ... RenderTargetView 생성 부분

  // Set the viewport
  ZeroMemory(&m_screenViewport, sizeof(D3D11_VIEWPORT));
  m_screenViewport.TopLeftX = 0;
  m_screenViewport.TopLeftY = 0;
  m_screenViewport.Width = float(m_screenWidth); // /2 등을 통해 왼쪽에서만 렌더링도 가능
  m_screenViewport.Height = float(m_screenHeight);
  // m_screenViewport.Width = static_cast<float>(m_screenHeight);
  m_screenViewport.MinDepth = 0.0f;
  m_screenViewport.MaxDepth = 1.0f; // Note: important for depth buffering
  
  // 뷰포트 적용
  m_context->RSSetViewports(1, &m_screenViewport);

}
```

뷰포트 설정 초기화<br>

- viewport?<br>
  : 래스터라이저(RS) 단계에서 쓰이는<br>
  '출력되는 좌표계'를 '어떻게 자를지/변환할지'가 표시되는 영역<br>
  (D3D11_VIEWPORT)<br>

  - 직접적인 영역이 아닌, RS 단계에서<br>
    최종적으로 '픽셀 위치'를 구하기 위한 일종의 규칙<br>

  - 화면이라는 '도화지'에서 <br>
    '이 안'에서만 쓰세요<br>
    같은 느낌이라 생각해보자<br>
    (그렇기에 Width나 Height를 절반으로 줄이면<br>
    그에 맞게 '절반'에만 출력하게 됨)<br>

- Depth Buffer를 사용하기 위해서는 Depth 설정에 유의할 것<br>

- Context의 RSSetViewports 를 통해<br>
  Viewport를 설정한다<br>
  (RS 단계(NDC -> 화면 좌표계)에서 뷰포트의<br>
  범위를 이용하기에 출력이 바뀌게 된다)<br>

## AppBase::InitDirect3D() 5. RasterizerState 생성 부분

```
bool AppBase::InitDirect3D() {
  ... Viewport 설정 부분

  // 설정을 위해 _DESC 구조체
  // Create a rasterizer state
  D3D11_RASTERIZER_DESC rastDesc;
  ZeroMemory(&rastDesc, sizeof(D3D11_RASTERIZER_DESC)); // Need this
  rastDesc.FillMode = D3D11_FILL_MODE::D3D11_FILL_SOLID;
  // rastDesc.FillMode = D3D11_FILL_MODE::D3D11_FILL_WIREFRAME;
  rastDesc.CullMode = D3D11_CULL_MODE::D3D11_CULL_NONE;
  rastDesc.FrontCounterClockwise = false;

  m_device->CreateRasterizerState(&rastDesc, &m_rasterizerSate);

}
```

관련 _Desc 구조체를 생성하고<br>
그 내용을 세팅한 후<br>
device에 '생성'을 요구한다<br>
(리소스 생성이므로 Device에 요구)<br>

- D3D11_FILL_WIREFRAME<br>
  : 와이어 프레임만 그리는 경우<br>
   (디버깅, 정점 확인 등에 사용)<br>
   (언리얼에서 와이어프레임 표시하는 것처럼 나온다)<br>

- CullMode<br>
  : None이면 모든 삼각형을 그림<br>
    (front / back은 아래 옵션과 같이 이용)<br>
    front : 정면 x<br>
    back : 후면 x<br>

- FrontCounterClockwise<br>
  : 정면이 시계/반시계인지 세팅하는 옵션<br>
   (false 가 '시계')<br>
   (렌더링하는 정점의 방향을 결정한다)<br>

이렇게 만든 RasterizerState 는 렌더링에서 사용한다<br>

## AppBase::InitDirect3D() 6. Depth Buffer 생성 부분

```
bool AppBase::InitDirect3D() {
  ... RasterizerState 생성 부분

  // Create depth buffer
  // Depth 저장을 위해 Texture 2D 사용
  D3D11_TEXTURE2D_DESC depthStencilBufferDesc;
  depthStencilBufferDesc.Width = m_screenWidth; // 해상도 맞춰준다
  depthStencilBufferDesc.Height = m_screenHeight;
  depthStencilBufferDesc.MipLevels = 1;
  depthStencilBufferDesc.ArraySize = 1;
  depthStencilBufferDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT; // 24비트 사용, UNORM + Stencil 8비트 에 UINT로

  // MSAA의 경우 옵션을 맞춰준다
  if (numQualityLevels > 0) {
      depthStencilBufferDesc.SampleDesc.Count = 4; // how many multisamples
      depthStencilBufferDesc.SampleDesc.Quality = numQualityLevels - 1;
  } else {
      depthStencilBufferDesc.SampleDesc.Count = 1; // how many multisamples
      depthStencilBufferDesc.SampleDesc.Quality = 0;
  }
  // 메모리 사용처 - CPU,GPU가 접근이 가능한지를 설정
  // Default 면 GPU가 읽고 쓸 수 있음
  // IMMUTABLE 이면 GPU가 읽기만 (CPU는 못읽음)
  // DYNAMIC 이면 GPU는 읽기만 CPU는 쓸수만 있음 - 물리 엔진을 CPU에서 계산 등
  // STAGING 이면 GPU -> CPU 일때 사용 (Compute 쉐이딩 등에서 사용)
  depthStencilBufferDesc.Usage = D3D11_USAGE_DEFAULT; 
  depthStencilBufferDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL; // Depth + Stencil 버퍼로 사용
  depthStencilBufferDesc.CPUAccessFlags = 0;
  depthStencilBufferDesc.MiscFlags = 0;

  if (FAILED(m_device->CreateTexture2D(&depthStencilBufferDesc, 0,
                                      m_depthStencilBuffer.GetAddressOf()))) {
      cout << "CreateTexture2D() failed." << endl;
  }

  if (FAILED(
          m_device->CreateDepthStencilView(m_depthStencilBuffer.Get(), 0, &m_depthStencilView))) {
      cout << "CreateDepthStencilView() failed." << endl;
  }

}
```

DepthBuffer : Depth 값을 '저장'하는 버퍼<br>
그렇기에 Texture2D를 사용하여 구현<br>
(RS 진행 후, Z 값이 '평면'에서 같아지기에<br>
 깊이 값을 따로 저장하여 나중에 그릴 순서를 잡아준다)<br>
(그렇기에 z 값 검사할때 보통 투명 객체는 별도처리 하는 편)<br>
(뒤에서 앞으로 그린다던가)<br>

- DXGI_FORMAT_D24_UNORM_S8_UINT<br>
  : 텍스쳐 포맷으로 24비트 사용<br>
   형식은 UNORM<br>
   Stencil 에 8비트, 그 형식을 UINT로<br>
   (32비트를 결국 다 사용)<br>

- Usage<br>
  : 텍스쳐 2D의 사용처<br>
    (사실상 2차원 메모리 공간을 '어떻게 사용할지'에 대한 설정)<br>
    - D3D11_USAGE_DEFAULT : GPU가 읽고 씀<br>
    - D3D11_USAGE_IMMUTABLE : GPU가 읽기만 (CPU는 못읽음) -> 만들 때 초기화할 것<br>
    - D3D11_USAGE_DYNAMIC : GPU는 읽기만 CPU는 쓸수만(물리 엔진 CPU 계산 등)<br>
    - D3D11_USAGE_STAGING : GPU -> CPU 복사에 사용(Compute Shading 등)<br>

- BindFlag : D3D11_BIND_DEPTH_STENCIL<br>
  (Depth + Stencil 버퍼로도 사용)<br>

- Device를 통해 GPU에 메모리를 잡아달라고 한다<br>
  (Texture2D는 GPU의 메모리 뭉치다)<br>

- Depth Buffer는 Texture2D로 선언<br>
  RTV처럼 그냥 Texture2D로 공간 잡아놓고<br>
  다르게 사용하는 것<br>
  (결국 얘도 버퍼하나 잡아놓고 나중에 그 안에 있는걸<br>
  Depth 값으로 인식해서 쓴단 말)<br>

- CreateDepthStencilView(m_depthStencilBuffer.Get(), 0, &m_depthStencilView)<br>
  : m_depthStencilBuffer를<br>
  '깊이/스텐실 버퍼'로 해석해서 사용한다고 Device에 알려주는 것<br>

  - Depth Stencil View?<br>
    : OM 스테이지에 바인딩할 수 있는 핸들<br>
      (그냥 OM에서 깊이값 사용할때, 제공하기 위한 view이다)<br>

- Stencil??<br>
  : 일종의 '마스크'용 버퍼<br>
    GPU 파이프라인에서 '그릴 픽셀'과 '필요 없는 픽셀'을 구분하기 위한<br>
    커스텀 규칙을 제공하는 버퍼<br>
    (보통 StencilFunc 옵션을 통해 비교하며<br>
    이를 사용자가 비교 함수를 통해 설정한다)<br>
    - 원하는 영역을 렌더링하면서 '색'은 안남기고 Stencil 버퍼 값만 세팅(=1)<br>
    - 조건 부 렌더링(Stencil Test 이용)을 통하여 Stencil == 1 인 녀석들만 렌더링 한다<br>
    - 이후, OM 단계에서 통합할땐, Stencil 값을 세팅한 녀석들만 그려짐<br>
    - 거울, 미니맵, UI 마스크, 아웃라인 등 다양한 연출에 사용 가능<br>


## AppBase::InitDirect3D() 7. Depth Stencil state 생성 부분

```
bool AppBase::InitDirect3D() {
  ... Depth Buffer 생성 부분

  // Depth Stencil View에 대한 상태 설정 (_DESC)
  // Create depth stencil state

  D3D11_DEPTH_STENCIL_DESC depthStencilDesc;
  ZeroMemory(&depthStencilDesc, sizeof(D3D11_DEPTH_STENCIL_DESC));
  depthStencilDesc.DepthEnable = true; // false
  depthStencilDesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK::D3D11_DEPTH_WRITE_MASK_ALL; // 껏다 키거나 할때 고려
  depthStencilDesc.DepthFunc = D3D11_COMPARISON_FUNC::D3D11_COMPARISON_LESS_EQUAL; // Depth 비교 방식 (지금은 작거나 같을때 그려주기 - 가까운걸 그려주는 옵션이다)
  if (FAILED(m_device->CreateDepthStencilState(&depthStencilDesc,
                                              m_depthStencilState.GetAddressOf()))) {
      cout << "CreateDepthStencilState() failed." << endl;
  }

  return true;
}
```

위의 DSV(Depth Stencil View)를 어떻게 사용할지<br>
_DESC 구조체와 함께 설정<br>

- DepthEnable : 깊이값 사용 여부<br>

- DepthWriteMask : Depth 사용여부를 껐다 킨다던가 할때 고려하는 옵션<br>

- DepthFunc : Depth 비교 방식<br>

- Depth Stencil state?<br>
  : 깊이 / 스텐실을 어떻게 사용할지에 대한 설정 상태<br>
   Device->CreateDepthStencilState을 통해 생성하고<br>
   나중에 OM 스테이지에서 해당 설정을 사용<br>