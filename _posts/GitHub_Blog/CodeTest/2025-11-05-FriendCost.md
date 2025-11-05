---
title: "백준 Gold 4 친구비"
date : "2025-11-05 10:30:00 +0900"
last_modified_at: "2025-11-05T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
---

## 친구비 (백준 Gold 4)
<https://www.acmicpc.net/problem/16562><br>

준석이가 다른 N명의 친구를 사귀는데 구하는 비용 N개가 주어지고<br>
이미 친구 사이인 M 개의 정보와 K의 돈이 주어진다<br>

준석이가 모든 사람과 친구가 되기 위한 최소 금액을 구하는 문제<br>

- `친구의 친구`가 친구로 판정됨<br>
- K 이상의 돈을 사용해야 하는 경우는 "Oh no" 출력<br>

## 풀이 방법

**분리 집합**을 이용하여 풀 수 있는 문제이다<br>

- 준석이는 '모든 이' 와 친구가 되고 싶어하며<br>
  그 각각의 비용 N이 주어지기에<br>
  준석이를 0번 노드로 생각하여 배치<br>

- m개가 주어진 경우, 해당하는 노드들을 분리 집합으로 미리 연결<br>

- M개의 친구 관계 외에는 별도의 친구 관계가 없기에<br>
  n[i]의 비용을 사용해야만 친구가 될 수 있는 경우가 존재<br>
  - 따라서 각각의 친구 비용을 통해<br>
    start : 0(준석이) , target : i 번째 친구, cost : n[i]<br>
	라는 식의 구조체로 만들 수 있음<br>

- 각각의 edge를 정렬하고<br>
  이미 Union된 경우가 아닌 비용을 더해나간다<br>

- 최종 비용이 K 보다 크다면 실패 처리<br>
  그 외에는 비용을 출력<br>

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

bool Union(vector<int>& pv, int a, int b)
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
	int n, m, k;
	cin >> n >> m >> k;

	vector<int> parent(n + 1,0);
	vector<edge> edges;

	for (int i = 0; i < n; i++)
	{
		edge e;
		cin >> e.cost;
		e.s = 0;
		e.t = i + 1;
		
		edges.push_back(e);
		parent[i + 1] = i + 1;
	}

	for (int i = 0; i < m; i++)
	{
		int t1, t2;
		cin >> t1 >> t2;
		Union(parent, t1, t2);
	}

	sort(edges.begin(), edges.end(), [](auto& a, auto& b) {return a.cost < b.cost; });

	int ans = 0;

	for (auto& e : edges)
	{
		if (Union(parent, e.s, e.t))
		{
			ans += e.cost;
		}
	}

	if (ans > k)
		cout << "Oh no";
	else
		cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/80d6cf50-7623-4b19-b695-fac9cde83ad6)](https://github.com/user-attachments/assets/80d6cf50-7623-4b19-b695-fac9cde83ad6){: .image-popup}<br>

처음에는 '친구'가 많은 '친구'를 사귀는 것이 '유리'하다고 생각하였지만<br>
자세히 생각해보니<br>
'모든 친구'를 사귀는 것이 문제의 요구 사항이었기에 해당 부분은 제외해도 된다고 생각하였다<br>

또한 비용을 기준으로 정렬하기에<br>
'이미 친구'인데 또 비용을 지불하는 경우의 수를 제외할 수 있다<br>
