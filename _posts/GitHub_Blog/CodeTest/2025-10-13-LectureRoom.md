---
title: "백준 Gold 5 강의실"
date : "2025-10-13 10:30:00 +0900"
last_modified_at: "2025-10-13T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그리디
  - 우선순위 큐
---

## 강의실 (백준 Gold 5)
<https://www.acmicpc.net/problem/1374><br>

N개의 강의가 주어질때<br>
최대한 적은 강의실을 빌려 모든 강의를 이루어지게 하는 문제<br>

## 풀이 방법

- 먼저 강의가 순서대로 주어지지 않으므로<br>
  시작 시간이 짧은 순서대로 정렬<br>

- '끝나는 시간'이 빠른 순으로 정렬되도록<br>
  우선순위 큐를 제작<br>

- 정렬된 데이터에서 하나씩 빼면서 우선순위 큐에 집어넣어 준다<br>
  다만 이때, 우선순위 큐의 Top에 존재하는 값이<br>
  '시작 시간' 이하의 값이라면 빼준다<br>

- 해당 과정마다 pq의 사이즈가 가장 컸는지를 확인하고<br>
  가장 큰 값을 answer로 기록<br>

## 제출 코드

```cpp
#include<iostream>
#include<queue>
#include<vector>
#include<algorithm>

using namespace std;

int main()
{
	int n;
	cin >> n;

	vector<pair<int,int>> iv;
	priority_queue<int,vector<int>,greater<int>> pq;

	for (int i = 0; i < n; i++)
	{
		int t,t1,t2;
		cin >> t >> t1>> t2;
		iv.push_back({t1,t2});
	}

	sort(iv.begin(), iv.end(), []
	(const pair<int, int>& a, const pair<int, int>& b)
		{
			if (a.first == b.first)
				return a.second < b.second;

			return a.first < b.first;
		});

	int ans = 0;

	for (int i = 0; i < iv.size(); i++)
	{
		if (pq.empty())
		{
			pq.push(iv[i].second);
		}
		else
		{
			while (pq.empty() == false &&
				pq.top() <= iv[i].first)
			{
				pq.pop();
			}

			pq.push(iv[i].second);
		}

		if (pq.size() > ans)
			ans = pq.size();
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/300a560c-b89d-4b5a-a0cb-e9683ba1fbe4)](https://github.com/user-attachments/assets/300a560c-b89d-4b5a-a0cb-e9683ba1fbe4){: .image-popup}<br>

pq는 `'해당 상황'에서 최적값을 찾는데 사용`해야 한다는 점을<br>
다시 깨닫는 문제였다<br>

- 새로운 강의가 시작할 때<br>
  '강의실'을 빌려야하는지 여부?<br>
  - 기존 강의실 중 '끝난 시간'이 있는지를 검사<br>
