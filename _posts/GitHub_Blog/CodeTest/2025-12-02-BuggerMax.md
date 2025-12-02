---
title: "백준 Gold 1 햄최몇?"
date : "2025-12-02 10:30:00 +0900"
last_modified_at: "2025-12-02T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 햄최몇? (백준 Gold 1)
<https://www.acmicpc.net/problem/19645><br>

주어지는 N개의 햄버거에 각각의 가치 V가 존재할때<br>
3명이서 나누되, 가장 적게 받는 가치를<br>
가장 높이 받는 방식을 구하기<br>

## 풀이 방법

세 명의 분배 값을 각각 a, b, c,<br>
모든 햄버거의 가치의 합을 maxx라 했을 때<br>

`c = maxx - a - b` 로 정의할 수 있음<br>

따라서 a,b 를 구하면 c 값을 알 수 있음!<br> 

**dp 정의**<br>
- dp[i][j] : i,j 로 햄버거를 분배할 수 있는지의 여부 파악<br>

- i,j로 햄버거를 분배하면 maxx - i - j 가 우리가 원하는 값이 될 것<br>
  - 물론 a = i, b = j, c = maxx - i - j 이지만<br>
    이중 가장 작은 값이 우리가 구할 값이 됨<br>
	(min(a,b,c))<br>
	그리고 이러한 값들 중 가장 큰 값이 정답<br>

**다음 상태 이동(점화식)**<br>
- 현재 햄버거를 nowBugger 라 하면<br>
  dp[i - nowBugger][j] 가 true일때, dp[i][j] 역시 true<br>
  또한 dp[i][j - nowBugger]가 true이면 dp[i][j]가 true<br>

- 각각, 다른 자에게 '배분'하는 경우로 해석 가능함<br>

**순회 방식**<br>
- 내림차순 순회<br>
  - 햄버거가 1개밖에 없기에 1번씩만 접근 가능하도록<br>

dp를 완성한 후<br>
2중 반복문을 돌며<br>

dp[a][b] 가 true인 경우에 대해<br>
min(a,b,c)를 계산해주면 된다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n;
	cin >> n;
	vector<int> buggers(n);
	int maxx = 0;

	for (int i = 0; i < n; i++)
	{
		cin >> buggers[i];
		maxx += buggers[i];
	}

	// dp[i][j] : i 와 j로 분배가 가능하지 파악
	//
	// maxx - i - j = 길원이가 받을 햄버거 양
	//
	vector<vector<bool>> dp(maxx + 1, vector<bool>(maxx + 1, false));
	dp[0][0] = true;

	for (int i = 0; i < n; i++)
	{
		int nowBugger = buggers[i];
		for (int j = maxx; j >= 0; j--)
		{
			for (int k = maxx; k >= 0; k--)
			{
				if (j >= nowBugger &&
					dp[j-nowBugger][k])
				{
					dp[j][k] = true;
				}

				if (k >= nowBugger &&
					dp[j][k - nowBugger])
				{
					dp[j][k] = true;
				}
			}
		}
	}

	int ans = 0;

	for (int i = 0; i <= maxx; i++)
	{
		for (int j = 0; j <= maxx - i; j++)
		{
			if (dp[i][j] == false)
				continue;

			int c = maxx - i - j;
			int v = min(i, j);
			v = min(v, c);

			if (v > ans)
				ans = v;
		}
	}

	cout << ans;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/3c26df16-e4f5-46e5-a73b-6960929c6fb1)](https://github.com/user-attachments/assets/3c26df16-e4f5-46e5-a73b-6960929c6fb1){: .image-popup}<br>

dp 정의 부분이 특히 어려웠다<br>
솔직히 처음 dp 정의도 감을 못잡고 있다가<br>
힌트를 받아서 문제를 풀 수 있었다<br>

- a,b,c 세 대상이 존재할 때<br>
  c = maxx - a - b<br>
  라는 부분을 떠올릴 수 없었기에<br>
  dp 정의 부분에서 막혔다<br>
