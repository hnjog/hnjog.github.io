---
title: "문승현 튜터님 강의 - '알고리즘 : 배열과 문자열'"
date : "2025-12-01 18:00:00 +0900"
last_modified_at: "2025-12-01T18:00:00"
categories:
  - C++
  - Algorithm
tags:
  - C++
  - Algorithm
  - 투포인터
  - 슬라이딩 윈도우
---

# 알고리즘 : 배열과 문자열

배열(Array/Vector)과 문자열(String)은 메모리에 연속적으로 저장되는 가장 기본적인 자료구조입니다.<br>
이 자료구조들의 핵심은 인덱스 접근(Random Access)이 O(1)로 빠르지만,<br>
중간 삽입/삭제는 O(N)으로 느리다는 점입니다.<br>
따라서 알고리즘 문제에서는 데이터를 복사하거나 이동시키는 것을 최소화하고,<br>
**인덱스(포인터)를 효율적으로 조작**하여 문제를 해결하는 패턴이 주를 이룹니다.<br>

## 패턴 1: 투 포인터 (Two Pointers)

### 개념

배열이나 문자열의 `양 끝(혹은 다른 위치)`에 `두 개의 포인터(인덱스 변수)`를 두고,<br>
특정 조건에 따라 서로 다가오거나 같은 방향으로 **이동하며 탐색**하는 기법입니다.<br>

### **장점**

- O(N^2)의 무식한 탐색(Brute Force)을 O(N)으로 줄일 수 있습니다.<br>
- 추가적인 메모리 공간을 거의 사용하지 않습니다.<br>

### 대표 예제 1: 문자열 뒤집기 (Reverse String)

가장 기초적인 투 포인터 활용입니다.<br>
`std::reverse`의 내부 구현 원리이기도 합니다.<br>

**문제**: 문자열이 주어졌을 때, 이를 뒤집으시오. (추가 메모리 사용 최소화)<br>

**해결 논리**:<br>

1. `left`는 0(시작점), `right`는 `size - 1`(끝점)에 둡니다.<br>
2. 두 포인터가 서로 교차하기 전까지 반복합니다.<br>
3. 두 문자의 값을 교환(Swap)하고, 포인터를 한 칸씩 안쪽으로 이동시킵니다.<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm> // swap

using namespace std;

void reverseString(vector<char>& s) {
    int left = 0;
    int right = s.size() - 1;

    // 두 포인터가 서로 교차하기 전까지 반복
    while (left < right) {
        swap(s[left], s[right]);
        left++;
        right--;
    }
}

// --- 테스트 코드 ---
int main() {
    vector<char> s = {'h','e','l','l','o'};
    reverseString(s);
    cout << "Result: ";
    for(char c : s) cout << c; 
    cout << endl; // Expected: olleh
    return 0;
}
```

### 대표 예제 2: **세 수의 합 (3Sum)**

투 포인터를 활용하여 특정 합을 만드는 조합을<br>
찾는 고전적인 문제입니다.<br>

**문제**: 정수 배열 `nums`가 주어졌을 때, `nums[i] + nums[j] + nums[k] == 0`을 만족하는 **중복 없는** 세 수의 조합을 모두 찾으시오.<br>

**해결 논리**:

1. **정렬**: 투 포인터 로직을 적용하기 위해 배열을 오름차순으로 `정렬`합니다. (O(Nlog N))<br>
2. **기준점 고정**: 반복문으로 `i`를 고정합니다. 이때 중복된 값은 건너뜁니다.<br>
3. **투 포인터 탐색**: 나머지 구간에서 `left`와 `right`를 이용해 합이 `nums[i]`가 되는 두 수를 찾습니다.<br>
4. **중복 처리**: 정답을 찾은 후에도, `left`와 `right`가 가리키는 값이 이전과 같다면 건너뛰어야 중복 조합을 방지할 수 있습니다.<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

vector<vector<int>> threeSum(vector<int>& nums) 
{
    sort(nums.begin(), nums.end()); // 1. 정렬 필수
    vector<vector<int>> result;
    int n = nums.size();

    for (int i = 0; i < n - 2; i++) 
    {
        // 기준점 i의 중복 값 건너뛰기
        if (i > 0 && nums[i] == nums[i - 1])
            continue;

        int left = i + 1;
        int right = n - 1;

        while (left < right) 
        {
            int sum = nums[i] + nums[left] + nums[right];

            if (sum == 0) 
            {
                result.push_back({nums[i], nums[left], nums[right]});
                
                // 정답을 찾은 후, 중복된 값들은 건너뛰기(동일한 값이 연속으로 있을 경우는 동일한 조합이 여러 번 만들어짐)
                while (left < right &&
                     nums[left] == nums[left + 1]) 
                    left++;

                while (left < right &&
                     nums[right] == nums[right - 1]) 
                    right--;
                
                left++;
                right--;
            } 
            else if (sum < 0) 
            {
                left++; // 합이 작으므로 더 큰 값을 더하기 위해 left 이동
            } 
            else 
            {
                right--; // 합이 크므로 더 작은 값을 더하기 위해 right 이동
            }
        }
    }
    return result;
}

// --- 테스트 코드 ---
int main() {
    vector<int> nums = {-1, 0, 1, 2, -1, -4};
    vector<vector<int>> res = threeSum(nums);
    cout << "Found: ";
    for (auto& v : res) cout << "[" << v[0] << "," << v[1] << "," << v[2] << "] ";
    cout << endl; // Expected: [-1,-1,2] [-1,0,1]
    return 0;
}
```

