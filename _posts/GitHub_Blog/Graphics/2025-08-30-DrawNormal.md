---
title: "Normal 그리기"
date : "2025-08-30 18:30:00 +0900"
last_modified_at: "2025-08-30T18:30:00"
categories:
  - Direct X
tags:
  - Normal
---

## 디버깅을 위한 Normal 렌더링<br>

지난번에는 WireFrame 이었고<br>
이번에는 정점의 Normal을 그리는 기능을 추가하였다<br>

- 필요한 이유?<br>
  - 뒤집힌 면 확인 : 노멀이 반대로 나올 수 있음<br>
  - 하드엣지 감지 : 같은 삼각형이나 노멀은 다르게 적용된 버텍스 스플릿(하드엣지) 상태를 감지<br>
  - 스케일 / 트랜스폼 이슈 : 각 노멀의 '크기'에 따른 몇몇 이슈 체크 가능<br>
  그 외에도 여러 상황에 대한 검증에 사용 가능<br>

## 예제 - Normal 을 버튼을 누름으로서 그림

<img width="2229" height="1733" alt="Image" src="https://github.com/user-attachments/assets/ed5647d0-30c8-455f-9a93-ea045bab34fb" /><br>

이미 Normal 에 관한 정보를 쉐이더로 보내는데<br>
그냥 그리면 되는거 아닌가?<br>

싶지만 DirectX 자체에서<br>
Normal을 분류하여 그리는 그런식의 기능은 따로 없음<br>

그렇기에 박스 처럼 따로 draw 를 호출하여 그려주어야 함<br>

- 별도의 노멀용 쉐이더(VS)<br>

```
#include "Common.hlsli"

cbuffer NormalVertexConstantBuffer : register(b0)
{
    matrix model;
    matrix invTranspose;
    matrix view;
    matrix projection;
    float scale; // 그려지는 선분의 길이 조절
};

PixelShaderInput main(VertexShaderInput input)
{
    PixelShaderInput output;
    float4 pos = float4(input.posModel, 1.0f);

    // Normal 먼저 변환
    float4 normal = float4(input.normalModel, 0.0f);
    output.normalWorld = mul(normal, invTranspose).xyz;
    output.normalWorld = normalize(output.normalWorld);
    
    pos = mul(pos, model);
    
    float t = input.texcoord.x;
    
    pos.xyz += output.normalWorld * t * scale; // 정점의 위치를 옮기고 TriangleList로 Line과 비슷한 모습 생성

    output.posWorld = pos.xyz;
    
    pos = mul(pos, view);
    pos = mul(pos, projection);

    output.posProj = pos;
    output.texcoord = input.texcoord;
    
    // 시작점은 노란색으로, 끝점에 가까울수록 빨간색
    output.color = float3(1.0, 1.0, 0.0) * (1.0 - t) + float3(1.0, 0.0, 0.0) * t;

    return output;
}
```

- 별도의 노멀용 쉐이더(PS)<br>
  (딱히 하는 일 없음)<br>

```
#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

float4 main(PixelShaderInput input) : SV_TARGET
{
    return float4(input.color, 1.0f);
}
```

- Mesh 클래스<br>
  (각 메시마다 가질 버텍스,인덱스 버퍼 및
  vertex constant, pixel constant 버퍼)

```
Mesh.h

#pragma once

#include <d3d11.h>
#include <windows.h>
#include <wrl.h> // ComPtr

namespace hlab {

using Microsoft::WRL::ComPtr;

// 같은 메쉬를 여러번 그릴 때 버퍼들을 재사용
struct Mesh {

    ComPtr<ID3D11Buffer> m_vertexBuffer;
    ComPtr<ID3D11Buffer> m_indexBuffer;
    ComPtr<ID3D11Buffer> m_vertexConstantBuffer;
    ComPtr<ID3D11Buffer> m_pixelConstantBuffer;

    UINT m_indexCount = 0;
};
} // namespace hlab
```

- Normal을 그려줄 요소들의 선언<br>
 : vs,ps, inputlayout(이건 박스와 동일한 옵션 사용)<br>

```
ExampleApp.h

// 노멀 쉐이더 쪽으로 보내줘야 할 Constant data들
struct NormalVertexConstantBuffer {
    Matrix model;
    Matrix invTranspose;
    Matrix view;
    Matrix projection;
    float scale = 0.1f;
    float dummy[3]; // 패딩용
};

{
// 노멀 벡터 그리기
ComPtr<ID3D11VertexShader> m_normalVertexShader;
ComPtr<ID3D11PixelShader> m_normalPixelShader;
//ComPtr<ID3D11InputLayout> m_normalInputLayout; // 같이 사용

shared_ptr<Mesh> m_normalLines;
NormalVertexConstantBuffer m_normalVertexConstantBufferData;
}
```

- Init<br>
  : 버텍스 버퍼, 인덱스 버퍼, C Buffer와 쉐이더를 생성<br>

