---
title: "백준 Gold 5 보물섬"
date : "2025-10-07 10:30:00 +0900"
last_modified_at: "2025-10-07T10:30:00"
categories:
  - 코딩 테스트
tags:
  - BFS
---

## 보물섬 (백준 Gold 5)
<https://www.acmicpc.net/problem/2589><br>

주어진 맵 데이터에서<br>
특정 지점에서 다른 지점까지 이동하는데 걸리는<br>
가장 긴 '`최단 거리`'를 구하는 문제<br>

## 풀이 방법
모든 칸에 BFS를 돌리는 방식으로 충분히 구할 수 있는 문제이다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<queue>

using namespace std;

int h, w;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

inline bool checkRightArrange(int nowY, int nowX)
{
	if (nowY < 0 || nowY >= h ||
		nowX < 0 || nowX >= w)
		return false;

	return true;
}

struct edge
{
	int y, x, cost;
};

int bfs(vector<string>& maps, int nowY, int nowX)
{
	if (checkRightArrange(nowY, nowX) == false)
		return 0;

	int ret = 0;

	if (maps[nowY][nowX] == 'W')
		return ret;

	vector<vector<bool>> check(h, vector<bool>(w, false));
	queue<edge> q;
	q.push({ nowY,nowX,0 });
	check[nowY][nowX] = true;

	vector<pair<int, int>> vValues;

	while (q.empty() == false)
	{
		int ny = q.front().y;
		int nx = q.front().x;
		int ncost = q.front().cost;
		q.pop();

		if (ncost >= ret)
		{
			if (ncost > ret)
			{
				ret = ncost;
				vValues.clear();
			}
			vValues.push_back({ ny,nx });
		}

		for (int i = 0; i < 4; i++)
		{
			int nextY = ny + dirY[i];
			int nextX = nx + dirX[i];

			if (checkRightArrange(nextY, nextX) == false)
				continue;

			if (maps[nextY][nextX] == 'W')
				continue;

			if (check[nextY][nextX])
				continue;

			check[nextY][nextX] = true;
			q.push({ nextY,nextX,ncost + 1 });
		}
	}

	return ret;
}

int main()
{
	cin.tie(nullptr);
	ios::sync_with_stdio(false);

	cin >> h >> w;

	vector<string> maps(h);
	for (int i = 0; i < h; i++)
		cin >> maps[i];

	int bestV = 0;

	vector<vector<bool>> visit(h, vector<bool>(w, false));

	for (int i = 0; i < h; i++)
	{
		for (int j = 0; j < w; j++)
		{
			if (maps[i][j] == 'L')
			{
				int v = bfs(maps, i, j);
				if (v > bestV)
					bestV = v;
			}
		}
	}

	cout << bestV;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/402e45ef-b241-4fb6-8dca-56a26da3eedb)](https://github.com/user-attachments/assets/402e45ef-b241-4fb6-8dca-56a26da3eedb){: .image-popup}<br>

다만 처음 코드에 시간초과가 발생하였기에<br>
로직적 문제 or 입력 관련인줄 알고 여러 시도를 해보았다<br>

- input 을 의식하여 cin.tie 와 ios::sys 쪽 코드 추가<br>

- 로직적 문제인가 싶어 BFS를 2중으로 하는 방식 고려<br>
  : 첫 BFS 이후, 가장 먼 지점을 찾아<br>
    해당 지점에서 다시 BFS를 통해 가장 먼 지점을 찾음<br>
  - 이후 탐색한 영역에 대한 체크(같은 섬을 반복 탐색 하지 않기 위해)<br>
  - 다만 이 방식은 먼 지점이 '여러 개'인 경우<br>
    탐색 시간이 늘어날 수 있어 포기<br>
	또한, 정확도 테스트에서 오답이 되었다<br>

계속해서 생각해보았으나<br>
답이 나오지 않자<br>
결국 GPT에게 조언을 구했는데...<br>

```cpp
bool checkRightArrange(int nowY, int nowX)
{
	if (nowY < 0 || nowY >= h ||
		nowX < 0 || nowX >= w)
		return false;

	return true;
}
```

이 함수에 inline을 붙이라고 하였다...<br>
그리고는 허무하게 통과했다...<br>

- 생각해보니 해당 함수는 노드당 약 4번 호출되고 있음<br>
  (함수 호출에 따른 오버헤드가 기하급수적으로 증가)<br>

- 기존에는 해당 방식을 그냥 손으로 쳤기에<br>
  문제를 고려하지 못하였음<br>

난이도 자체는 무척 쉬운 편이었으나<br>
함수 호출의 오버헤드와<br>
inline의 중요성에 대하여 다시 알 수 있는 문제였다<br>

- 물론 기본적으로 클래스에서 가능하면 inline 최적화를 해주긴하니<br>
  전역 함수나 Util용 static 함수에서 고려하자<br>