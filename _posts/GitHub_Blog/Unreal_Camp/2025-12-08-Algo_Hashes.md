---
title: "문승현 튜터님 강의 - '알고리즘 : 해시, 정렬, 탐색'"
date : "2025-12-08 18:00:00 +0900"
last_modified_at: "2025-12-08T18:00:00"
categories:
  - C++
  - Algorithm
tags:
  - C++
  - Algorithm
  - 해시
  - 정렬
  - 탐색
---

# 알고리즘 : 해시, 정렬, 탐색

데이터가 아무리 많아도 "순식간에 찾는 법"과<br>
"깔끔하게 정리하는 법"을 알면 문제는 단순해집니다.<br>

## **1. 해시 (Hash)**

### **1.1 개념 및 특징**

해시는 "번호표(Key)"만 건네주면 직원이 "내 짐(Value)"이<br>
있는 위치로 바로 가서 가져다주는 시스템입니다.<br>

거대한 도서관에서 책을 처음부터 끝까지 찾는 것(탐색 O(N))이 아니라,<br>
청구기호 하나로 바로 위치를 특정하는 것(해시 O(1))과 같습니다.<br>

- **시간 복잡도**<br>
    - 데이터 넣기/빼기/찾기: $O(1)$ (즉시 가능)<br>
- **언제 쓸까?**<br>
    - **빠른 검색:** 리스트를 다 뒤지기엔 시간이 부족할 때.<br>
    - **중복 확인:** "이거 아까 봤던 건가?" 체크할 때.<br>
    - **카운팅:** 투표 수, 단어 등장 횟수 세기.<br>

### **대표 예제 1 : 두 수의 합 (Two Sum)**<br>

해시맵(Dictionary)을 왜 써야 하는지 보여주는 가장 기본 문제입니다.<br>

- **문제:** 정수 배열 `nums`와 목표값 `target`이 주어집니다. 더해서 `target`이 되는 두 숫자의 인덱스를 찾으세요. (정답은 하나만 존재)<br>
- **입력:** `nums = [2, 7, 11, 15]`, `target = 9`<br>
- **출력:** `[0, 1]`<br>
- **해결 논리:**<br>
    1. 빈 해시맵(기록장)을 준비합니다.<br>
    2. 숫자를 하나씩 보면서, **`target - 현재숫자` (내 짝꿍)**가 기록장에 있는지 확인합니다.<br>
    3. 있으면? 짝꿍의 인덱스와 내 인덱스를 반환합니다.<br>
    4. 없으면? **현재 숫자와 인덱스**를 기록장에 적어두고 다음으로 넘어갑니다.<br>
