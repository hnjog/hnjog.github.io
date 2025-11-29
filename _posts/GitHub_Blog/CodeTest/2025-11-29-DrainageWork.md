---
title: "백준 Gold 4 배수 공사"
date : "2025-11-29 10:30:00 +0900"
last_modified_at: "2025-11-29T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 배수 공사 (백준 Gold 4)
<https://www.acmicpc.net/problem/15817><br>

임의의 파이프 종류 N과 원하는 길이 x가 존재할때<br>
x를 만들 수 있는 경우의 수를 구하는 문제<br>

- 파이프는 길이 l과 수량 c가 주어진다<br>

## 풀이 방법

**dp 정의**<br>

- dp[i][j] : i번째 파이프까지 사용하여 j길이를 만들 수 있는 경우의 수<br>

**다음 상태 이동(점화식)**<br>

- dp[i][j] += dp[i-1][j - nowL * k]<br>
  - k 개수만큼 돌면서 가능한 여부를 현재 값에 넣어줌<br>

- dp[0][0] = 1<br>
  (길이 0을 만드는 경우는 아무것도 사용하지 않는 것)<br>

**순회 방식**<br>
- 내림차순 선택<br>

- 오름차순도 가능은 해보이나<br>
  이 때는 0 대신 -1 같은 초기화를 해야 할 것으로 보인다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	/*
		dp[i][j] : i번째 파이프까지 사용하여 j 길이를 만들 수 있는 경우의 수

		점화식?
		dp[i][j] = dp[i][j-len[i]] + 1 (단 dp[i][j]를 만들 수 있다면)

		이거 for문 사용해서
		counts[i] 만큼 돌려야 함

		내림차순
	*/

	int n, x;
	cin >> n >> x;

	vector<int> lens(n);
	vector<int> counts(n);

	for (int i = 0; i < n; i++)
	{
		cin >> lens[i];
		cin >> counts[i];
	}

	vector<vector<int>> dp(n + 1, vector<int>(x + 1, 0));
	dp[0][0] = 1;

	for (int i = 1; i <= n; i++)
	{
		int nowL = lens[i - 1];
		int nowC = counts[i - 1];

		for (int j = x; j >= 0; j--)
		{
			for (int k = 0; k <= nowC; k++)
			{
				if (j - nowL * k < 0)
					continue;

				dp[i][j] += dp[i-1][j - nowL * k];
			}
		}
	}

	cout << dp[n][x];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/62484381-aeed-4cd9-bdeb-221df30b5d48)](https://github.com/user-attachments/assets/62484381-aeed-4cd9-bdeb-221df30b5d48){: .image-popup}<br>

배낭 문제에 조금 익숙해진 기분이 든다<br>