---
title: "백준 Gold 3 벽 부수고 이동하기 2"
date : "2025-10-24 10:30:00 +0900"
last_modified_at: "2025-10-24T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - BFS
---

## 벽 부수고 이동하기 2 (백준 Gold 3)
<https://www.acmicpc.net/problem/14442><br>

N x M 의 맵 정보가 주어지고<br>
K번 벽을 부술수 있을 때<br>
0,0 에서 N-1,M-1 에 도달하는 최소 비용을 구하는 문제<br>

- 도달하지 못하는 경우 -1 출력<br>

## 풀이 방법

K번 벽을 부술 수 있기에<br>
그에 관련된 내용 역시 visit에 저장하면 풀 수 있는 문제이다<br>

- 다만 k가 10 이하로 주어졌기에 고려 할수 있는 방식이기도 하다<br>
  n 과 m이 1000개 이하이기에<br>
  1000 * 1000 * 10 * 4(int 비트)= 4천만 bit 이므로<br>
  약 5mb 정도를 소모<br>
  (아마 조금 더 넉넉히 주어져도 메모리 용량이 512 mb 제한이기에<br>
  괜찮았을지도?)<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<queue>
#include<limits.h>

using namespace std;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

int n, m, k;

struct infos
{
	int y, x;
	int cost;
	int remainK;
};

int bfs(vector<string>& maps)
{
	vector<vector<vector<int>>> visited(n, vector<vector<int>>(m,vector<int>(k + 1,INT_MAX)));

	queue<infos> q;
	q.push({ 0,0,1,k });
	visited[0][0][k] = 1;

	while (q.empty() == false)
	{
		int nowy = q.front().y;
		int nowx = q.front().x;
		int nowcost = q.front().cost;
		int nowRk = q.front().remainK;
		q.pop();

		if (nowy == n - 1 &&
			nowx == m - 1)
		{
			return nowcost;
		}

		for (int i = 0; i < 4; i++)
		{
			int ny = nowy + dirY[i];
			int nx = nowx + dirX[i];
			int ncost = nowcost + 1;
			int nextRk = nowRk;
			
			if (ny < 0 || ny >= n ||
				nx < 0 || nx >= m)
				continue;

			if (maps[ny][nx] == '1')
			{
				if (nextRk == 0)
					continue;

				nextRk--;
			}

			if (visited[ny][nx][nextRk] <= ncost)
				continue;

			visited[ny][nx][nextRk] = ncost;
			q.push({ ny,nx,ncost,nextRk });
		}

	}

	return -1;
}

int main()
{
	cin >> n >> m >> k;

	vector<string> maps(n);
	for (int i = 0; i < n; i++)
		cin >> maps[i];

	cout << bfs(maps);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/59469a57-e3b3-4cf6-8cf3-ff80c283caf3)](https://github.com/user-attachments/assets/59469a57-e3b3-4cf6-8cf3-ff80c283caf3){: .image-popup}<br>
