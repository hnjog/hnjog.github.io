---
title: "Direct X InitDirect 2"
date : "2025-08-21 12:30:00 +0900"
last_modified_at: "2025-08-21T12:30:00"
categories:
  - Direct X
tags:
  - Direct 3D
---

## 들어가기 앞서
Init 관련 코드를 포함하기에<br>
문자량이 전반적으로 많아지고 있다고 느끼긴 한다<br>
일단 Init& Run 부분까지는 이렇게 진행을 하며<br>
차후 DX 포스팅 부분은 내가 이해한 개념과 간략한 코드 등을 위주로 해보려 한다<br>

## AppBase::InitGUI()

```
bool AppBase::InitGUI() {

    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImGuiIO &io = ImGui::GetIO();
    (void)io;
    io.DisplaySize = ImVec2(float(m_screenWidth), float(m_screenHeight));
    ImGui::StyleColorsLight();

    // Setup Platform/Renderer backends
    if (!ImGui_ImplDX11_Init(m_device.Get(), m_context.Get())) {
        return false;
    }

    if (!ImGui_ImplWin32_Init(m_mainWindow)) {
        return false;
    }

    return true;
}
```

AppBase의 Init의 가장 마지막인<br>
GUI에 대한 초기화 단계이다<br>

여기서는 ImGui를 사용<br>

- Imgui<br>
  : C++ 기반의 GUI 라이브러리<br>
    디버깅/툴 용 UI 제작 및 에디터 등에 사용 가능<br>
    매우 빠르게 테스트용 UI를 만들 수 있기에<br>
    선호되는 내부 디버깅용 라이브러리다<br>

- vcpkg로 Imgui를 설치<br>
  - vcpkg?<br>
    : C++의 패키징 매니저<br>
      imgui,boost 등의 라이브러리를 가볍게 설치 가능<br>
      (일반적으로 소스코드 같은 저장소 쪽 루트나 개발환경에 맞추어 로컬에 설치)<br>
  - vcpkg 설치법<br>
    : git clone https://github.com/microsoft/vcpkg.git 명령어를 통해<br>
      해당 위치의 vcpkg를 현재 폴더 위치에 복사<br>
      이후 vcpkg.bat를 실행하면 세팅 끝<br>
  - vcpkg를 이용한 라이브러리 설치법<br>
    : 'vcpkg'가 설치된 폴더의 위치에서<br>
       vcpkg install imgui:x64-windows<br>
       (:x64-windows - 동적 링크(DLL 기반))<br>
       이런식으로 해당 라이브러리 설치 가능<br>
  - vcpkg integrate install 를 통해<br>
    visual studio가 vcpkg 패키지를 인식하게 가능하다<br>
    (include/lib 경로에 세팅해준다)<br>
    -> 기본적으로 x64 등으로 빌드한다면 잡아줌<br>
    -> x86 등으로 빌드하는 경우는 '프로젝트 속성 > VC++ 디렉터리 > 포함 디렉터리 / 라이브러리 디렉터리'에<br>
       수동 추가 해야 한다<br>
       (아니면 설치할때 :x86-windows로 설치하던가)<br>


- ImGui_ImplDX11_Init <br>
  : Imgui가 사용하는 backend 초기화를 위해<br>
    Device와 Context를 넘겨준다<br>


## ExampleApp::Initialize() 1. 개념 정리
돌고 돌아 다시 온 ExampleApp::Initialize()<br>

AppBase 쪽은 사실 정형화된 Windows,DX,GUI 초기화에 가깝고<br>
ExampleApp 쪽의 Initialize 부분이<br>
우리가 커스텀할 여지가 많은 초기화 부분이다<br>

## 사용하는 개념들

- Buffer<br>
  : GPU 메모리에 넣을 데이터들<br>
   (Texture2D)<br>

- Vertex Buffer<br>
  : '정점'에 대한 정보를 담은 버퍼<br>
  (ex : 좌표,색상,Normal, UV 좌표 등)<br>

- Index Buffer<br>
  : '정점'의 '그리는 순서'를 저장하는 버퍼<br>
  ('동일한 정점'을 여러번 쓰지 않고<br>
  Index로 지정하여 재활용 한다)<br>
  원래 사각형 만들 때,<br>
  '삼각형 2개' -> 정점 6개 가 필요하나<br>
  서로 '붙는' 2개의 정점은<br>
  각각의 삼각형이 '공유'하여 사용할 수 있음<br>
  '따라서' 그리는 순서만 맞도록 설정하여 정점을 공유<br>
  (CCW/CW에 따라 그리는 순서를 다르게 해야 함)<br>

