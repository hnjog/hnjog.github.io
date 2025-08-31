---
title: "백준 Gold 4 녹색 옷 입은 애가 젤다지?"
date : "2025-08-31 09:00:00 +0900"
last_modified_at: "2025-08-31T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 그래프 탐색
  - BFS
  - 다익스트라
---

## 녹색 옷 입은 애가 젤다지? (백준 Gold 4)
<https://www.acmicpc.net/problem/4485><br>

주어지는 N 크기의 정사각형 맵에서<br>
N-1 까지 가는 최소 cost를 구하기<br>

- 이미 0,0 을 밟았고, 마지막에는 N-1,N-1 의 값을 밟아야 함<br>

다익스트라 알고리즘에 익숙하면<br>
그리 어렵지 않게 풀 수 있는 문제이다<br>

## 제출 코드
```
#include<iostream>
#include<vector>
#include<queue>
#include<limits.h>

using namespace std;

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0,1,0,-1 };

struct pos
{
	int y, x;
	int cost;
};

struct Compare
{
	bool operator()(const pos& a, const pos& b)
	{
		return a.cost > b.cost;
	}
};

int bfs(vector<vector<int>>& maps)
{
	int n = maps.size();

	priority_queue<pos, vector<pos>, Compare> pq;
	pq.push({ 0,0,0 });

	vector<vector<int>> visit(n, vector<int>(n, INT_MAX));

	while (pq.empty() == false)
	{
		int nowY = pq.top().y;
		int nowX = pq.top().x;
		int nowCost = pq.top().cost;
		pq.pop();

		if (nowY < 0 || nowY >= n ||
			nowX < 0 || nowX >= n)
			continue;

		// 현재 밟은 도둑 루피
		nowCost += maps[nowY][nowX];
		
		if (visit[nowY][nowX] <= nowCost)
			continue;
		
		visit[nowY][nowX] = nowCost;

		if (nowY == n - 1 && nowX == n - 1)
		{
			return nowCost;
		}

		for (int i = 0; i < 4; i++)
		{
			pq.push({ nowY + dirY[i],nowX + dirX[i],nowCost });
		}
	}

	return 0;
}

int main()
{
	int t = 0;
	while (true)
	{
		t++;
		int n;
		cin >> n;
		if (n == 0)
			break;

		vector<vector<int>> maps(n, vector<int>(n));
		for (int i = 0; i < n; i++)
			for (int j = 0; j < n; j++)
				cin >> maps[i][j];

		cout << "Problem " << t << ": " << bfs(maps) << '\n';
	}

	return 0;
}
```

## 결과
<img width="1148" height="120" alt="Image" src="https://github.com/user-attachments/assets/9dad9b2f-be02-4cd9-98ce-a984b90cd31d" /><br>

메모리 초과는 visit 값의 <= 부분을 실수로 < 로 처리한 코드이다<br>
아마 같은 값의 visit가 반복되며 루프가 지나치게 길어진 것으로 추정된다<br>