- **정답 예시 코드**<br>

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> memo; // Key: 값, Value: 인덱스
    
    for (int i = 0; i < nums.size(); i++) {
        int partner = target - nums[i];
        
        // 1. 짝꿍 찾기
        if (memo.find(partner) != memo.end()) {
            return {memo[partner], i};
        }
        
        // 2. 나를 기록하기
        memo[nums[i]] = i;
    }
    return {};
}
/* 테스트 케이스
Input: nums = [3, 2, 4], target = 6
Logic: 
    - 3 확인: (6-3=3) 없음 -> 기록 {3:0}
    - 2 확인: (6-2=4) 없음 -> 기록 {3:0, 2:1}
    - 4 확인: (6-4=2) 있음! -> {1, 2} 리턴
Result: [1, 2]
*/
```


### 대표 예제 2 : 완주하지 못한 선수 (프로그래머스)

데이터의 개수를 세는(Counting) 전형적인 해시 문제입니다.<br>

- **문제:** 마라톤 참가자 명단과 완주자 명단이 주어집니다. 완주하지 못한 단 한 명의 이름을 찾으세요. (동명이인이 있을 수 있음)<br>
- **입력:** `participant = ["leo", "kiki", "eden"]`, `completion = ["eden", "kiki"]`<br>
- **출력:** `"leo"`<br>
- **해결 논리:**<br>
    1. 해시맵을 이용해 **출석부**를 만듭니다. (Key: 이름, Value: 인원수)<br>
    2. 참가자 명단을 돌며 이름을 부를 때마다 카운트를 +1 합니다.<br>
    3. 완주자 명단을 돌며 이름을 부를 때마다 카운트를 -1 합니다.<br>
    4. 마지막에 카운트가 0이 아닌 사람(남은 사람)이 범인입니다.<br>
- **정답 예시 코드**<br>

```cpp
string solution(vector<string> participant, vector<string> completion) {
    unordered_map<string, int> map;
    
    // 1. 참가자 등록
    for (string name : participant) map[name]++;
    
    // 2. 완주자 체크
    for (string name : completion) map[name]--;
    
    // 3. 미완주자 찾기
    for (auto pair : map) {
        if (pair.second > 0) return pair.first;
    }
    return "";
}
/* 테스트 케이스
Input: ["mislav", "stanko", "mislav", "ana"], ["stanko", "ana", "mislav"]
Logic: mislav는 2명인데 완주는 1명 -> map["mislav"] == 1
Result: "mislav"
*/
```


### **대표 예제 3 (상): 가장 긴 연속 시퀀스 (Longest Consecutive Sequence)**

정렬 없이 O(N)으로 풀어야 해서 해시셋(Set)의 O(1) 검색 능력을 활용해야 하는 문제입니다.<br>

- **문제:** 정렬되지 않은 정수 배열에서, 연속된 숫자(1, 2, 3...)로 이루어진 가장 긴 구간의 길이를 구하세요.<br>
- **입력:** `[100, 4, 200, 1, 3, 2]`<br>
- **출력:** `4` (1, 2, 3, 4가 가장 긺)<br>
- **해결 논리:**<br>
    1. 모든 숫자를 `Set`에 넣습니다. (중복 제거 및 빠른 존재 확인)<br>
    2. Set에 있는 숫자를 하나씩 꺼냅니다. 이때, **내 앞 숫자(num - 1)가 존재하지 않는 경우**만 시작점으로 봅니다. (예: 2는 1이 있으니 시작점이 아님)<br>
    3. 시작점을 찾았다면, `num + 1`이 있는지 계속 물어보며 길이를 늘려갑니다.<br>
    4. 최대 길이를 갱신합니다.<br>
- 정답 예시 코드<br>

```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int longest = 0;
    
    for (int num : s) {
        // 내 앞의 숫자가 없으면 내가 시작점!
        if (s.find(num - 1) == s.end()) {
            int currentNum = num;
            int currentStreak = 1;
            
            while (s.find(currentNum + 1) != s.end()) {
                currentNum += 1;
                currentStreak += 1;
            }
            longest = max(longest, currentStreak);
        }
    }
    return longest;
}
/* 테스트 케이스
Input: [0, 3, 7, 2, 5, 8, 4, 6, 0, 1]
Logic: 0부터 시작 -> 0,1,2,3,4,5,6,7,8 존재 -> 길이 9
Result: 9
*/
```
    

## **2. 정렬 (Sorting)**

### **2.1 개념 및 특징**

"도서관 책 정리" 또는 "카드 오름차순 정리"입니다.<br>
데이터를 순서대로 나열해두면 찾기도 쉽고(이진 탐색), 비교하기도 쉬워집니다.<br>

- **시간 복잡도**<br>
    - 일반적인 효율적 정렬: O(N log N)<br>
- **언제 쓸까?**<br>
    - **순서가 중요할 때:** 랭킹, 최솟값/최댓값 구하기.<br>
    - **그리디 알고리즘:** "가장 큰 것부터 처리하자" 같은 전략을 쓸 때.<br>

### **대표 예제 1 : 유효한 애너그램 (Valid Anagram)**

정렬을 하면 데이터의 모양이 같아진다는 성질을 이용합니다.<br>

- **문제:** 두 문자열 `s`와 `t`가 주어졌을 때, 두 단어가 애너그램(철자 구성이 같음)인지 판별하세요.<br>
- **입력:** `s = "anagram", t = "nagaram"`<br>
- **출력:** `true`<br>
- **해결 논리:**<br>
    1. 두 문자열의 길이가 다르면 바로 False.<br>
    2. 두 문자열을 각각 정렬합니다. (예: "cat", "act" -> 둘 다 "act"가 됨)<br>
    3. 정렬된 결과가 똑같은지 비교합니다.<br>
- **정답 예시 코드**<br>

```cpp
bool isAnagram(string s, string t) {
    if (s.length() != t.length()) return false;
    sort(s.begin(), s.end());
    sort(t.begin(), t.end());
    return s == t;
}
/* 테스트 케이스
Input: s="rat", t="car"
Logic: 정렬 시 "art" != "acr"
Result: False
*/
```

### 대표 예제 2 : 가장 큰 수 (Largest Number)

단순 크기 비교가 아닌 '커스텀 비교(Custom Comparator)'를 정의해야 하는 문제입니다.<br>

- **문제:** 0 또는 양의 정수가 담긴 배열을 이어 붙여 만들 수 있는 가장 큰 수를 문자열로 반환하세요.<br>
- **입력:** `[3, 30, 34, 5, 9]`<br>
- **출력:** `"9534330"`<br>
- **해결 논리:**<br>
    1. 숫자를 문자열로 변환합니다.<br>
    2. 단순 사전순이 아니라 "A와 B를 붙였을 때 뭐가 더 큰가?"를 기준으로 정렬합니다.<br>
        - 3 vs 30 : "330" vs "303" -> 3이 앞에 와야 함.<br>
    3. 정렬된 문자열을 순서대로 이어 붙입니다.<br>
    4. (예외 처리) 맨 앞이 "0"이면 전체가 0이므로 "0" 반환.<br>
- **정답 예시 코드**<br>

```cpp
static bool compare(string a, string b) {
    return a + b > b + a; // 내림차순 기준
}