### 대표 예제 3: 빗물 가두기 (Trapping Rain Water)

투 포인터를 양 끝에서 좁혀오며 최대 높이를 갱신하는 심화 문제입니다.<br>

**문제**: 높이가 다른 벽들이 있을 때, 비가 온 후 벽 사이에 가둘 수 있는 물의 총량을 구하시오.<br>

**해결 논리**:<br>

1. 특정 위치 `i`에 물이 고이려면, 그 위치를 기준으로 왼쪽에서 가장 높은 벽(`leftMax`)과 오른쪽에서 가장 높은 벽(`rightMax`)이 필요합니다. 물은 그 두 벽 중 **더 낮은 벽의 높이**만큼 찰 수 있습니다.<br>
2. `left`와 `right` 포인터를 양 끝에 두고 시작합니다.<br>
3. 두 포인터 중 **높이가 더 낮은 쪽을 안쪽으로 이동**시킵니다. 왜냐하면 물의 높이는 낮은 벽에 의해 제한되기 때문입니다.<br>
4. 이동하면서 만나는 벽이 현재까지의 최대 높이보다 낮으면 -> **물 채우기 (`Max - 현재높이`)**<br>
5. 현재까지의 최대 높이보다 높으면 -> **최대 높이 갱신 (`Max = 현재높이`)**<br>


- 진행 방식<br>
  - 오른쪽, 왼쪽 벽 중 더 높은쪽을 내버려 두고,<br>
    낮은 쪽에서 진행함<br>
  - 진행 중, 현재 높이보다 더 높으면 최대 높이 갱신<br>
  - 낮다면, 그 차이만큼 물이 담긴다는 뜻이므로 물을 담아준다<br>
  - 최고 벽이 이미 있으면서 동시에<br>
    '낮은 쪽'에서 진행하기에<br>
    '반대 쪽'에 이미 현재 높이보다 '큰' 벽이 있다는 것이 전제하임<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int trap(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int leftMax = 0, rightMax = 0;
    int water = 0;

    while (left < right) {
        // 왼쪽 벽이 더 낮거나 같으면 왼쪽을 처리 (물 높이는 왼쪽 벽에 의해 결정됨)
        if (height[left] <= height[right]) 
        {
            if (height[left] >= leftMax) 
            {
                leftMax = height[left]; // 최대 높이 갱신 (물 못 담음)
            } 
            else
            {
                water += leftMax - height[left]; // 물 담기
            }
            left++;
        } 
        // 오른쪽 벽이 더 낮으면 오른쪽을 처리
        else 
        {
            if (height[right] >= rightMax) 
            {
                rightMax = height[right];
            } 
            else 
            {
                water += rightMax - height[right];
            }
            right--;
        }
    }
    return water;
}

