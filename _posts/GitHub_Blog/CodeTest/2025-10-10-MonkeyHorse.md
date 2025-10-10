---
title: "백준 Gold 3 말이 되고픈 원숭이"
date : "2025-10-10 10:30:00 +0900"
last_modified_at: "2025-10-10T10:30:00"
categories:
  - 코딩 테스트
tags:
  - BFS
---

## 말이 되고픈 원숭이 (백준 Gold 3)
<https://www.acmicpc.net/problem/1600><br>

`체스의 말 기물`처럼 움직일 수 있는 횟수가 k번 주어지고<br>
상하좌우로 1칸씩 이동이 가능할때<br>
0,0 에서 w-1,h-1 로 이동할 수 있는 `최단 이동 횟수`를 구하는 문제<br>

## 풀이 방법

최단 거리이기에 BFS 탐색이 어울리지만<br>
k번 뛸 수 있을 때의 가능성 역시 고려해야 한다<br>

그렇기에 visit 를 3차원 배열로 만들어<br>
말 기물 점프를 사용한 방문 비용 배열을 통해<br>
visit 여부를 검사하였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>

using namespace std;

int k;
int w, h;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

const int hY[8] = { 2,2,1,1,-1,-1,-2,-2 };
const int hX[8] = { 1,-1,2,-2,2,-2,1,-1 };

struct infos
{
	int y, x;
	int cost;
	int useK;
};

inline bool checkRange(int y, int x)
{
	if (y < 0 || y >= h ||
		x < 0 || x >= w)
		return false;

	return true;
}

inline bool checkCan(const vector<vector<int>>& maps, vector<vector<vector<int>>>& visit, infos& n)
{
	if (checkRange(n.y, n.x) == false)
		return false;

	if (maps[n.y][n.x] == 1)
		return false;

	if (visit[n.useK][n.y][n.x] <= n.cost)
		return false;

	return true;
}

struct Compare
{
	bool operator()(const infos& a, const infos& b)
	{
		if (a.cost == b.cost)
			return a.useK > b.useK;

		return a.cost > b.cost;
	}
};

int bfs(vector<vector<int>>& maps)
{
	vector<vector<vector<int>>> visit(k + 1, vector<vector<int>>(h, vector<int>(w, 5000000)));

	priority_queue<infos, vector<infos>, Compare> q;
	q.push({ 0,0,0,0 });

	while (q.empty() == false)
	{
		infos i = q.top();
		q.pop();

		if (checkCan(maps, visit, i) == false)
			continue;

		visit[i.useK][i.y][i.x] = i.cost;

		if (i.y == h - 1 &&
			i.x == w - 1)
		{
			return i.cost;
		}

		for (int j = 0; j < 4; j++)
		{
			q.push({ i.y + dirY[j],i.x + dirX[j],i.cost + 1,i.useK });
		}

		if (i.useK < k)
		{
			for (int j = 0; j < 8; j++)
			{
				q.push({ i.y + hY[j],i.x + hX[j],i.cost + 1,i.useK + 1 });
			}
		}
	}

	return -1;
}

int main()
{
	cin >> k;
	cin >> w >> h;
	vector<vector<int>> maps(h, vector<int>(w, -1));
	for (int i = 0; i < h; i++)
	{
		for (int j = 0; j < w; j++)
		{
			cin >> maps[i][j];
		}
	}

	cout << bfs(maps);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/d3d80897-fd7e-48b0-bc1e-6f3b86f04041)](https://github.com/user-attachments/assets/d3d80897-fd7e-48b0-bc1e-6f3b86f04041){: .image-popup}<br>

사실 더 간단하게 풀 수 있을지도 모른다고 생각하였으나<br>
pq를 사용하는 것이 조금 더 좋아보였다<br>

- 아마 일반 BFS로도 가능하지 않을까?<br>

- 또한 next 시점에서 visit 및 검사 체크를 하는 것이<br>
  메모리 측면에서는 더 좋을지 모른다<br>
