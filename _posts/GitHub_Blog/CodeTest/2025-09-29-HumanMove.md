---
title: "백준 Gold 4 인구 이동"
date : "2025-09-29 10:30:00 +0900"
last_modified_at: "2025-09-29T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 시뮬레이션
  - 그래프 탐색
  - BFS
---

## 인구 이동 (백준 Gold 4)
<https://www.acmicpc.net/problem/16234><br>

NxN개의 나라 정보가 주어질때<br>
(각 나라는 1x1 크기)<br>
인구 이동이 며칠 동안 발생하는지를 구하는 문제<br>

조건들<br>
- 이웃나라들끼리 인구비교를 하였을때<br>
  두 인구수의 차이가 L 이상 R 이하인 경우<br>
  '국경'을 개방<br>

- 모든 나라들에 대하여 위에 대한 조건 판별 후<br>
  국경이 개방된 나라들의 인구수가 평균이 된다<br>
  (국경 개방 나라 인구수 / 국경 개방 나라 수)<br>

- 이후 국경을 폐쇄하고<br>
  하루가 지나간다<br>

## 풀이 방법
요점은 `'인구 이동의 총 발생 일자'`이며<br>
조건이 맞지 않으면 그냥 바로 0일로 끝날수도 있다는 점이다<br>

나는 BFS 탐색 방식을 사용하여 국경 개방 여부를 파악하였다<br>

- 해당 일자에 대한 Visit 배열을 사용하여<br>
  방문 여부 검사<br>
  (조건에 맞지 않으면, 애초에 그나라는 그날 국경 개방이 불가능함)<br>

- 두 인접 칸에서 인구수가 l이상 r 이하인지 검사<br>
- 동시에 국경을 개방한 나라들의 위치를 저장해둠<br>
- 저장한 위치들의 개수가 2개 이상이면 평균을 낸다<br>

- BFS가 성공하였다면, 하루가 흘러가고<br>
  아니라면 종료하고 반환<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>

using namespace std;

int n, l, r;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

bool bfs(vector<vector<int>>& maps, vector<vector<bool>>& visit, int y, int x)
{
	if (visit[y][x])
		return false;

	long sumValue = 0;
	vector<pair<int, int>> vecs;
	queue<pair<int, int>> q;

	q.push({ y,x });

	visit[y][x] = true;

	while (q.empty() == false)
	{
		int nowY = q.front().first;
		int nowX = q.front().second;
		q.pop();

		vecs.push_back({ nowY,nowX });
		sumValue += maps[nowY][nowX];

		for (int i = 0; i < 4; i++)
		{
			int ny = nowY + dirY[i];
			int nx = nowX + dirX[i];

			if (ny < 0 || ny >= n ||
				nx < 0 || nx >= n)
				continue;

			if (visit[ny][nx])
				continue;

			int mV = abs(maps[nowY][nowX] - maps[ny][nx]);

			if (mV < l || mV > r)
				continue;

			visit[ny][nx] = true;
			q.push({ ny,nx });
		}
	}

	if (vecs.size() <= 1)
		return false;

	int v = sumValue / vecs.size();
	for (auto& p : vecs)
	{
		maps[p.first][p.second] = v;
	}

	return true;
}

int main()
{
	cin >> n >> l >> r;
	vector<vector<int>> maps(n, vector<int>(n, 0));

	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			cin >> maps[i][j];
	int answer = 0;

	while (true)
	{
		bool dayNotChanged = true;

		vector<vector<bool>> visit(n, vector<bool>(n, false));

		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < n; j++)
			{
				if (visit[i][j] == false)
				{
					if (bfs(maps, visit, i, j))
					{
						dayNotChanged = false;
					}
				}
			}
		}
		if (dayNotChanged)
			break;

		answer++;
	}

	cout << answer;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/70a25b77-c9e9-4d12-819b-37f0604ac79e)](https://github.com/user-attachments/assets/70a25b77-c9e9-4d12-819b-37f0604ac79e){: .image-popup}<br>

일반적인 BFS 문제와는 살짝 달라 당황하였지만<br>
문제를 침착하게 보니 풀 수 있었다<br>
