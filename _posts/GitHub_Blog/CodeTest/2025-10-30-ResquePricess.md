---
title: "백준 Gold 5 공주님을 구해라!"
date : "2025-10-30 10:30:00 +0900"
last_modified_at: "2025-10-30T10:30:00"
categories:
  - 코딩 테스트
tags:
  - BFS
---

## 공주님을 구해라! (백준 Gold 5)
<https://www.acmicpc.net/problem/17836><br>

0,0 에서 n-1,m-1 까지 가기 위한 최단 경로를 구하는 문제<br>

- 격자 그래프 형식으로 주어짐<br>
- n x m 데이터가 주어짐<br>
- 1은 벽이며 기본적으로 통과할 수 없음<br>
- 2는 전설의 검이며, 획득 시 모든 벽을 부수며 이동 가능<br>
- 제한 시간 t 내에 구할 수 없다면 "Fail" 출력<br>

## 풀이 방법

bfs를 2번 돌려 값을 비교하면 풀 수 있다<br>

- 검을 안먹고 최단 시간내로 주파<br>
- 검을 목적지로 한 후,<br>
  (공주님 위치 - 검의 위치)를 더해주어 값을 구하기<br>
  - 검을 획득한 순간 '벽'의 의미가 없어지므로<br>
    최단거리 주파가 가능해짐<br>
	(공주.y - 검.y , 공부.x - 검.x)<br>
	현재 시간에 이 거리를 더해주는 것으로 문제를 풀 수 있음<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<limits.h>

using namespace std;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

struct infos
{
	int y, x;
	int cost;
};

int n, m, t;

int bfs(vector<vector<int>>& maps, int targetY,int targetX, bool bFindSword)
{
	queue<infos> q;
	q.push({ 0,0,0 });

	vector<vector<int>> visit(n, vector<int>(m, INT_MAX));
	visit[0][0] = 0;

	while (q.empty() == false)
	{
		int nowY = q.front().y;
		int nowX = q.front().x;
		int nowCost = q.front().cost;
		q.pop();

		if (nowY == targetY &&
			nowX == targetX)
		{
			if (bFindSword)
			{
				int f = (n - 1 - targetY) + (m - 1 - targetX);
				return nowCost + f;
			}

			return nowCost;
		}

		for (int i = 0; i < 4; i++)
		{
			int ny = dirY[i] + nowY;
			int nx = dirX[i] + nowX;
			int ncost = nowCost + 1;

			if (ny < 0 || ny >= n ||
				nx < 0 || nx >= m)
				continue;

			if (maps[ny][nx] == 1)
				continue;

			if (ncost > t)
				continue;

			if (visit[ny][nx] <= ncost)
				continue;

			visit[ny][nx] = ncost;
			q.push({ ny,nx,ncost });
		}
	}

	return INT_MAX;
}

int main()
{
	cin >> n >> m >> t;
	vector<vector<int>> maps(n, vector<int>(m, 0));

	pair<int, int> sPos;

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			cin >> maps[i][j];
			if (maps[i][j] == 2)
				sPos = { i,j };
		}
	}

	int b = bfs(maps, n - 1, m - 1, false);
	int b2 = bfs(maps, sPos.first, sPos.second, true);

	if (b > b2)
		b = b2;

	if (b <= t)
		cout << b;
	else
		cout << "Fail";

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/75d801ca-488d-4ce3-b831-37a01eea4cc4)](https://github.com/user-attachments/assets/75d801ca-488d-4ce3-b831-37a01eea4cc4){: .image-popup}<br>