- Constant Buffer (or Uniform Buffer)<br>
  : GPU 쉐이더에 일정 기간동안 '고정적'으로 넘길 값을 담은 버퍼<br>
  (ex : model,view,projection 행렬 정보, 조명 방향 등)<br>
  매 정점마다 달라지는 것이 '아닌'<br>
  Draw Call 단위로 동일하게 적용되는 값들을 담는다<br>
  (DX에서 CBuffer 라고도 말함)<br>
  (OpenGL에선 Uniform Buffer Object 라 한다)<br>

- Shader<br>
  : GPU에서 실행되는 일종의 프로그램<br>

- Vertex Shader<br>
  : 정점 하나하나를 입력으로 받아 처리하는 쉐이더<br>
  (파이프라인에서 이 단계를 VS라 함)<br>
  - 정점의 좌표를 Model -> World -> View -> Clip으로 변환<br>
  - 조명 계산을 위한 Normal 변환<br>
  - 텍스쳐 좌표를 다음 과정으로 전달<br>
    등의 처리<br>
  - 출력값 <br>
  : 보통 NDC(정규화된 장치 좌표)와<br>
    Pixel Shader로 넘길 보간 데이터들<br>

#### 좌표계의 의미<br>

| 단계    | 공간(Space)                               | 변환 수식                                                                                                       | 특징 / 의미                                                                                                                                                                         |
| ----- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Object(Local) Space**                 | 모델 정점 좌표 `v_local`                                                                                          | - 모델이 자기 기준(모델 원점)에서 갖는 좌표<br>- 예: 큐브라면 (±1, ±1, ±1)<br>- 모델링 툴(Blender, Maya 등)에서 만들어진 좌표계                                                                                     |
| **2** | **World Space**                         | `v_world = WorldMatrix * v_local`                                                                           | - 월드 내 배치(위치, 회전, 스케일)가 반영된 좌표<br>- 모든 객체가 **하나의 공통 월드 좌표계** 안에 놓임<br>- 광원, 충돌 처리, 물리 연산 등이 이 공간에서 계산됨                                                                          |
| **3** | **View (Eye/Camera) Space**             | `v_view = ViewMatrix * v_world`                                                                             | - 카메라가 (0,0,0) 원점에 있고 +Z or -Z 축을 바라보는 좌표계로 변환<br>- 즉, 카메라 입장에서 바라본 장면<br>- 카메라 이동은 사실상 월드 전체를 반대로 이동/회전시키는 것                                                                   |
| **4** | **Clip Space**                          | `v_clip = ProjectionMatrix * v_view`                                                                        | - 투영(Perspective/Orthographic) 행렬을 곱한 결과<br>- 아직 `w`로 나누지 않은 상태 (`(x, y, z, w)`)<br>- 좌표 유효 범위: `-w ≤ x,y,z ≤ w` (이걸 **View Frustum**이라 함)<br>- 이 범위를 벗어나면 **Clipping 단계**에서 잘림 |
| **5** | **NDC (Normalized Device Coordinates)** | `v_ndc = (x_clip/w, y_clip/w, z_clip/w)`                                                                    | - Clip 좌표를 `w`로 나눠 정규화<br>- 좌표 범위: `[-1, 1]` (x,y,z 모두)<br>- `(-1,-1)` = 왼쪽 아래, `(1,1)` = 오른쪽 위, z는 보통 \[0,1] 또는 \[-1,1] API에 따라 다름<br>- 이 단계까지는 해상도와 무관                        |
| **6** | **Viewport / Screen Space**             | `v_screen.x = (v_ndc.x+1)/2 * Width`<br>`v_screen.y = (1-v_ndc.y)/2 * Height`<br>`v_screen.z = DepthBuffer` | - 실제 모니터 해상도 픽셀 좌표<br>- NDC → 픽셀 단위 좌표로 매핑<br>- Y축 뒤집힘 주의 (DirectX vs OpenGL 차이 존재)<br>- 최종적으로 \*\*백버퍼(Render Target)\*\*에 기록됨                                                  |