// --- 테스트 코드 ---
int main() {
    vector<int> heights = {0,1,0,2,1,0,1,3,2,1,2,1};
    cout << "Trapped Water: " << trap(heights) << endl; // Expected: 6
    return 0;
}
```

## **패턴 2: 슬라이딩 윈도우 (Sliding Window)**

투 포인터가 두 개의 점(Point)을 움직인다면,<br>
슬라이딩 윈도우는 범위(Range)를 다룹니다. <br>

마치 창문을 옆으로 미는 것 같다고 해서 슬라이딩 윈도우라고 부르지만,<br>
문제 풀이에서는 "고무줄"로 이해하면 훨씬 쉽습니다.<br>

우리는 조건을 만족하는 '최적의 구간'을 찾아야 합니다.<br>

**1. 늘리기 (Expand - Right 이동):**<br>
    - 고무줄의 오른쪽 끝(`right`)을 잡아당겨 범위를 넓힙니다.<br>
    - 새로운 데이터를 범위 안으로 포함시킵니다. (예: 합계 증가, 문자 추가)<br>
**2. 조건 확인 및 줄이기 (Check & Contract - Left 이동):**<br>
    - 범위 안의 데이터가 **조건을 만족**했나요? (예: 합이 target 이상이 됨)<br>
    - 그렇다면, 이제는 고무줄의 왼쪽 끝(`left`)을 당겨서 범위를 줄여봅니다.<br>
    - **왜?** 더 짧은 길이로도 조건을 만족하는지 확인해서 **최솟값(정답)을 갱신**하기 위해서입니다.<br>

### 대표 예제 1: 최소 크기 부분 배열 합

**문제**: 합이 `target` 이상인 가장 짧은 부분 배열의 길이를 구하시오.<br>

**해결 논리**:<br>

1. `right` 포인터를 오른쪽으로 한 칸씩 이동하며 윈도우를 확장합니다.<br>
2. 새로 들어온 값(`nums[right]`)을 `sum`에 더해 현재 윈도우의 합을 키웁니다.<br>
3. 현재 합(`sum`)이 `target` 이상이 되면, "조건을 만족하는 유효한 윈도우"가 된 것입니다.<br>
4. 이제 **가장 짧은 길이**를 찾기 위해 `while` 문으로 윈도우를 왼쪽에서부터 줄여나갑니다.<br>
5. 현재 길이(`right - left + 1`)와 `minLength`를 비교해 최솟값을 갱신합니다.<br>
6. 맨 왼쪽 값(`nums[left]`)을 `sum`에서 빼고 `left`를 증가시킵니다.<br>
7. 이 과정을 합이 다시 `target`보다 작아질 때까지 반복합니다.<br>

- 오른쪽으로 한칸씩 더해감<br>
- 그 합이 Target이상이라면 현 시점에서 '가장 짧은 길이' 구하기<br>
    - 이후 left에서 하나씩 빼주면서 Target 미만의 값으로 만들기<br>
- 끝까지 반복하기<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits> // INT_MAX

using namespace std;

int minSubArrayLen(int target, vector<int>& nums) {
    int n = nums.size();
    int minLength = INT_MAX;
    int left = 0, sum = 0;

    for (int right = 0; right < n; right++) {
        // 1. 윈도우 확장: 오른쪽 값을 더함
        sum += nums[right];

        // 2. 윈도우 축소: 조건(합 >= target)을 만족하는 동안 왼쪽을 줄임
        while (sum >= target) {
            minLength = min(minLength, right - left + 1); // 길이 갱신
            sum -= nums[left]; // 왼쪽 값 빼기
            left++;            // 왼쪽 포인터 이동
        }
    }
    return (minLength == INT_MAX) ? 0 : minLength;
}

// --- 테스트 코드 ---
int main() {
    vector<int> nums = {2, 3, 1, 2, 4, 3};
    int target = 7;
    
    int result = minSubArrayLen(target, nums);
    
    cout << "Input: target = " << target << ", nums = [2,3,1,2,4,3]" << endl;
    cout << "Output: " << result << endl; // Expected: 2 (부분 배열 [4,3])
    
    return 0;
}
```

