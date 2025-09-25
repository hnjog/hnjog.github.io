---
title: "백준 Gold 1 복제 로봇"
date : "2025-09-25 10:30:00 +0900"
last_modified_at: "2025-09-25T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - 그래프 탐색
  - 최소 스패닝 트리
  - MST
  - 크루스칼
  - Union Find
  - BFS
---

## 복제 로봇 (백준 Gold 1)
<https://www.acmicpc.net/problem/1944><br>

로봇을 움직여 맵에 존재하는 모든 Key를 얻는 최소 이동 거리를 구하는 문제<br>

- 로봇 자가 복제 될 수 있기에<br>
  상하좌우 4방향으로 복제할 수 있다<br>

- 로봇이 Key를 회수하더라도 딱히 원래 자리로 돌아올 필요는 없음<br>

## 풀이 방법
이전과 비슷한 `'그래프 탐색 + MST'` 계열의 문제이다<br>

이전 처럼 푸는 단계를 정해두고 풀어보았다<br>

### 1. 로봇과 Key의 위치를 각각 저장<br>

탐색에 사용할 정보를 String으로 받은 후<br>
각각의 위치를 저장해두었다<br>

### 2. 로봇이 Key의 위치를 탐색하는 그래프 탐색 코드 작성<br>
나는 Priority Queue 기반의 BFS 탐색을 사용<br>
(다익스트라와 유사)<br>

- 이때, 유의할 점은 Key와 Key 사이의 거리 또한 구해야 한다는 점이다<br>
  (Key에서 Key로 이동하는 방법만 가능하거나, 이쪽이 더 저렴한 비용일 수 있음)<br>

### 3. 이후 만들어진 경로를 통하여 MST를 작성<br>

로봇의 위치를 0번 인덱스<br>
각 Key들에게 임의의 인덱스를 주어<br>
연결 여부와 비용을 확인하는<br>
크루스칼 알고리즘을 사용<br>

모두 연결되었는지를 확인한다<br>

## 첫 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<queue>
#include<algorithm>

using namespace std;

int n, m;

struct posInfo
{
	int y, x;
	int idx;
};

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

struct Edge
{
	int s, t,cost;
};

void FindShortDis(const vector<string>& maps, const posInfo& start, const posInfo& target, vector<Edge>& outEdges)
{
	struct qInfos
	{
		int y, x;
		int cost;
	};

	struct Compare
	{
		bool operator()(qInfos& a, qInfos& b)
		{
			return a.cost > b.cost;
		}
	};

	const int dirY[4] = { -1,0,1,0 };
	const int dirX[4] = { 0,1,0,-1 };

	priority_queue<qInfos, vector<qInfos>, Compare> pq;
	vector<vector<bool>> visit(n, vector<bool>(n, false));
	pq.push({ start.y,start.x,0 });

	while (pq.empty() == false)
	{
		int cy = pq.top().y;
		int cx = pq.top().x;
		int cCost = pq.top().cost;
		pq.pop();

		if (cy < 0 || cy >= n ||
			cx < 0 || cx >= n)
			continue;

		if (maps[cy][cx] == '1')
			continue;

		if (visit[cy][cx])
			continue;

		visit[cy][cx] = true;

		if (cy == target.y &&
			cx == target.x)
		{
			outEdges.emplace_back(Edge{start.idx,target.idx,cCost});
			return;
		}

		for (int i = 0; i < 4; i++)
		{
			pq.push(qInfos{cy + dirY[i],cx + dirX[i],cCost + 1});
		}
	}

}

int main()
{
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	cin >> n >> m;
	vector<string> maps(n);

	vector<posInfo> keys;
	keys.reserve(m + 1);

	int flag = 1;
	for (int i = 0; i < n; i++)
	{
		cin >> maps[i];
		for (int j = 0; j < n; j++)
		{
			if (maps[i][j] == 'S' ||
				maps[i][j] == 'K')
			{
				posInfo p;
				p.y = i;
				p.x = j;
				p.idx = maps[i][j] == 'S' ? 0 : flag++;
				keys.emplace_back(p);
			}
			
		}
	}

	vector<Edge> sEdges;

	for (int i = 0; i <= m; i++)
	{
		for (int j = i + 1; j <= m; j++)
		{
			FindShortDis(maps, keys[i], keys[j], sEdges);
		}
	}

	sort(sEdges.begin(), sEdges.end(), []
	(const Edge& a, const Edge& b)
		{
			return a.cost < b.cost;
		}
	);

	int answer = 0;
	int kCount = 0;
	vector<int> pv(m + 1);
	for (int i = 0; i <= m; i++)
		pv[i] = i;

	for (auto& e : sEdges)
	{
		if (Union(pv, e.s, e.t))
		{
			answer += e.cost;
			kCount++;
			if (kCount == m)
				break;
		}
	}

	int s = FindParent(pv,0);

	for (int i = 1; i <= m; i++)
	{
		if (s != FindParent(pv, pv[i]))
		{
			cout << -1;
			return 0;
		}
	}

	cout << answer;

	return 0;
}
```

## 틀린 이유 1 - 시간초과

모든 Key들에 대하여 '일일이' BFS 탐색을 하였기에<br>
지나치게 느려졌다<br>

따라서 해당 부분을 수정하기 위하여<br>
Map에 Flag를 넣어 인덱스값을 쉽게 가져오려 하였다<br>

## 다음 제출 코드

```cpp
...