string largestNumber(vector<int>& nums) {
    vector<string> s_nums;
    for(int i : nums) s_nums.push_back(to_string(i));
    
    sort(s_nums.begin(), s_nums.end(), compare);
    
    if(s_nums[0] == "0") return "0";
    
    string ans = "";
    for(string s : s_nums) ans += s;
    return ans;
}
/* 테스트 케이스
Input: [10, 2]
Logic: "210" vs "102" -> 2가 10보다 우선 -> "210"
Result: "210"
*/
```


### 대표 예제 3 : 구간 병합 (Merge Intervals)

정렬을 통해 문제의 난이도를 확 낮추는 대표적인 유형입니다.<br>

- **문제:** 겹치는 구간들을 모두 하나로 합쳐서 반환하세요.<br>
- **입력:** `[[1,3],[2,6],[8,10],[15,18]]`<br>
- **출력:** `[[1,6],[8,10],[15,18]]`<br>
- **해결 논리:**<br>
    1. 구간의 **시작 시간**을 기준으로 오름차순 정렬합니다. (핵심)<br>
    2. 결과 배열에 첫 구간을 넣고 시작합니다.<br>
    3. 현재 구간의 시작점이 결과 배열의 마지막 구간의 **종료점**보다 앞서 있다면(겹친다면)?<br>
        - 종료점을 둘 중 더 큰 값으로 합칩니다(Merge).<br>
    4. 안 겹친다면? 그냥 결과 배열에 추가합니다.<br>
- **정답 예시 코드**<br>

```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    if (intervals.empty()) return {};
    sort(intervals.begin(), intervals.end()); // 시작점 기준 정렬
    
    vector<vector<int>> res;
    res.push_back(intervals[0]);
    
    for (int i = 1; i < intervals.size(); i++) {
        // res.back()[1]: 마지막 구간의 끝나는 시간
        // intervals[i][0]: 현재 구간의 시작 시간
        if (res.back()[1] >= intervals[i][0]) {
            res.back()[1] = max(res.back()[1], intervals[i][1]); // 병합
        } else {
            res.push_back(intervals[i]); // 추가
        }
    }
    return res;
}
/* 테스트 케이스
Input: [[1,4],[4,5]]
Logic: 4와 4가 겹침(접함) -> [1,5]로 병합
Result: [[1,5]]
*/
```


## 3. 탐색 (Search) - 이진 탐색

## **3.1 개념 및 특징**

"업-다운(Up-Down) 게임의 필승 전략"입니다.<br>
범위를 절반씩 뚝뚝 잘라내며 찾기 때문에 데이터가 10억 개라도 약 30번(log 10억) 만에 찾을 수 있습니다.<br>

- **필수 조건:** 데이터가 반드시 **정렬**되어 있어야 합니다.<br>
- **시간 복잡도:** O(log N)<br>

### **대표 예제 1 : 이진 탐색 (Binary Search)**

가장 기본적인 구현 문제입니다.<br>

- **문제:** 오름차순 정렬된 `nums`에서 `target`의 인덱스를 찾으세요. (없으면 -1)<br>
- **입력:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`<br>
- **출력:** `4`<br>
- **해결 논리:**<br>
1. `Left`(시작)와 `Right`(끝) 포인터를 설정합니다.<br>
    1. `Mid`(중간) 값이 `target`보다 작으면? 답은 오른쪽에 있습니다. (`Left = Mid + 1`)<br>
    2. `Mid` 값이 `target`보다 크면? 답은 왼쪽에 있습니다. (`Right = Mid - 1`)<br>
    3. 같으면 `Mid`를 반환합니다.<br>
