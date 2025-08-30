---
title: "백준 Gold 1 구슬 탈출 2"
date : "2025-08-30 09:00:00 +0900"
last_modified_at: "2025-08-30T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 그래프 탐색
  - BFS
---

## 구슬 탈출 2 (백준 Gold 1)
<https://www.acmicpc.net/problem/13460><br>

NxM 사각형 미로에서<br>
빨간 구슬(R)과 파란 구슬(B)을 동시에 같은 방향으로 움직이며<br>
빨간 구슬이 목적지인(O)에 도달하도록 하는 최소 횟수를 구하기<br>

### 문제 유의 사항들

- 파란 구슬과 빨간 구슬은 동시에 움직이나<br>
  같은 위치에 존재할 수는 없다<br>

- 구슬들은 해당 방향으로 쭉 나간다<br>

- 구슬들이 더 이상 움직이지 않는 것을 1회로 친다<br>

- 파란 구슬이 먼저 목적지에 도착하거나, 빨간 구슬과<br>
  같은 횟수로 나가게 된다면 실패 처리<br>

- 시도 횟수가 10회를 넘어가게 된 경우도 실패 처리<br>

## 첫 제출 코드

```
#include<iostream>
#include<vector>
#include<queue>
#include<string>
#include<limits.h>

using namespace std;

struct posInfo
{
	int y, x;
};

enum dir
{
	D_down,
	D_right,
	D_up,
	D_left
};

struct moveInfo
{
	posInfo Red, Blue;
	int cost;
	dir to;
};

struct Compare
{
	bool operator()(const moveInfo& a, const moveInfo& b)
	{
		return a.cost > b.cost;
	}
};

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0,1,0,-1 };

posInfo Red, Blue,Gole;
int n, m;

int BFS(const vector<vector<char>>& maps)
{
	priority_queue<moveInfo, vector<moveInfo>, Compare> pq;
	pq.push({ Red,Blue,1 ,D_down });
	pq.push({ Red,Blue,1 ,D_right });
	pq.push({ Red,Blue,1 ,D_up });
	pq.push({ Red,Blue,1 ,D_left });

	vector<vector<int>> visited(n,vector<int>(m,INT_MAX));

	while (pq.empty() == false)
	{
		posInfo nowRed = pq.top().Red;
		posInfo nowBlue = pq.top().Blue;
		int nowCost = pq.top().cost;
		dir nowDir = pq.top().to;
		pq.pop();

		posInfo nextRed{ nowRed.y + dirY[nowDir],nowRed.x + dirX[nowDir] };
		posInfo nextBlue{ nowBlue.y + dirY[nowDir],nowBlue.x + dirX[nowDir] };

		bool isContinue = true;

		// 두 위치는 같이 존재할 수 없음
		if (nextRed.y == nextBlue.y &&
			nextRed.x == nextBlue.x)
			continue;

		if (maps[nextRed.y][nextRed.x] == 'O')
		{
			// 해당 방향으로 계속 굴렸을때 파란 구슬이 빠져나오는지 확인해야 함
			posInfo checkBlue = nextBlue;
			bool isFailed = false;
			while (true)
			{
				if (maps[checkBlue.y][checkBlue.x] == 'O')
				{
					isFailed = true;
					break;
				}

				if (maps[checkBlue.y][checkBlue.x] == '#')
				{
					break;
				}

				checkBlue.y += dirY[nowDir];
				checkBlue.x += dirX[nowDir];
			}

			if (isFailed)
				continue;

			return nowCost;
		}

		if (maps[nextRed.y][nextRed.x] == '#')
		{
			if (visited[nowRed.y][nowRed.x] < nowCost)
				continue;

			visited[nowRed.y][nowRed.x] = nowCost;
			nextRed = nowRed;
			nextBlue = nowBlue;
			isContinue = false;
		}

		if (maps[nextBlue.y][nextBlue.x] == '#')
		{
			// 파란공 제자리에 막힘
			nextBlue = nowBlue;
		}

		if (isContinue)
		{
			pq.push({ nextRed,nextBlue,nowCost,nowDir });
		}
		else
		{
			for (int i = 0; i < 4; i++)
			{
				if (i == nowDir)
					continue;

				pq.push({ nextRed,nextBlue,nowCost + 1,(dir)i });
			}
		}
	}

	return -1;
}

int main()
{
	cin >> n >> m;
	vector<vector<char>> maps;
	for (int i = 0; i < n; i++)
	{
		string str;
		cin >> str;
		maps.push_back(vector<char>());
		for (int j = 0; j < m;j++)
		{
			char c = str[j];
			maps[i].push_back(c);

			if (c == 'B')
			{
				Blue.y = i;
				Blue.x = j;
			}

			if (c == 'R')
			{
				Red.y = i;
				Red.x = j;
			}

			if (c == 'O')
			{
				Gole.y = i;
				Gole.x = j;
			}
		}
	}

	cout << BFS(maps);

	return 0;
}
```

### 틀린 이유?

각각 1칸씩 진행하면서<br>
처리하는 방식이지만 이 방식으로 찾아내지 못하는 경우의 수가 존재하였다<br>

- '빨간 구슬'은 가만히 있는데 '파란 구슬'만 움직이는 경우<br>
  풀 수 있는 테스트 케이스를 통과하지 못함<br>

