---
title: "밉맵(Mipmap)과 LOD"
date : "2025-10-13 20:00:00 +0900"
last_modified_at: "2025-10-13T20:00:00"
categories:
  - 그래픽스
  - 수학
  - DirectX
tags:
  - Mipmap
  - Level Of Detail
---

## Mipmap

[![Image](https://github.com/user-attachments/assets/10224a01-a53a-49c8-b609-0f97243b0960)](https://github.com/user-attachments/assets/10224a01-a53a-49c8-b609-0f97243b0960){: .image-popup}<br>

텍스쳐링 기술의 하나로<br>
'텍스쳐'를 효율적으로 표시하기 위한 기술<br>

- 하나의 텍스쳐의 `여러 해상도`를 미리 만들어 두고<br>
  관측자(카메라)와의 거리를 계산하여 사용할 해상도의 텍스쳐를 선택하는 방식<br>
  (언리얼 같은 엔진은 이러한 해상도에 따른 축소본들을 자동으로 생성해줌)<br>
  (DirectX에서도 준비할 수 있다)<br>

ex)<br>

| 레벨      | 해상도       | 설명         |
| ------- | --------- | ---------- |
| Level 0 | 1024×1024 | 원본 텍스처     |
| Level 1 | 512×512   | 절반 크기      |
| Level 2 | 256×256   | 1/4 크기     |
| Level 3 | 128×128   | ...        |
| ...     | ...       | 마지막은 1×1까지 |


(위 사진처럼 점점 이미지가 절반이 되기에<br>
 이미지 피라이드 라는 단어로도 표현됨)<br>


[![Image](https://github.com/user-attachments/assets/62862964-4b63-4799-8805-d343b3c96c61)](https://github.com/user-attachments/assets/62862964-4b63-4799-8805-d343b3c96c61){: .image-popup}<br>

- 왜 사용하는가?<br>
  - 멀리 있는 물체에 고해상도를 쓰는 것은 '낭비'인 동시에<br>
    무늬가 깜빡이거나 패턴이 깨지는 `원근 감쇠(Distance Attenuation) 현상`이<br>
    발생 가능하며, 이를 예방하기 위함<br>
    (사진처럼 Mipmap을 사용하는 쪽이 끝 부분이 깔끔함)<br>
  - 또한 작은 텍스쳐를 사용함으로서 메모리 및 캐시에 대한 이점을 가지게 됨<br>

- 어디에 활용하는가<br>
  - Level Of Detail (LOD)<br>
  - PBR<br>

## 예제 - Mipmap

[![Image](https://github.com/user-attachments/assets/9e71fcc6-f79b-402c-bc0e-9190b0034edb)](https://github.com/user-attachments/assets/9e71fcc6-f79b-402c-bc0e-9190b0034edb){: .image-popup}<br>


[![Image](https://github.com/user-attachments/assets/895146f6-bee5-4483-b4b1-0be811cec7e6)](https://github.com/user-attachments/assets/895146f6-bee5-4483-b4b1-0be811cec7e6){: .image-popup}<br>

- Mipmap Level을 수정함으로서 텍스쳐의 디테일이 달라지는 모습<br>
  Detail이 높은 텍스쳐에서 낮은 텍스쳐로 갈수록<br>
  텍스쳐보단 '하나의 색'에 가까워지는 모습<br>

## Mipmap 구현 방식

1. 임시 사용할 스테이징 텍스쳐를 만들기<br>
2. Cpu에서 스테이징 텍스쳐로 복사<br>
3. 스테이징 텍스쳐에서 진짜로 사용할 텍스쳐로 복사<br>

```cpp
void BasicMeshGroup::Initialize(ComPtr<ID3D11Device> &device,
                                ComPtr<ID3D11DeviceContext> &context,
                                const std::vector<MeshData> &meshes) {

    // Sampler 만들기
    D3D11_SAMPLER_DESC sampDesc;
    ZeroMemory(&sampDesc, sizeof(sampDesc));
    sampDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;
    // sampDesc.Filter = D3D11_FILTER_MIN_MAG_LINEAR_MIP_POINT;
    ...

}
```

- D3D11_FILTER_MIN_MAG_LINEAR_MIP_POINT 옵션 사용시<br>
  해상도가 낮아지는 것이 더 직관적으로 보인다<br>
  (변화의 중간이 없이 계단처럼 뚝뚝 끊기도록 설정)<br>
  (위에서 본 사진처럼 조건에 맞는 텍스쳐를 하나만 골라 바로 적용)<br>
  (Linear는 거리에 따라 더 세부적인 보간 처리를 해줌)<br>

### Min Mag

[![Image](https://github.com/user-attachments/assets/6990933a-ccf1-48ca-81ff-1ab0c2038fb2)](https://github.com/user-attachments/assets/6990933a-ccf1-48ca-81ff-1ab0c2038fb2){: .image-popup}<br>

- Mag : 텍스쳐를 원본보다 크게 그릴 때<br>
- Min : 텍스쳐를 원본보다 작게 그릴 때<br>

각각 적용되는 필터 방식을 설정하는 것이<br>
위에서 본 MIN_MAG 옵션<br>
(각각 다르게 설정도 가능하다)<br>

- '텍스쳐'를 이루는 각 픽셀을 `'Texel'(텍셀)` 이라 함<br>

### Subresources

[![Image](https://github.com/user-attachments/assets/4c6ddaef-7400-47de-99d1-a144a6bf29bf)](https://github.com/user-attachments/assets/4c6ddaef-7400-47de-99d1-a144a6bf29bf){: .image-popup}<br>

- SubResource의 일종의 '좌표축'들<br>
  (Array Slice, Mip Slice)<br>

[![Image](https://github.com/user-attachments/assets/3f8d98e5-57ce-46ab-9cd4-691a4de5b7fb)](https://github.com/user-attachments/assets/3f8d98e5-57ce-46ab-9cd4-691a4de5b7fb){: .image-popup}<br>

- 지정된 각각의 서브리소스들<br>


```vbnet
📦 Texture2DArray 리소스
 ├─ [0] 텍스처
 │   ├─ Mip 0 → Sub 0
 │   ├─ Mip 1 → Sub 1
 │   ├─ Mip 2 → Sub 2
 │   └─ Mip 3 → Sub 3
 └─ [1] 텍스처
     ├─ Mip 0 → Sub 4
     ├─ Mip 1 → Sub 5
     ├─ Mip 2 → Sub 6
     └─ Mip 3 → Sub 7
```

- GPU에 올려져 있는 메모리 묶음<br>

- 텍스쳐 자체는 '리소스' 중 하나 이지만<br>
  각각의 '해상도'가 '달라지는' 경우<br>
  이를 '구분'해야 하기에<br>
  이러한 '각각'을 표현하기 위해<br>
  `서브리소스`라는 표현을 사용<br>

| 용어                      | 설명                                         |
| ----------------------- | ------------------------------------------ |
| **Resource (리소스)**      | GPU 메모리 안의 큰 데이터 단위 (예: Texture, Buffer 등) |
| **Subresource (서브리소스)** | 그 내부의 “부분 단위” (예: 특정 밉맵, 특정 배열 요소)         |


- 특정한 Mipmap만 업데이트할 수 있으며<br>
  특정한 요소의 인덱스를 통해 지정이 가능하기에<br>
  '일부'만 제어할 때 특히 유용한 단위<br>

- Mipmap은 서브리소스를 통해 '표현'될 수 있음<br>

- 예시 코드<br>

```cpp
ComPtr<ID3D11Texture2D>
CreateStagingTexture(ComPtr<ID3D11Device> &device,
                     ComPtr<ID3D11DeviceContext> &context, const int width,
                     const int height, const std::vector<uint8_t> &image,
                     const int mipLevels = 1, const int arraySize = 1) {

    // 스테이징 텍스춰 만들기
    D3D11_TEXTURE2D_DESC txtDesc;
    ZeroMemory(&txtDesc, sizeof(txtDesc));
    txtDesc.Width = width;
    txtDesc.Height = height;
    txtDesc.MipLevels = mipLevels;
    txtDesc.ArraySize = arraySize;
    txtDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    txtDesc.SampleDesc.Count = 1;
    txtDesc.Usage = D3D11_USAGE_STAGING;
    txtDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE | D3D11_CPU_ACCESS_READ;

    ComPtr<ID3D11Texture2D> stagingTexture;
    if (FAILED(device->CreateTexture2D(&txtDesc, nullptr,
                                       stagingTexture.GetAddressOf()))) {
        cout << "Failed()" << endl;
    }

    // CPU에서 이미지 데이터 복사
    D3D11_MAPPED_SUBRESOURCE ms;
    context->Map(stagingTexture.Get(), NULL, D3D11_MAP_WRITE, NULL, &ms);
    uint8_t *pData = (uint8_t *)ms.pData;
    for (UINT h = 0; h < UINT(height); h++) { // 가로줄 한 줄씩 복사
        memcpy(&pData[h * ms.RowPitch], &image[h * width * 4],
               width * sizeof(uint8_t) * 4);
    }
    context->Unmap(stagingTexture.Get(), NULL);

    return stagingTexture;
}

---

void D3D11Utils::CreateTexture(
    ComPtr<ID3D11Device> &device, ComPtr<ID3D11DeviceContext> &context,
    const std::string filename, ComPtr<ID3D11Texture2D> &texture,
    ComPtr<ID3D11ShaderResourceView> &textureResourceView) {

    int width, height;
    std::vector<uint8_t> image;

    ReadImage(filename, image, width, height);

    // 스테이징 텍스춰 만들고 CPU에서 이미지를 복사합니다.
    ComPtr<ID3D11Texture2D> stagingTexture =
        CreateStagingTexture(device, context, width, height, image);

    // 실제로 사용할 텍스춰 설정
    D3D11_TEXTURE2D_DESC txtDesc;
    ZeroMemory(&txtDesc, sizeof(txtDesc));
    txtDesc.Width = width;
    txtDesc.Height = height;
    txtDesc.MipLevels = 0; // 밉맵 레벨 최대
    txtDesc.ArraySize = 1;
    txtDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    txtDesc.SampleDesc.Count = 1;
    txtDesc.Usage = D3D11_USAGE_DEFAULT; // 스테이징 텍스춰로부터 복사 가능
    txtDesc.BindFlags = D3D11_BIND_SHADER_RESOURCE | D3D11_BIND_RENDER_TARGET;
    txtDesc.MiscFlags = D3D11_RESOURCE_MISC_GENERATE_MIPS; // 밉맵 사용
    txtDesc.CPUAccessFlags = 0;

    // 초기 데이터 없이 텍스춰 생성 (전부 검은색)
    device->CreateTexture2D(&txtDesc, nullptr, texture.GetAddressOf());

    // 실제로 생성된 MipLevels를 확인해보고 싶을 경우
    // texture->GetDesc(&txtDesc);
    // cout << txtDesc.MipLevels << endl;

    // 스테이징 텍스춰로부터 가장 해상도가 높은 이미지 복사
    context->CopySubresourceRegion(texture.Get(), 0, 0, 0, 0,
                                   stagingTexture.Get(), 0, nullptr);

    // ResourceView 만들기
    device->CreateShaderResourceView(texture.Get(), 0,
                                     textureResourceView.GetAddressOf());

    // 해상도를 낮춰가며 밉맵 생성
    context->GenerateMips(textureResourceView.Get());

    // HLSL 쉐이더 안에서는 SampleLevel() 사용
}
```

- 스테이징 텍스쳐를 만들기<br>
  - D3D11_USAGE_STAGING : CPU와의 통신을 위하여 임시로 데이터를 담아둠<br>
    그렇기에 CPU 접근 Access 플래그가<br>
    D3D11_CPU_ACCESS_WRITE | D3D11_CPU_ACCESS_READ<br>
    (1번 단계)<br>

- 이후 Map,Unmap을 통해 CPU로부터 텍스쳐 데이터를 읽어들임<br>
  (해상도가 일치하지 않기에 memcpy로 일일이 복사)<br>
  (2번 단계)<br>

- 이후 실제 사용할 텍스쳐를 구현하기<br>
  - MipLevel = 0 일 경우, 가능한 모든 Mipmap을 만듦<br>
  - Usage를 Defalut로 잡아 스테이징으로부터 복사 가능하도록<br>
  - BindFlag는 D3D11_BIND_SHADER_RESOURCE | D3D11_BIND_RENDER_TARGET 를 사용<br>
    (GenerateMips를 사용하려면 '읽기 + 쓰기'가 모두 가능해야 함)<br>
  - MiscFlag를 D3D11_RESOURCE_MISC_GENERATE_MIPS로 설정(밉맵 만들려면 설정해야함)<br>
  - GPU로부터 복사해오기에 CpuFlag는 0으로 설정<br>
  - 이후 생성할 때, nullptr을 통해 검은색으로 밀어버림<br>

- 스테이징 텍스쳐로부터 해상도가 가장 높은걸 복사<br>

### 쉐이더 코드 (Hlsl)

```cpp
PixelShaderOutput main(PixelShaderInput input)
{
    float3 toEye = normalize(eyeWorld - input.posWorld);

    float3 color = float3(0.0, 0.0, 0.0);

    ...

    if (useTexture)
    {
        // diffuse *= g_texture0.Sample(g_sampler, input.texcoord);
        diffuse *= g_texture0.SampleLevel(g_sampler, input.texcoord, mipmapLevel);

        // Specular texture를 별도로 사용할 수도 있습니다.
    }

    PixelShaderOutput output;
    output.pixelColor = diffuse + specular;
    output.indexColor = indexColor;
    
    return output;
}

```

- SampleLevel 을 통해 mipmapLevel에 따른 텍스쳐를 사용<br>
 (Sample은 적절한 Lod Level을 내부적으로 계산하여 샘플링함)<br>
 (위의 SubResource를 통해 만들어진 Mipmap중 하나를 선택하며<br>
  mipmapLevel이 그 중간값이면 내부적으로 보간한 값을 선택)<br>

## 예제 - Hlsl 코드를 수정하여 거리에 따른 Mipmap 변화

```cpp
 if (useTexture)
 {
     // diffuse *= g_texture0.Sample(g_sampler, input.texcoord);
     float dist = length(eyeWorld - input.posWorld);
     float distMin = 5.0;
     float distMax = 10.0;
     float lod = 10.0 * saturate((dist - distMin) / (distMax - distMin));
     diffuse *= g_texture0.SampleLevel(g_sampler, input.texcoord, lod);

     // Specular texture를 별도로 사용할 수도 있습니다.
 }
```

- Mipmap Level이 상승할수록 텍스쳐 품질이 낮아짐<br>
- 거리가 가까울수록 mipmap 값이 낮아지도록<br>

- 결과<br>

[![Image](https://github.com/user-attachments/assets/95c8434e-187f-4d80-92fd-9a118c01de63)](https://github.com/user-attachments/assets/95c8434e-187f-4d80-92fd-9a118c01de63){: .image-popup}<br>

거리가 멀어지니 지구본의 텍스쳐가<br>
다소 흐려진 모습이다<br>

### 이미지 출처
- Wiki