- **정답 예시 코드**<br>

```cpp
int search(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while(left <= right){
        int mid = left + (right - left) / 2;
        if(nums[mid] == target) return mid;
        else if(nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
/* 테스트 케이스
Input: [-1, 0, 3, 5, 9, 12], 2
Logic: 끝까지 탐색해도 2는 없음
Result: -1
*/
```

### 대표 예제 2 (중): 회전된 정렬 배열 탐색 (Search in Rotated Sorted Array)

배열이 중간에 잘려서 순서가 뒤바뀐 상태에서의 탐색입니다.<br>

- **문제:** `[4,5,6,7,0,1,2]` 처럼 회전된 배열에서 `target`을 찾으세요. 시간 복잡도는 $O(\log N)$이어야 합니다.<br>
- **입력:** `nums = [4,5,6,7,0,1,2]`, `target = 0`<br>
- **출력:** `4` (인덱스)<br>
- **해결 논리:**<br>
    1. 반으로 잘랐을 때, **적어도 한쪽 절반은 반드시 정렬되어 있다**는 점을 이용합니다.<br>
    2. 왼쪽이 정렬되어 있는가? (`nums[left] <= nums[mid]`)<br>
        - Target이 그 범위 안에 있으면 왼쪽 탐색, 아니면 오른쪽 탐색.<br>
    3. 오른쪽이 정렬되어 있는가?<br>
        - Target이 그 범위 안에 있으면 오른쪽 탐색, 아니면 왼쪽 탐색.<br>
- **정답 예시 코드**<br>

```cpp
int search(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while(left <= right){
        int mid = left + (right - left) / 2;
        if(nums[mid] == target) return mid;
        
        // 왼쪽 절반이 정렬된 상태라면
        if(nums[left] <= nums[mid]) {
            if(nums[left] <= target && target < nums[mid]) 
                right = mid - 1;
            else 
                left = mid + 1;
        } 
        // 오른쪽 절반이 정렬된 상태라면
        else {
            if(nums[mid] < target && target <= nums[right]) 
                left = mid + 1;
            else 
                right = mid - 1;
        }
    }
    return -1;
}
/* 테스트 케이스
Input: [4,5,6,7,0,1,2], 3
Logic: 어느 구간에도 3은 없음
Result: -1
*/
```
    
### 대표 예제 3 (상): 입국 심사 (Parametric Search)

탐색의 대상을 '인덱스'가 아닌 **'정답(시간)'**으로 바꾸는 생각의 전환이 필요한 문제입니다.

