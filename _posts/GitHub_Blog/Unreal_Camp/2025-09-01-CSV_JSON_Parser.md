---
title: "CSV - Json Parser"
date : "2025-09-01 15:00:00 +0900"
last_modified_at: "2025-09-01T15:00:00"
categories:
  - C++
tags:
  - 인코딩
  - UTF-8
  - Json
  - CSV
---

## CSV -> Json -> C++ 파이프라인
팀 프로젝트 TextRPG에서<br>
CSV -> Json으로 파싱한 후<br>
그것을 C++로 읽어 들이면 데이터 기반 프로그래밍이 되지 않을까 싶어<br>
간략하게 ChapGPT의 힘을 빌려 간단하게 제작하는 것이 이번 목표였다<br>

그리고 이번 제작 중 각종 이슈가 있었기에<br>
가볍게 TIL을 남겨볼까 한다<br>

### 선요약
- 내부의 '코어 프로젝트'는 UTF-8(BOM) (std::string)을 고정하되<br>
  OS/ 콘솔 / 파일 경계 등 필요할 때 UTF-16(std::wstring)으로 변환<br>

- 소스파일(h/cpp)의 인코딩은 UTF-8로 저장(보통 65001)<br>
  MSVC (Microsoft Visual C++ 컴파일러) 에게 /utf-8 명령어를 사용<br>
  (프로젝트 속성 → C/C++ → 명령줄 → 추가 옵션: /utf-8)<br>
  - BOM: 파일 맨 앞의 마커. UTF-8(EF BB BF), UTF-16LE(FF FE), UTF-16BE(FE FF)<br>

- 콘솔은 SetConsoleOutputCP(CP_UTF8); 후 std::cout만 쓴다(혼용 금지)<br> 
  (같이 사용하면 출력시 글자가 '깨지기' 쉬움)<br>

- 입력 파일에서 BOM 감지한 후, 제거하여 UTF-8 정규화 후 파싱<br>

## CSV->Json->C++ 파이프라인 제작 이유? (들어가기 앞서)

기본적으로 '기획자 / 밸런스 디자이너'분들과의 협업을 고려한 방식이다<br>

- CSV (스프레드 시트)가 접근성이 좋으며 익숙함<br>
- 게임 코드 쪽에서 Json 기반이 파싱하기 쉬운 편<br>

그렇기에 '툴'을 이용하여<br>
기획 측 데이터(CSV) -> 게임 쪽 사용 데이터(Json) -> 실제 게임(C++)로 이용<br>

### 장점?

- 비개발자 친화<br>
  (CSV만 수정하면 되기에 비개발자가 C++ 코드를 만질 필요 x)<br>
  (데이터 수정에 따른 재빌드가 없어짐)<br>

- 다양한 타이밍의 검증이 가능<br>
  (데이터 입력 실수 -> 데이터 수치가 이상 -> 실제 인게임에서 밸런스 테스트 하며 검증)<br>
  (단순한 실수 등을 '에러' 반환으로 피드백이 가능)<br>

- 데이터 버전 관리<br>
  (CSV,Json 파일의 변경까지 Git으로 관리 가능)<br>


## 인코딩과 문자열

| 구분             | 내용                 | 언제 쓰나                       | 장점                | 주의점              |
| -------------- | ------------------ | --------------------------- | ----------------- | ---------------- |
| UTF-8          | 가변 길이, ASCII 호환    | CSV/JSON/네트워크/내부 데이터        | 이식성·툴 호환 최고       | 콘솔/OS 호출 시 변환 필요 |
| UTF-16         | 2바이트 기반(Win 친화)    | Win API, UE/Unity, 한글 경로/콘솔 | 한글 확실, Win API 직결 | 크로스플랫폼 부담, 메모리↑  |
| `std::string`  | 바이트 컨테이너(보통 UTF-8) | **프로젝트 코어 전반**              | 단순, 생태계 친화        | 출력/OS 경계에서 변환 필요 |
| `std::wstring` | Windows에선 UTF-16   | 파일경로/콘솔/GUI 등 **Win W API** | 한글 안전, Win API 호환 | 팀 협업·이식성 저하 가능   |


- Windows의 파일 경로/ 콘솔/ GUI / 레지스트리 등의 Win32 W API 호출 시<br>
  Wstring 사용을 고려해야 함<br>
  (+ 엔진이 UTF-16 기반일때도 (UE/Unity))<br>

- CSV/Json/ 네트워크/ 직렬화 등의 일반적인 데이터 로직에선<br>
  string (UTF-8) 고려<br>

다행히도<br>
게임 엔진 쪽에서는 '엔진이 제공하는 문자열 타입 및 IO' 사용 시<br>
이러한 인코딩을 직접 신경쓸 일은 별로 없음<br>

| 엔진            | 주 문자열 타입                  | 내부 인코딩/개념                  | 개발자가 보통 쓰는 것                                                        | 변환을 신경쓸 타이밍                                                   |
| ------------- | ------------------------- | -------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------- |
| Unreal Engine | `FString`/`FText`/`TCHAR` | 플랫폼 추상(윈도우에선 사실상 UTF-16계열) | `TEXT("…")`, `FString`, `UE_LOG`, `FFileHelper::LoadFileToString` 등 | OS API 직접 호출, Raw `char*` 라이브러리(UTF-8)와 연동, 네트워크/툴에서 바이트 다룰 때 |
| Unity         | C# `string`               | UTF-16                     | `File.ReadAllText(path, Encoding.UTF8)`, `JsonUtility`/Newtonsoft   | 네이티브 플러그인(C/C++)와 마샬링할 때, 바이너리/바이트 버퍼 직접 다룰 때                 |
| 기타(사내/커스텀)    | 엔진 타입(각자 다름)              | 대개 문자열 추상 제공               | 엔진의 FS/로깅/리소스 API                                                   | OS·서드파티·툴과 “경계”에서                                             |

