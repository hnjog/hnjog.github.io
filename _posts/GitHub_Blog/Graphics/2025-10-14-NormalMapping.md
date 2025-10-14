---
title: "Normal Mapping 1"
date : "2025-10-14 20:00:00 +0900"
last_modified_at: "2025-10-14T20:00:00"
categories:
  - 그래픽스
  - DirectX
tags:
  - Normal Mapping
---

## Normal Mapping

[![Image](https://github.com/user-attachments/assets/da9ce32d-3394-4f9c-bfad-d2c4e4a84abc)](https://github.com/user-attachments/assets/da9ce32d-3394-4f9c-bfad-d2c4e4a84abc){: .image-popup}<br>

메시의 폴리곤을 늘리지 않고<br>
표면의 '미세한 기복'을 '노멀' 방향만 바꾸어<br>
`마치 있는 것` 처럼 표현하는 일종의 **트릭**<br>

[![Image](https://github.com/user-attachments/assets/7b277d26-4835-4596-b7d4-c83f3982e1c0)](https://github.com/user-attachments/assets/7b277d26-4835-4596-b7d4-c83f3982e1c0){: .image-popup}<br>

- 텍스쳐의 RGB에 XYZ 노멀 벡터를 담아<br>
  쉐이더에서 TBN으로 월드/뷰 행렬 변환 후<br>
  라이팅 계산에서 주로 사용<br>

- 별도의 폴리곤 증가 없이 세세한 표현이 (주로 빛)<br>
  가능하기에 성능적인 효율이 좋음<br>

- 표현의 한계는 존재하나 디테일을 효율 좋게 표현 가능<br>

[![Image](https://github.com/user-attachments/assets/8b0ad015-600b-4384-b13c-b21ebf138ec0)](https://github.com/user-attachments/assets/8b0ad015-600b-4384-b13c-b21ebf138ec0){: .image-popup}<br>

[![Image](https://github.com/user-attachments/assets/2915414a-2626-4dc5-a32c-a59cfc3af05e)](https://github.com/user-attachments/assets/2915414a-2626-4dc5-a32c-a59cfc3af05e){: .image-popup}<br>

- GL 과 DX 용으로 Normal 이 분리되어 있는 편이 많음<br>
  (텍스쳐 좌표계의 차이로 180도 회전의 차이가 존재)<br>

### Albedo? Diffuse?

- Albedo 대신 Diffuse, 그 반대도 보통 사용 가능<br>
  그렇다고 개념이 완전히 같은 것은 아님<br>
  (또 Albedo는 base color 같은 용어와 혼용됨)<br>
  - Albedo : 입사해서 들어오는 빛을 '반사'해내는 `비율`<br>
    (빛/그림자/AO/하이라이트가 전혀 들어가지 않은 순수한 표면 색)<br>
  - Diffuse Color : diffuse 반사에 의해 우리가 보게되는 `색`<br>
    (과거의 Diffuse 소스들은 종종 위의 요소들이 '섞여'있었음 - 당시의 기술적 한계)<br>
  - 최신 PBR에서는 Albedo를 기반으로 생각하며 Diffuse는 약간 legacy 같은 느낌<br>


## 예제 1 - 픽셀 쉐이더를 이용한 구체 그리기

[![Image](https://github.com/user-attachments/assets/97cea313-3cd8-4cd7-a699-d606d95c5c84)](https://github.com/user-attachments/assets/97cea313-3cd8-4cd7-a699-d606d95c5c84){: .image-popup}<br>

텍스쳐 좌표를 이용하여 풀 수 있는 예제<br>

```cpp
pixelshader.hlsl

float3 normalWorld = input.normalWorld;

if (useNormalMap) // NormalWorld를 교체
{
    float3 center = float3(0.5, 0.5, 0.3);
    float2 f = input.texcoord - center.xy;
    float f2 = dot(f, f);
    if(f2 < 0.3 * 0.3)
    {
        float z = sqrt(0.3 * 0.3 - f2);
        // DX 좌표계를 고려
        float3 p = float3(f * float2(1, -1), -z);
        normalWorld = normalize(p);
    }
    else
    {
        normalWorld = float3(0, 0, -1);
    }
}
```

- f : 중앙으로부터 현재 텍스쳐 좌표로부터의 오프셋<br>
- f2 : 중심으로부터의 거리 제곱을 통해 거리 파악<br>
- 0.3 : 임의로 잡은 구의 반지름<br>
- float2 좌표가 1,-1 인 이유는 dx 좌표계의 특성 때문!<br>
- z값이 음수인 이유는 해당 normal들이 카메라를 바라보고 있기 때문<br>