VS의 결과물은 Clip 좌표계<br>
RS는 입력으로 NDC 좌표계를 받음<br>
(Clip->NDC는 GPU 내부에서 처리해준다)<br>
(VS->RS 라고 그 중간 단계가 없는 것은 아니다)<br>

ViewPort와 Screen Space는 매우 유사하나<br>
더 좁게 나누자면<br>
ViewPort : GPU에서 계산된 렌더 타겟 픽셀 기준의 좌표<br>
Screen : 실제 OS/ 모니터에서 보이는 최종 윈도우 픽셀 좌표<br>
(ex : Viewport는 화면을 기준으로 잡았으나<br>
사용자가 '창 모드'를 키면 Screen 좌표는<br>
그에 맞게 최종 설정된다)<br>

- Pixel Shader<br>
  : 래스터라이저(RS)가 만든 픽셀 단위의 데이터를 입력으로 받아 처리하는 쉐이더<br>
  (파이프라인에서 이 단계를 PS라 함)<br>
  - 최종 색상(Color) 계산<br>
  - 텍스쳐 샘플링<br>
  - 조명/그림자 연산<br>
    등의 처리<br>
  - 출력값<br>
    : 렌더 타겟(RTV가 가리키는 버퍼)에 쓸 색상값<br>

## ExampleApp::Initialize() 2. 기하 정보 만들기

```
bool ExampleApp::Initialize() {

    // 기초 초기화 완료
    if (!AppBase::Initialize())
        return false;

    // Geometry 정의
    auto [vertices, indices] = MakeBox();

    ...
}
```

- auto [vertices, indices] = MakeBox();를 통해 '기하정보'를 획득<br>
  (여기서는 정육면체)<br>


### MakeBox() 코드

```
auto MakeBox() {

    vector<Vector3> positions;
    vector<Vector3> colors;
    vector<Vector3> normals;

    const float scale = 1.0f;

    // 윗면
    positions.push_back(Vector3(-1.0f, 1.0f, -1.0f) * scale);
    positions.push_back(Vector3(-1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, -1.0f) * scale);
    colors.push_back(Vector3(1.0f, 0.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(0.0f, 1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, 1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, 1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, 1.0f, 0.0f));

    // 아랫면
    positions.push_back(Vector3(-1.0f, -1.0f, -1.0f) * scale);
    positions.push_back(Vector3(-1.0f, -1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, -1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, -1.0f, -1.0f) * scale);
    colors.push_back(Vector3(1.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(1.0f, 1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, -1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, -1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, -1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, -1.0f, 0.0f));

    // 앞면
    positions.push_back(Vector3(-1.0f, -1.0f, 1.0f) * scale);
    positions.push_back(Vector3(-1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, -1.0f, 1.0f) * scale);
    colors.push_back(Vector3(0.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 0.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 0.0f));
    normals.push_back(Vector3(0.0f, 0.0f, -1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, -1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, -1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, -1.0f));

    // 뒷면
    positions.push_back(Vector3(-1.0f, -1.0f, -1.0f) * scale);
    positions.push_back(Vector3(-1.0f, 1.0f, -1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, -1.0f) * scale);
    positions.push_back(Vector3(1.0f, -1.0f, -1.0f) * scale);
    colors.push_back(Vector3(0.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 0.0f, 1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, 1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, 1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, 1.0f));
    normals.push_back(Vector3(0.0f, 0.0f, 1.0f));

    // 왼쪽
    positions.push_back(Vector3(1.0f, -1.0f, -1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, -1.0f) * scale);
    positions.push_back(Vector3(1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(1.0f, -1.0f, 1.0f) * scale);
    colors.push_back(Vector3(1.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 1.0f));
    colors.push_back(Vector3(1.0f, 0.0f, 1.0f));
    normals.push_back(Vector3(-1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(-1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(-1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(-1.0f, 0.0f, 0.0f));

    // 오른쪽
    positions.push_back(Vector3(-1.0f, -1.0f, -1.0f) * scale);
    positions.push_back(Vector3(-1.0f, 1.0f, -1.0f) * scale);
    positions.push_back(Vector3(-1.0f, 1.0f, 1.0f) * scale);
    positions.push_back(Vector3(-1.0f, -1.0f, 1.0f) * scale);
    colors.push_back(Vector3(0.0f, 1.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 1.0f));
    colors.push_back(Vector3(0.0f, 1.0f, 1.0f));
    normals.push_back(Vector3(1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(1.0f, 0.0f, 0.0f));
    normals.push_back(Vector3(1.0f, 0.0f, 0.0f));

    vector<Vertex> vertices;
    for (size_t i = 0; i < positions.size(); i++) {
        Vertex v;
        v.position = positions[i];
        v.color = colors[i];
        vertices.push_back(v);
    }

    vector<uint16_t> indices = {
        0, 1, 2,  0, 2,  3,  // 윗면
        4, 5, 6,  4, 6,  7,  // 아랫면
        8, 9, 10, 8, 10, 11, // 앞면
        12, 13, 14, 12, 14, 15, // 뒷면
        16, 17, 18, 16, 18, 19, // 왼쪽
        20, 21, 22, 20, 22, 23, // 오른쪽
    };

    return tuple{vertices, indices};
}
```