- **문제:** 입국심사대별 처리 시간 `times`가 주어지고 `n`명이 대기 중입니다. 모든 사람이 심사를 마치는 **최소 시간**을 구하세요.<br>
- **입력:** `n = 6`, `times = [7, 10]`<br>
- **출력:** `28`<br>
- **해결 논리:**<br>
    1. 질문을 바꿉니다: **"X분 안에 n명을 모두 처리할 수 있는가?"** (`Yes/No` 결정 문제)<br>
    2. 시간의 범위 (1분 ~ 가장 오래 걸리는 경우)를 잡고 이진 탐색을 수행합니다.<br>
    3. `Mid` 시간 동안 처리 가능한 인원수를 계산합니다.<br>
        - 인원이 `n`보다 많거나 같으면? -> 시간은 충분함. 더 줄여보자 (`Right` 이동).<br>
        - 인원이 부족하면? -> 시간이 더 필요함 (`Left` 이동).<br>
- **정답 예시 코드**<br>

```cpp
long long solution(int n, vector<int> times) {
    long long answer = 0;
    sort(times.begin(), times.end());
    
    long long left = 1;
    long long right = (long long)times.back() * n; // 최악의 경우
    
    while(left <= right){
        long long mid = (left + right) / 2; // 주어진 시간
        long long count = 0;
        
        for(int t : times) count += mid / t; // 각 심사관이 처리한 수
        
        if(count >= n) {
            answer = mid;   // 일단 저장 (가능한 시간임)
            right = mid - 1; // 최소 시간을 찾기 위해 줄여봄
        } else {
            left = mid + 1; // 불가능하니 시간 늘림
        }
    }
    return answer;
}
/* 테스트 케이스
Input: n=6, times=[7, 10]
Logic: 
    - 28분: 28/7=4명, 28/10=2명 -> 합 6명 (성공) -> 더 줄여봄
    - 27분: 27/7=3명, 27/10=2명 -> 합 5명 (실패) -> 다시 늘림
Result: 28
*/
```

## 면접 준비

### 객체 복사를 막는 방법은 어떤 방법이 있을까요? 왜 객체 복사를 막아야 할까요?

## 1. 왜 객체 복사를 막아야 할까요?

C++에서 클래스를 만들면 컴파일러는 기본적으로 **복사 생성자**와 **복사 대입 연산자**를 자동으로 만들어줍니다.<br>
하지만 이 "자동 복사"가 치명적인 문제를 일으키는 경우가 있습니다.<br>

### ① 자원 관리 문제 (Double Free / Dangling Pointer)

객체가 메모리(포인터), 파일 핸들, 네트워크 소켓 등을 관리하고 있다고 가정해 봅시다.<br>
단순 복사(Shallow Copy, 얕은 복사)가 일어나면 **두 객체가 같은 자원을 가리키게 됩니다.**<br>

- **문제 상황:** 객체 A를 복사해 객체 B를 만듭니다. 둘 다 같은 메모리 주소를 가리킵니다.<br>
- **소멸 시점:** 객체 A가 소멸하며 메모리를 해제(`delete`)합니다. 이어 객체 B가 소멸할 때 **이미 해제된 메모리를 또 해제하려고 시도**합니다.<br>
- **결과:** 프로그램이 즉시 강제 종료됩니다. (Double Free Error)<br>

### ② 논리적 유일성 (Identity)

세상에 하나만 존재해야 하는 객체들이 있습니다.<br>

- **예:** `Mutex`(잠금 장치), `PrinterController`(프린터 제어권), `std::unique_ptr`, `Singleton` 등.<br>
- **이유:** "잠금 장치"를 복사해서 두 개로 만들면, 어떤 것이 진짜 잠금인지 알 수 없게 되어 동기화가 깨집니다. 이런 객체는 복사 자체가 논리적으로 모순입니다.<br>

### ③ 불필요한 성능 저하 방지

매우 큰 데이터를 담고 있는 객체(예: 수 기가바이트의 이미지 데이터)를 실수로 복사하게 되면,<br>
시스템 멈춤 현상(Freezing)이 발생할 수 있습니다. <br>

이를 원천적으로 차단하여 참조(`&`)나 이동(`move`)만 사용하도록 강제해야 합니다.<br>

## 2. 객체 복사를 막는 방법

C++ 표준의 변화에 따라 방법이 진화했습니다.<br>

- 11 이전 : 'private' 처리<br>
- 11 이후 : '=delete' 처리<br>
  + Unique_ptr 같은 복사를 막아둔(NotCopyable) 멤버를 클래스가 포함<br>
