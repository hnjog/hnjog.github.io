---
title: "백준 Gold 5 수강 과목"
date : "2025-11-14 10:30:00 +0900"
last_modified_at: "2025-11-14T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 수강 과목 (백준 Gold 5)
<https://www.acmicpc.net/problem/17845><br>

수강 과목의 개수 K과 총 공부 시간 N이 주어진다<br>
과목마다 중요도 l과 공부시간 T가 주어질때<br>
N을 최대한 활용한 최대한의 중요도를 구하는 문제<br>

## 풀이 방법
이전에 풀었던 것과 매우 유사한 문제이다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int n, k;
	cin >> n >> k;

	vector<int> weights, values;

	for (int i = 0; i < k; i++)
	{
		int l, t;
		cin >> l >> t;
		values.push_back(l);
		weights.push_back(t);
	}

	vector<int> dp(n + 1);

	for (int i = 0; i < k; i++)
	{
		for (int j = n; j >= weights[i]; j--)
		{
			dp[j] = max(dp[j], dp[j - weights[i]] + values[i]);
		}
	}

	cout << dp[n];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/7d211c7c-be79-434a-b45a-029e396277d0)](https://github.com/user-attachments/assets/7d211c7c-be79-434a-b45a-029e396277d0){: .image-popup}<br>

최근에는 배낭 문제에 대한 코드 스니펫을 기반으로<br>
개념을 먼저 잡아보려고 한다<br>