정육면체 Box 만드는 코드<br>
각각의 position과 color, normal을 고려하여<br>
각각의 위치에 넣어주면 된다<br>
이후 Index를 CCW/CW에 맞게 설정<br>

## ExampleApp::Initialize() 3. Vertex Buffer 만들기

```
bool ExampleApp::Initialize() {
    // MakeBox...

    AppBase::CreateVertexBuffer(vertices, m_vertexBuffer);

    ...
}
```

그릴 정보의 정점(vertices)를 기반으로 <br>
Vertex Buffer를 만든다<br>

### AppBase::CreateVertexBuffer 코드

```
template <typename T_VERTEX>
void CreateVertexBuffer(const vector<T_VERTEX> &vertices, ComPtr<ID3D11Buffer> &vertexBuffer) {

    // D3D11_USAGE enumeration (d3d11.h)
    // https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_usage

    D3D11_BUFFER_DESC bufferDesc;
    ZeroMemory(&bufferDesc, sizeof(bufferDesc));
    bufferDesc.Usage = D3D11_USAGE_IMMUTABLE; // 초기화 후 변경X
    bufferDesc.ByteWidth = UINT(sizeof(T_VERTEX) * vertices.size());
    bufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
    bufferDesc.CPUAccessFlags = 0; // 0 if no CPU access is necessary.
    bufferDesc.StructureByteStride = sizeof(T_VERTEX);

    D3D11_SUBRESOURCE_DATA vertexBufferData = {0}; // MS 예제에서 초기화하는 방식
    vertexBufferData.pSysMem = vertices.data();
    vertexBufferData.SysMemPitch = 0;
    vertexBufferData.SysMemSlicePitch = 0;

    const HRESULT hr =
        m_device->CreateBuffer(&bufferDesc, &vertexBufferData, vertexBuffer.GetAddressOf());
    if (FAILED(hr)) {
        std::cout << "CreateBuffer() failed. " << std::hex << hr << std::endl;
    };
}
```

- Template 함수로 선언한 이유?<br>
  : 일반 함수라면 Vertex 구조체에 추가적인 정보가 들어오는 경우<br>
    추가적인 수정이 필요하거나 함수 오버로딩이 발생하기에<br>
    템플릿 함수를 사용<br>
    (ex : 지금 pos,color 값만 있지만 normal, uv 좌표 등이<br>
    vertex에 추가될 수 있음)<br>

- Vertex 버퍼는 ID3D11Buffer 타입<br>

- D3D11_BUFFER_DESC 로 버퍼 세팅<br>
  (당장은 모델 하나만 사용하기에 IMMUTABLE)<br>
  (CPUAccessFlags : 0 - 어차피 IMMUTABLE)<br>
  (StructureByteStride : 하나의 요소가 가지는 크기)<br>

- D3D11_SUBRESOURCE_DATA?<br>
  : 버퍼나 텍스쳐 등을 생성할 때<br>
   '초기 데이터'를 전달하기 위한 구조체<br>
   (GPU 메모리 잡은 후, 그 안에 당장 넣을 데이터를 뜻함)<br>
   (어떤 데이터를 어떻게 보낼지를 설정한다 봐도 된다)<br>
   - psystem : 초기 데이터 포인터(CPU의 메모리 주소)<br>
   - SysMemPitch : 한 행(row)의 메모리 간격(byte 단위, 2D 텍스쳐용)<br>
   - SysMemSlicePitch : 한 슬라이스(layer)의 메모리 간격(byte 단위, 2D 텍스쳐용)<br>
     (여기서는 1D 데이터이므로 둘 다 0 설정)<br>

