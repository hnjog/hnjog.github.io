---
title: "Direct X RunDirect"
date : "2025-08-21 16:30:00 +0900"
last_modified_at: "2025-08-21T16:30:00"
categories:
  - Direct X
tags:
  - Direct 3D
---

## 다시 Main.CPP로
```
int main() {
    hlab::ExampleApp exampleApp;

    if (!exampleApp.Initialize()) {
        cout << "Initialization failed." << endl;
        return -1;
    }

    return exampleApp.Run();
}
```

이제 초기화는 다 했으니<br>
Run을 돌려볼 시간!<br>

## AppBase::Run()

```
int AppBase::Run() {

    // Main message loop
    MSG msg = {0};
    while (WM_QUIT != msg.message) {
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        } else {
            ImGui_ImplDX11_NewFrame(); // GUI 프레임 시작
            ImGui_ImplWin32_NewFrame();

            ImGui::NewFrame(); // 어떤 것들을 렌더링 할지 기록 시작
            ImGui::Begin("Scene Control");

            // ImGui가 측정해주는 Framerate 출력
            ImGui::Text("Average %.3f ms/frame (%.1f FPS)", 1000.0f / ImGui::GetIO().Framerate,
                        ImGui::GetIO().Framerate);

            UpdateGUI(); // 추가적으로 사용할 GUI

            ImGui::End();
            ImGui::Render(); // 렌더링할 것들 기록 끝

            Update(ImGui::GetIO().DeltaTime); // 애니메이션 같은 변화

            Render(); // 우리가 구현한 렌더링

            ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData()); // GUI 렌더링

            // Switch the back buffer and the front buffer
            // 주의: ImGui RenderDrawData() 다음에 Present() 호출
            m_swapChain->Present(1, 0);
        }
    }

    return 0;
}
```

virtual이 아님!<br>
기본적으로 해당 프레임워크를 기반으로<br>
UpdateGUI,Update, Render 등을 override하면 된다<br>

매 프레임당<br>
while을 돌리면서<br>
프로그램 로직을 진행<br>

AppBase의 순수 가상 함수들<br>
virtual void UpdateGUI() = 0;<br>
virtual void Update(float dt) = 0;<br>
virtual void Render() = 0;<br>

Run 자체는 일종의 프로그램 흐름으로 냅두고<br>
해당하는 순수 가상함수를 구현함으로서<br>
Direct 3D 프로그래밍의 기반을 만들 수 있음<br>

ex)<br>
MainGame 만들고<br>
Update내에 모든 요소들의 Update를 돌린다는 등<br>

- Framerate : 프레임 rate를 제공하는 Imgui 함수<br>
- DeltaTime : DeltaTime을 제공하는 Imgui 함수<br>
  ('DeltaTime'은 '바로 전 프레임'을 실행할때 걸린 시간을 의미)<br>


## ExampleApp::UpdateGUI()

```
void ExampleApp::UpdateGUI() {
    ImGui::Checkbox("usePerspectiveProjection", &m_usePerspectiveProjection);
}
```

- Checkbox를 통해<br>
  bool 변수의 체크여부를 UI로 띄운후<br>
  사용자가 bool 변수를 수정할 수 있도록 한다<br>

## ExampleApp::Update()

```
void ExampleApp::Update(float dt) {

    static float rot = 0.0f;
    rot += dt;

    // 모델의 변환
    m_constantBufferData.model = Matrix::CreateScale(0.5f) * Matrix::CreateRotationY(rot) *
                                 Matrix::CreateTranslation(Vector3(0.0f, -0.3f, 1.0f));
    m_constantBufferData.model = m_constantBufferData.model.Transpose();

    using namespace DirectX;

    // 시점 변환
    m_constantBufferData.view =
        XMMatrixLookAtLH({0.0f, 0.0f, -1.0f}, {0.0f, 0.0f, 1.0f}, {0.0f, 1.0f, 0.0f});
    m_constantBufferData.view = m_constantBufferData.view.Transpose();

    // 프로젝션
    const float aspect = AppBase::GetAspectRatio();
    if (m_usePerspectiveProjection) {
        const float fovAngleY = 70.0f * XM_PI / 180.0f;
        m_constantBufferData.projection =
            XMMatrixPerspectiveFovLH(fovAngleY, aspect, 0.01f, 100.0f);
    } else {
        m_constantBufferData.projection =
            XMMatrixOrthographicOffCenterLH(-aspect, aspect, -1.0f, 1.0f, 0.1f, 10.0f);
    }
    m_constantBufferData.projection = m_constantBufferData.projection.Transpose();

    // Constant를 CPU에서 GPU로 복사
    AppBase::UpdateBuffer(m_constantBufferData, m_constantBuffer);
}
```

