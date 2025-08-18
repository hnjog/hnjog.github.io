---
title: "백준 Silver 1 쉬운 최단거리"
last_modified_at: "2025-08-18T10:00:00"
categories:
  - 코딩 테스트
tags:
  - BFS
---

## 쉬운 최단거리 (백준 Silver 1)
<https://www.acmicpc.net/problem/14940><br>

일반적인 BFS 문제였다<br>
2라는 시작점을 찾은 후<br>
그에 맞추어 각각의 상하좌우를 탐색하며<br>
최단거리를 갱신하였다<br>

- 다만 '0'인 경우 '벽'이며 동시에<br>
  결과 역시 0으로 출력해야 하기에<br>
  출력 처리에 신경써야 한다<br>

- 또한, '도달하지 못한 곳'은<br>
  maps가 1인 동시에 '방문처리'되지 않은 곳이므로<br>
  visit 배열을 추가하여 처리해주었다<br>

## 결과
<img width="1145" height="192" alt="Image" src="https://github.com/user-attachments/assets/a7341161-94f0-4fd7-b346-1b7fc52fae2e" /><br>

앞서 주의한 부분으로 인하여<br>
조금 틀렸으나 코드와 문제를 다시 보고 해결하였다<br>

개인적으로는 예제가 조금 더 있었으면 어떨까 싶었지만<br>
반례를 찾는 것도 공부법 중 하나라 한다<br>


### 제출 코드

```
#include<iostream>
#include<vector>
#include<queue>
#include<limits.h>

using namespace std;

int startY, startX;

struct infos
{
	int y, x;
	int cost;
};

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0,1,0,-1 };

void func(const vector<vector<int>>& maps)
{
	int n = maps.size();
	int m = maps[0].size();
	vector<vector<int>> result(n,vector<int>(m,INT_MAX));
	vector<vector<bool>> visited(n,vector<bool>(m,false));

	queue<infos> q;
	q.push({ startY,startX,0 });

	while (q.empty() == false)
	{
		int nowY = q.front().y;
		int nowX = q.front().x;
		int nowCost = q.front().cost;
		q.pop();

		if (nowY < 0 || nowY >= n ||
			nowX < 0 || nowX >= m)
			continue;

		if (maps[nowY][nowX] == 0)
		{
			continue;
		}

		if (result[nowY][nowX] <= nowCost)
			continue;

		result[nowY][nowX] = nowCost;
		visited[nowY][nowX] = true;

		for (int i = 0; i < 4; i++)
		{
			int ny = nowY + dirY[i];
			int nx = nowX + dirX[i];

			q.push({ ny,nx,nowCost + 1 });
		}
	}

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			if (maps[i][j] == 1 && visited[i][j] == false)
				cout << -1 << ' ';
			else if (result[i][j] == INT_MAX)
				cout << 0 << ' ';
			else
				cout << result[i][j] << ' ';
		}

		cout << '\n';
	}

}

int main()
{
	int n, m;
	cin >> n >> m;

	vector<vector<int>> vec(n, vector<int>(m, 0));

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			cin >> vec[i][j];
			if (vec[i][j] == 2)
			{
				startY = i;
				startX = j;
			}
		}
	}

	func(vec);
}
```