- m_device->CreateBuffer 를 통해<br>
  Device에 Buffer 생성 요청<br>
  (GPU에 메모리 할당 요청)<br>

### ID3D11Buffer Vs ID3D11Texture2D?

| 구분           | **ID3D11Buffer**                                                                                                                                                | **ID3D11Texture2D**                                                                                                                                                           |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **메모리 배치**   | 1차원 **선형 배열**                                                                                                                                                   | 2차원 **텍셀 그리드** (타일/스위즐 등 GPU 친화 배치)                                                                                                                                           |
| **포맷 지정**    | 옵션 (Raw, Structured, 정수/실수)                                                                                                                                     | **DXGI\_FORMAT 필수** (R8G8B8A8, R32\_FLOAT 등)                                                                                                                                  |
| **차원/레벨**    | 1D, mipmap 없음                                                                                                                                                   | 2D (배열/큐브맵/MSAA 가능), mipmap 지원                                                                                                                                                |
| **셰이더 접근**   | `Buffer<T>`, `StructuredBuffer<T>`, `ByteAddressBuffer`, `RWStructuredBuffer`                                                                                   | `Texture2D<T>`, `RWTexture2D<T>` (샘플러/LOD/필터링 지원)                                                                                                                             |
| **필터링/샘플링**  | 불가 (그냥 로드)                                                                                                                                                      | 가능 (포인트/선형/애니소, 샘플러 상태 활용)                                                                                                                                                    |
| **렌더 타겟/깊이** | 불가                                                                                                                                                              | 가능 (`RenderTargetView`, `DepthStencilView`)                                                                                                                                   |
| **UAV(SRV)** | 둘 다 가능 (바인드 플래그 필요)                                                                                                                                             | 둘 다 가능 (바인드 플래그 필요)                                                                                                                                                           |
| **주요 활용 예시** | - **정점/인덱스/상수 데이터** 저장<br>- **구조체 배열, 플래그 테이블, GPGPU 계산(스캔/리덕션)**<br>- **Append/Consume 큐** (StructuredBuffer + UAV)<br>- **정밀한 1D 랜덤 액세스** (ByteAddressBuffer) | - **렌더 타겟/깊이 버퍼/포스트프로세스 중간 결과(G-Buffer)**<br>- **샘플링/필터링/LOD 기반 룩업** (예: 라이트 쿠키, LUT)<br>- **2D 지역성 활용, 자연스러운 정수 좌표 접근**<br>- **RWTexture2D 기반 이미지 처리** (블러, 컨볼루션, 타일드 라이팅 등) |


두 인터페이스 모두<br>
ID3D11Resource 를 상속받는 리소스 타입이며<br>
'버퍼'라는 넓은 개념에 들어간다<br>

요약<br>
- ID3D11Buffer : 1차원 데이터 컨테이너, 계산/ 정점 데이터 등에 적합 (일부 파이프라인은 Buffer를 고정으로 요구)<br>
- ID3D11Texture2D : 2차원 데이터 컨테이너, 샘플링/렌더타겟/이미지 연상 등에 적합<br>

## ExampleApp::Initialize() 4. Index Buffer 만들기

```
bool ExampleApp::Initialize() {
    // Vertex Buffer...

    // 인덱스 버퍼 만들기
    m_indexCount = UINT(indices.size());

    AppBase::CreateIndexBuffer(indices, m_indexBuffer);

    ...
}
```

인덱스의 길이를 통해 m_indexCount를 계산하고<br>
AppBase::CreateIndexBuffer 를 통해 Index Buffer를 계산<br>

### AppBase::CreateIndexBuffer 코드

```
void AppBase::CreateIndexBuffer(const std::vector<uint16_t> &indices,
                                ComPtr<ID3D11Buffer> &m_indexBuffer) {
    D3D11_BUFFER_DESC bufferDesc = {};
    bufferDesc.Usage = D3D11_USAGE_IMMUTABLE; // 초기화 후 변경X
    bufferDesc.ByteWidth = UINT(sizeof(uint16_t) * indices.size());
    bufferDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
    bufferDesc.CPUAccessFlags = 0; // 0 if no CPU access is necessary.
    bufferDesc.StructureByteStride = sizeof(uint16_t);

    D3D11_SUBRESOURCE_DATA indexBufferData = {0};
    indexBufferData.pSysMem = indices.data();
    indexBufferData.SysMemPitch = 0;
    indexBufferData.SysMemSlicePitch = 0;

    m_device->CreateBuffer(&bufferDesc, &indexBufferData, m_indexBuffer.GetAddressOf());
}
```

