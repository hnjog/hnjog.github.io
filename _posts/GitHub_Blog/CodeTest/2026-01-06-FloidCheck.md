---
title: "백준 Gold 2 궁금한 민호"
date : "2026-01-06 10:30:00 +0900"
last_modified_at: "2026-01-06T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 플로이드-워셜
  - 그래프 이론
  - 최단 경로
---

## 궁금한 민호 (백준 Gold 2)
<https://www.acmicpc.net/problem/1507><br>

특정한 N개의 도시가 주어지고<br>
그 다음 NxN 개의 경로 소모값이 주어진다<br>

해당 경로 소모값을 기반으로<br>
'원래 존재하였던 도로' 비용의 총합을 구하는 문제<br>

- 만약, 경로 소모값이 '이상'하다면 -1 출력<br>

## 풀이 방법  

플로이드-워셜 or 다익스트라 계열의 문제라고 생각하였다<br>

- dist[i] + w == dist[j] 이라면<br>
  w가 포함되는 간선이 '최소 비용'에 포함<br>
  - 이전에 특정 기준점에서 시작하는 다익스트라 문제에서 힌트를 얻었다<br>

- 플로이드-워셜의 dist[i][j]는<br>
  'i->j'까지의 *최단거리*<br>
  - dist[i][k] + dist[k][j] < dist[i][j] 라면 경로를 갱신하는 알고리즘<br>

- 따라서 `dist[i][k] + dist[k][j] == dist[i][j]` 라면<br>
  dist[i][j]는 '원래 있던 도로'가 아니라는 뜻!<br>

- 그렇기에 '원래 있던 도로'가 아닌 녀석들은 대상에서 제외하고<br>
  원래 있던 도로로 '체크'된 녀석들의 합을 구하면 됨!<br>

*다만 문제가 하나 존재한다*<br>

- 주어진 N x N 의 그래프가 '플로이드-워셜'의 결과물이라는 '보장'이 없다!<br>

- 그렇기에 `dist[i][k] + dist[k][j] == dist[i][j]` 를 검사하면서<br>
  dist[i][k] + dist[k][j] < dist[i][j] 인지를 검사하고<br>
  이 조건에 들어간다면 '-1'을 출력하는 쪽으로 진행하였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n;
	cin >> n;

	vector<vector<int>> maps(n, vector<int>(n, 0));

	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			cin >> maps[i][j];

	vector<vector<bool>> origin(n, vector<bool>(n, true));

	for (int k = 0; k < n; k++)
	{
		for (int i = 0; i < n; i++)
		{
			if (i == k)
				continue;

			for (int j = 0; j < n; j++)
			{
				if (i == j || j == k)
					continue;

				if (maps[i][k] + maps[k][j] == maps[i][j])
				{
					origin[i][j] = false;
				}
				else if (maps[i][k] + maps[k][j] < maps[i][j])
				{
					cout << -1;
					return 0;
				}
			}
		}
	}

	int ret = 0;

	for (int i = 0; i < n; i++)
	{
		for (int j = i + 1; j < n; j++)
		{
			if (origin[i][j])
			{
				ret += maps[i][j];
			}
		}
	}


	cout << ret;

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/49536094-9681-4fa6-a8c0-83b9cc5d8543)](https://github.com/user-attachments/assets/49536094-9681-4fa6-a8c0-83b9cc5d8543){: .image-popup}<br>

문제의 설명이 다소 모호한 부분이 있다고 생각하지만<br>
반대로 문제의 난이도를 높였다고 생각한다<br>

- 때로는 문제의 설명을 읽어보는 것도 좋지만,<br>
  입력값과 출력값을 보고 문제를 판단하는 것도 하나의 방법임을 알았다<br>