물체의 회전 같은 애니메이션을 구현하는 곳<br>

model의 변환 행렬을 만들어주고<br>
view 행렬을 통해 '시점'을 정한 후<br>
projection을 XMMatrix를 통해 구한다<br>

- XMMatrixLookAtLH<br>
  : DirectXMath 에서 제공하는 뷰 행렬 생성 함수<br>

<img width="668" height="550" alt="Image" src="https://github.com/user-attachments/assets/b56ec294-1958-4c5a-b0a5-d42bf7d518b4" /><br>

3차원 벡터 3개를 지정하면 뷰 행렬을 얻을 수 있다<br>
(카메라 위치, 카메라의 방향, 카메라 기준으로 '위' 방향)<br>

- XMMatrixPerspectiveFovLH<br>
  : DirectXMath 라이브러리에서 제공하는 함수<br>
  원근 투영 행렬을 생성한다<br>
  (시야각이 필요)<br>

- XMMatrixOrthographicOffCenterLH<br>
  : DirectXMath 라이브러리에서 제공하는 함수<br>
  직교 투영 행렬을 생성한다<br>

<img width="888" height="650" alt="Image" src="https://github.com/user-attachments/assets/406cbd21-da6e-4b1b-9997-c3f6e7ec4e33" /><br>

(각각의 변환 방법에 대한 시각적 표현)<br>

ConstantBuffer를 갱신한 후,<br>
해당 데이터를 AppBase::UpdateBuffer 를 사용하여<br>
GPU로 보낸다<br>
(정확히는 GPU에 세팅이 되어있는 위치로 보낸다)<br>

- DirectX는 왼손 좌표계이며<br>
  Row Major Matrix를 사용하는 점을 항상 숙지!<br>
  (Unreal도 이렇다)<br>
  그런데<br>
  HLSL은 Column Major Matrix를 사용한다<br>
  => 각 Model,View,Projection의 마지막에 Transpose(전치)를<br>
     사용하는 이유!<br>
     (주의할 것)<br>

### AppBase::UpdateBuffer

```
template <typename T_DATA>
void UpdateBuffer(const T_DATA &bufferData, ComPtr<ID3D11Buffer> &buffer) {
    D3D11_MAPPED_SUBRESOURCE ms;
    m_context->Map(buffer.Get(), NULL, D3D11_MAP_WRITE_DISCARD, NULL, &ms);
    memcpy(ms.pData, &bufferData, sizeof(bufferData));
    m_context->Unmap(buffer.Get(), NULL);
}
```

- D3D11_MAPPED_SUBRESOURCE?<br>
  : GPU 메모리에 CPU가 접근할 수 있도록<br>
  '매핑'하였을 때 반환받는 구조체 타입<br>
  (pData 부분을 memcpy 등을 통해 직접 쓰거나 읽으면<br>
  GPU 리소스에도 반영)<br>

- m_context->Map<br>
  : CPU가 GPU 메모리에 있는 리소스에 접근할 수 있도록 임시로 포인터를 할당해주는 함수<br>

  구성요소들<br>
  pResource : CPU에서 접근하려는 GPU 리소스 - buffer.Get()<br>
  Subresource : 서브리소스 인덱스, 버퍼라면 0 - NULL<br>
  MapType : CPU가 접근할 방식 설정 - D3D11_MAP_WRITE_DISCARD(기존 데이터 버리고 덮어씀)<br>
  MapFlags : 동기화 플래그 - NULL<br>
  pMappedResource : 결과 구조체 - &ms<br>

