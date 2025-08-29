---
title: "WireFrame 그리기"
date : "2025-08-29 18:30:00 +0900"
last_modified_at: "2025-08-29T18:30:00"
categories:
  - Direct X
tags:
  - WireFrame
---

## 디버깅을 위한 WireFrame 그리기

3D 모델을 렌더링 하기전,<br>
이러한 모델을 WireFrame 방식으로 그리는 방식을 알아보자<br>

- WireFrame?<br>
  : 3D 메시를 삼각형으로 그림으로서<br>
    '그물망'처럼 표현하는 시각화 방식<br>
    후처리 없이 순수한 '기하'형태에 따른 구조만을 표시한다<br>

- 필요한 이유?<br>
  메시(Geometry)의 최적화 상태의 확인,<br>
  모델링이 삼각형이 얼마나 사용되었는지에 대한 시각적 확인,<br>
  불필요한 폴리곤이 들어갔는지, LOD 등이 잘 작동하는지를 확인<br>
  (그 외에도 Z-Fighting 현상이나 UV 등에 대한 것도 확인할 수 있다)<br>
  (충돌 / Occlusion(가림처리) 문제도 확인해볼 수 있음)<br>
  
이러한 체킹을 시각적으로 빠르게<br>
할 수 있기에 WireFrame 모드는 여러 엔진이나<br>
그래픽스 툴에서 기본 제공된다<br>

## 예제 - WireFrame을 버튼을 누름으로서 전환

- 일반적인 렌더링<br>

<img width="2225" height="1721" alt="Image" src="https://github.com/user-attachments/assets/994e106f-a02a-40c6-b88e-55c6be31ce12" /><br>

- WireFrame<br>

<img width="2233" height="1717" alt="Image" src="https://github.com/user-attachments/assets/de6e602a-7369-4a84-8174-2d7284a304e7" /><br>

이러한 버튼체크에 따라서<br>
Mode를 바꿔주는 것이 이번 예제의 목표<br>

Rasterize State 의<br>
D3D11_FILL_MODE::D3D11_FILL_WIREFRAME가 이번 예제의 핵심<br>
(D3D11_CULL_MODE::D3D11_CULL_NONE 도 같이 자주 사용,<br>
투명하게 뒤쪽도 보이게 하기 위하여)<br>

- RS 단계에서 설정하는 이유?<br>
  : Vs 에서 기하의 모양을 정의하고<br>
    이후 래스터라이저에서 이러한 기하 모양을<br>
    '픽셀' 단위로 바꾸는 과정에서 설정한다<br>
    (이후 FillMode로 설정된 Pixel 들만이 Pixel Shader로 간다)<br>

```
AppBase.h

// TODO: RasterizerState를 2개 만들기
ComPtr<ID3D11RasterizerState> m_solidRasterizerSate; // 일반적인 렌더링용 RSState
ComPtr<ID3D11RasterizerState> m_wireRasterizerSate;  // WireFrame용 RsState
bool m_drawAsWire = false;

---
AppBase - Init()

// Create a rasterizer state
D3D11_RASTERIZER_DESC rastWireDesc;
ZeroMemory(&rastWireDesc, sizeof(D3D11_RASTERIZER_DESC)); // Need this
rastWireDesc.FillMode = D3D11_FILL_MODE::D3D11_FILL_WIREFRAME; // WireFrame Mode
rastWireDesc.CullMode = D3D11_CULL_MODE::D3D11_CULL_NONE;      // 와이어 프레임으로 '투명하게' 뒤쪽까지 보이기 위해 사용
rastWireDesc.FrontCounterClockwise = false;
rastWireDesc.DepthClipEnable = true; // <- zNear, zFar 확인에 필요

m_device->CreateRasterizerState(&rastWireDesc,
                                m_wireRasterizerSate.GetAddressOf());
```


이후 Render에서 bool 설정에 따라<br>
RsSetState를 바꿔가며 설정하면 된다<br>
(매프레임 호출?<br>
그래픽스 파이프라인에서 Set...() 시리즈는<br>
그렇게 부담되는 연산량은 아니다)<br>

```
ExampleApp - Render

if (m_drawAsWire)
    m_context->RSSetState(m_wireRasterizerSate.Get());
else
    m_context->RSSetState(m_solidRasterizerSate.Get());
```