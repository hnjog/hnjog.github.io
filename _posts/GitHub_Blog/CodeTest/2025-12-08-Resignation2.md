---
title: "백준 Gold 5 퇴사 2"
date : "2025-12-08 10:30:00 +0900"
last_modified_at: "2025-12-08T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
---

## 퇴사 2 (백준 Gold 5)
<https://www.acmicpc.net/problem/15486><br>

N+1 날에 퇴사를 하기 위해<br>
N일 동안 상담을 진행하며 가장 많은 비용을 얻으려 하는 문제<br>

- 상담은 진행 기간과 그 비용이 존재한다<br>
  - 첫날의 상담이 '3일', '10'에 비용을 얻을 수 있다면<br>
    해당 상담 진행시 2,3 번째 일자의 상담은 진행할 수 없다<br>

## 풀이 방법

**dp 정의**<br>
- dp[i] : i번째 날부터 '마지막 날'까지 얻을 수 있는 최대 금액<br>

**다음 상태 이동(점화식)**<br>

- dp[i] = max(dp[i+1],dp[i+t[i]] + p[i])<br>

- dp[i+1]<br>
 : 현재 상담을 선택하지 않음<br>
 - 그렇기에 다음 날 기준의 최대 비용을 가져옴<br>

- dp[i+t[i]] + p[i]<br>
 : 현재 상담을 선택했을 때<br>
   상담이 진행된 일자 기준의 비용과 현재 상담 비용을 더한 값을 구함<br>

- 다만 i+t[i]가 N을 넘어서는 경우는<br>
  dp[i+1]을 가져오도록 한다<br>

**순회 방식**<br>
- i를 내림차순으로 진행<br>

- 다음 날의 최대 비용과 비교하는 방식<br>
  (현재 상담을 진행하지 않는 것과 진행함으로서<br>
  포기할 수 있는 그 다음의 상담들을 고려)<br>

- dp[0]에 결과값이 저장<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n;
	cin >> n;
	vector<int> times, pays;
	for (int i = 0; i < n; i++)
	{
		int t, p;
		cin >> t >> p;

		times.push_back(t);
		pays.push_back(p);
	}

	vector<int> dp(n+1, 0);

	for (int i = n-1; i >= 0; i--)
	{
		if (i + times[i] > n)
		{
			dp[i] = dp[i + 1];
		}
		else
		{
			dp[i] = max(dp[i + 1], dp[i + times[i]] + pays[i]);
		}
	}

	cout << dp[0];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/66b4c611-4390-41b9-b274-0efd3c557d76)](https://github.com/user-attachments/assets/66b4c611-4390-41b9-b274-0efd3c557d76){: .image-popup}<br>

처음에는<br>
- dp[i] : i번째 날까지 진행했을 때 얻을 수 있는 금액<br>
  
으로 dp 정의를 하고 진행하다 보니<br>
'배낭 문제'식으로 문제를 풀려고 하였다<br>

그러다 보니 첫 예제만 통과하고 나머지 문제에서 조금씩 틀리다 보니<br>
문제의 정의가 헷갈리게 되었고<br>
핵심적으로는 '이전 상태'를 고려하지 않는 상황이 되었다<br>

- 1일에 3, 2일에 5 걸리는 상담이 있는데<br>
  8일동안 둘 다 진행할 수 있는것으로 코드가 진행<br>
  - 실제로는 1일에 상담 받을 시, 2,3일은 불가하므로<br>
    잘못된 로직 구현<br>

따라서 해당 dp정의가 '틀림'을 알 수 있었고<br>
새로운 dp를 찾아야 하였다<br>

- 코드 자체는 매우 간단하지만<br>
  dp 재정의와 로직 검증에 시간을 사용한 문제였다<br>
