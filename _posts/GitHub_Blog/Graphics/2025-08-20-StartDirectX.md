---
title: "Direct X Init 1"
last_modified_at: "2025-08-20T14:30:00"
categories:
  - Direct X
tags:
  - Windows
  - Direct 3D
---

## 들어가기 앞서
개인적으로 포스팅을 하는 글이며<br>
기본적으로는 [Honglab](https://www.honglab.ai/enrollments){:target="_blank" rel="noopener noreferrer"}의 강의 글을 통해<br>
공부하는 용도이다<br>

## 시작 전에
사실 이거 말고도 이전 강의가 하나 더 있지만<br>
Component Object Model (COM)에 관한 내용<br>
(DirectX에서 유용하게 설명할 수 있으나 <br>
그래픽스와 직접적인 연관은 없기에 짧게 짚고 넘어간다)<br>

- COM ? <br>
  : 마이크로소프트가 제공하는 일종의 인터페이스<br>
  언어/컴파일러가 달라도 같은 인터페이스를 제공하는 Windows용 규약<br>

- WRL::ComPtr<br>
  : COM 객체용 스마트 포인터<br>
    IUnknown을 상속받으면 해당 포인터로 관리 가능<br>
    C++의 스마트 포인터 중 하나인 Shared_ptr과 유사하게 사용 가능<br>
    (Device, Context, Buffer ,Texture, SwapChain 등이 <br>
    IUnknown을 상속받기에 ComPtr로 관리가 가능하다)<br>
    (생포인터)

# DirectX 초기화

전반적으로 DX 자체는 버전은 12까지 나왔으나<br>
초기화 관련으로는 어느정도 틀이 잡혀있기에<br>
관련 개념과 초기화 방식 위주로 집고 넘어갈 예정이다<br>

(현재 코드 기준은 DX 11)<br>

## main.cpp

```
#include <iostream>
#include <memory>
#include <windows.h>

#include "ExampleApp.h"

using namespace std;

// main()은 앱을 초기화하고 실행시키는 기능만 합니다.
// 콘솔창이 있으면 디버깅에 편리합니다.
// 디버깅할 때 애매한 값들을 cout으로 출력해서 확인해보세요.
int main() {
    hlab::ExampleApp exampleApp;

    if (!exampleApp.Initialize()) {
        cout << "Initialization failed." << endl;
        return -1;
    }

    return exampleApp.Run();
}
```

기본적인 프레임 워크로서<br>
App 이라는 클래스를 생성 후<br>
해당 Initialize와 Run을 호출시킨다<br>
(사실상 main은 '프로그램 진입점'의 역할만 수행)<br>

초기화의 성공 여부를 확인하고<br>
프로그램을 실행시킨다<br>

## ExampleApp.h

```
class ExampleApp : public AppBase {
  public:
    ExampleApp(); // indexCount (렌더링시 vertex 렌더링 횟수 - 나중에는 뺌) 초기화 및 AppBase() 호출

    virtual bool Initialize() override;
    virtual void UpdateGUI() override;
    virtual void Update(float dt) override;    
    virtual void Render() override;

  protected:
    ComPtr<ID3D11VertexShader> m_colorVertexShader;
    ComPtr<ID3D11PixelShader> m_colorPixelShader;
    ComPtr<ID3D11InputLayout> m_colorInputLayout;

    ComPtr<ID3D11Buffer> m_vertexBuffer;
    ComPtr<ID3D11Buffer> m_indexBuffer;
    ComPtr<ID3D11Buffer> m_constantBuffer;
    UINT m_indexCount;

    ModelViewProjectionConstantBuffer m_constantBufferData;

    bool m_usePerspectiveProjection = true;
};
```

이미 AppBase에서 선언된 내용을 override하여 주로 사용할 클래스<br>
그 외에도 렌더링용 VS 나 PS 등을 가지고 있다<br>
이번에는 '초기화'에 내용이 집중되어 있기에<br>
일부 요소는 나중에 설명할 예정<br>

(hlsl 파일이 존재하나 역시 이번엔 안다룬다)<br>

## AppBase.h

```
// 모든 예제들이 공통적으로 사용할 기능들을 가지고 있는
// 부모 클래스
class AppBase {
  public:
    AppBase(); // 여기서 해상도 세팅, m_mainWindow,m_screenViewport를 초기화
    virtual ~AppBase();

    float GetAspectRatio() const;

    int Run();

    virtual bool Initialize();
    virtual void UpdateGUI() = 0;
    virtual void Update(float dt) = 0;
    virtual void Render() = 0;

    virtual LRESULT MsgProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam);
    
    ...
    다양한 멤버 함수

  public:
    // 변수 이름 붙이는 규칙은 VS DX11/12 기본 템플릿을 따릅니다.
    // 다만 변수 이름을 줄이기 위해 d3d는 생략했습니다.
    // 예: m_d3dDevice -> m_device
    int m_screenWidth; // 렌더링할 최종 화면의 해상도
    int m_screenHeight;
    HWND m_mainWindow;

    ComPtr<ID3D11Device> m_device;
    ComPtr<ID3D11DeviceContext> m_context;
    ComPtr<ID3D11RenderTargetView> m_renderTargetView;
    ComPtr<IDXGISwapChain> m_swapChain;
    ComPtr<ID3D11RasterizerState> m_rasterizerSate;

    // Depth buffer 관련
    ComPtr<ID3D11Texture2D> m_depthStencilBuffer;
    ComPtr<ID3D11DepthStencilView> m_depthStencilView;
    ComPtr<ID3D11DepthStencilState> m_depthStencilState;

    D3D11_VIEWPORT m_screenViewport;

}
```

다양한 예제를 위한 기본 베이스가 될 클래스<br>
Device, Context, 렌더 타겟과 스왑 체인,<br>
DepthView와 State, 뷰포트 등을 변수로 가진다<br>

멤버 함수를 다 적으면 100줄이 넘어가기에 스킵<br>
(버퍼, Init, 입력 등에 관한 다양한 함수 존재)<br>

## AppBase 의 생성자

```
// RegisterClassEx()에서 멤버 함수를 직접 등록할 수가 없기 때문에
// 클래스의 멤버 함수에서 간접적으로 메시지를 처리할 수 있도록 도와줍니다.
AppBase *g_appBase = nullptr;

// 전역 함수
// RegisterClassEx()에서 실제로 등록될 콜백 함수
LRESULT WINAPI WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {

    // 클래스의 멤버함수는 콜백으로 등록이 안되기에
    // 전역 함수를 사용 하고
    // 전역 변수인 g_appBase를 이용해서 간접적으로 멤버 함수 호출
    return g_appBase->MsgProc(hWnd, msg, wParam, lParam);
}

// 생성자
AppBase::AppBase()
    : m_screenWidth(1280), m_screenHeight(960), m_mainWindow(0),
      m_screenViewport(D3D11_VIEWPORT()) {

    g_appBase = this;
}
```

생성자에서 g_appBase 라는 전역변수에 자신을 넣어줌<br>
InitWindow()에서 콜백용 함수를 받을 때 사용하는 함수는<br>
클래스의 멤버함수가 등록이 안되기에<br>

임의로 전역 변수 + 함수 를 선언하여<br>
클래스의 함수를 호출하도록 한다<br>
(일반적으로 WndProc 라는 함수명을 사용)<br>

- 프로그램의 메시지 큐에 OS가 메시지를 전달해줌<br>
  그렇기에 프로그램은 WM_KEYDOWN(키입력),<br>
  WM_MOUSEMOVE(마우스 움직임) 같은<br>
  입력처리를 직접 구현하지 않고 OS로부터<br>
  메시지를 전달받는다<br>
  (WndProc에서 호출하는 MsgProc 를 통해<br>
  각 메시지(입력값)에 따른 처리를 할 예정)<br>


## ExampleApp::Initialize (1)

```
bool ExampleApp::Initialize() {

  // 부모 클래스에서 전반적인 초기화 진행
  if (!AppBase::Initialize())
      return false;

  ...
}

```

일단 AppBase가 전반적인 Device나 Context 등을 생성하며<br>
Windows의 창을 띄우는 등의 역할을 해줄 예정<br>

AppBase의 초기화를 먼저 보고 다시 오자<br>


## AppBase::Initialize()

```
bool AppBase::Initialize() {

    if (!InitMainWindow()) // Windows를 초기화하고 생성
        return false;

    if (!InitDirect3D()) // Direct 3D를 통해 만든 window에 어떻게 그림을 그릴지 세팅 초기화
        return false;

    if (!InitGUI())     // Imgui 초기화
        return false;

    return true;
}
```

초기화를 각각의 함수에 위임한다<br>

### AppBase::InitMainWindow()

```
bool AppBase::InitMainWindow() {
/ 구조체 제작
    WNDCLASSEX wc = {sizeof(WNDCLASSEX),
                     CS_CLASSDC,
                     WndProc, // 콜백함수 등록 (전역 함수)
                     0L,
                     0L,
                     GetModuleHandle(NULL),
                     NULL,
                     NULL,
                     NULL,
                     NULL,
                     L"HongLabGraphics", // lpszClassName, L-string (Wide Character)
                     NULL};

    // The RegisterClass function has been superseded by the RegisterClassEx function.
    // https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerclassa?redirectedfrom=MSDN
    if (!RegisterClassEx(&wc)) {
        cout << "RegisterClassEx() failed." << endl;
        return false;
    }

    // 툴바까지 포함한 윈도우 전체 해상도가 아니라
    // 우리가 실제로 그리는 해상도가 width x height가 되도록
    // 윈도우를 만들 해상도를 다시 계산해서 CreateWindow()에서 사용

    // 우리가 원하는 그림이 그려질 부분의 해상도
    RECT wr = {0, 0, m_screenWidth, m_screenHeight};

    // 필요한 윈도우 크기(해상도) 계산
    // wr의 값이 바뀜
    AdjustWindowRect(&wr, WS_OVERLAPPEDWINDOW, false);

    // 윈도우를 만들때 위에서 계산한 wr 사용
    m_mainWindow = CreateWindow(wc.lpszClassName, L"HongLabGraphics Example", WS_OVERLAPPEDWINDOW,
                                100,                // 윈도우 좌측 상단의 x 좌표
                                100,                // 윈도우 좌측 상단의 y 좌표
                                wr.right - wr.left, // 윈도우 가로 방향 해상도
                                wr.bottom - wr.top, // 윈도우 세로 방향 해상도
                                NULL, NULL, wc.hInstance, NULL);

    if (!m_mainWindow) {
        cout << "CreateWindow() failed." << endl;
        return false;
    }

    ShowWindow(m_mainWindow, SW_SHOWDEFAULT); // 윈도우 창을 띄움
    UpdateWindow(m_mainWindow);

    return true;
}
```

게임 엔진이나, 앱 등에서는 기본적으로 처리해주는 과정<br>

- WNDCLASSEX wc 를 생성하는 과정에서<br>
  이전에 말하였던 WndProc 함수를 넣어준다<br>
  (콜백함수 등록)<br>

- L"" : Wide Character로 이루어진 문자열<br>
  (더 큰 크기의 문자 를 통해 더 많은 개수의 문자 표현 가능)<br>

- RegisterClassEx : 윈도우를 '등록'<br>
  이후, CreateWindow 로 실제 윈도우를 만든다<br>

- 툴바 크기를 포함하지 않고 렌더링을 하기 위하여<br>
  wr.right - wr.left, // 윈도우 가로 방향 해상도<br>
  wr.bottom - wr.top, // 윈도우 세로 방향 해상도<br>
  를 통해 윈도우 크기를 재계산<br>
  (툴바의 크기를 화면 해상도에 포함시키지 않음)<br>

- ShowWindow : 실제 윈도우 창을 띄운다<br>

