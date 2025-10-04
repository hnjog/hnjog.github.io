---
title: "기하 분할(Tessellation)"
date : "2025-10-04 12:00:00 +0900"
last_modified_at: "2025-10-04T12:00:00"
categories:
  - 그래픽스
  - 수학
  - DirectX
tags:
  - Tessellation
  - 기하 분할
---

## 기하 분할(Tessellation)

[![Image](https://github.com/user-attachments/assets/d2711346-fd35-4d2c-8aa7-e85b896ce637)](https://github.com/user-attachments/assets/d2711346-fd35-4d2c-8aa7-e85b896ce637){: .image-popup}<br>

개념적으로 **'하나의 면'을 다른 조각으로 쪼개는 것**을 의미<br>

DX 파이프라인의 하나의 단계<br>
VS -> Hull Shader -> **Tessellation** -> Domain Shader -> GS 순서<br>

- 이것을 이용해 시점과의 거리를 통한 '정밀도'의 차이를 구현 가능<br>
  (Level Of Detail, LOD)<br>

### Tessellation와 관련 스테이지

- Hull Shader(HS)<br>
 - VS에서 Patch 단위로 입력을 받음<br>
 - 출력은 '제어점' 과 테셀레이션 팩터<br>
 - DX에서 [patchconstantfunc] 속성을 통해 Tessellation Factor를 계산하여 넘김<br>

- Tessellation<br>
 - 넘겨받은 Tessellation Factor 를 기반으로 삼각형 /사각형 /도형 의 Patch를 세분화<br>
 - 출력으로 수정된 결과물을 반환함<br>
 - 고정 파이프라인이라 코드 수정은 불가능, 옵션을 주어 원하는 결과를 유도<br>

- Domain Shader(DS)<br>
 - Tessellation에서 생성한 '도메인 위치'를 받음<br>
 - 실제 3D 공간 좌표로 변환하여 GS로 넘겨줌<br>
 - 보간 (Barycentric 등), 곡면 방정식 등의 적용이 이루어짐<br>


그렇기에 테셀레이션을 이용하려면<br>
Hull Shader 와 Domain Shader를 작성해야 함<br>

HS & DS 는 같이 구현해야 함<br>
GS 를 사용할 때, HS와 DS가 옵션이듯<br>
HS와 DS 를 이용할 때도 GS는 필수로 구현하지 않아도 됨<br>
(테셀레이션을 구현함에 따라 충분히 모델에 대한 세부적인 표현이 가능하기도 함)<br>

### 최신 쉐이더 방식?

[![Image](https://github.com/user-attachments/assets/709fecb5-c91c-4e60-8109-920c0bf0d495)](https://github.com/user-attachments/assets/709fecb5-c91c-4e60-8109-920c0bf0d495){: .image-popup}<br>

과거의 DX 레거시의 경우는 이전에 배운 내용이지만<br>
'쉐이더'가 다소 많이 필요하게 됨<br>
(DX에서 세부적으로 작성하는 쉐이더가 5개나 된다...)<br>

최신 쉐이더의 경우는<br>
쉐이더 단계를 단순화 하는 쪽으로 변하는 경향이 있다고 한다<br>
(3개의 쉐이더만 관리하면 된다!)<br>

## 예제 코드 

주어지는 `컨트롤 포인트`를 이용하여 테셀레이션을 시도해보자!<br>

- Control Point?<br>

[![Image](https://github.com/user-attachments/assets/b4884b58-ba81-4593-96b7-2dc924aafe3e)](https://github.com/user-attachments/assets/b4884b58-ba81-4593-96b7-2dc924aafe3e){: .image-popup}<br>

그래픽스에서 b처럼 세부적인 폴리곤들을 일일이 지정하는 것보다<br>
a처럼 '대략적인 지점'을 통해 상세적인 모델로 만들기 위해<br>
제공되는 것들을 컨트롤 포인트라 한다<br>
(제어가 가능한 요소)<br>

이러한 컨트롤 포인트를 다루는 것이 Hull Shader<br>

[![Image](https://github.com/user-attachments/assets/9543e1a5-d241-4302-b935-7d3c78c30df6)](https://github.com/user-attachments/assets/9543e1a5-d241-4302-b935-7d3c78c30df6){: .image-popup}<br>

그림처럼 작은 사각형/삼각형 들을 Patch라 하고<br>
저들을 모아 '곡면'을 정의할 수 있음<br>

Hull 이란 표현은 '껍데기'이며<br>
Patch들이 모여 만들어진 것이 '곡면'의 껍데기 역할을 하고 있기에<br>
Hull Shader 라고 부름<br>


### 생성 과정

```cpp
.h

// Hull shader -> Tessellation stage -> Domain shader
ComPtr<ID3D11HullShader> m_hullShader;
ComPtr<ID3D11DomainShader> m_domainShader;

---

.cpp

// 각각 내부에서 device->CreateHullShader / device->CreateDomainShader 호출
D3D11Utils::CreateHullShader(device, L"TessellatedQuadHS.hlsl",
                              m_hullShader);

D3D11Utils::CreateDomainShader(device, L"TessellatedQuadDS.hlsl",
                                m_domainShader);

```

### Render시 추가 처리

```cpp
// Hull shader
context->HSSetShader(m_hullShader.Get(), 0, 0);
context->HSSetConstantBuffers(0, 1, m_constantBuffer.GetAddressOf());

// Domain shader
context->DSSetShader(m_domainShader.Get(), 0, 0);
context->DSSetSamplers(0, 1, m_samplerState.GetAddressOf());
context->DSSetConstantBuffers(0, 1, m_constantBuffer.GetAddressOf());

// 토폴로지를 4개의 Control Point로 설정
context->IASetPrimitiveTopology(
    D3D_PRIMITIVE_TOPOLOGY_4_CONTROL_POINT_PATCHLIST);
context->Draw(m_indexCount, 0);

// HS, DS를 사용하지 않는 다른 물체들을 위해 nullptr로 설정
context->HSSetShader(nullptr, 0, 0);
context->DSSetShader(nullptr, 0, 0);
```

다른 쉐이더 와 비슷<br>
Topology 설정이 다소 특이한데<br>

테셀레이션이 '패치' 단위로 동작하기에<br>
Control Point 집합을 넘겨줘야 함<br>
-> DX에서 테셀레이션용의 별도 Topology 를 구분함<br>

- D3D_PRIMITIVE_TOPOLOGY_(N : 1~32)_CONTROL_POINT_PATCHLIST<br>
  (해당하는 N의 개수에 따라 삼각형, 사각형, 오각형 ... 등의 각 패치를 구분)<br>
  (곡선은 보통 2로 결정)<br>


## Hull Shader

```cpp
cbuffer ConstantData : register(b0)
{
    float3 eyeWorld;
    float width;
    Matrix model;
    Matrix view;
    Matrix proj;
    float time = 0.0f;
    float3 padding;
    float4 edges;
    float2 inside;
    float2 padding2;
};

struct VertexOut
{
    float4 pos : POSITION;
};

struct HullOut
{
    float3 pos : POSITION;
};

struct PatchConstOutput
{
    float edges[4] : SV_TessFactor;
    float inside[2] : SV_InsideTessFactor;
};

PatchConstOutput MyPatchConstantFunc(InputPatch<VertexOut, 4> patch,
                                     uint patchID : SV_PrimitiveID)
{
    PatchConstOutput pt;
    
    pt.edges[0] = edges[0];
    pt.edges[1] = edges[1];
    pt.edges[2] = edges[2];
    pt.edges[3] = edges[3];	
    pt.inside[0] = inside[0];
    pt.inside[1] = inside[1];
	
    return pt;
}

[domain("quad")]
[partitioning("integer")]
[outputtopology("triangle_cw")]
[outputcontrolpoints(4)]
[patchconstantfunc("MyPatchConstantFunc")]
[maxtessfactor(64.0f)]
HullOut main(InputPatch<VertexOut, 4> p,
           uint i : SV_OutputControlPointID,
           uint patchId : SV_PrimitiveID)
{
    HullOut hout;
	
    hout.pos = p[i].pos.xyz;

    return hout;
}

```

- VS의 입력값 - Pos를 받음<br>

- edges, inside<br>
  : 가장자리를 쪼개는 수치와 xy 방향을 쪼개는 용도의 수치를 constant로 받음<br>
  (GUI 에서 받음)<br>

- HS의 출력은 크게 2가지로<br>
  - VS 받은 위치에서 변환한 위치에 대한 값(HullOut)<br>
  - PatchConstantFunc를 추가로 반환<br>
    (MyPatchConstantFunc 가 PatchConstOutput 를 반환)<br>
    (어떻게 쪼개줄지에 대한 데이터)<br>

- float edges[4] : SV_TessFactor<br>
  : 쿼드 패치의 4변에 대한 팩터<br>
    (외곽선을 나눌 세분화의 정도, 분할 선 수)<br>

- float inside[2] : SV_InsideTessFactor<br>
  : 패치 내부를 얼마나 쪼갤지에 대한, 함수가 채워야 하는 값<br>
   (내부의 점 밀도)<br>

- [domain("quad")]<br>
 : 쿼드 도메인(UV 2D)에서 테셀레이션<br>
 (사각형)<br>

- [partitioning("integer")]<br>
 : 테셀 팩터를 정수 그리드로 해석(쪼개는 선 수가 정수)<br>
   fractional_even/odd를 쓰면 부분/홀수 분할<br>

- [outputtopology("triangle_cw")]<br>
   : 테셀레이터가 생성한 조각을 시계방향 삼각형으로 래스터라이즈하도록 명시<br>

- [outputcontrolpoints(4)]<br>
  : 이 패치는 제어점 4개(IA: 4_CONTROL_POINT_PATCHLIST)를 사용<br>

- [patchconstantfunc("MyPatchConstantFunc")]<br>
  : 위의 패치 상수 함수를 테셀 팩터 제공용으로 사용<br>

- [maxtessfactor(64.0f)]<br>
 : 하드웨어 상한(실제 최대는 HW/드라이버에 의해 클램프되며 DX11 일반적 최대는 64)<br>

- SV_OutputControlPointID<br>
  : 현재 몇 번째 제어점을 처리 중인지(0~3)<br>
  (여기선 단순 패스스루로 제어점 좌표를 DS로 전달)<br>

- InputPatch<VertexOut, 4> patch<br>
  :VertexOut 의 4개짜리 배열<br>


## Domain Shader

```cpp
cbuffer ConstantData : register(b0)
{
    float3 eyeWorld;
    float width;
    Matrix model;
    Matrix view;
    Matrix proj;
    float time = 0.0f;
    float3 padding;
};

struct PatchConstOutput
{
    float edges[4] : SV_TessFactor;
    float inside[2] : SV_InsideTessFactor;
};

struct HullOut
{
    float3 pos : POSITION;
};

struct DomainOut
{
    float4 pos : SV_POSITION;
};

[domain("quad")]
DomainOut main(PatchConstOutput patchConst,
             float2 uv : SV_DomainLocation,
             const OutputPatch<HullOut, 4> quad)
{
    DomainOut dout;

	// Bilinear interpolation.
    float3 v1 = lerp(quad[0].pos, quad[1].pos, uv.x);
    float3 v2 = lerp(quad[2].pos, quad[3].pos, uv.x);
    float3 p = lerp(v1, v2, uv.y);
    
    dout.pos = float4(p, 1.0);
    dout.pos = mul(dout.pos, view);
    dout.pos = mul(dout.pos, proj);
	
    return dout;
}

```

도메인 쉐이더는
- Hull Shader 쪽에서의<br>
  Output Control Points<br>
  (현재 우리의 4개인 각 꼭짓점들 위치 + <br>
  쪼개진 Vertex 들의 좌표값)<br>

- Tessellator Stage 의<br>
  Output Textrue Coordinates<br>
  (각 쪼개진 Vertex의 텍스쳐의 좌표)<br>

2개를 받아<br>
최종 정점 위치를 반환<br>

- DomainOut을 통해 좌표를 반환<br>

- 현재는 Bilinear Interpolation을 통하여<br>
  Pos를 반환<br>
 
- Domain Shader는 각 Vertex의 개수만큼 호출됨<br>

-> Patch 하나를 그리기 위해<br>
   HS 와 DS는 여러번 호출됨<br>

## 예제 목표 - 테셀레이션 구현해보기

[![Image](https://github.com/user-attachments/assets/78d06a53-d3a1-4ad7-a633-03ba0e7565af)](https://github.com/user-attachments/assets/78d06a53-d3a1-4ad7-a633-03ba0e7565af){: .image-popup}<br>

해당 모형을 시선에 따라 변화하는 LOD로서 구현해보기<br>

```cpp
HS.Hlsl

PatchConstOutput MyPatchConstantFunc(InputPatch<VertexOut, 4> patch,
                                     uint patchID : SV_PrimitiveID)
{
    float3 center = (patch[0].pos + patch[1].pos + patch[2].pos + patch[3].pos).xyz * 0.25f;
    float dist = length(center - eyeWorld);
    float distMin = 0.5;
    float distMax = 2.0;
    float tess = 64.0 * saturate((distMax - dist) / (distMax - distMin)) + 1.0;
    
    PatchConstOutput pt;
    
    pt.edges[0] = tess;
    pt.edges[1] = tess;
    pt.edges[2] = tess;
    pt.edges[3] = tess;
    pt.inside[0] = tess;
    pt.inside[1] = tess;
	
    return pt;
}
```

- center : 주어진 Control point 들의 위치 평균(중앙)<br>

- 눈과의 거리를 통하여<br>
  분할할 선의 수와(각 4변 : edges[4])<br>
  내부 점의 배치 밀도를 조정(uv : inside[2])<br>

결과<br>

[![Image](https://github.com/user-attachments/assets/2cba2b5f-1855-4fa2-9ac9-c57d8170484c)](https://github.com/user-attachments/assets/2cba2b5f-1855-4fa2-9ac9-c57d8170484c){: .image-popup}<br>
