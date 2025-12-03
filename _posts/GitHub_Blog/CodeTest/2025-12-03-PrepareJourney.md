---
title: "백준 Gold 3 백남이의 여행 준비"
date : "2025-12-03 10:30:00 +0900"
last_modified_at: "2025-12-03T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 백남이의 여행 준비 (백준 Gold 3)
<https://www.acmicpc.net/problem/23061><br>

물건 N개와 배낭 M개가 주어질 때<br>
가장 '효율적'인 배낭을 구하는 문제<br>

- 효율성은 (배낭에 넣을 수 있는 최대 가치) / (배낭이 담을 수 있는 무게)로 정해진다<br>

## 풀이 방법

dp를 기반으로 문제를 풀 수 있다!<br>

**dp 정의**<br>
- dp[i] : i 무게로 얻을 수 있는 최대 가치<br>

- 본래, 2차원 dp로 할까 싶었으나 1차원 dp로 풀 수 있어 수정하였다<br>

- 또한, 가방의 최대 무게가 1000000 이기에, 최댓값을 그에 맞게 제한<br>

**다음 상태 이동(점화식)**<br>
- 일반적인 배낭 문제 방식으로 접근 가능<br>
- dp[i] = max(dp[i], dp[i - w[j]] + v[j])<br>
  w를 담을 수 있다면, 그 가치인 v를 고려하여 계산<br>

**순회 방식**<br>
- 0/1 배낭 문제이며, 1차원 dp로 수정하였기에<br>
  내림차순으로 접근해야 한다<br>

이후,<br>
- dp[배낭무게] 를 통해 '배낭에 넣을 수 있는 최대 가치'를 구하고<br>
  다시 배낭무게로 나누면 문제를 풀 수 있다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	// 주어지는 가방 크기 와 가치들을 분석했을때 나오는 가치 효용성을 구하는 방식
	// 
	// - 가방에 넣은 총 가치 / 가방이 최대로 넣을 수 있는 크기 중 가장 큰 가방 선택
	// 
	// dp[i][j] : i번째 물품까지, j 무게로 얻을 수 있는 최대 가치
	// - 1차원 dp로 되는가를 고민해야 함
	//
	// dp[i] : i무게로 얻을 수 있는 최대 가치
	// (내림차순)
	// 
	// 이후, 각각의 가방에서 
	// dp[k1] / w1 이 방식으로 효용성이 제일 높은것을 선택하기
	//

	int n, m;
	cin >> n >> m;
	vector<int> weights,values;

	int maxWeight = 0;
	for (int i = 0; i < n; i++)
	{
		int w, v;
		cin >> w >> v;
		weights.push_back(w);
		values.push_back(v);

		maxWeight += w;
	}

	vector<int> packs(m);

	for (int i = 0; i < m; i++)
	{
		cin >> packs[i];
	}

	if (maxWeight > 1000000)
		maxWeight = 1000000;

	vector<int> dp(maxWeight + 1, 0);

	for (int i = 0; i < n; i++)
	{
		int nowW = weights[i];
		int nowV = values[i];

		for (int j = maxWeight; j >= nowW; j--)
		{
			dp[j] = max(dp[j], dp[j - nowW] + nowV);
		}
	}

	// idx, 총 효율
	pair<int, double> ans;
	ans.first = 0;
	ans.second = 0.0;

	for (int i = 0; i < m; i++)
	{
		int w = packs[i];
		if (w > maxWeight)
			w = maxWeight;

		double v = dp[w] / (double)packs[i];

		if (ans.second < v)
		{
			ans.first = i;
			ans.second = v;
		}
	}

	cout << ans.first + 1;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/35bddce2-0593-4597-b45c-00588050d33b)](https://github.com/user-attachments/assets/35bddce2-0593-4597-b45c-00588050d33b){: .image-popup}<br>

- 런타임 에러는 packs[i]가 maxWeight를 넘어설 수 있단 걸 나중에 알아<br>
  해당 부분을 수정하여 고칠 수 있었다<br>
