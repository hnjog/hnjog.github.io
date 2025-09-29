---
title: "기하 쉐이더(GS)"
date : "2025-09-29 12:00:00 +0900"
last_modified_at: "2025-09-29T12:00:00"
categories:
  - 그래픽스
  - 수학
  - DirectX
tags:
  - 기하 쉐이더
  - Geometry Shader
---

## 기하 쉐이더(Geometry Shader,GS)

[![Image](https://github.com/user-attachments/assets/1faeaa91-7e77-4ed7-847b-a1956c273316)](https://github.com/user-attachments/assets/1faeaa91-7e77-4ed7-847b-a1956c273316){: .image-popup}<br>

기하 쉐이더는 VertexShaer와 PixelShader 사이에 존재하는 하나의 `스테이지`<br>

- 입력 받는 것 : `Primitive`(도형)<br>
- 출력 하는 것 : `수정된 Primitive`<br>
- 해당 과정을 통해, `동적인 수정 or 일부를 제거`하는 쉐이딩 단계이다<br>

기하 정보와 관련된<br>
`VS ~ GS` 를 별도로<br>
**Geometry Pipeline**이라 퉁치기도 함<br>

요점은 대략적인 Mesh를 받은 후<br>
- 고퀄리티 Mesh를 GPU에서 만듦(CPU -> GPU를 가볍게)<br>
- SubDivision 같은 기술을 이용하여 실시간 메시를 변형 가능<br>

IA에서 이미 삼각형이 정점 3개가 필요하다는 사실을 알고 있기에<br>
GS에 들어오는 것은 여러 정점 데이터가 한번에 들어올 수 있음<br>
(Primitive)<br>

### 구체적으로 하는 일? 사례?

- **추가적인 생성**<br>
  : Vertex 하나를 '사각형'으로 만들어줄 수 있음<br>
    (추가적인 폴리곤 생성으로 더 부드럽거나 독특한 도형으로 변형)<br>

- **LOD (Level of Detail)**<br>
  : 카메라와의 거리를 통해 삼각형을 늘리거나 줄임<br>

### 유의할 점들

- 동적인 Primitive 수정이 가능하지만<br>
  종종 병목 현상의 원인이 되기도 함<br>
  (최신 그래픽스의 경우, Compute Shader, Mesh Shader 등을 고려하는 편)<br>

- 한번의 GS 호출에서 출력하는 Vertex에 제한 존재<br>
  (D3D11 : 약 1024)<br>

- 쉐이더를 GSSetShader로 등록 시,<br>
  한번의 Draw 콜에서 '계속' 쉐이더로 남아있기에<br>
  다른 물체들이 GS를 사용하지 않는다면<br>
  GSSetShader(nullptr,0,0) 같은 방식으로<br>
  GS쉐이더의 사용을 해제해야 함<br>

```cpp
Render()
{
  context->GSSetConstantBuffers(0, 1, m_constantBuffer.GetAddressOf());
  context->GSSetShader(m_geometryShader.Get(), 0, 0);

  // draw Call...

  context->GSSetShader(nullptr, 0, 0);
}
```

이전 세부 단계는 나중에 다룰 예정<br>
(Hull,Domain, Tessellator)<br>



## 예제 - GS를 이용하여 정점을 삼각형으로 변환


[![Image](https://github.com/user-attachments/assets/2b21a314-835e-47b8-8a74-9e342e660361)](https://github.com/user-attachments/assets/2b21a314-835e-47b8-8a74-9e342e660361){: .image-popup}<br>

```cpp
// Geometry-Shader Object
// https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-geometry-shader

// Stream-Output Object
// https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-so-type

cbuffer BillboardPointsConstantData : register(b0)
{
    float3 eyeWorld;
    float width;
    Matrix model; // For vertex shader
    Matrix view; // For vertex shader
    Matrix proj; // For vertex shader
};

struct GeometryShaderInput
{
    float4 pos : SV_POSITION;
};

struct PixelShaderInput
{
    float4 pos : SV_POSITION; // not POSITION
    uint primID : SV_PrimitiveID;
};

//TODO: PointStream -> TriangleStream
[maxvertexcount(100)] // 최대 출력 Vertex 갯수
void main(point GeometryShaderInput input[1], uint primID : SV_PrimitiveID,
                              inout PointStream<PixelShaderInput> outputStream)
{
    PixelShaderInput output;
    
    output.pos = input[0].pos;
    
    for (int i = 0; i < 100; i ++)
    {
        output.pos = input[0].pos + float4(0.0, 0.003, 0.0, 0.0) * float(i);
        output.pos = mul(output.pos, view);
        output.pos = mul(output.pos, proj);
        output.primID = primID;

        outputStream.Append(output);
    }
}
```

- VS -> GS -> PS 이기에<br>
  VS에서 입력을 받고<br>
  GS에서는 PS의 결과물로 return 한다<br>
  (inout)<br>

- 예제 목표는 저 PointStream을 Triangle로 바꾸는 것<br>

- point GeometryShaderInput input[1]<br>
 - point : 입력 Primitive를 지정 (Point : 점 1개, line : 선 1개 등)<br>
 - GeometryShaderInput : 우리가 정의한 입력 구조체 타입<br>
 - [1] : 도형을 구성하는 정점 개수(Point라서 점 1개)<br>

- SV_PrimitiveID?<br>
 : 현재 쉐이더에서 처리 중인 도형의 고유 ID<br>
 (삼각형의 단위를 표현)<br>
 (PS 의 Output이며, ps에서 사용 가능)<br>
 (ex : 3번째 삼각형마다 색 변경 등)<br>
 (저 primID는 GPU가 넣어주는 값이기에 VS에서 생각할 필요 없음)<br>

- [maxvertexcount(100)] <br>
 : 최대 출력 Vertex (아까 말했듯 32비트 환경에선 1024로 제한됨)<br>

- 어차피 POINTLIST로 IA에서 설정하였고<br>
  정점 5개를 받아 각각 물체를 만들어주는 것이 목표<br>

### 예제 수정 코드

```cpp
[maxvertexcount(100)] // 최대 출력 Vertex 갯수
void main(point GeometryShaderInput input[1], uint primID : SV_PrimitiveID,
                              inout TriangleStream<PixelShaderInput> outputStream)
{
    
    float hw = 0.5 * width;
    
    PixelShaderInput o1;
    
    o1.pos = input[0].pos + float4(-hw, -hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    o1.primID = primID;
    outputStream.Append(o1);

    o1.pos = input[0].pos + float4(-hw, hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    outputStream.Append(o1);

    o1.pos = input[0].pos + float4(hw, hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    outputStream.Append(o1);

    outputStream.RestartStrip();

    o1.pos = input[0].pos + float4(-hw, -hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    outputStream.Append(o1);

    o1.pos = input[0].pos + float4(hw, hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    outputStream.Append(o1);

    o1.pos = input[0].pos + float4(hw, -hw, 0.0, 0.0);
    o1.pos = mul(o1.pos, view);
    o1.pos = mul(o1.pos, proj);
    outputStream.Append(o1);
}

```

- 삼각형으로 출력할 때는<br>
  output에 맞게 각 요소에<br>
  수정<br>
  - 시계 방향으로 위치를 조작<br>
  - 이후 outputStream에 Append<br>
  - 삼각형을 그린 후, RestartStrip을 통하여 끊어주기<br>

- 내부적으로 그릴때 TriangleStrip을 사용하기에<br>
  2가지 방식으로 구현이 가능하다<br>
  - POINTLIST 처럼 각각의 독립적인 3각형 2개를 합침 - 현재 방식<br>
  - Strip을 이용하여 정점 4개만을 이용하여 그리는 방식<br>

```cpp
만약 4개로 그린다면?
// 좌 하
o1.pos = input[0].pos + float4(-hw, -hw, 0.0, 0.0);
o1.pos = mul(o1.pos, view);
o1.pos = mul(o1.pos, proj);
o1.primID = primID;
outputStream.Append(o1);

// 좌 상
o1.pos = input[0].pos + float4(-hw, hw, 0.0, 0.0);
o1.pos = mul(o1.pos, view);
o1.pos = mul(o1.pos, proj);
outputStream.Append(o1);

// 우 하
o1.pos = input[0].pos + float4(hw, -hw, 0.0, 0.0);
o1.pos = mul(o1.pos, view);
o1.pos = mul(o1.pos, proj);
outputStream.Append(o1);

// 우 상
o1.pos = input[0].pos + float4(hw, hw, 0.0, 0.0);
o1.pos = mul(o1.pos, view);
o1.pos = mul(o1.pos, proj);
outputStream.Append(o1);
    
```

결과<br>

[![Image](https://github.com/user-attachments/assets/be8e1d3f-21ab-4083-b10c-db9e6b66095e)](https://github.com/user-attachments/assets/be8e1d3f-21ab-4083-b10c-db9e6b66095e){: .image-popup}<br>

- 첫 예제라 비교적 가벼운 내용이었다<br>