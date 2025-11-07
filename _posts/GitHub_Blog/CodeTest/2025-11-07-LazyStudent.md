---
title: "백준 Gold 5 귀찮은 해강이"
date : "2025-11-07 10:30:00 +0900"
last_modified_at: "2025-11-07T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
---

## 귀찮은 해강이 (백준 Gold 5)
<https://www.acmicpc.net/problem/24391><br>

강의실 N개와 강의실이 같은 건물에 존재하는지에 대한 정보가 M개 주어질때<br>
특정 시간표대로 이동하였을때, 건물 밖으로 나오는 최소 횟수를 구하는 문제<br>

- 처음 건물에 들어가는 이동 횟수는 세지 않음<br>

## 풀이 방법

요점은 '같은 건물'이라면<br>
반드시 나가지 않는다는 점이므로<br>
'특정한 집합' 안에 존재하는지를 확인하는 문제<br>

즉, 분리 집합 문제이다<br>

- M개 주어지는 데이터를 통해 각각의 데이터를 Union 시키며<br>

- 이후 강의 시간표를 따라가며, 이전 강의실과 현재 강의실의 부모가 같은지를 확인하면 된다<br>
  (FindParent)<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

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

int main()
{
	int n, m;
	cin >> n >> m;
	vector<int> pv(n);

	for (int i = 0; i < n; i++)
		pv[i] = i;

	for (int i = 0; i < m; i++)
	{
		int s, t;
		cin >> s >> t;
		s--;
		t--;

		Union(pv,s, t);
	}

	int ans = 0;

	int now = 0;
	int prev = -1;

	for (int i = 0; i < n; i++)
	{
		int t;
		cin >> t;
		now = t - 1;

		if (prev == -1)
		{
			prev = now;
			continue;
		}

		if (FindParent(pv, now) != FindParent(pv, prev))
		{
			ans++;
			prev = now;
		}

	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/a9c1e9e9-46b2-4ac6-b53e-50032681d57d)](https://github.com/user-attachments/assets/a9c1e9e9-46b2-4ac6-b53e-50032681d57d){: .image-popup}<br>

최근에는 스파르타 팀 프로젝트 막바지이기에<br>
코드 스니펫과 개념을 복습하는 느낌으로 분리 집합 문제를 주로 풀고 있다<br>
