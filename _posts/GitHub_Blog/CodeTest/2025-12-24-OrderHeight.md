---
title: "백준 Gold 4 키 순서"
date : "2025-12-24 10:30:00 +0900"
last_modified_at: "2025-12-24T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 플로이드-워셜
---

## 키 순서 (백준 Gold 4)
<https://www.acmicpc.net/problem/2458><br>

학생 n명과 그 키를 비교한 결과 m이 주어진다<br>
m은 a,b 가 주어지며, 이는 a가 b보다 작다는 뜻이다<br>

이러한 데이터가 주어질 때, '자신'의 '키 순서'를 정확히 알 수 있는<br>
학생의 수를 구하는 문제<br>

[![Image](https://github.com/user-attachments/assets/8633a57b-e585-4a0e-aaf1-7dce210c2bc5)](https://github.com/user-attachments/assets/8633a57b-e585-4a0e-aaf1-7dce210c2bc5){: .image-popup}<br>

위의 예시를 보면<br>
'4'번째 학생만이 자신의 정확한 키 순서를 알 수 있음<br>

- 1,3,5는 자신보다 키가 작음<br>
- 2,6 은 자신보다 키가 큼<br>

## 풀이 방법  

처음에는 '순서'를 따라 푸는 '위상정렬' 계열의 문제인줄 알았으나<br>
조금 더 생각해보니 현재 '자신'의 위치를 파악하는 계열의 문제라 생각하였다<br>

- 자신의 순서? -> 자신의 위치를 파악<br>
- 자신의 위치를 파악? -> 다른 위치들에서 본 '자신의 위치'?<br>
- 모든 위치에 대하여 '현재 위치'를 파악할 수 있나?<br>
  -> 플로이드-워셜 계열?<br>

다만 최단거리를 아는 것이<br>
도움이 될까...? 싶었는데<br>

조금 더 생각을 해보았다<br>

- 주어진 '비교'를 '단방향'으로 진행<br>
  (위의 예시를 보자면 3번에서 1,5를 알수 없어야 하므로)<br>
- 이런 edge들을 기반으로 플로이드-워셜 진행<br>
  - 단방향이기에, '자신'을 가리키는 것도 확인 (maps[i][j],maps[j][i])<br>
  - 둘중 하나라도 초기값이 아니라면 해당 노드에서 자신의 거리를 알 수 있음<br>
  - maps[i][j]와 maps[j][i]가 둘 다, 초기값인 경우<br>
    해당 위치에서 자신의 위치를 알 수 없다는 것<br>
	- 즉, i는 자신의 정확한 '키 순서'를 모름!<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

const int initV = 1e9;

int main()
{
	int n, m;
	cin >> n >> m;
	vector<vector<int>> maps(n, vector<int>(n, initV));

	for (int i = 0; i < n; i++)
		maps[i][i] = 0;

	for (int i = 0; i < m; i++)
	{
		int a, b;
		cin >> a >> b;
		a--;
		b--;
		maps[a][b] = 1;
	}

	for (int k = 0; k < n; k++)
	{
		for (int i = 0; i < n; i++)
		{
			for (int j = 0; j < n; j++)
			{
				maps[i][j] = min(maps[i][j], maps[i][k] + maps[k][j]);
			}
		}
	}

	int count = 0;

	for (int i = 0; i < n; i++)
	{
		bool bFound = true;

		for (int j = 0; j < n; j++)
		{
			if (maps[i][j] == initV &&
				maps[j][i] == initV)
			{
				bFound = false;
				break;
			}
		}

		if (bFound)
			count++;
	}

	cout << count;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/dbce0bde-7f9e-42d5-901a-38c1135f03da)](https://github.com/user-attachments/assets/dbce0bde-7f9e-42d5-901a-38c1135f03da){: .image-popup}<br>

플로이드-워셜 문제에서<br>
유의할 점은 다음과 같다!<br>

- i : 시작점<br>
- j : 목표점<br>
- k : 경유점<br>

따라서<br>
k를 '가장 바깥쪽'의 반복문으로 빼야<br>
'모든 루트'에서 0,1,...,n 까지<br>
순서대로 갱신하기 때문<br>

- k : 0 일때 i,j를 한번씩 돌림<br>
  k : 1 일때 i,j를 한번씩 돌림 <- 이때 위의 0번째 노드에 대한 데이터를 재사용함<br>