_DESC를 통해 버퍼의 용도 등을 설정한 후<br>
device를 통해 버퍼를 생성<br>

- IndexBuffer는 항상 자연수값을 받기에<br>
  uint16_t를 사용하고<br>
  '템플릿 함수'로 선언하지 않음<br>
  (create Vertex와 pixel이 헤더에 있는 이유)<br>

## ExampleApp::Initialize() 5. Constant Buffer 만들기

```
bool ExampleApp::Initialize() {
    // Index Buffer...

    // ConstantBuffer 만들기
    m_constantBufferData.model = Matrix();
    m_constantBufferData.view = Matrix();
    m_constantBufferData.projection = Matrix();

    AppBase::CreateConstantBuffer(m_constantBufferData, m_constantBuffer);

    ...
}
```

model,view,projection은<br>
정점마다 바뀌는 것이 아닌 drawCall 동안 유지되기에<br>
Constant 버퍼에 넣어준다<br>
(지금은 임시로 빈 배열을 넣어줌)<br>

### AppBase::CreateConstantBuffer 코드

```
template <typename T_CONSTANT>
void CreateConstantBuffer(const T_CONSTANT &constantBufferData,
                          ComPtr<ID3D11Buffer> &constantBuffer) {
    D3D11_BUFFER_DESC cbDesc;
    cbDesc.ByteWidth = sizeof(constantBufferData);
    cbDesc.Usage = D3D11_USAGE_DYNAMIC;
    cbDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    cbDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
    cbDesc.MiscFlags = 0;
    cbDesc.StructureByteStride = 0;

    // Fill in the subresource data.
    D3D11_SUBRESOURCE_DATA InitData;
    InitData.pSysMem = &constantBufferData;
    InitData.SysMemPitch = 0;
    InitData.SysMemSlicePitch = 0;

    m_device->CreateBuffer(&cbDesc, &InitData, constantBuffer.GetAddressOf());
}
```

- Constant에 넣어야 하는 정보가 추가될 수 있으므로<br>
  Template 선언<br>

- Usage_Dynamic 설정된 이유?<br>
  : 회전 행렬 등이 적용되면<br>
    해당 constant 정보가 변할 수 있기에<br>
    바뀐 행렬을 drawcall 때마다 보내줘야 함<br>
    따라서 Dynamic 설정 + CPU_ACCESS_WRITE<br>


## ExampleApp::Initialize() 6. Vertex Shader 생성

```
bool ExampleApp::Initialize() {
    // Constant Buffer...

    // 쉐이더 만들기

    // Input-layout objects describe how vertex buffer data is streamed into the
    // IA(Input-Assembler) pipeline stage.
    // https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-iasetinputlayout

    // Input-Assembler Stage
    // The purpose of the input-assembler stage is to read primitive data
    // (points, lines and/or triangles) from user-filled buffers and assemble
    // the data into primitives that will be used by the other pipeline stages.
    // https://learn.microsoft.com/en-us/windows/win32/direct3d11/d3d10-graphics-programming-guide-input-assembler-stage

    vector<D3D11_INPUT_ELEMENT_DESC> inputElements = {
        {"POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0},
        {"COLOR", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 4 * 3, D3D11_INPUT_PER_VERTEX_DATA, 0},
    };

    AppBase::CreateVertexShaderAndInputLayout(L"ColorVertexShader.hlsl", inputElements,
                                              m_colorVertexShader, m_colorInputLayout);

    ...
}
```

- D3D11_INPUT_ELEMENT_DESC<br>
  : 정점 데이터의 구조를 GPU에게 알려주기 위한 구조체<br>
  ex)<br>
  SemanticName : 시멘틱이름 - "POSITION"<br>
  SemanticIndex : 동일 이름 시멘틱 구분용 인덱스 - 0<br>
  Format : 데이터 포맷 - DXGI_FORMAT_R32G32B32_FLOAT<br>
  InputSlot : 입력 슬롯(여러 버퍼 구분용) - 0<br> 
  AlignedByteOffset : 버퍼에서 이 요소가 시작하는 바이트 오프셋 - 0<br> 
  (바로 아래의 COLOR는 Position의 크기를 건너띄어 시작한다)<br>
  InputSlotClass : 데이터의 종류 - D3D11_INPUT_PER_VERTEX_DATA<br>
  InstanceDataStepRate : 몇 개 인스턴스를 그릴 때마다 데이터를 다음으로 넘길지를 지정 - 0<br>
  (이 옵션은 여러 인스턴스를 다양한 버퍼 데이터를 사용할 때 활용)<br>
  (0 - 모든 인스턴스가 같은 옵션 사용)<br>

