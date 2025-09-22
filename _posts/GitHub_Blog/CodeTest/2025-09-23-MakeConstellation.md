---
title: "백준 Gold 3 별자리 만들기"
date : "2025-09-23 10:30:00 +0900"
last_modified_at: "2025-09-23T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - 최소 스패닝 트리
  - MST
  - 크루스칼
  - Union Find
---

## 별자리 만들기 (백준 Gold 3)
<https://www.acmicpc.net/problem/4386><br>

주어지는 n개의 별자리의 **'좌표'**가 양의 실수로 주어졌을 때<br>
각 별들을 이어 '별자리'를 만드는 **최소 비용**을 구하는 문제<br>

## 풀이 방법

이전에 보았던 MST 계열의 문제이다<br>

따라서 문제의 유의점은 다음과 같다<br>

- 주어지는 것은 '실수'이며, 거리를 구해야 한다<br>
  따라서 `Sqrt((y1 -y2)^2 + (x1-x2)^2)` 를 통해<br>
  거리를 구함 (**피타고라스**)<br>

- 출력을 2자릿수만 해야 하므로<br>
  `fixed`와 `setPrecision`을 통해 고정<br>

- 주어지는 것이 별들의 **좌표**이기에<br>
  가능한 '모든 별'의 좌표를 구한 후<br>
  정렬시켜 구한다<br>
  (다행히 n이 1000개 이하라 이 방식을 통해<br>
  풀 수 있다)<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include <iomanip>
#include <math.h>

using namespace std;

struct starInfo
{
	double y, x;
};

struct starEdgeInfo
{
	int s1, s2;
	double distance;
};

double GetDistance(starInfo& a, starInfo& b)
{
	return sqrt((a.y - b.y) * (a.y - b.y) + (a.x - b.x) * (a.x - b.x));
}

int FindParents(vector<int>& pv, int x)
{
	if (pv[x] == x)
		return x;

	return pv[x] = FindParents(pv, pv[x]);
}

bool Union(vector<int>& pv, int a, int b)
{
	a = FindParents(pv, a);
	b = FindParents(pv, b);

	if (a == b)
		return false;

	pv[a] = b;
	return true;
}

int main()
{
	int n;
	cin >> n;
	vector<starInfo> infoVec(n);
	vector<int> parents(n);
	for (int i = 0; i < n; i++)
	{
		cin >> infoVec[i].x;
		cin >> infoVec[i].y;

		parents[i] = i;
	}

	vector<starEdgeInfo> edgeInfo;

	for (int i = 0; i < n; i++)
	{
		for (int j = i + 1; j < n; j++)
		{
			double dis = GetDistance(infoVec[i], infoVec[j]);
			edgeInfo.push_back({ i,j,dis });
		}
	}

	sort(edgeInfo.begin(), edgeInfo.end(), []
	(const starEdgeInfo& a, const starEdgeInfo& b)
		{
			return a.distance < b.distance;
		});

	double answer = 0;

	for (const starEdgeInfo& e : edgeInfo)
	{
		if (Union(parents, e.s1, e.s2))
		{
			answer += e.distance;
		}
	}

	cout << fixed << setprecision(2) << answer;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/1d842f3d-dd5e-4ff1-8404-75d7d91d87cc)](https://github.com/user-attachments/assets/1d842f3d-dd5e-4ff1-8404-75d7d91d87cc){: .image-popup}<br>

만약 n의 수가 굉장히 많았다면<br>
문제가 더 까다로웠을 것이다<br>

코테에서 본 문제가 아마 이랬던것 같은데<br>
그때는 먼저 좌표를 '정렬'한 후<br>
근처의 녀석들끼리 거리를 구하는 방식을 통해<br>
간선의 개수를 어느정도 줄여놓는 것이 효과적일 것 같다<br>
(분할 정복을 통해 일정 범위 내에 존재하는 녀석들끼리?)<br>
(굳이 그게 아니더라도 반복문을 돌릴때 범위를 임의로 지정하면 된다)<br>