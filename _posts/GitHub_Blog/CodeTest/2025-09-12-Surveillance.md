---
title: "백준 Gold 3 감시"
date : "2025-09-12 09:00:00 +0900"
last_modified_at: "2025-09-12T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 구현
  - 브루트포스
  - 백트래킹
---

## 감시 (백준 Gold 3)
<https://www.acmicpc.net/problem/15683><br>

[![Image](https://github.com/user-attachments/assets/7d9e971b-04eb-463e-b443-2e955ee29a80)](https://github.com/user-attachments/assets/7d9e971b-04eb-463e-b443-2e955ee29a80){: .image-popup}<br>

5가지 모양의 CCTV가 주어졌을 때,<br>
CCTV들로 감시하지 못하는 총 사각지대의 영역을 구하는 문제<br>

- CCTV는 마음대로 회전시킬 수 있다<br>
- CCTV는 CCTV를 통과하여 감시할 수 있음<br>
- 다만 '벽'을 만나는 경우는 그 뒤의 공간은 감시할 수 없음<br>

## 풀이 방법

일단 여러 '방향'에 대한 전체적인 검사가 필요하기에<br>
'백트래킹' 문제이다<br>

유의할 점 몇가지가 존재한다<br>

일단 나는 cctv를 카메라로 인식하였기에<br>
변수명에 카메라 등을 사용하였다<br>
(설명도 그렇게...)<br>

- 일단 벽은 6번이며 카메라가 아니다<br>
  (바보 같은 소리지만 이것 때문에 시간 초과가 났었다...)<br>

- 2번 카메라는 방향을 한번만 돌린다면 나머지 방향은 체크할 필요가 없다<br>
  (<-> 을 가로로 한번만 돌리면 4방향이 아니라 2방향만 봐도 됨)<br>

- 마찬가지로 5번 카메라는 방향을 돌릴 필요조차 없다<br>
  (한번에 전방위를 다보므로)<br>

- 특정한 영역은 '한 번'보면 감시 영역이 되지만<br>
  자신을 보는 다른 카메라가 있는지를 알아야 한다<br>
  - 나는 0인 값이 -를 주어 -1,-2 이런식으로 보는 카메라의 개수를<br>
    표기하였다<br>

- 백트래킹이기에, 해당 방향을 다 보았다면 카메라를 돌리면서<br>
  영역들을 원래대로 되돌려 놓아야 한다<br>
  다만 다시 loop를 돌기보다는<br>
  이전에 '표시'한 영역들을 따로 보관하고 있다가<br>
  그 녀석들만 체크해주는 방식을 사용하였다<br>
  (8x8이기에 메모리 초과는 발생하지 않을 것이라 가정)<br>

- 재귀를 돌던 중 0을 return 받으면<br>
  사실상 더 돌 필요가 없으므로 종료<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<limits.h>
#include<algorithm>

using namespace std;

struct pos
{
	int y, x;
};

int n, m;

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0, 1,0,-1 };

void sprayView(vector<vector<int>>& map, const pos& p, int mainDir,vector<pos>& ret)
{
	int y = p.y;
	int x = p.x;

	int ny = y, nx = x;

	while (true)
	{
		ny += dirY[mainDir];
		nx += dirX[mainDir];

		if (ny < 0 || ny >= n ||
			nx < 0 || nx >= m)
			break;

		if (map[ny][nx] <= 0)
		{
			map[ny][nx]--;
			ret.push_back({ ny,nx });
		}

		if (map[ny][nx] == 6)
			break;
	}
}

vector<pos> setView(vector<vector<int>>& map, const pos& p, int mainDir)
{
	vector<pos> ret;

	switch (map[p.y][p.x])
	{
	case 1:
	{
		sprayView(map, p, mainDir, ret);
	}
	break;
	case 2:
	{
		sprayView(map, p, mainDir, ret);
		int subDir = (mainDir + 2) % 4;
		sprayView(map, p, subDir, ret);
	}
	break;
	case 3:
	{
		sprayView(map, p, mainDir, ret);
		int subDir = mainDir - 1;
		if (subDir < 0)
			subDir = 3;
		sprayView(map, p, subDir, ret);
	}
	break;
	case 4:
	{
		sprayView(map, p, mainDir, ret);
		int subDir = mainDir - 1;
		if (subDir < 0)
			subDir = 3;
		sprayView(map, p, subDir, ret);

		int subDir2 = mainDir + 1;
		if (subDir2 > 3)
			subDir2 = 0;
		sprayView(map, p, subDir2, ret);
	}
	break;
	case 5:
	{
		sprayView(map, p, mainDir, ret);
		int subDir = mainDir - 1;
		if (subDir < 0)
			subDir = 3;
		sprayView(map, p, subDir, ret);

		int subDir2 = mainDir + 1;
		if (subDir2 > 3)
			subDir2 = 0;
		sprayView(map, p, subDir2, ret);

		int subDir3 = mainDir + 2;
		if (subDir3 > 3)
			subDir3 = 0;
		sprayView(map, p, subDir3, ret);
	}
	break;
	}

	return ret;
}

int recur(vector<vector<int>>& map, vector<pos>& cameraPoss, int idx)
{
	// -1 -2 처럼 보고 있는 시야를 중첩할 예정시킬 생각임
	int cSize = cameraPoss.size();
	if (idx == cSize)
	{
		int ret = 0;
		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < m; j++)
			{
				if (map[i][j] == 0)
					ret++;
			}
		}

		return ret;
	}

	int ret = INT_MAX;
	for (int i = 0; i < 4; i++)
	{
		vector<pos> poss = setView(map, cameraPoss[idx], i);
		ret = min(ret, recur(map, cameraPoss, idx + 1));
		if (ret == 0)
			return 0;

		for (auto& p : poss)
		{
			int y = p.y;
			int x = p.x;
			map[y][x]++;
		}

		if (map[cameraPoss[idx].y][cameraPoss[idx].x] == 2 &&
			i > 2)
			break;
	}

	return ret;
}

int main()
{
	cin >> n >> m;
	vector<vector<int>> map(n, vector<int>(m, 0));
	vector<pos> cameraPoss, crossSet;

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++)
		{
			cin >> map[i][j];
			if (map[i][j] == 6)
				continue;

			if (map[i][j] > 0)
			{
				if (map[i][j] == 5)
					crossSet.push_back({ i,j });
				else
					cameraPoss.push_back({ i,j });
			}
		}
	}

	for (auto& p : crossSet)
	{
		setView(map, p, 0);
	}

	cout << recur(map, cameraPoss, 0);


	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/54ef5414-7ea1-43be-9513-eca015461f5b)](https://github.com/user-attachments/assets/54ef5414-7ea1-43be-9513-eca015461f5b){: .image-popup}<br>

처음 시간초과들에서 <br>

```cpp
map[y][x] == 6
```

부분을 체크하지 않아 시간초과가 발생하였다<br>
나중에 벽까지 넣고 있단걸 알았고 그제서야<br>
시간초과를 벗어날 수 있었다<br>