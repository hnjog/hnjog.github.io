---
title: "백준 Gold 4 테트로미노"
date : "2025-09-10 09:00:00 +0900"
last_modified_at: "2025-09-10T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 구현
  - 브루트포스
---

## 테트로미노 (백준 Gold 4)
<https://www.acmicpc.net/problem/14500><br>

[![Image](https://github.com/user-attachments/assets/01167340-7d20-442c-83b6-9f0e97c41047)](https://github.com/user-attachments/assets/01167340-7d20-442c-83b6-9f0e97c41047){: .image-popup}<br>


정사각형 4개를 모여 붙인 도형을 테트로미노라 한다<br>
임의의 '맵'에서 다음과 같이 5가지 도형을<br>
배치하여 도형이 '덮은 수들의 합'이 가장 큰 것을 구하는 문제<br>

- 각 도형은 적절히 '회전/대칭' 시킬 수 있음<br>

## 풀이 방법

단순히 브루트 포스 문제인 것처럼 보이나<br>
조금 더 자세히 생각해보면<br>

저 4가지 모형을 하나의 점에 두고<br>
각각의 모형들을 회전 '대칭' 시켜보면<br>
사실상 시작점에서 '뻣어나가는' 길이 4개 짜리 dfs라고 봐도 좋다<br>
(단, 'ㅜ' 모양 제외)<br>

- 'ㅜ' 모양이 아닌 것들은 백트래킹을 통해 구할 수 있다<br>

- 'ㅜ' 모양의 경우는 1번 전진하였을때<br>
  다른 방향 2개를 잡아 검사하는 쪽으로 예외처리를 해주면 된다<br>


## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

const int dirY[4] = { -1,0,1,0 };
const int dirX[4] = { 0, 1,0,-1 };

int n, m;

int best(vector<vector<int>>& map, vector<vector<bool>>& visit, int startY, int startX, int count,int nowSum)
{
	if (startY < 0 || startY >= n ||
		startX < 0 || startX >= m)
		return 0;

	if (visit[startY][startX])
		return 0;

	if (count == 3)
	{
		return nowSum + map[startY][startX];
	}

	int ret = 0;
	visit[startY][startX] = true;
	
	for (int i = 0; i < 4; i++)
	{
		int ny = startY + dirY[i];
		int nx = startX + dirX[i];

		ret = max(ret, best(map, visit, ny, nx, count + 1, nowSum + map[startY][startX]));
	}

	// ㅜ 모양에 대한 예외처리
	if (count == 1)
	{
		for (int i = 0; i < 4; i++)
		{
			int ny = startY + dirY[i];
			int nx = startX + dirX[i];
			int j = i + 1;

			if (j > 3)
				j = 0;
			int nny = startY + dirY[j];
			int nnx = startX + dirX[j];

			if (ny < 0 || ny >= n ||
				nx < 0 || nx >= m ||
				nny < 0 || nny >= n ||
				nnx < 0 || nnx >= m ||
				visit[ny][nx] ||
				visit[nny][nnx])
				continue;

			ret = max(ret, nowSum + map[startY][startX] + map[ny][nx] + map[nny][nnx]);
		}
	}

	visit[startY][startX] = false;

	return ret;
}

int main()
{
	cin >> n >> m;
	vector<vector<int>> map(n,vector<int>(m,-1));
	vector<vector<bool>> visit(n,vector<bool>(m,false));

	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			cin >> map[i][j];
	
	int bestV = 0;
	
	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			bestV = max(bestV, best(map,visit, i, j, 0, 0));

	cout << bestV;
	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/a9331601-f580-4370-b734-b83c569f5214)](https://github.com/user-attachments/assets/a9331601-f580-4370-b734-b83c569f5214){: .image-popup}<br>

주어지는 예제의 범위가 약 500개 내외인 경우<br>
보통은 백트래킹이나 브루트 포스를 감안하여 접근해보는 방식도 존재한다<br>
(1000개 이상인 경우, 해당 알고리즘 적용 시 시간초과가 발생할 것이 뻔하므로!)<br>

그렇기에 보통 브루트 포스 문제가<br>
'판단력'과 '구현력'을 요구하는 경우가 많은 듯하다<br>