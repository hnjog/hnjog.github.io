---
title: "Cube Mapping"
date : "2025-09-08 14:00:00 +0900"
last_modified_at: "2025-09-08T14:00:00"
categories:
  - Direct X
tags:
  - Cube Mapping
---

## Cube Mapping

[![Image](https://github.com/user-attachments/assets/52c61d7f-3849-4db6-939a-038e12fef1b7)](https://github.com/user-attachments/assets/52c61d7f-3849-4db6-939a-038e12fef1b7){: .image-popup}<br>

플레이어를 거대한 '큐브' 안에 가두고<br>
각 면에 텍스쳐를 발라<br>
'배경' 등을 표현하여 사용<br>

- Skybox / Skymap 등에 대한 구현은 대부분 큐브맵 기반<br>
- 6방향 텍스쳐를 이용한 '무한히 먼 배경'을 표현<br>
- 단순한 샘플링으로 인한 하드웨어 친화적 (빠르다!)<br>
- skybox, reflection prove 등 다양한 그래픽에 응용 가능<br>
- 현대 게임 엔진에서는 '원형'을 기준으로한 텍스쳐를 받아<br>
  '큐브맵'으로 변환한 후 이용한다고 함<br>

이전 강의 기준으로<br>
배경을 구현해본 적은 있다<br>
(texture를 직접 배치하는 방식)<br>

- 따라서 DirectX가 제공하는 기능을 이용하여<br>
  예제를 구현해보자<br>

- .dds?<br>
  : Direct X 에서 '이미지'를 저장할때 사용하는 포맷<br>
   (2D,3D 텍스쳐(volume), 큐브맵 등을 저장 가능)<br>

- dds를 이용하여 큐브맵 구현<br>

- 직접 만들려한다면 '파노라마' 같은 360도 공간 촬영 이미지 필요<br>
  (해보겠다면 Panorama to cubemap 같은 키워드를 기억해두자)<br>
  (Humus 같은 사이트도 존재한다)<br>

## 큐브맵 구현 코드 부분

```cpp
void ExampleApp::InitializeCubeMapping() {

    // texassemble.exe cube -w 2048 -h 2048 -o saintpeters.dds posx.jpg negx.jpg
    // posy.jpg negy.jpg posz.jpg negz.jpg texassemble.exe cube -w 2048 -h 2048
    // -o skybox.dds right.jpg left.jpg top.jpg bottom.jpg front.jpg back.jpg -y
    // https://github.com/Microsoft/DirectXTex/wiki/Texassemble

    // .dds 파일 읽어들여서 초기화
    ComPtr<ID3D11Texture2D> texture;
    auto hr = CreateDDSTextureFromFileEx(
        // this->m_device.Get(), L"./SaintPetersBasilica/saintpeters.dds", 0,
        this->m_device.Get(), L"./skybox/skybox.dds", 0, D3D11_USAGE_DEFAULT,
        D3D11_BIND_SHADER_RESOURCE, 0,
        D3D11_RESOURCE_MISC_TEXTURECUBE, // 큐브맵용 텍스춰
        DDS_LOADER_FLAGS(false), (ID3D11Resource **)texture.GetAddressOf(),
        this->m_cubeMapping.cubemapResourceView.GetAddressOf(), nullptr);

    if (FAILED(hr)) {
        std::cout << "CreateDDSTextureFromFileEx() failed" << std::endl;
    }

    m_cubeMapping.cubeMesh = std::make_shared<Mesh>();

    m_BasicVertexConstantBufferData.model = Matrix();
    m_BasicVertexConstantBufferData.view = Matrix();
    m_BasicVertexConstantBufferData.projection = Matrix();
    ComPtr<ID3D11Buffer> vertexConstantBuffer;
    ComPtr<ID3D11Buffer> pixelConstantBuffer;
    AppBase::CreateConstantBuffer(m_BasicVertexConstantBufferData,
                                  m_cubeMapping.cubeMesh->vertexConstantBuffer);
    AppBase::CreateConstantBuffer(m_BasicPixelConstantBufferData,
                                  m_cubeMapping.cubeMesh->pixelConstantBuffer);

    // 커다란 박스 초기화
    // - 세상이 커다란 박스 안에 갇혀 있는 구조입니다.
    // - D3D11_CULL_MODE::D3D11_CULL_NONE 또는 삼각형 뒤집기
    // - 예시) std::reverse(myvector.begin(),myvector.end());
    MeshData cubeMeshData = GeometryGenerator::MakeBox(20.0f);
    std::reverse(cubeMeshData.indices.begin(), cubeMeshData.indices.end());

    AppBase::CreateVertexBuffer(cubeMeshData.vertices,
                                m_cubeMapping.cubeMesh->vertexBuffer);
    m_cubeMapping.cubeMesh->m_indexCount = UINT(cubeMeshData.indices.size());
    AppBase::CreateIndexBuffer(cubeMeshData.indices,
                               m_cubeMapping.cubeMesh->indexBuffer);

    // 쉐이더 초기화

    // 다른 쉐이더와 동일한 InputLayout 입니다.
    // 실제로는 "POSITION"만 사용합니다.
    vector<D3D11_INPUT_ELEMENT_DESC> basicInputElements = {
        {"POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0,
         D3D11_INPUT_PER_VERTEX_DATA, 0},
        {"NORMAL", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 4 * 3,
         D3D11_INPUT_PER_VERTEX_DATA, 0},
        {"TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, 4 * 3 + 4 * 3,
         D3D11_INPUT_PER_VERTEX_DATA, 0},
    };

    AppBase::CreateVertexShaderAndInputLayout(
        L"CubeMappingVertexShader.hlsl", basicInputElements,
        m_cubeMapping.vertexShader, m_cubeMapping.inputLayout);

    AppBase::CreatePixelShader(L"CubeMappingPixelShader.hlsl",
                               m_cubeMapping.pixelShader);

    // 기타
    // - 텍스춰 샘플러도 다른 텍스춰와 같이 사용
}
```

[Texassemble?](https://github.com/Microsoft/DirectXTex/wiki/Texassemble){: .image-popup}<br>

- DirectXTex 라이브러리에 포함된 명령줄 유틸리티<br>
  (텍스쳐들을 '조립'하여 하나의 dds 텍스쳐를 만들어줌)<br>

- texassemble.exe를 cmd를 통해서 열면 도움말이 나온다<br>

ex)<br>

```sql
texassemble.exe cube -w 2048 -h 2048 -o saintpeters.dds posx.jpg negx.jpg posy.jpg negy.jpg posz.jpg negz.jpg
```

이런식으로 6장의 이미지를 dds 큐브맵으로 조립시킨다<br>
(-o : 저장할 파일 이름)<br>

그외에도<br>
volume 텍스쳐, texture array 생성 등에도 이용 가능<br>

```cpp
#pragma once

#include <wrl.h>

#include "GeometryGenerator.h"
#include "Material.h"
#include "Vertex.h"

namespace hlab {

using Microsoft::WRL::ComPtr;

struct CubeMapping {

    std::shared_ptr<Mesh> cubeMesh;

    ComPtr<ID3D11ShaderResourceView> cubemapResourceView; 

    ComPtr<ID3D11VertexShader> vertexShader;
    ComPtr<ID3D11PixelShader> pixelShader;
    ComPtr<ID3D11InputLayout> inputLayout;
};
} // namespace hlab
```

CubeMapping 구조체를 이용<br>

- CreateDDSTextureFromFileEx?<br>
  : 텍스쳐와 srv 지정<br>
  (텍스쳐는 임시용을 사용하고<br>
  srv를 가져다 저장하는 목적이 크다)<br>

- D3D11_RESOURCE_MISC_TEXTURECUBE를 통하여<br>
  큐브맵용 텍스쳐라는 것을 지정<br>

- '기본적'인 큐브매핑에선 픽셀 쉐이더에 constants를 사용할 필요는 없음<br>
  (나중에 응용하는 경우 제외)<br>
  그래서 형식적으로 생성만 해준다<br>

- 플레이어부터 모든 것이 cubemap에 갇혀있는 구조이므로<br>
  '인덱스'를 뒤집거나 cull_mode를 D3D11_CULL_NONE로 바꾸어<br>
  삼각형을 뒤집어 주어야한다<br>
  (그래야 '안쪽'인 큐브맵이 제대로 보일테니)<br>
  (예제에서는 'indices'를 reverse 해준다)<br>

- Normal과 TexCoord 부분은 사용하지 않지만<br>
  VertexShaderInput 에 '통일'하기 위하여<br>
  맞춰준다<br>
  (Position만 사용)<br>
  (Position만 사용하도록 shader input을 추가하면 연산을 깔끔하게 줄일 수 있음)<br>
  (그래도 기능이 다르므로 shader용 hlsl은 분리)<br>

### 큐브맵 Shader 코드

```cpp
Vertex Shader
#include "Common.hlsli"

cbuffer BasicVertexConstantBuffer : register(b0)
{
    matrix model;
    matrix invTranspose;
    matrix view;
    matrix projection;
};

PixelShaderInput main(VertexShaderInput input)
{
    // 불필요한 멤버들도 VertexShaderInput을 통일시켰기 때문에 채워줘야 합니다.
    
    PixelShaderInput output;
    float4 pos = float4(input.posModel, 1.0f);

    pos = mul(pos, model); // Identity

    output.posWorld = pos.xyz;
    
    float4 normal = float4(input.normalModel, 0.0f);
    output.normalWorld = mul(normal, invTranspose).xyz;
    output.normalWorld = normalize(output.normalWorld);

    pos = mul(pos, view);
    
    pos = mul(pos, projection);
    output.posProj = pos;

    output.texcoord = input.texcoord;
    output.color = float3(1.0, 1.0, 0.0);

    return output;
}
```

- 모델 좌표에서 World 좌표로 변환한 시점의 posWorld만 미리 저장해둔다<br>

```cpp
Pixel Shader
#include "Common.hlsli" // 쉐이더에서도 include 사용 가능

TextureCube g_textureCube0 : register(t0);
SamplerState g_sampler : register(s0);

float4 main(PixelShaderInput input) : SV_TARGET
{
    // 주의: 텍스춰 좌표가 float3 입니다.
    return g_textureCube0.Sample(g_sampler, input.posWorld.xyz);
}
```

- 큐브맵의 posWorld 만을 이용하여 텍스쳐링을 진행한다<br>

- 텍스쳐 좌표가 3차원 좌표(float3)임에 유의<br>
  (자세히 보면 TextureCube라는 타입으로 받고 있다)<br>

[![Image](https://github.com/user-attachments/assets/4415514b-2219-41f1-ba54-87ff903fc8a0)](https://github.com/user-attachments/assets/4415514b-2219-41f1-ba54-87ff903fc8a0){: .image-popup}<br>

- 우리 캐릭터나 물체는 '큐브맵' 내부에 존재하기에<br>
  6장의 정사각형 텍스쳐를 '한 점'(원점)에서 바라본 방향으로<br>
  샘플링을 하는 방식이기 때문<br>
  (그렇기에 해당 텍스쳐 좌표는 '쳐다보는 방향'으로 받는다)<br>
  (관측자는 큐브맵의 중앙에 있다는 가정)<br>
  (어차피 도달하지 못하는 것이 skymap 같은 것이니)<br>

- 따라서 '방향'을<br>
  위치 좌표 - '원점' 이지만<br>
  '원점'이 0,0,0 으로 표현하기에<br>
  위치 좌표를 방향 좌표처럼 이용하는 모습<br>

- 실제로는 normalize를 하는것이<br>
  더 안정적이고 '방향'을 표현하는 좋은 방법일 수 있다<br>
  (return g_textureCube0.Sample(g_sampler, normalize(input.posWorld.xyz));)<br>

### 큐브맵에서 사용할 Model 변환은 함부로 건들지 말 것
```cpp
ExamplaApp.cpp - Update()
{
  m_BasicVertexConstantBufferData.model =
      Matrix::CreateScale(m_modelScaling) *
      Matrix::CreateRotationY(m_modelRotation.y) *
      Matrix::CreateRotationX(m_modelRotation.x) *
      Matrix::CreateRotationZ(m_modelRotation.z) *
      Matrix::CreateTranslation(m_modelTranslation);
  m_BasicVertexConstantBufferData.model =
      m_BasicVertexConstantBufferData.model.Transpose();

  m_BasicVertexConstantBufferData.invTranspose =
      m_BasicVertexConstantBufferData.model;
  m_BasicVertexConstantBufferData.invTranspose.Translation(Vector3(0.0f));
  m_BasicVertexConstantBufferData.invTranspose =
      m_BasicVertexConstantBufferData.invTranspose.Transpose().Invert();
  ...일반적인 모델 좌표 업데이트 버퍼 완료...!

  // 큐브매핑을 위한 ConstantBuffers
  m_BasicVertexConstantBufferData.model = Matrix();
  // Transpose()도 생략 가능
  // 시점(View) 변환에선 '이동'도 취소해야 함 (이번 예제는 움직이지 않기에 pass)

  AppBase::UpdateBuffer(m_BasicVertexConstantBufferData,
                        m_cubeMapping.cubeMesh->vertexConstantBuffer);
}

```

- 큐브 매핑에서 '변환'에 주의할것!<br>
- Model 변환은 전체적인 '배경'에 해당하는 큐브맵에 적용하지 말아야 함!<br>
- View의 경우 '이동' 등에 따라 '배경'이 움직이면 안되므로 해당 부분 적용을 취소해야 함<br>

- 차후 큐브맵을 다루게 되는 경우, 다시 다룰 부분이므로 유의할 것<br>

## 큐브맵 Render 구현 예제

```cpp
ExamplaApp.cpp - Render()
{
  ...

  // 큐브매핑
  // TODO:
  m_context->VSSetShader(m_cubeMapping.vertexShader.Get(), 0, 0);
  m_context->VSSetConstantBuffers(
      0, 1, m_cubeMapping.cubeMesh->vertexConstantBuffer.GetAddressOf());

  m_context->PSSetSamplers(0, 1, m_samplerState.GetAddressOf());
  m_context->PSSetShader(m_cubeMapping.pixelShader.Get(), 0, 0);
  // Views[] 행렬을 이용하여 적용할 수도 있음
  m_context->PSSetShaderResources(0, 1, m_cubeMapping.cubemapResourceView.GetAddressOf());

  m_context->IASetInputLayout(m_cubeMapping.inputLayout.Get());
  m_context->IASetVertexBuffers(
      0, 1, m_cubeMapping.cubeMesh->vertexBuffer.GetAddressOf(),
                                &stride, &offset);
  m_context->IASetIndexBuffer(m_cubeMapping.cubeMesh->indexBuffer.Get(),
                              DXGI_FORMAT_R32_UINT,
                              0);
  m_context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

  m_context->DrawIndexed(m_cubeMapping.cubeMesh->m_indexCount, 0, 0);

... // 이후에 모델들을 그려줌
}
```

- IA 버퍼 세팅<br>
- 버텍스 쉐이더 세팅<br>
- PS의 srv 등의 세팅<br>
- drawIndex<br>

```cpp
// Views[] 행렬을 이용하여 적용할 수도 있음
  m_context->PSSetShaderResources(0, 1, m_cubeMapping.cubemapResourceView.GetAddressOf());
```

이전에 저장해둔 cubeMapResourceView(SRV)를 이용<br>

- 일단 createTeture를 통해 Gpu 리소스가 할당됨<br>
  그리고 Srv를 만들때 해당 리소스를 여전히 참조하고 있기에<br>
  리소스의 수명 보장<br>
  (참조 카운트가 유지되는 중)<br>
- MISC_TEXTURECUBE로 플래그를 준 srv를<br>
  Cubemap로 받은 후<br>
  pixel shader에서 sample 하는 것<br>

```cpp

ID3D11ShaderResourceView *views[1] = {
    m_cubeMapping.cubemapResourceView.Get()};
  m_context->PSSetShaderResources(0, 1, views);
```

- srv를 하나 이상 사용할 수 있기에 배열로 만들어 사용 가능<br>

- 해당 '그리는' 코드를 모델 이후에 넣어도 정상 동작함<br>
  (배경을 '가장 늦게' 그려서 다 덮어 씌우는거 아닌가?)<br>
  (Depth Buffer가 있어서 괜찮다!)<br>

- cubemap을 만들때 '지나치게 크게 만들면', 그에 맞게 farz 값을<br>
  수정해야 절두체에 cubemap이 반영된다<br>
  (다만 이건 우리가 실제 '큐브맵'을 기하적으로<br>
  그려주는 방식이기에 발생하는 문제임을 유의하자)<br>
  
- 기하를 sphere로 바꾸어도 딱히 문제는 딱히 없는 편<br>
  (근데 사실 sphere로 바꾸고 폴리곤을 많이 잡아도 크게 나아지는 모습은 없다)<br>
  (-> 텍스쳐링에서 중요한건 텍스쳐의 품질 이라는 점을 알아두자!)<br>

## 결과물

[![Image](https://github.com/user-attachments/assets/3bb34eae-56fa-45a3-8e63-8176f6ed30fe)](https://github.com/user-attachments/assets/3bb34eae-56fa-45a3-8e63-8176f6ed30fe){: .image-popup}<br>

큐브맵이 잘 먹는 모습이다!<br>

여담으로<br>
최근에는 GPU가 많이 발전하였기에<br>
pixel shader에서 여러 효과를 다양하게 사용할 수 있음<br>
-> 이걸 감안하여 여러 멋진 효과를 만들 수 있다<br>

- 이전 시간에 배운 Rim Light를 추가로 적용해보면?<br>

[![Image](https://github.com/user-attachments/assets/e431ee73-f3f4-4400-88a8-51072ea66b5c)](https://github.com/user-attachments/assets/e431ee73-f3f4-4400-88a8-51072ea66b5c){: .image-popup}<br>

살짝 주황색에 가까운 빛이 젤다의 테두리 근처에 보여<br>
좀 더 어울리는 모습이라 생각한다!<br>