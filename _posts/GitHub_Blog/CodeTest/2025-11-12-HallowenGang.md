---
title: "백준 Gold 2 할로윈의 양아치"
date : "2025-11-12 10:30:00 +0900"
last_modified_at: "2025-11-12T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 할로윈의 양아치 (백준 Gold 2)
<https://www.acmicpc.net/problem/20303><br>

스브러스가 사탕을 빼앗으려 할때<br>
K명을 울리지 않으면서 사탕을 빼앗을 수 있는 최대 수<br>

- 친구의 친구들까지 한번에 빼았는다<br>
  (강제)<br>

## 풀이 방법
'분리 집합'을 중간 과정으로 사용하여 DP를 적용하는 문제<br>

- 특정 친구들이 연결되어 있기에<br>
   하나의 노드로 묶어 관리<br>
   - 총 사탕의 양<br>
   - 총 친구 수<br>

- Union-Find를 통해 묶는 것을 완료하였다면<br>
  각각의 총 친구수와 사탕의 양을 각각<br>
  Weight와 Values로 해석 가능<br>

- 배낭 문제를 푸는 방식으로<br>
  K 미만 내로 최대한 가능한 값을 구함<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

struct fInfo
{
	int p;
	int val;
	int weight;
};

int FindParent(vector<fInfo>& pv, int x)
{
	if (pv[x].p == x)
		return x;

	return pv[x].p = FindParent(pv, pv[x].p);
}

bool Union(vector<fInfo>& pv, int a, int b)
{
	a = FindParent(pv, a);
	b = FindParent(pv, b);

	if (a == b)
		return false;

	pv[a].p = b;
	pv[b].val += pv[a].val;
	pv[b].weight += pv[a].weight;
	pv[a].val = 0;
	pv[a].weight = 0;

	return true;
}

int Knapsack(const vector<int>& weights, const vector<int>& values, int k)
{
	vector<int> dp(k + 1, 0);

	int wSize = weights.size();

	for (int i = 0; i < wSize; i++)
	{
		// 총 용량을 k부터 시작하고
		// 현재 무게의 물건을 담을 수 있는지를 체크
		for (int j = k; j >= weights[i]; j--)
		{
			dp[j] = max(dp[j], dp[j - weights[i]] + values[i]);
		}
	}

	return dp[k];
}

int main()
{
	int n, m, k;
	cin >> n >> m >> k;
	k--;

	vector<fInfo> pv(n);

	for (int i = 0; i < n; i++)
	{
		int c;
		cin >> c;
		pv[i].p = i;
		pv[i].val = c;
		pv[i].weight = 1;
	}

	for (int i = 0; i < m; i++)
	{
		int t1, t2;
		cin >> t1 >> t2;
		t1--;
		t2--;
		Union(pv, t1, t2);
	}

	vector<int> weights, values;
	for (fInfo& f : pv)
	{
		if (f.val != 0)
		{
			weights.push_back(f.weight);
			values.push_back(f.val);
		}
	}

	cout << Knapsack(weights, values, k);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/f0b0d3b4-25d3-4f41-b6ba-d2a432e7858e)](https://github.com/user-attachments/assets/f0b0d3b4-25d3-4f41-b6ba-d2a432e7858e){: .image-popup}<br>

푸는 방식을 미리 생각하여 다행히 문제를 순서대로 풀 수 있었다<br>