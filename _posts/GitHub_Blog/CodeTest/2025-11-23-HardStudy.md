---
title: "백준 Gold 5 벼락치기"
date : "2025-11-23 10:30:00 +0900"
last_modified_at: "2025-11-23T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 벼락치기 (백준 Gold 5)
<https://www.acmicpc.net/problem/29704><br>

주어지는 문제 N개와 T의 남은 시간을 통해<br>
최소 벌금 금액을 구하는 문제<br>

- 문제는 소요일수 d와 벌금 m으로 이루어져 N개 주어짐<br>

## 풀이 방법

**점화식**<br>
- dp[i] : dp[i] 일자에서 내야하는 최소 벌금<br>
  - 배낭 문제이지만, '이전 상태'만을 알면 되기에<br>
    1차원으로 풀 수 있다고 판단하였다<br>

**다음 상태 이동**<br>
- dp[j] = min(dp[j], dp[j - nowT] - nowV)<br>
  - 최소 dp이기에 초기값을 'MaxV'로 설정<br>
  - 또한 Min 과 - 연산<br>

**순회 방식**<br>
- 1차원이며, 최종 t값을 각 문제마다 1번만 갱신해야 하기에<br>
  t -> nowTime 으로 갱신<br>
  - 해당 문제를 풀 수 있는 날자들 중<br>
    가장 작은 값으로 갱신하는 방식<br>


## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n, t;
	cin >> n >> t;

	int maxV = 0;
	vector<int> vals, times;
	for (int i = 0; i < n; i++)
	{
		int d, m;
		cin >> d >> m;
		times.push_back(d);
		vals.push_back(m);
		maxV += m;
	}

	// dp[i] : i번째 일에서 가장 작은 수치의 값
	vector<int> dp(t + 1, maxV);
	
	for (int i = 0; i < n; i++)
	{
		int nowV = vals[i];
		int nowT = times[i];

		for (int j = t; j >= nowT; j--)
		{
			dp[j] = min(dp[j], dp[j - nowT] - nowV);
		}
	}

	cout << dp[t];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/7c034204-6c69-4e09-8dec-b814bf7aecb7)](https://github.com/user-attachments/assets/7c034204-6c69-4e09-8dec-b814bf7aecb7){: .image-popup}<br>

배낭 문제를 비교적 가볍게 푸는 방식은 조금 익숙해진 듯하다<br>
