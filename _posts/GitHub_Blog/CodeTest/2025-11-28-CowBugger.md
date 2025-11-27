---
title: "백준 Gold 4 카우버거 알바생"
date : "2025-11-28 10:30:00 +0900"
last_modified_at: "2025-11-28T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 카우버거 알바생 (백준 Gold 4)
<https://www.acmicpc.net/problem/17208><br>

n개의 주문과 남아있는 버거 m, 튀김 k 가 주어질때<br>
가능한 많은 주문을 처리하는 문제<br>

- 주문은 1개당 1번만 처리 가능<br>

## 풀이 방법

**dp 정의**<br>
- dp[i][j] : i 버거와 j 튀김까지 사용하여 처리할 수 있는 최대 주문 횟수<br>

- 조금 특이하게도<br>
  i번쨰를 처리하는 것이 아닌 2개의 자원을 동시에 관리하는 문제이다<br>

**다음 상태 이동(점화식)**<br>

- dp[i][j] = max(dp[i][j],dp[i- ch[k]][j - fr[k]] + 1);
- 주문을 별도의 for문 돌려야 하기에 3차원 반복<br>


**순회 방식**<br>
- 2차원 이지만, 목표값에 1번만 접근하기 위해<br>
  내림차순을 사용하여 0/1 조건을 만족<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	/*
		남아있는 치즈버거와 감자튀김을 이용해 모두 처리하는 문제

		정의
		dp[i][j] : i 치즈버거와 j 감자튀김을 소모했을때 처리할 수 있는 최대 주문 횟수

		점화식
		dp[i][j] = max(dp[i][j], dp[i-cho[k]][j-fry[k]] + 1)

		이후 순회하며 max 인 값 찾기

	*/

	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int n, m, k;
	cin >> n >> m >> k;

	vector<pair<int, int>> orders(n);

	for (int i = 0; i < n; i++)
	{
		cin >> orders[i].first >> orders[i].second;
	}

	vector<vector<int>> dp(m + 1, vector<int>(k + 1, 0));

	for (int i = 0; i < n; i++)
	{
		int nowCh = orders[i].first;
		int nowFr = orders[i].second;

		for (int j = m; j >= nowCh; j--)
		{
			for (int v = k; v >= nowFr; v--)
			{
				dp[j][v] = max(dp[j][v], dp[j - nowCh][v - nowFr] + 1);
			}
		}
	}

	int ans = 0;

	for (auto& v : dp)
	{
		for (int i : v)
		{
			if (i > ans)
				ans = i;
		}
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/c3806ce0-7499-4776-a35d-0cd269434cfa)](https://github.com/user-attachments/assets/c3806ce0-7499-4776-a35d-0cd269434cfa){: .image-popup}<br>

dp 정의를 직관적으로 잡긴 하였다<br>
2개의 조건이지만 개수를 같이 세야 하기에 내림차순 정렬 역시 고민해본 문제<br>

- 이미 정답처리이긴 하나<br>
  dp[m][k]로 정답을 구할 수 있다<br>
  - '이미 최댓값'처리가 되어 있기에<br>
	마지막에 for문을 또 돌린 필요는 없었다<br>