- SemanticName을 VS의 Semantic과 맞춰줘야 한다<br>
  SematicName + Index 로 VS에서 COLOR0 등의 Semantic으로 연결<br>
  (참고 : https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics)<br>

### AppBase::CreateVertexShaderAndInputLayout 코드

```
void AppBase::CreateVertexShaderAndInputLayout(
    const wstring &filename, const vector<D3D11_INPUT_ELEMENT_DESC> &inputElements,
    ComPtr<ID3D11VertexShader> &vertexShader, ComPtr<ID3D11InputLayout> &inputLayout) {

    ComPtr<ID3DBlob> shaderBlob;
    ComPtr<ID3DBlob> errorBlob;

    // 주의: 쉐이더의 시작점의 이름이 "main"인 함수로 지정
    HRESULT hr =
        D3DCompileFromFile(filename.c_str(), 0, 0, "main", "vs_5_0", 0, 0, &shaderBlob, &errorBlob);

    CheckResult(hr, errorBlob.Get());

    m_device->CreateVertexShader(shaderBlob->GetBufferPointer(), shaderBlob->GetBufferSize(), NULL,
                                 &vertexShader);

    m_device->CreateInputLayout(inputElements.data(), UINT(inputElements.size()),
                                shaderBlob->GetBufferPointer(), shaderBlob->GetBufferSize(),
                                &inputLayout);
}
```

- ID3DBlob<br>
  : Direct3D의 바이너리 데이터 컨테이너<br>
  단순한 메모리 버퍼로 '임시 데이터' 등을 담는데 사용<br>
  GetBufferPointer() - 내부 메모리 시작 주소 반환<br>
  GetBufferSize() - 버퍼 크기(바이트 단위) 반환<br>

- D3DCompileFromFile<br>
  : HLSL 셰이더를 파일로부터 읽어와 컴파일하는 함수<br>
  (지금은 vs 이므로 셰이더 모델(버전)을 vs_5_0 로 설정)<br>
  (0,0 - 매크로나 include 를 넣어줄 수 있으나 여기선 pass)<br>
  (main - 셰이더의 엔트리 포인트가 될 함수)<br>
  (shaderBlob - 성공 시의 바이트 코드 반환)<br>
  (errorBlob - 실패 시의 에러 메시지 반환)<br>

- CheckResult를 통해 성공여부 체크<br>
  (파일이름 틀린 경우, 에러 메시지 등을 체크)<br>
  (AppBase에 있는 체크용 함수다)<br>

- CreateVertexShader를 통해 Device에<br>
  쉐이더 객체를 실제로 생성한다<br>

- CreateInputLayout를 통해<br>
  만들어진 쉐이더 객체와 inputElements를 연결<br>

## ExampleApp::Initialize() 7. Pixel Shader 생성

```
bool ExampleApp::Initialize() {
    // Vertex Shader...

    AppBase::CreatePixelShader(L"ColorPixelShader.hlsl", m_colorPixelShader);

    return true;
}
```

별도의 hlsl 파일로 픽셀 쉐이더를 생성<br>


### AppBase::CreatePixelShader 코드

```
void AppBase::CreatePixelShader(const wstring &filename, ComPtr<ID3D11PixelShader> &pixelShader) {
    ComPtr<ID3DBlob> shaderBlob;
    ComPtr<ID3DBlob> errorBlob;

    // 주의: 쉐이더의 시작점의 이름이 "main"인 함수로 지정
    HRESULT hr =
        D3DCompileFromFile(filename.c_str(), 0, 0, "main", "ps_5_0", 0, 0, &shaderBlob, &errorBlob);

    CheckResult(hr, errorBlob.Get());

    m_device->CreatePixelShader(shaderBlob->GetBufferPointer(), shaderBlob->GetBufferSize(), NULL,
                                &pixelShader);
}
```

vs에 유사하나 ps로 지정<br>