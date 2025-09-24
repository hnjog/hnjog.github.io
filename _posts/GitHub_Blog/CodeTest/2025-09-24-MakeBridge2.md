---
title: "백준 Gold 1 다리 만들기 2"
date : "2025-09-24 10:30:00 +0900"
last_modified_at: "2025-09-24T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - 그래프 탐색
  - 최소 스패닝 트리
  - MST
  - 크루스칼
  - Union Find
  - DFS
---

## 다리 만들기 2 (백준 Gold 1)
<https://www.acmicpc.net/problem/17472><br>

N,M이 주어지고<br>
N X M 으로 이루어진 맵 정보가 주어진다<br>
0은 바다, 1은 섬의 일부이다<br>

각각의 섬들에 다리를 놓았을 때<br>
모든 섬이 연결되는 최소 거리를 구하시오<br>

유의사항<br>

- 다리의 길이는 최소 2 이상이여야 한다<br>
- 다리가 겹치더라도 별개의 거리로 계산한다<br>
- 다리의 '방향'이 다른경우는 근처에 있어도 연결되지 않은것으로 침<br>
  (ex : 위쪽을 향한 다리인데 한칸 왼쪽에 다른 섬이 있어도 연결 X)<br>

## 풀이 방법
`그래프 탐색 + MST` 로 푸는 문제로 판단할 수 있었다<br>
('섬'의 판단 여부 및 '최소 연결 거리')<br>

그렇기에 먼저 문제를 푸는 것을<br>
총 3개의 단계로 나누었다<br>

1. 각각의 섬에 깃발(Flag)을 세워 **구분**하기<br>
   : map의 값이 1인 섬들에서 DFS를 통해 탐색하여 확인<br>

2. 이후 각각의 섬을 연결하는 '다리'를 만들고 저장하기<br>
   : 방향을 하나 정한후, 쭉 탐색하여 자신의 섬이 아니라면 Edge에 저장<br>

3. 만든 다리들을 크루스칼을 통해 짧은 다리만 남기기 + 모든 섬이 연결됬는지 확인<br>
   : Union-Find를 통한 탐색 및 모든 부모가 같은지를 체크<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

using namespace std;

int n, m;

struct edge
{
	int s, t;
	int cost;
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

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0,1,0,-1 };

void MapMark(vector<vector<int>>& maps, int y, int x, int flag)
{
	if (y < 0 || y >= n ||
		x < 0 || x >= m)
		return;

	if (maps[y][x] != 1)
		return;

	maps[y][x] = flag;

	for (int i = 0; i < 4; i++)
	{
		MapMark(maps, y + dirY[i], x + dirX[i], flag);
	}
}

void MakeBridge(vector<vector<int>>& maps, vector<edge>& edgeVec, int y, int x, int flag,int dir, int dis)
{
	if (y < 0 || y >= n ||
		x < 0 || x >= m)
		return;

	if (maps[y][x] != 0)
	{
		if (maps[y][x] != flag &&
			dis > 1)
		{
			edgeVec.push_back({ flag,maps[y][x],dis });
		}

		return;
	}

	MakeBridge(maps, edgeVec, y + dirY[dir], x + dirX[dir], flag, dir, dis + 1);
}

void CheckBridge(vector<vector<int>>& maps, vector<edge>& edgeVec, int y, int x, int flag)
{
	for (int i = 0; i < 4; i++)
	{
		MakeBridge(maps, edgeVec, y + dirY[i], x + dirX[i], flag, i, 0);
	}
}

int main()
{
	cin >> n >> m;

	vector<vector<int>> maps(n, vector<int>(m, -1));
	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			cin >> maps[i][j];

	int flag = 2;
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			if (maps[i][j] == 1)
				MapMark(maps, i, j, flag++);
		}
	}

	vector<edge> edges;

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			if (maps[i][j] > 1)
			{
				CheckBridge(maps, edges, i, j, maps[i][j]);
			}
		}
	}

	sort(edges.begin(), edges.end(), []
	(const edge& a, const edge& b)
		{
			return a.cost < b.cost;
		});

	vector<int> parents(flag -2);
	for (int i = 0; i < flag-2; i++)
	{
		parents[i] = i;
	}

	int answer = 0;
	for (auto& e : edges)
	{
		if (Union(parents, e.s - 2, e.t - 2))
		{
			answer += e.cost;
		}
	}

	int p = FindParent(parents,0);
	for (int i = 1; i < flag - 2; i++)
	{
		if (p != FindParent(parents,i))
		{
			cout << -1;
			return 0;
		}
	}
	cout << answer;

	return 0;
}
```


## 결과
[![Image](https://github.com/user-attachments/assets/531aedd6-2317-4aa3-ba8d-75af55a79b00)](https://github.com/user-attachments/assets/531aedd6-2317-4aa3-ba8d-75af55a79b00){: .image-popup}<br>

밑의 틀린 이유는<br>
처음 부모 검사 코드가 이랬기 때문이다<br>

```cpp
int p = parents[0];
	for (int i = 1; i < flag - 2; i++)
	{
		if (p != parents[i])
		{
			cout << -1;
			return 0;
		}
	}
```

유니온 파인드의 경우<br>
FindParent가 호출되면서<br>
이전의 부모값들이 갱신되는데<br>

그걸 깜빡하고 저렇게 체크를 하니<br>
갱신을 하지 않고 체크하여 올바른 값이 아니라 -1을 출력한 케이스가 존재하기 때문<br>

푸는 단계를 잘 정하고 풀었기에<br>
비교적 쉽게 접근할 수 있었다<br>