---
title: "그래픽스 디버깅"
date : "2025-08-25 12:30:00 +0900"
last_modified_at: "2025-08-25T12:30:00"
categories:
  - HLSL
tags:
  - 그래픽스 디버거
---

## 사전 세팅

- Windows의 '시스템' -> '선택적 기능'에서<br>
  '그래픽 도구'가 설치되어 있는지 확인하기<br>

<img width="1444" height="1529" alt="Image" src="https://github.com/user-attachments/assets/3ac376c3-cd6b-47d8-a46d-0419995326ae" /><br>


- 컴파일 플래그에서 DEBUG를 설정<br>
  (D3DCOMPILE_SKIP_OPTIMIZATION : 최적화 스킵<br>
  일부 최적화는 컴파일러가 내부적으로 코드를 바꿀 가능성이 존재하기에<br>
  안정적인 디버깅을 위해 최적화를 꺼준다)<br>

```
    UINT compileFlags = 0;
#if defined(DEBUG) || defined(_DEBUG)
    compileFlags = D3DCOMPILE_DEBUG | D3DCOMPILE_SKIP_OPTIMIZATION;
#endif
```

## 그래픽 디버깅

<img width="2070" height="838" alt="Image" src="https://github.com/user-attachments/assets/fc9e783b-f19c-4a65-8689-34bea126f1c9" /><br>

Visual Studio에서<br>
디버그 -> 그래픽 -> '그래픽 디버깅 시작' 을 클릭<br>


<img width="1696" height="1101" alt="Image" src="https://github.com/user-attachments/assets/477024cb-4458-4bd1-866e-1aacf92dce83" /><br>

report에 캡쳐해놓은 프레임을 확인할 수 있으며<br>
스크린샷에 찍어놓은 프레임을 더블클릭하면

<img width="3819" height="2029" alt="Image" src="https://github.com/user-attachments/assets/da40ff46-9fb4-4fa3-aa95-68192f46c815" /><br>

그래픽스 애널라이저를 열어<br>
해당 프레임의 렌더링 상태를 분석할 수 있다<br>

- 이벤트 리스트<br>
  : 해당 프레임에서 실행된 모든 Draw/ Dispatch 호출을 시간순으로 보여준다<br>
  각 호출을 선택하여 그 프레임의 '파이프라인 상태'를 확인 가능함<br>

- 파이프라인 뷰<br>
  : IA->VS->RS->PS->OM 같은 GPU 파이프라인 단계별 상태를 시각적으로 확인 가능<br>
    각 단계의 쉐이더나 데이터의 상태 체크<br>

- 리소스 뷰<br>
  : 버퍼, 텍스쳐, 상태 객체 등을 열어 GPU 리소스 내부값 확인 가능<br>
   Constant Buffer의 내용을 확인 가능하기에 입력값 디버깅에 유용<br>

- 픽셀 히스토리<br>
  : 특정 픽셀을 클릭하면 그 픽셀의 결정 과정을 순서대로 볼 수 있음<br>
    (DrawCall, 쉐이더 결과 등)<br>
    특정 픽셀의 색이 이상한 경우 원인 분석에 유용<br>

- 쉐이더 디버깅<br>
  : 선택한 Draw 호출의 쉐이더 코드(HLSL)을 단계별로 실행 가능(BreakPoint,변수확인)<br>
    CPU 디버깅 하듯 GPU 쉐이더의 내부 상태를 확인 가능<br>

<img width="3025" height="1849" alt="Image" src="https://github.com/user-attachments/assets/090bae3c-5f84-4403-8c4e-320d5cc0e153" /><br>

- 픽셀 히스토리 예제<br>
  처음에는 ClearRenderTargetView 이후<br>
  픽셀 쉐이더에 의하여 값이 정해짐<br>
  또한 Pixel 쉐이더 시점의 Input값 등을 확인 가능<br>

