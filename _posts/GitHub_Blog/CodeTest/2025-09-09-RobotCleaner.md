---
title: "백준 Gold 5 로봇 청소기"
date : "2025-09-09 11:00:00 +0900"
last_modified_at: "2025-09-09T11:00:00"
categories:
  - 코딩 테스트
tags:
  - 구현
  - 시뮬레이션
---

## 로봇 청소기 (백준 Gold 5)
<https://www.acmicpc.net/problem/14503><br>

로봇 청소기와 방의 상태가 주어졌을 때,<br>
'청소하는 영역'의 개수를 구하는 문제<br>

## 풀이 방법

기본 규칙은 다음과 같다<br>

- 현재 칸이 청소되지 않은 경우 청소<br>
- 주변 4칸(상하좌우)가 청소할 칸이 없는 경우<br>
  - 바라보는 방향을 유지한채로 '후진'할 수 있으면 후진<br>
  - 후진하려는데 '벽'에 부딪히면 작동 종료후 결산<br>
- 주변 4칸에 청소할 칸이 있는 경우<br>
  - 반시계 90도 회전<br>
  - 바라보는 방향 기준으로 '정면' 칸이 청소할 칸이라면 전진<br>

유의할 점이 몇가지 있다<br>

- 블럭을 '청소 안한 칸', '벽' 그리고 '청소한 칸'으로 나눌 것<br>
  : '청소한 칸'은 '전진' 고려 대상은 아니지만<br>
    후진을 할 수는 있다<br>

- 좌표의 기준이 좌상단이다<br>
  그렇기에 dir 'Up'과 같은 것을 사용하려면<br>
  -1 을 해주어야 한다<br>
  (이것때문에 삽질 좀 하였다)<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

enum dir
{
	d_Up = 0,
	d_Right,
	d_Down,
	d_Left
};

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0, 1,0,-1 };

class Robot
{
public:
	Robot(int y, int x, int d)
		:posY(y),
		posX(x),
		nowDir(d),
		amount(0)
	{
	}

	int Work(vector<vector<int>>& maps)
	{
		int n = maps.size();
		int m = maps[0].size();

		while (true)
		{
			// 현재 칸이 청소되지 않은 경우 청소
			if (maps[posY][posX] == 0)
			{
				amount++;
				maps[posY][posX] = 2;
				continue;
			}

			if (SearchNonClear(maps) == false)
			{
				if (CheckBack(maps))
				{
					// 후진 가능
					posY -= dirY[nowDir];
					posX -= dirX[nowDir];
				}
				else
				{
					// 후진 못하면 작동 정지
					break;
				}
			}
			else // 주변 4칸에 청소되지 않은 빈칸 존재
			{
				nowDir--;
				if (nowDir < 0)
					nowDir = d_Left;

				// 정면 한칸 체크
				if (CheckFront(maps))
				{
					// 더럽다면 전진
					posY += dirY[nowDir];
					posX += dirX[nowDir];
				}
			}
		}

		return amount;
	}

protected:
	bool SearchNonClear(vector<vector<int>>& map)
	{
		int n = map.size();
		int m = map[0].size();

		for (int i = 0; i <= d_Left; i++)
		{
			int ny = posY + dirY[i];
			int nx = posX + dirX[i];

			if (ny < 0 || ny >= n ||
				nx < 0 || nx >= m)
				continue;

			if (map[ny][nx] == 0)
				return true;
		}

		return false;
	}

	bool CheckFront(vector<vector<int>>& map)
	{
		int n = map.size();
		int m = map[0].size();

		int ny = posY+ dirY[nowDir];
		int nx = posX+ dirX[nowDir];
		
		if (map[ny][nx] == 0)
			return true;
		
		return false;
	}

	bool CheckBack(vector<vector<int>>& map)
	{
		int n = map.size();
		int m = map[0].size();

		int ny = posY - dirY[nowDir];
		int nx = posX - dirX[nowDir];

		if (map[ny][nx] == 1)
			return false;

		return true;
	}

protected:
	int posY;
	int posX;
	int nowDir;
	int amount;
};

int main()
{
	int n, m;
	cin >> n >> m;

	int y, x, d;
	cin >> y >> x >> d;

	Robot robot(y, x, d);

	vector<vector<int>> map(n, vector<int>(m, -1));

	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			cin >> map[i][j];

	cout << robot.Work(map);
	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/546f2acd-22d1-4f26-be4e-54e2b3cc992a)](https://github.com/user-attachments/assets/546f2acd-22d1-4f26-be4e-54e2b3cc992a){: .image-popup}<br>

main에 구현하여도 되지만<br>
클래스로 만들어 '자동 청소'되는 느낌을 주려해보았다<br>

구현 문제는 각 '요구'와 '제한'을 잘 읽어야 풀 수 있는 문제임을<br>
다시 실감하는 문제였다<br>