- m_context->Unmap<br>
  : CPU가 수정한 내용 등을 반영하고, GPU가 다시 메모리를 쓸 수 있도록 잠금 해제하는 함수<br>

  구성요소들<br>
  pResource : 이전에 Map했던 리소스 - buffer.Get()<br>
  Subresource(UINT) : Map 하였던 서브리소스 인덱스 - NULL<br>

## ExampleApp::Render()

### DirectX의 그래픽 파이프라인

기본적인 단계는 IA -> VS -> RS -> PS -> OM<br>

<img width="419" height="697" alt="Image" src="https://github.com/user-attachments/assets/5fe11daa-1972-48fd-92c6-f9271bb45eb3" /><br>

(세부적으로 보았을 때)<br>

일반적으로 Memory Resource가 그래픽 파이프라인으로 향하는<br>
(<-)<br>
것이 DX 프로그래머가 데이터를 주거나 수정할 수 있는 부분<br>

#### 각 단계의 정리 표<br>

| 단계                              | 주요 역할                                       | 입력 → 출력                 | 주요 개념 / API                                               |
| ------------------------------- | ------------------------------------------- | ----------------------- | --------------------------------------------------------- |
| **Input Assembler (IA)**        | 정점(Vertex)/인덱스 데이터를 가져와 GPU에 전달             | 정점 버퍼, 인덱스 버퍼 → 정점 스트림  | `ID3D11Buffer`, InputLayout                               |
| **Vertex Shader (VS)**          | 정점 변환 (Model→World→View→Projection) 및 속성 계산 | 정점 입력 → 변환된 정점          | HLSL `VS`, `XMMatrixLookAtLH`, `XMMatrixPerspectiveFovLH` |
| **Hull Shader (HS)** *(옵션)*     | 테셀레이션 단계 제어 (패치 세분화)                        | 패치 → 제어점                | HLSL `HS`                                                 |
| **Tessellator** *(옵션)*          | 패치 분할, 곡면 생성                                | 제어점 → 분할된 점             | 하드웨어 고정 단계                                                |
| **Domain Shader (DS)** *(옵션)*   | 테셀레이션 후 좌표 계산                               | 세분화된 점 → 위치             | HLSL `DS`                                                 |
| **Geometry Shader (GS)** *(옵션)* | 정점 집합(프리미티브)을 받아 새로운 정점/프리미티브 생성            | 삼각형, 선 등 → 수정/추가된 프리미티브 | HLSL `GS`                                                 |
| **Rasterizer (RS)**             | 3D → 2D 변환, 클리핑, 뷰포트 변환                     | 클립 공간 정점 → 픽셀 후보        | 뷰포트, 카메라 클리핑                                              |
| **Pixel Shader (PS)**           | 픽셀 색상 계산 (조명, 텍스처, 머티리얼 등)                  | 픽셀 입력 → 픽셀 색상           | HLSL `PS`                                                 |
| **Output Merger (OM)**          | 최종 픽셀을 렌더타겟/깊이버퍼와 합성                        | 픽셀 색상 → 프레임버퍼           | 블렌딩, 깊이/스텐실 테스트                                           |

#### 단계 별 관계도(흐름도)

```
[CPU App Code]
    |
    v
[Input Assembler] -- 정점 데이터(Vertex Buffer, Index Buffer)
    |
    v
[Vertex Shader] -- 좌표 변환 (World→View→Projection)
    |
    v
 (Hull Shader)  --> (Tessellator) --> (Domain Shader)  [*테셀레이션 사용 시*]
    |
    v
(Geometry Shader) [옵션]
    |
    v
[Rasterizer] -- 3D → 2D 화면 공간 변환
    |
    v
[Pixel Shader] -- 픽셀 색상, 텍스처 샘플링
    |
    v
[Output Merger] -- 깊이/스텐실/블렌딩 처리
    |
    v
[Back Buffer / SwapChain]
    |
    v
[화면 출력]
```