그래도 다음과 같은 상황에서는 고려해볼 것<br>

- OS 의 API를 직접 호출하거나<br>
- 외부 라이브러리 / 플러그인 사용(char* = UTF-8 기대) : 엔진 문자열과 인코딩이 다를 수 있음<br>
- CSV/Json 같은 텍스트 파일을 raw 바이트로 직접 읽을 때 : 파일의 인코딩이 여러가지 섞여있을 수 있음<br>
  (요번처럼 BOM 감지 후, UTF-8 로 정규화 후 파싱 권장)<br>
- 콘솔/로그 를 엔진 바깥으로 내보낼때<br>

## 이슈와 해결

- 한글이 깨지는 경우<br>
  : 콘솔, vs 내부에서 표시 인코딩이 ANSI 되었다<br>
  처음에는 데이터를 잘못 파싱한줄 알았으나<br>
  '표시 인코딩'의 불일치로 인한 것<br>
  
- 일부 문자열이 출력이 스킵되는 경우<br>
  : cout와 wcout의 혼용으로 인한 문제<br>
    처음에는 '한글'을 wstring으로 저장하였기에<br>
    해당 부분을 출력하고자 wcout와 cout 를 같이 사용했었음<br>
    
=> SetConsoleOutputCP(CP_UTF8);<br>
  SetConsoleCP(CP_UTF8); 를 사용하여 해결<br>
  (+ MSVC - /utf-8 명령어 추가)<br>

(원래는 ANSI 경로 방식이 아닌<br>
WIDE 방식을 사용하는 것이 정석)<br>

#### ANSI 경로 vs Wide 경로

| 구분          | 의미                   | 대표 API/스트림                                                                              | 인코딩 해석                         |
| ----------- | -------------------- | --------------------------------------------------------------------------------------- | ------------------------------ |
| **ANSI 경로** | `char` 기반, 코드페이지에 의존 | `WriteConsoleA`, `ReadConsoleA`, `CreateFileA`, `printf`, `scanf`, `std::cout/std::cin` | **현재 코드페이지(CP)** 로 바이트를 문자로 해석 |
| **Wide 경로** | `wchar_t`(UTF-16) 기반 | `WriteConsoleW`, `ReadConsoleW`, `CreateFileW`, `std::wcout/std::wcin`                  | **UTF-16** 직통 (코드페이지 무관)       |


## 결과

- CSV<br>

<img width="735" height="197" alt="Image" src="https://github.com/user-attachments/assets/6469bfb9-ca4d-4958-939d-883bd6699b62" /><br>

- Json<br>

```
[{"Effect":"Heal","Idx":"1","Name":"회복포션","Type":"Consume","Value":"50"},
{"Effect":"Refrain","Idx":"2","Name":"마나포션","Type":"Consume","Value":"30"}]
```

- C++ <br>

```
void DataManager::LoadItemsJson(const JsonValue& root)
{
	ItemDataVector.clear();

	// 우리가 쓰는 스키마: 최상위가 배열
	if (root.type != JsonValue::Type::Array)
		return;

	ItemDataVector.reserve(root.arr.size());
	for (const auto& obj : root.arr) {
		if (obj.type != JsonValue::Type::Object)
			continue;

		ItemBase it;

		const JsonValue* pIdx = obj.get("Idx");
		if (pIdx)
		{
			if (pIdx->type == JsonValue::Type::String)
				it.idx = static_cast<int>(std::strtol(pIdx->str.c_str(), nullptr, 10));
			else if (pIdx->type == JsonValue::Type::Number)
				it.idx = static_cast<int>(pIdx->number);
		}

		const JsonValue* pName = obj.get("Name");
		if (pName && pName->type == JsonValue::Type::String)
			it.name = (pName->str);

		const JsonValue* pEffect = obj.get("Effect");
		if (pEffect && pEffect->type == JsonValue::Type::String)
			it.effect = (pEffect->str);

		const JsonValue* pType = obj.get("Type");
		if (pType && pType->type == JsonValue::Type::String)
		{
			it.type = ParseItemType(pType->str);
		}

		const JsonValue* pValue = obj.get("Value");
		if (pValue)
		{
			if (pValue->type == JsonValue::Type::String)
				it.value = static_cast<int>(std::strtol(pValue->str.c_str(), nullptr, 10));
			else if (pValue->type == JsonValue::Type::Number)
				it.value = static_cast<int>(pValue->number);
		}

		ItemDataVector.push_back(it); // 이후 Move를 통해 실제 관리할 Manager가 가져감
	}
}
```

- 실제 출력<br>

<img width="1703" height="873" alt="Image" src="https://github.com/user-attachments/assets/f40604a7-eb13-48a6-a745-225dbc56ca08" /><br>

```
출력 코드

void ItemManager::PrintAllItems()
{
	for (auto& item : ItemDatas)
	{
		std::cout << "==========================" << '\n';
		std::cout << "아이템 이름 : " << item.name << '\n';
		std::cout << "아이템 효과 : " << item.effect << '\n';
		std::cout << "아이템 수치 : " << item.value << '\n';
		std::cout << "아이템 인덱스 : " << item.idx << '\n';
	}
}
```