### 대표 예제 2: 중복 문자가 없는 가장 긴 부분 문자열

**문제**: 문자열에서 문자가 중복되지 않는 가장 긴 부분 문자열의 길이를 구하시오.<br>

**해결 논리**:<br>

1. 현재 윈도우 안에 어떤 문자가 있는지 빠르게 확인하기 위해 `unordered_set` (또는 배열)을 사용합니다.<br>
2. `right`가 가리키는 문자가 현재 윈도우(Set)에 **없으면** 안전하게 추가하고 `right` 확장합니다. (최대 길이 갱신)<br>
3. 만약 현재 문자 `s[right]`가 이미 있다면(중복 발생), 중복이 사라질 때까지 `left`를 이동하여 제거합니다.<br>
4. 중복이 해소되면 다시 `right`를 추가합니다.<br>

- 기본적으론 아까와 비슷함<br>
  - 0번째부터 오른쪽 진행<br>
  - 조건 만족 시, 왼쪽부터 제거 실행<br>
    (set 크기를 통해 갱신 시도)<br>

- 인덱스를 통해<br>
  left,right 검사<br>

```cpp
#include <iostream>
#include <string>
#include <unordered_set>
#include <algorithm>

using namespace std;

int lengthOfLongestSubstring(string s) {
    int maxLength = 0;
    int left = 0;
    unordered_set<char> window;

    for (int right = 0; right < s.length(); right++) {
        // 현재 문자(s[right])가 이미 윈도우에 있다면, 그 문자가 없어질 때까지 left 이동
        while (window.count(s[right])) {
            window.erase(s[left]);
            left++;
        }
        
        // 중복이 해결되었으므로 현재 문자 추가
        window.insert(s[right]);
        // 길이 갱신
        maxLength = max(maxLength, right - left + 1);
    }
    return maxLength;
}

// --- 테스트 코드 ---
int main() {
    string s1 = "abcabcbb";
    string s2 = "bbbbb";
    
    cout << "Input: " << s1 << " -> Output: " << lengthOfLongestSubstring(s1) << endl; 
    // Expected: 3 ("abc")
    
    cout << "Input: " << s2 << " -> Output: " << lengthOfLongestSubstring(s2) << endl; 
    // Expected: 1 ("b")
    
    return 0;
}
```

### 대표 예제 3 : 최소 부분 문자열 (Minimum Substring)

**문제:** 문자열 `s`와 `t`가 주어집니다. `s`의 부분 문자열 중에서 `t`의 모든 문자를 포함하는<br>
가장 짧은 구간을 구하시오.(예: `s = "ADOBECODEBANC"`, `t = "ABC"` → 정답 `"BANC"`)<br>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <climits>

using namespace std;

string minWindow(string s, string t) {
    // [1단계] 준비: 쇼핑 리스트 작성
    vector<int> map(128, 0);
    for (char c : t) map[c]++;

    int left = 0, right = 0;
    int minLen = INT_MAX; 
    int startIdx = 0;
    int counter = t.size(); // 필수 문자 총 개수

    // [2단계] 탐색: 카트 채우기 (Right 확장)
    while (right < s.size()) 
    {
        char rChar = s[right];
        
        // 필요한 문자였다면 미션 카운트 감소
        if (map[rChar] > 0) counter--;
        
        // 재고 차감 (필요 없던 건 음수로 내려감)
        map[rChar]--;
        right++;

        // [3단계 & 4단계] 검증 및 최적화: 
        // 필수 문자를 다 모았을 때(counter == 0)만 실행
        while (counter == 0) 
        {
            // 현재 길이가 최소라면 기록
            if (right - left < minLen) 
            {
                startIdx = left;
                minLen = right - left;
            }

            // 왼쪽 요소 빼기 (다이어트)
            char lChar = s[left];
            map[lChar]++; // 뺐으니 map 수치 복구
            
            // 방금 뺀 게 '여유분(음수)'이 아니라 '필수템(양수)'이 되었다면?
            if (map[lChar] > 0) 
            {
                counter++; // 다시 부족 상태로 전환 -> while 루프 종료
            }
            left++;
        }
    }

    return (minLen == INT_MAX) ? "" : s.substr(startIdx, minLen);
}