- Tesselation?<br>

<img width="485" height="374" alt="Image" src="https://github.com/user-attachments/assets/490582ca-af4b-4ac8-9d64-4897bd0667c0" /><br>

DX11 부터 지원<br>
모델의 '표면'을 GPU가 '자동'으로 세분화하여<br>
더 많은 '폴리곤'을 생성하는 기술<br>

- 3D 모델이 가까이서 보았을때 '울퉁불퉁'한 현상을 방지하기 위함<br>
  (low polygon)<br>
- 그렇기에 삼각형을 세분화하여 더 부드럽고 디테일한 메쉬를 만든다<br>


- 이미지 보는 그래픽스 단계<br>
<img width="324" height="828" alt="Image" src="https://github.com/user-attachments/assets/afcb38f3-ad04-4e1b-bb4c-c230a844486f" /><br>




### 코드
```
void ExampleApp::Render() {

    // IA: Input-Assembler stage
    // VS: Vertex Shader
    // PS: Pixel Shader
    // RS: Rasterizer stage
    // OM: Output-Merger stage

    m_context->RSSetViewports(1, &m_screenViewport);

    float clearColor[4] = {0.0f, 0.0f, 0.0f, 1.0f};
    m_context->ClearRenderTargetView(m_renderTargetView.Get(), clearColor);
    m_context->ClearDepthStencilView(m_depthStencilView.Get(),
                                     D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);

    // 비교: Depth Buffer를 사용하지 않는 경우
    // m_context->OMSetRenderTargets(1, m_renderTargetView.GetAddressOf(), nullptr);
    m_context->OMSetRenderTargets(1, m_renderTargetView.GetAddressOf(), m_depthStencilView.Get());

    m_context->OMSetDepthStencilState(m_depthStencilState.Get(), 0);

    // 어떤 쉐이더를 사용할지 설정
    m_context->VSSetShader(m_colorVertexShader.Get(), 0, 0);

    /* 경우에 따라서는 포인터의 배열을 넣어줄 수도 있습니다.
    ID3D11Buffer *pptr[1] = {
        m_constantBuffer.Get(),
    };
    m_context->VSSetConstantBuffers(0, 1, pptr); */

    m_context->VSSetConstantBuffers(0, 1, m_constantBuffer.GetAddressOf());
    m_context->PSSetShader(m_colorPixelShader.Get(), 0, 0);

    m_context->RSSetState(m_rasterizerSate.Get());

    // 버텍스/인덱스 버퍼 설정
    UINT stride = sizeof(Vertex);
    UINT offset = 0;
    m_context->IASetInputLayout(m_colorInputLayout.Get());
    m_context->IASetVertexBuffers(0, 1, m_vertexBuffer.GetAddressOf(), &stride, &offset);
    m_context->IASetIndexBuffer(m_indexBuffer.Get(), DXGI_FORMAT_R16_UINT, 0);
    m_context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    m_context->DrawIndexed(m_indexCount, 0, 0);
}
```

DX 자료에선<br>
함수 이름을 통해 '어떤 단계'에서 사용할 데이터인지 판별이 가능하다<br>

- ..Set..() 함수<br>
  : 보통 어떻게 렌더링을 해야할지 그래픽스 파이프라인의<br>
   여러가지 옵션들을 설정만 해주는 것<br>
   ('설정'하는 거싱기에 '문맥'(Context)로 볼 수 있음)<br>
   (실제 이러한 옵션을 사용하는건 DrawIndex 호출 이후)<br>
   (그렇기에 이러한 계열 함수들은 IA이전에 호출하여도<br>
   기본적으론 괜찮다)<br>

- m_context->ClearRenderTargetView 이전에 그려놓았던<br>
  RTV(백버퍼)를 깨끗히 비워준다<br>
  ClearDepthStencilView 도 호출하면 Depth/Stencil도 초기화<br>
  (Depth는 1로, Stencil은 0으로 초기화)<br>

