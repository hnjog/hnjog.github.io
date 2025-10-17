---
title: "백준 Gold 3 욕심쟁이 판다"
date : "2025-10-17 10:30:00 +0900"
last_modified_at: "2025-10-17T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 다이나믹 프로그래밍
  - DFS
---

## 욕심쟁이 판다 (백준 Gold 3)
<https://www.acmicpc.net/problem/1937><br>

N x N 크기의 맵 데이터가 주어지고<br>
특정 위치에 판다를 풀어놓았을 때<br>
판다가 먹이를 먹는 칸의 가장 큰 개수를 구하는 문제<br>

- 판다는 시작 칸부터 먹는다<br>
- 판다는 상하좌우 중 하나로 이동<br>
- 판다는 반드시 현재 칸보다 많은 칸인 경우에만 먹이를 먹음<br>

## 풀이 방법

'해당 칸'에서 '탐색'한 결과를 다른 탐색에서도 이용할 수 있는<br>
DP 문제이다<br>

판다는 `이전 단계`로 돌아가지 않으며<br>
각 단계별로 탐색의 결과가 나뉜다<br>
또한, 시작 위치에 따라 각 탐색 결과가 **바뀌지 않음!**<br>

- dp[i][j]<br>
 : i,j에서 dfs 탐색을 진행하여<br>
   가장 깊은 탐색 결과를 저장<br>

- 재귀를 통해 탐색하며<br>
  이미 dp[i][j]에 값이 있다면 바로 가져옴<br>
  아니라면<br>
  그 결과를 dp[i][j]에 저장해둠<br>

- 모든 위치를 각각 dfs를 진행하되<br>
  각각 저장된 값을 이용하기에<br>
  점점 더 캐싱된 값이 많아짐<br>

- 이후 '자기자신'을 포함하기에 ans + 1 을 출력<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

const int dirY[4] = { 0,-1,0,1 };
const int dirX[4] = { -1,0,1,0 };

int n;

int bestValue(const vector<vector<int>>& maps, vector<vector<int>>& dp,int nowY,int nowX)
{
	if (dp[nowY][nowX] >= 0)
		return dp[nowY][nowX];

	int best = 0;

	for (int i = 0; i < 4; i++)
	{
		int ny = nowY + dirY[i];
		int nx = nowX + dirX[i];

		if (ny < 0 || ny >= n ||
			nx < 0 || nx >= n)
			continue;

		if (maps[ny][nx] <= maps[nowY][nowX])
			continue;

		int t = bestValue(maps, dp, ny, nx) + 1;

		if (t > best)
			best = t;
	}

	return dp[nowY][nowX] = best;
}

int main()
{
	cin >> n;
	vector<vector<int>> maps(n, vector<int>(n, 0));
	vector<vector<int>> dp(n, vector<int>(n, -1));

	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			cin >> maps[i][j];

	int ans = 0;

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < n; j++)
		{
			int v = bestValue(maps, dp, i, j);
			if (v > ans)
				ans = v;
		}
	}

	cout << ans + 1;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/c8975553-6cfd-44fd-b9e0-39e84198d459)](https://github.com/user-attachments/assets/c8975553-6cfd-44fd-b9e0-39e84198d459){: .image-popup}<br>

- dp를 적용하는 방식인 각 map 데이터 활용<br>
- 재귀를 통한 탐색 결과 저장<br>

양측 모두 구현이 까다로웠던 문제이다<br>