<img width="3829" height="2077" alt="Image" src="https://github.com/user-attachments/assets/d8db0098-9093-4326-83ac-cae54142cf8d" /><br>

- 쉐이더 디버깅 예제<br>
  픽셀 히스토리 부분의<br>
  각 쉐이더의 Start 버튼을 눌러 실행 가능<br>
  (VS는 삼각형이기에 각 정점마다 확인 가능<br>
  PS는 해당 Pixel에서 가능)<br>
  실제 VS 디버깅 처럼 f10,f11 등을 통해<br>
  한줄씩 코드 실행 및 조사식을 통한 변수값 확인이 가능하다<br>

- 우측 화면은 어셈블러를 볼 수 있음<br>
  (DXBC 를 통해 어셈블러로 컴파일된 코드 확인 가능)<br>

  NVidia 그래픽스 카드 사용시,<br>
  Nsight를 VS에 설치해서 디버깅 정보를 보는것도 가능<br>
  (범용 그래픽스 디버거는 RenderDoc가 유명<br>
  RenderDoc.org에서 무료 다운 가능하다 함)<br>

## 쉐이더를 컴파일하는 방법?

- D3DCompileFromFile<br>
  : Shader 파일이름을 직접 정해주어 컴파일시킨다<br>
    (파일 이름, 지정 함수, 어떤 쉐이더 모델 사용 등등을 매개변수로 지정)<br>
   이후 성공한 Blob을 바탕으로<br>
   실제 Shader나 InputLayout을 생성한다<br>
   (런타임 컴파일 - 직접 CSO를 만들라고 호출함)<br>

```
    HRESULT hr = D3DCompileFromFile(filename.c_str(), 0, 0, "main", "vs_5_0",
                                    compileFlags, 0, &shaderBlob, &errorBlob);

   CheckResult(hr, errorBlob.Get());

   m_device->CreateVertexShader(shaderBlob->GetBufferPointer(),
                                shaderBlob->GetBufferSize(), NULL,
                                &vertexShader);

   m_device->CreateInputLayout(inputElements.data(),
                               UINT(inputElements.size()),
                               shaderBlob->GetBufferPointer(),
                               shaderBlob->GetBufferSize(), &inputLayout);                              
```

- vs 프로젝트 내부에 HLSL가 존재하는 경우<br>
  해당 파일들의 '속성'을 HLSL 쉐이더로 지정하면<br>
  (속성 : HLSL 컴파일러 설정, 이후 쉐이더 타입을 설정)<br>
  빌드할 시, 빌드 결과에 cso / hlsl.h 같은 산출물을 생성함<br>
  (cso : compiled shader object)<br>
  그렇기에 cmd와 fxc.exe 를 사용하여<br>
  cso 파일을 만들고<br>
  그 데이터를 읽어와 Vertex Shader와 Input Layout 등을 만들어 줄 수 있음<br>
  (미리 cso 파일을 만들어 놓는 경우 유용)<br>

```
// 참고: 수동으로 컴파일 하기
// "fxc.exe"의 위치는 각자 다를 수도 있습니다.
//"C:\Program Files (x86)\Windows Kits\10\bin\10.0.19041.0\x86\fxc.exe"
// C:\Users\jmhong\HongLabGraphicsPart2\06_GraphicsPipeline_Step4_Shaders\ColorVertexShader.hlsl
// /T "vs_5_0" /E "main" /Fo "ColorVertexShader.cso" /Fx
// "ColorVertexShader.asm"

// ifstream input("ColorVertexShader.cso", ios::binary);
// vector<unsigned char> buffer(istreambuf_iterator<char>(input), {});
// m_device->CreateVertexShader(buffer.data(), buffer.size(), NULL,
//                              &vertexShader);

// m_device->CreateInputLayout(inputElements.data(),
//                             UINT(inputElements.size()), buffer.data(),
//                             buffer.size(), &inputLayout);
```