- OMSetRenderTargets : RTV를 설정<br>
  (이전에 세팅해둔 RTV를 가져다 쓴다)<br>

  - RTV는 하나 만들었는데 버퍼는 2개(front, back)<br>
    : 근데 그러면 RTV가 가리킨 백버퍼가 SwapChain이 Present()할때<br>
      다시 새로운 BackBuffer를 가리킴??<br>
  - 정확히는 조금 다름<br>
    RTV는 Front Buffer를 가리킬 수 없으며<br>
    '가리키는 것은' '몇 번째 백버퍼'를 가리키는 것임<br>

```
초기 상태:
  Front -> Buffer[0]
  Back  -> Buffer[1]  ← RTV가 여기 연결됨

Present() 후:
  Front -> Buffer[1]
  Back  -> Buffer[0]  ← 이제 여기에 그려야 함
```

위와 같이<br>
RTV를 별도로 건드리지 않아도 계속 알아서 백버퍼에 그려주는 것<br>
(DX12나 실제 엔진 등에서는<br>
백버퍼의 개수만큼 여러 RTV를 만들어두고<br>
해당 프레임에 사용할 RTV 포인터를 교체하는 방식을 사용)<br>

(DX11에서는 드라이버가 해주기에 0으로 냅둬도 괜찮기는 하다<br>
그렇다고 이게 원칙은 아니며,<br>
테스트 예제 같은 곳에서 사용하는 방식임)<br>

- VSSetShader로 사용할 Vertex Shader를 고른다<br>
 이후 ConstantBuffer, Pixel Shader, RSState를 세팅<br>

- IA를 통해 InputLayout, vertex Buffer, Index Buffer를 세팅<br>

- 그리는 방식을 TOPOLOGY_TRIANGLELIST 로 설정<br>
  (정점 3개당 삼각형 1개)<br>
  (다양한 방식이 존재한다)<br>

<img width="787" height="630" alt="Image" src="https://github.com/user-attachments/assets/86646421-12fe-4840-bc51-8b7fa6deabcb" /><br>

1. TRIANGLELIST를 통해 그리며<br>
2. TRIANGLESTRIP을 통해 그리는 경우<br>
   - 홀수로 그리는 경우와 짝수는 경우에 대하여<br>
     (홀수/짝수 의 상황에 따라 삼각형을 앞/뒤가 뒤집힌다...)<br>

- Render에서는 IA 세팅 후, DrawIndexed를 호출이 끝<br>
  (DrawIndexed를 통해 그리는 요소를 정해줄 수도 있음)<br>
  (지금은 0,0으로 전체를 그린다)<br>

- Set 함수들은 설정을 DX가 기억하기에<br>
  한 번 설정한 이후라면 설정이 유지됨<br>
  (Start Render() 등으로 함수를 뺼수도?)<br>
  (Shader는 번갈아가며 사용할 수 있기에 Render에서 계속 호출하는 편)<br>

- Render 함수 이후 Run 쪽의 마지막 쪽에서<br>
  m_swapChain->Present(1, 0); 를 호출하여 있다<br>
  -> 몇 줄 차이 안나는데 GPU가 벌써 다 그렸나???<br>
  Present가 호출되어도 내부적으로 '동기화'를 기다린다<br>
  GPU가 렌더링이 끝날때까지 대기한다<br>
  (Vsync 옵션이 켜져있으면<br>
  다 그리고 나서도 모니터 주사율에 맞춰서 호출되고<br>
  아니면 렌더링이 '끝나자 마자' 바로 화면을 교체한다)<br>
  (그런데 이러면 '화면'에서 일반 프레임과 이전 프레임이 같이 보이는<br>
  '짤리는' 현상이 나타날 수도 있으니 주의하자)<br>

  Present(1,0)<br>
  1 : SyncInterval - 모니터 수직 동기화 옵션<br>
  (1이면 다음 수직 동기 신호까지 대기)<br>
  0 : Flag - 프레젠트 동작 제어 플래그<br>
  (0이면 기본 동작, 재시작 or 실패코드 등의 처리 가능)<br>
