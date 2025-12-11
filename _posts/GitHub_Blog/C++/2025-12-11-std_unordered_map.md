---
title: "std::unordered_map"
date : "2025-12-11 14:00:00 +0900"
last_modified_at: "2025-12-11T14:00:00"
categories:
  - C++
tags:
  - C++
  - unordered_map
---

## std::unordered_map

- 키 - 값 (Key - Value) 쌍을 저장하는 컨테이너<br>
  (내부 원소 타입은 pair<>)<br>
  - 일반적으로 '유일 키' 방식<br>

- 해시 테이블을 기반으로 작동<br>
  - Key를 Hash 함수에 넣어 Hash 값을 만든 후,<br>
    bucket_count 만큼 나누어 지정된 위치에 요소를 저장<br>
  - '임의의 위치'에 저장하기에<br>
    Iterator 기반 탐색 시, '정렬되지 않은' 상태로 순회<br>
    ('unordered'라 말하는 이유)<br>

- 삽입/삭제/접근 이 모두 O(1)!<br>
  - 다만 '해시 충돌'이 발생하지 않는 가정!<br>

- 해시 충돌?<br>
  : '다른 key'임에도 해시 함수가 내놓은 Hash 값이 같은 경우<br>
  - 이때, 동일한 '위치'에 요소를 저장해야 하는 상황이 발생<br>
  - unordered_map은 내부적으로 '연결 리스트'를 사용하는 '체이닝' 방식으로<br>
    해당 문제를 해결<br>
    - 다만, 해시 충돌이 너무 잦게 발생하면 '연결 리스트' 탐색으로 인하여<br>
      점점 탐색이 느려질 수 있음<br>

- 내부적으로 vector의 capacity와 비슷하게<br>
  bucket_count 라는 요소가 존재<br>
  - 실제 요소의 개수가 버켓 카운트 이상이 되면<br>
    (실제로는 (저장 원소 개수) / (버캣 개수) > 1.0)<br>
    충돌 확률이 높아지기에 'rehash'가 발생함!
  - reserve로 초기에 설정 가능<br>
    (임의의 size 예상이 가능하다면, reserve를 통해<br>
    rehash 빈도를 줄일 수 있음)<br>

