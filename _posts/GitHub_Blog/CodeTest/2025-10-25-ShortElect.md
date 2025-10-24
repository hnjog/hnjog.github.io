---
title: "백준 Gold 4 전력난"
date : "2025-10-25 10:30:00 +0900"
last_modified_at: "2025-10-25T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - MST
  - 크루스칼
---

## 전력난 (백준 Gold 4)
<https://www.acmicpc.net/problem/6497><br>

모든 집을 왕복이 가능한 정도로만 '가로수'를 비추어<br>
기존 비용에서 최대한 줄일 수 있는 비용을 구하는 문제<br>

## 풀이 방법
`모든 노드`를 연결하되<br>
그 비용이 **최소**가 되게 하는 `최소 스패닝 트리`(MST) 문제이다<br>

- MST를 통해 연결한 경우의 비용과<br>
  원래 비용의 차를 구하면 풀 수 있는 문제<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

using namespace std;

int FindParent(vector<int>& pv, int x)
{
	if (pv[x] == x)
		return x;

	return pv[x] = FindParent(pv, pv[x]);
}

bool Union(vector<int>& pv, int a,int b)
{
	a = FindParent(pv, a);
	b = FindParent(pv, b);

	if (a == b)
		return false;

	pv[a] = b;

	return true;
}

struct edge
{
	int s, t, cost;
};

int main()
{
	while (true)
	{
		int m, n;
		cin >> m >> n;

		if (m == 0 && n == 0)
			break;

		vector<int> pv(m);
		for (int i = 0; i < m; i++)
			pv[i] = i;

		vector<edge> edges(n);
		int full = 0;
		for (int i = 0; i < n; i++)
		{
			cin >> edges[i].s >> edges[i].t >> edges[i].cost;
			full += edges[i].cost;
		}

		sort(edges.begin(), edges.end(), []
		(const auto& a, const auto& b)
			{
				return a.cost < b.cost;
			});

		int ans = 0;
		for (edge& e : edges)
		{
			if (Union(pv, e.s, e.t))
			{
				ans += e.cost;
			}
		}

		cout << full - ans << '\n';
	}
	

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/ae0a633d-549a-4f65-b465-cff2bb735bd0)](https://github.com/user-attachments/assets/ae0a633d-549a-4f65-b465-cff2bb735bd0){: .image-popup}<br>

크루스칼의 코드 스니펫을 완전히 익히니<br>
여러모로 MST 문제를 풀때 도움이 되는 것 같다<br>