```
ExampleApp.cpp

bool ExampleApp::Initialize() {
  ...

  std::vector<Vertex> normalVertices;
  std::vector<uint16_t> normalIndices;

  // 박스의 vertex 하나당 2개씩 만들어준다
  for (size_t i = 0; i < meshData.vertices.size(); i++) {

      auto v = meshData.vertices[i];

      v.texcoord.x = 0.0f; // 시작점 표시
      normalVertices.push_back(v);

      v.texcoord.x = 1.0f; // 끝점 표시
      normalVertices.push_back(v);

      normalIndices.push_back(uint16_t(2 * i));
      normalIndices.push_back(uint16_t(2 * i + 1));
  }

  // 노멀용 버텍스 버퍼 생성
  AppBase::CreateVertexBuffer(normalVertices,
                            m_normalLines->m_vertexBuffer);

  // 노멀용 인덱스 버퍼 생성                           
  m_normalLines->m_indexCount = UINT(normalIndices.size());
  AppBase::CreateIndexBuffer(normalIndices, m_normalLines->m_indexBuffer);

  // 노멀용 Vertex Shader 생성(InputLayout은 공용으로 사용)
  AppBase::CreateVertexShaderAndInputLayout(
      L"NormalVertexShader.hlsl", basicInputElements, m_normalVertexShader,
      m_basicInputLayout);

  // 노멀용 Pixel Shader 생성
  AppBase::CreatePixelShader(L"NormalPixelShader.hlsl", m_normalPixelShader);

  AppBase::CreateConstantBuffer(m_normalVertexConstantBufferData,
                                m_normalLines->m_vertexConstantBuffer);
}
```

- Update<br>
  : NormalConstantBuffer의 데이터를 업데이트<br>
  
```
ExampleApp.cpp

// 노멀 벡터 그리기
if (m_drawNormals) {
    // 기존 사용한 박스의 변환을 그대로 사용
    m_normalVertexConstantBufferData.model =
        m_BasicVertexConstantBufferData.model;

    m_normalVertexConstantBufferData.invTranspose =
        m_BasicVertexConstantBufferData.invTranspose;

    m_normalVertexConstantBufferData.view =
        m_BasicVertexConstantBufferData.view;

    m_normalVertexConstantBufferData.projection =
        m_BasicVertexConstantBufferData.projection;

    // Constant를 CPU에서 GPU로 복사
    AppBase::UpdateBuffer(m_normalVertexConstantBufferData,
                          m_normalLines->m_vertexConstantBuffer);
}
```

- Render<br>
  : Shader 및 버퍼에 대한 설정,<br>
    Draw Call<br>

```
ExampleApp.cpp

UINT stride = sizeof(Vertex);
UINT offset = 0;

// 노멀 벡터 그리기
if (m_drawNormals) {

    // TODO: 여기에 필요한 내용들 작성
    m_context->VSSetShader(m_normalVertexShader.Get(), 0, 0);
    m_context->VSSetConstantBuffers(
        0, 1, m_normalLines->m_vertexConstantBuffer.GetAddressOf());

    m_context->PSSetShader(m_normalPixelShader.Get(), 0, 0);

    // 버텍스/인덱스 버퍼 설정

    m_context->IASetVertexBuffers(
        0, 1, m_normalLines->m_vertexBuffer.GetAddressOf(), &stride,
        &offset);
    m_context->IASetIndexBuffer(m_normalLines->m_indexBuffer.Get(),
                                DXGI_FORMAT_R16_UINT, 0);
    
    m_context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_LINELIST);
    m_context->DrawIndexed(m_normalLines->m_indexCount, 0, 0);
}
```

### 이미 두 ConstantBuffer 똑같이 model ,view ,projection을 사용중인데 중복아닌가?

Normal 쪽의 <br>
matrix model;<br>
matrix invTranspose;<br>
matrix view;<br>
matrix projection;<br>

요소들을 제거하고<br>
mesh에 존재하는 요소를 사용하면 된다<br>

```
ID3D11Buffer *pptr[2] = {m_mesh->m_vertexConstantBuffer.Get(),
                        m_normalLines->m_vertexConstantBuffer.Get()};

m_context->VSSetConstantBuffers(0, 2, pptr);
```

이후 Shader의 코드를 수정<br>

```
// Normal vertex Shader (HLSL)

cbuffer BasicVertexConstantBuffer : register(b0)
{
    matrix model;
    matrix invTranspose;
    matrix view;
    matrix projection;
};

cbuffer NormalVertexConstantBuffer : register(b1)
{
    float scale; // 그려지는 선분의 길이 조절
};
```

그외에도 Scale 값이 변하지 않는다면 굳이 Update 할 필요가 없긴 하다<br>
(애초에 Update에 있는 내용을 UpdateGUI 쪽으로 넘겨주거나<br>
 Bool 변수를 추가(더티 플래그)하여 Scale 값 변환을 체크 가능)<br>