void FindShortDis(const vector<string>& maps, const posInfo& start, vector<Edge>& outEdges)
{
	...

	char startFlag = maps[start.y][start.x];

	priority_queue<qInfos, vector<qInfos>, Compare> pq;
	vector<vector<bool>> visit(n, vector<bool>(n, false));
	pq.push({ start.y,start.x,0 });

	while (pq.empty() == false)
	{
		int cy = pq.top().y;
		int cx = pq.top().x;
		int cCost = pq.top().cost;
		pq.pop();

		if (cy < 0 || cy >= n ||
			cx < 0 || cx >= n)
			continue;

		if (maps[cy][cx] == '1')
			continue;

		if (visit[cy][cx])
			continue;

		visit[cy][cx] = true;

		if (maps[cy][cx] >= '3' &&
			maps[cy][cx] != startFlag)
		{
			outEdges.emplace_back(Edge{start.idx - 2,(maps[cy][cx] - '0' - 2),cCost});
			continue;
		}

		for (int i = 0; i < 4; i++)
		{
			pq.push(qInfos{cy + dirY[i],cx + dirX[i],cCost + 1});
		}
	}
}

int main()
{
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	cin >> n >> m;
	vector<string> maps(n);

	vector<posInfo> keys;
	keys.reserve(m + 1);

	int flag = 3;
	for (int i = 0; i < n; i++)
	{
		cin >> maps[i];
		for (int j = 0; j < n; j++)
		{
			if (maps[i][j] == 'S' ||
				maps[i][j] == 'K')
			{
				posInfo p;
				p.y = i;
				p.x = j;
				if (maps[i][j] == 'S')
				{
					maps[i][j] = '2';
					p.idx = 2;
				}
				else
				{
					p.idx = flag;
					maps[i][j] = '0' + flag;
					flag++;
				}
				
				keys.emplace_back(p);
			}
			
		}
	}

	vector<Edge> sEdges;

	for (int i = 0; i <= m; i++)
	{
		FindShortDis(maps, keys[i], sEdges);
	}

	...

	return 0;
}
```

## 틀린 이유 2 - M 값은 10 이상이 될 수 있음
Flag를 저렇게 넣고 보니<br>
M은 총 250까지 들어올 있는데<br>
10 이상 부터는 값이 '이상'해진다는 문제를 뒤늦게 발견하였다<br>

따라서 그냥 map을 통하여 <위치,idx>를 관리하는 쪽으로 바꾸었다<br>

## 최종 제출 코드

```cpp
...

void FindShortDis(const vector<string>& maps, const posInfo& start, vector<Edge>& outEdges, map<pair<int, int>, int>& keyMaps)
{
	...

	priority_queue<qInfos, vector<qInfos>, Compare> pq;
	vector<vector<bool>> visit(n, vector<bool>(n, false));
	pq.push({ start.y,start.x,0 });

	while (pq.empty() == false)
	{
		...

		if (maps[cy][cx] == 'K' &&
			keyMaps[{cy, cx}] != start.idx)
		{
			outEdges.emplace_back(Edge{start.idx,keyMaps[{cy, cx}],cCost});
			continue;
		}

		...
	}
}

int main()
{
	...

	map<pair<int, int>, int> keyMaps;

	int flag = 1;
	for (int i = 0; i < n; i++)
	{
		cin >> maps[i];
		for (int j = 0; j < n; j++)
		{
			if (maps[i][j] == 'S' ||
				maps[i][j] == 'K')
			{
				posInfo p;
				p.y = i;
				p.x = j;
				if (maps[i][j] == 'S')
				{
					p.idx = 0;
				}
				else
				{
					p.idx = flag;
					keyMaps[{i, j}] = flag;
					flag++;
				}
				
				keys.emplace_back(p);
			}
			
		}
	}

	...
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/4734a767-fe80-4d74-a68c-9b4c260e64aa)](https://github.com/user-attachments/assets/4734a767-fe80-4d74-a68c-9b4c260e64aa){: .image-popup}<br>

풀이 과정 자체는 올바르게 잡았지만<br>
세부 구현이 깔끔하지 못하여<br>
시간을 좀 잡아먹었다<br>

