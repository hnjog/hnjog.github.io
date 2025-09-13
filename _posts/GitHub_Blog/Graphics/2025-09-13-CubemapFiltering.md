---
title: "CubeMap Filtering tool"
date : "2025-09-13 16:00:00 +0900"
last_modified_at: "2025-09-13T16:00:00"
categories:
  - Direct X
  - Graphics
tags:
  - CubeMap Filtering Tool
  - CMFT
---

## cmft Studio 사용법 소개

[![Image](https://github.com/user-attachments/assets/4f61df1e-9ee5-403c-ab5b-dcadb9ac40f3)](https://github.com/user-attachments/assets/4f61df1e-9ee5-403c-ab5b-dcadb9ac40f3){: .image-popup}<br>

이전 시간에 다루었던 큐브맵들을<br>
(Diffuse, Specular IBL 시리즈)<br>
직접 만들려 할 때<br>
사용할 수 있는 프로그램<br>

[cmft studio Git](https://github.com/dariomanesku/cmftStudio){:target="_blank"}<br>

- 큐브맵 필터링 툴<br>

- 오픈 소스이기에 직접 git에서 받아 빌드하여 사용해도 됨<br>

- 아니면 이미 readme 쪽에 이미 빌드된 파일을 다운 받을 수 있다<br>

### 큐브맵으로 사용할 원본에 대한 주의사항?

상 하 전 후 좌 우<br>
각각 6개의 이미지를 묶어서 큐브맵으로 만든다<br>

- texassemble.exe로 .dds 파일로 만듦<br>

다만 이렇게 만들고 cmft studio를 사용하려 할때<br>
주의점이 있음<br>

- 6개의 텍스쳐를 다 치고나서 마지막에 BGRA를 입력해야 함<br>
  (일반적인 텍스쳐 포맷은 RGBA임)<br>

```bat
texassemble cube ^
  -f B8G8R8A8_UNORM ^
  -o env_cubemap.dds ^
  -y ^
  right.jpg left.jpg top.jpg bottom.jpg front.jpg back.jpg
```

- 일부 시각적 라이브러리들은 BGRA 포맷을 사용하는 경우가 존재함<br>

### 사용법
exe 파일과 conf 파일을 텍스쳐와 .dds 파일이 있는곳에 같이둔후<br>
exe 파일을 실행하기<br>

그러면 이러한 화면이 뜬다<br>

[![Image](https://github.com/user-attachments/assets/8039ca2d-5833-4fae-97a4-a33240ce1d6e)](https://github.com/user-attachments/assets/8039ca2d-5833-4fae-97a4-a33240ce1d6e){: .image-popup}<br>

오른쪽에 있는 이미지를 클릭하면 이러한 ui가 뜬다<br>

[![Image](https://github.com/user-attachments/assets/f066aa95-3213-4b9a-8c6e-2e0e652253cd)](https://github.com/user-attachments/assets/f066aa95-3213-4b9a-8c6e-2e0e652253cd){: .image-popup}<br>

- Skybox : 원본 이미지가 들어가야할 자리<br>
- Radiance : Specular을 표현(반사되어 나오는 빛)<br>
- Irradiance : Diffuse를 표현(사방으로 들어오는 빛)<br>

각각 Browse를 통해 이미지를 불러들일 수 있음<br>

- 같은 경로에 두었다면 바로 읽어들일 수 있음<br>
  (아까 만들었던 BGRA 포맷을 불러오자)<br>


[![Image](https://github.com/user-attachments/assets/9ab05174-2ac0-4248-b997-d6a9aa400df8)](https://github.com/user-attachments/assets/9ab05174-2ac0-4248-b997-d6a9aa400df8){: .image-popup}<br>

배경이 바뀐 모습이 보인다<br>

[![Image](https://github.com/user-attachments/assets/065a7e94-fc88-4f7d-a3c4-d844f45d3120)](https://github.com/user-attachments/assets/065a7e94-fc88-4f7d-a3c4-d844f45d3120){: .image-popup}<br>

이후 Radiance 쪽에서 Fillter Skybox with cmft 를 누르고<br>
process 를 누르면 된다<br>

- 다만 가끔씩 스레드가 계속 작동중이라면서 로딩이 끝나지 않을때가 존재<br>
  (그냥 껏다 다시 실행해보자)<br>

[![Image](https://github.com/user-attachments/assets/1b1a2adb-78ae-4c28-a7cc-9f303976c7c0)](https://github.com/user-attachments/assets/1b1a2adb-78ae-4c28-a7cc-9f303976c7c0){: .image-popup}<br>

로딩이 끝났을때의 모습<br>
Reflection 처럼 코팅된 모습이 보인다<br>
('반사된' 듯한 모습 : Radiance)<br>

- 그렇기에 IBL에서 Specular로 사용할 수 있음<br>

[![Image](https://github.com/user-attachments/assets/044cd48b-2bbb-40e6-9a26-42388ccde746)](https://github.com/user-attachments/assets/044cd48b-2bbb-40e6-9a26-42388ccde746){: .image-popup}<br>

Irradiance도 같은 방식으로 진행!<br>

Fillter Skybox with cmft 를 누르고<br>
process 를 누르면 된다<br>

[![Image](https://github.com/user-attachments/assets/b16b8e3b-5f4a-49db-90be-88074f609913)](https://github.com/user-attachments/assets/b16b8e3b-5f4a-49db-90be-88074f609913){: .image-popup}<br>

Irradiance에도 이전에 보았던<br>

Diffuse IBL 처럼 뿌연 Blur 같은 모습이 되었다<br>

- 역시 Diffuse IBL로 사용이 가능<br>

[![Image](https://github.com/user-attachments/assets/b9e75fa5-802d-4784-b9e5-21f21d78476c)](https://github.com/user-attachments/assets/b9e75fa5-802d-4784-b9e5-21f21d78476c){: .image-popup}<br>

- Save를 눌러 저장해주자<br>

[![Image](https://github.com/user-attachments/assets/40de8a0f-6899-4079-beca-c5f06ffcf7bc)](https://github.com/user-attachments/assets/40de8a0f-6899-4079-beca-c5f06ffcf7bc){: .image-popup}<br>

- Name에서 사용할 IBL 타입으로 이름을 바꿔주자<br>
- 파일 타입 고르기 (우리는 DX에서 사용할 것이므로 .dds)<br>
- outputType 을 Cubemap으로 설정 : 환경맵으로 사용할 예정이므로<br>
- RGRA8 로 포맷 설정하기 (float4)<br>
- 이후 저장을 하면 IBL 준비가 끝난다<br>

다만 이전에도 말했듯<br>
이것이 완벽한 IBL은 아님<br>
나중에 다시 IBL에 대하여 공부할 예정이다<br>