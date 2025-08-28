---
title: "Phong,Blinn Phong 비교"
date : "2025-08-28 19:30:00 +0900"
last_modified_at: "2025-08-28T19:30:00"
categories:
  - Direct X
tags:
  - Phong
  - BlinnPhong
---

## Phong vs Blinn Phong?

### Phong Reflection Model
물체의 반사 벡터와 시선 벡터 사이의 각도를 이용하여<br>
Specular(반짝임)을 계산하는 방식<br>

반사 벡터가 '픽셀'마다 달라지므로<br>
각 픽셀마다 반사 벡터를 구해야 한다<br>
(비용이 높은 편)<br>

더 작고, 날카로운 Specular 하이라이트를 표현<br>

### Bline Phong Model
Phong 모델이 매번 '반사 벡터'를 구해야 했기에<br>
70~80 년대 GPU에서 부담이 되었음<br>

그렇기에 반사 벡터 대신<br>
Half Vector를 사용하여 계산을 단순화<br>
(Reflection 대신, 눈과 빛의 방향을 이용)<br>

눈과 빛의 방향은 고정되어 있기에<br>
모든 픽셀에서 동일한 처리가 가능<br>

그래픽스 최적화 모델<br>
(물론 현대의 GPU에선 미미한 차이가 나긴 한다)<br>
(그래도 이쪽이 더 효율적)<br>

하이라이트가 더 넓고 부드럽게 나타난다<br>

#### 비교용 HLSL 코드

```
float3 BlinnPhong(float3 lightStrength, float3 lightVec, float3 normal,
                   float3 toEye, Material mat)
{
    if (useBlinnPhong)
    {
        float3 halfway = normalize(toEye + lightVec);
        float hdotn = dot(halfway, normal);
        float3 specular = mat.specular * pow(max(hdotn, 0.0f), mat.shininess * 2.0); // Shiness 값을 조금 더 크게 사용
        return mat.ambient + (mat.diffuse + specular) * lightStrength;
    }
    else
    {
        float3 r = -reflect(lightVec, normal);
        float3 specular = mat.specular * pow(max(dot(toEye, r), 0.0f), mat.shininess);
        return mat.ambient + (mat.diffuse + specular) * lightStrength;
    }
}
```

## 비교 이미지

비교는 SpotLight 를 사용하였다<br>

### 비교 이미지 1

- Phong<br>

<img width="2237" height="1717" alt="Image" src="https://github.com/user-attachments/assets/190cd9be-6c73-42c5-a264-39169f5faac5" /><br>

중앙 부분에 더 하이라이트가 집중되어 보인다<br>


- Blinn Phong<br>

<img width="2235" height="1715" alt="Image" src="https://github.com/user-attachments/assets/a40813c4-0e3c-4ff9-9da7-71bbbe425702" /><br>

전반적으로 더 부드러워 보인다<br>

### 비교 이미지 2

보는 시점의 각도를 다르게 해봤다<br>

- Phong<br>

<img width="2237" height="1723" alt="Image" src="https://github.com/user-attachments/assets/0b0312f1-abbc-46b2-8242-a9c41ce84489" /><br>

SpotLight의 경계선이 날카롭게 끊어짐<br>
(손전등이 비치지 않는 부분에 대한 날카로운 표현 가능)<br>

- Blinn Phong<br>

<img width="2229" height="1717" alt="Image" src="https://github.com/user-attachments/assets/7de8efc8-caf2-4156-8444-43761c89ce8d" /><br>

비교적 넓은 범위까지 퍼지는 편이다<br>
(뚝 끊긴다기 보단 좀 더 멀리서 쏜 조명 같은 느낌이 든다)<br>