// --- 테스트 코드 ---
int main() {
    string s = "ADOBECODEBANC";
    string t = "ABC";
    
    // Expected: "BANC"
    // BANC는 ABC를 모두 포함하며 길이가 4로 가장 짧음
    cout << "Result: " << minWindow(s, t) << endl; 
    return 0;
}
```

**1. 준비 단계: "쇼핑 리스트 작성" (Map 초기화)**<br>

먼저 무엇이 필요한지 정확히 파악해야 합니다.<br>

- `vector<int> map(128, 0)`을 만듭니다. (아스키코드 전체를 커버하는 크기)<br>
- 타겟 문자열 `t`를 순회하며 `map`에 개수를 적립합니다.<br>
    - 예: `t = "ABC"`라면 `map['A']=1`, `map['B']=1`, `map['C']=1`이 됩니다.<br>
- **변수 `counter`:** 총 찾아야 하는 '필수 문자의 개수'를 저장합니다. (여기서는 3개)<br>

**2. 탐색 단계: "일단 카트에 담기" (Window 확장 - Right)**<br>

`right` 포인터를 오른쪽으로 한 칸씩 옮기며 `s`의 문자를 윈도우(카트)에 넣습니다. 이때 두 가지 판단을 합니다.<br>

- **판단 1: 이게 필요한 물건인가?**<br>
    - 만약 `map[s[right]] > 0`이라면, 쇼핑 리스트에 남아있던 물건입니다.<br>
    - **액션:** `counter`를 1 줄입니다. (미션 달성에 가까워짐!)<br>
- **판단 2: 재고 관리**<br>
    - 무조건 `map[s[right]]` 값을 1 줄입니다.<br>
    - **핵심:** 원래 1이었던 게 0이 되면 "개수 충족", 0이었던 게 -1이 되면 "과잉(여유분)" 상태가 됩니다.<br>

**3. 검증 단계: "미션 달성 확인" (Valid Window)**<br>

`counter == 0`이 되는 순간, 현재 윈도우(`left ~ right`) 안에는 `t`의 모든 문자가 들어있습니다.<br>

- 이때가 바로 "정답 후보"입니다.<br>
- 현재 길이가 지금까지 찾은 `minLen`보다 작다면, 정답을 갱신(기록)합니다.<br>

**4. 최적화 단계: "다이어트" (Window 축소 - Left)**<br>

이제 `counter == 0`인 상태(조건 만족 상태)를 유지하면서, 윈도우의 왼쪽(`left`)을 줄여봅니다. 불필요한 군더더기를 쳐내는 과정입니다.<br>

- `left` 위치의 문자를 윈도우 밖으로 뺍니다.<br>
- 뺀 문자의 `map` 값을 다시 1 더해줍니다. (원상복구)<br>
- **핵심 판단:**<br>
    - 만약 뺀 문자의 `map` 값이 여전히 0 이하(음수 또는 0)라면?<br>
        - "아, 걔는 있어도 그만 없어도 그만인(혹은 여분이었던) 애였어."<br>
        - **결과:** `counter`는 그대로 0입니다. `while` 문이 계속 돌며 `left`를 더 줄일 수 있습니다.<br>
    - 만약 뺀 문자의 `map` 값이 0보다 커진다(양수)면?<br>
        - "앗! 방금 뺀 건 **꼭 있어야 하는 필수템**이었는데!"<br>
        - **결과:** 이제 다시 부족해졌으므로 `counter`를 1 늘립니다. `while` 문이 깨지고, 다시 2. 탐색 단계(Right 확장)로 돌아가서 잃어버린 문자를 찾으러 갑니다.<br>