- `rehash`?<br>
  : bucket_count의 개수를 늘리고<br>
    기존의 데이터를 다시 해싱하며<br>
    '새로운' 버켓 위치에 배치하는 과정<br>
    모든 요소를 전부 건드리기에 비용이 큰 작업(O(N))<br>
    - unordered_map 사용시, 어느 순간 성능이 저하되는 가장 큰 이유!<br>

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    // 선언: Key는 string, Value는 int
    std::unordered_map<std::string, int> scores;

    // 1. 데이터 삽입
    scores["Alice"] = 90;
    scores["Bob"] = 85;
    scores.insert({"Charlie", 95});

    // 2. 데이터 수정 (Key가 이미 존재하면 Value 업데이트)
    scores["Alice"] = 92; 

    // 3. 데이터 검색 (안전한 방법)
    std::string target = "Bob";
    
    // find()는 데이터를 못 찾으면 end() iterator를 반환함
    if (scores.find(target) != scores.end()) {
        std::cout << target << "의 점수: " << scores[target] << std::endl;
    } else {
        std::cout << target << "을(를) 찾을 수 없습니다." << std::endl;
    }

    // 4. 전체 순회 (Iterator 사용)
    // 주의: 출력 순서는 입력 순서와 다르며, 뒤죽박죽일 수 있음
    std::cout << "\n--- 전체 명단 ---" << std::endl;
    for (const auto& pair : scores) {
        std::cout << "이름: " << pair.first << ", 점수: " << pair.second << std::endl;
    }

    // 5. 대괄호 [] 연산자의 주의점
    // 존재하지 않는 키를 []로 접근하면, 기본값(0)으로 자동 생성해버림
    std::cout << "\nDavid 점수 확인: " << scores["David"] << std::endl; // David가 0으로 생성됨
    std::cout << "현재 크기: " << scores.size() << std::endl; // 크기가 1 늘어남

    return 0;
}
```

### Hash 에 대하여
([Hash Table?](https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/Week1_Hash_Table/))<br>

- Hash 함수와 Hash 값<br>
  - Hash Function <br>
    : 임의의 데이터를 특정한 타입의 '값'으로 반환하는 함수<br>
    - 기초적으론 나눗셈이나 곱셈을 기반으로 버킷 위치를 정하거나<br>
      속도와 '충돌 방지'를 위한 것(MurMurHash, FNV 등)<br>
      아니면 역추적을 방지위한 해시 함수 등(MD5, SHA 시리즈)<br>
       매우 다양한 종류의 해시 함수가 존재<br>
  - Hash Value<br>
   : 해시 함수에서 내놓은 결과물<br>

- Hash Collision<br>
  : 서로 다른 '데이터'를 '해시 함수'에 집어넣었는데<br>
    그 결과값(해시 값)이 같은 현상<br>
    - 함수는 '다른 입력'으로 '같은 결과'를 충분히 내놓을 수 있음<br>
      (모든 입력에 대하여 결과가 1:1로 매핑되는 것이 아니기에)<br>

- 해시 충돌의 해결법들<br>
  - Chaining<br>
    : 데이터를 집어넣는 '같은 위치'에<br>
      여러 값을 집어넣을 수 있게 하는 방식<br>
      - 연결 리스트를 통해 값들을 이어 붙인 방식<br>
      - 충돌이 너무 잦을 시, 접근이 '연결 리스트' 탐색과 비슷하게 됨<br>
  - Open Addressing(개방 주소법)<br>
    : 충돌이 나면 '다른 빈칸'을 찾아<br>
      그 위치에 값을 집어넣는 방식<br>
      - '다른 빈칸'을 정하는 방식이 다양함<br>
        +1, 제곱, 나누기 등등<br>


### unordered_map vs TMap(Unreal)?

- 해시 충돌 처리 기법의 차이<br>
  : unordered_map은 '체이닝'<br>
  TMap은 '개방 주소법'을 사용 (속도를 위한 '배열 기반')<br>
  - 배열을 기반으로 하기에<br>
    '캐시 히트율'이 높아짐<br>

- 또한 TMap은 Unreal 최적화가 되어 있음<br>
  - Reflection<br>
  - GC<br>

정리 표<br>

| 비교 항목 | std::unordered_map (C++ STL) | TMap (Unreal Engine) |
| :--- | :--- | :--- |
| **기반 자료구조** | 해시 테이블 (Hash Table) | 해시 테이블 (Hash Table) |
| **충돌 해결 (Collision)** | **체이닝 (Chaining)** (연결 리스트 사용) | **개방 주소법 (Open Addressing)** (선형 조사 등) |
| **메모리 구조** | **노드 기반** (비연속적, 파편화 발생) | **배열(TArray) 기반** (연속적, 메모리 친화적) |
| **캐시 효율 (Cache)** | 낮음 (포인터 점프 발생, Cache Miss 증가) | **높음** (데이터가 모여있어 Cache Hit 유리) |
| **데이터 순회 속도** | 상대적으로 느림 | **매우 빠름** (인덱스 접근) |
| **메모리 할당 횟수** | 데이터 추가 시마다 발생 (`new` 호출 잦음) | 배열 크기가 찰 때만 재할당 (상대적으로 적음) |
| **언리얼 GC 연동** | 불가능 (`UObject*` 보호 못함, 댕글링 포인터 위험) | **가능** (`UPROPERTY` 지정 시 객체 참조 관리) |
| **직렬화/리플렉션** | 별도 변환 필요 | 언리얼 시스템 완벽 지원 |
| **삭제 비용** | O(1) (단순 링크 해제) | 상대적으로 복잡 (구멍(Hole) 관리 및 데이터 이동 가능성) |