따라서 해당 방향으로 '끝'까지 진행하는 쪽으로 코드를 수정해야 하였다<br>

## 풀이방법
한 방향으로 '끝'까지 진행하는 방식<br>

또한 <br>
priority_queue를 이용하여 최단거리를 구하려 하였고<br>
map을 이용하여, '붉은 구슬'의 위치가 동일함에도<br>
'파란 구슬'의 위치가 다른 케이스를 구분하였다<br>

- 두 구슬이 한 벽면에서 만나는 경우에 대한 처리 (ex : #RB..)<br>
  : 특정 방면으로 'R'이나 'B'가 벽에 '닿은' 경우에 대한 bool 변수를<br>
    추가하여 검사하였다<br>
	(같은 방향 진행 시,둘 중 하나만이 먼저 벽에 닿으므로)<br>
	이후, R == B 인 경우<br>
	벽에 닿지 않은 구슬을 dir 의 역방향으로 한칸 떼어 주었다<br>

## 최종 제출 코드
```
#include<iostream>
#include<vector>
#include<queue>
#include<string>
#include<map>
#include<set>
#include<limits.h>

using namespace std;

struct posInfo
{
	int y, x;

	bool operator==(const posInfo& other)
	{
		return y == other.y && x == other.x;
	}

	posInfo operator+(const posInfo& other)
	{
		return { y + other.y,x + other.x };
	}

};

enum dir
{
	D_down,
	D_right,
	D_up,
	D_left
};

struct moveInfo
{
	posInfo Red, Blue;
	int cost;
};

struct Compare
{
	bool operator()(const moveInfo& a, const moveInfo& b)
	{
		return a.cost > b.cost;
	}
};

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0,1,0,-1 };

posInfo Red, Blue, Gole;
int n, m;

int BFS(const vector<vector<char>>& maps)
{
	priority_queue<moveInfo, vector<moveInfo>, Compare> pq;
	pq.push({ Red,Blue,0 });

	map <pair<int, int>, set< pair<int, int>>> visited;

	while (pq.empty() == false)
	{
		posInfo nowRed = pq.top().Red;
		posInfo nowBlue = pq.top().Blue;
		int nowCost = pq.top().cost;
		pq.pop();

		// 두 위치는 같이 존재할 수 없음
		if (nowRed == nowBlue)
			continue;

		if (nowCost > 10)
			continue;

		if (nowRed == Gole)
		{
			return nowCost;
		}

		if (nowBlue == Gole)
			continue;

		if (visited[{nowRed.y, nowRed.x}].find({ nowBlue.y,nowBlue.x }) != visited[{nowRed.y, nowRed.x}].end())
		{
			continue;
		}

		visited[{nowRed.y, nowRed.x}].insert({ nowBlue.y,nowBlue.x });

		for (int i = 0; i < 4; i++)
		{
			posInfo nextRed = nowRed;
			posInfo nextBlue = nowBlue;

			bool bMoveRed = false;
			bool bStuckRed = false;

			bool bMoveBlue = false;
			bool bStuckBlue = false;

			while (bMoveRed == false ||
				bMoveBlue == false)
			{
				if(bMoveRed == false)
					nextRed = nextRed + posInfo{ dirY[i],dirX[i] };

				if (maps[nextRed.y][nextRed.x] == '#')
				{
					bMoveRed = true;
					bStuckRed = true;
					nextRed = nextRed + posInfo{ -dirY[i],-dirX[i] };
				}
				else if (nextRed == Gole)
				{
					bMoveRed = true;
				}

				if(bMoveBlue == false)
					nextBlue = nextBlue + posInfo{ dirY[i],dirX[i] };

				if (maps[nextBlue.y][nextBlue.x] == '#')
				{
					bMoveBlue = true;
					bStuckBlue = true;
					nextBlue = nextBlue + posInfo{ -dirY[i],-dirX[i] };
				}
				else if (nextBlue == Gole)
				{
					bMoveBlue = true;
				}
				
				if (nextRed == nextBlue)
				{
					if (bStuckRed)
					{
						nextBlue = nextBlue + posInfo{ -dirY[i],-dirX[i] };
					}
					else if(bStuckBlue)
					{
						nextRed = nextRed + posInfo{ -dirY[i],-dirX[i] };
					}
					break;
				}
			}

			pq.push({ nextRed,nextBlue,nowCost + 1 });
		}
	}

	return -1;
}

int main()
{
	cin >> n >> m;
	vector<vector<char>> maps;
	for (int i = 0; i < n; i++)
	{
		string str;
		cin >> str;
		maps.push_back(vector<char>());
		for (int j = 0; j < m; j++)
		{
			char c = str[j];
			maps[i].push_back(c);

			if (c == 'B')
			{
				Blue.y = i;
				Blue.x = j;
			}

			if (c == 'R')
			{
				Red.y = i;
				Red.x = j;
			}

			if (c == 'O')
			{
				Gole.y = i;
				Gole.x = j;
			}
		}
	}

	cout << BFS(maps);

	return 0;
}
```

## 결과
<img width="1152" height="161" alt="Image" src="https://github.com/user-attachments/assets/45092e8b-56bb-42ca-8a17-dc10bf13092a" /><br>

세부 구현 사항이 상당히 까다로웠던 문제였다<br>