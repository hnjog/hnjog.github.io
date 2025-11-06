---
title: "백준 Gold 5 소셜 네트워킹 어플리케이션"
date : "2025-11-06 10:30:00 +0900"
last_modified_at: "2025-11-06T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
---

## 소셜 네트워킹 어플리케이션 (백준 Gold 5)
<https://www.acmicpc.net/problem/7511><br>

특정한 친구 수 N 과<br>
친구 관계에 대한 설명이 k개 주어졌을 때<br>
두 사람이 친구인지 묻는 M개의 질문에 답하는 문제<br>

- 친구라면 1, 아니라면 0을 출력<br>

## 풀이 방법

분리 집합 문제로서<br>
N개의 Parent Vector를 만들고<br>
K개에 대하여 서로 Union 해준 후<br>
M개의 쌍에 대하여 서로 '같은 부모'를 가지는 지<br>
검사하면 풀 수 있다!<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

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

int main()
{
	ios_base::sync_with_stdio(false);
	cin.tie(nullptr);

	int t;
	cin >> t;
	int idx = 1;
	while (idx <= t)
	{
		int n, k;
		cin >> n >> k;

		vector<int> pv(n);
		for (int i = 0; i < n; i++)
			pv[i] = i;

		for (int i = 0; i < k; i++)
		{
			int a, b;
			cin >> a >> b;
			Union(pv, a, b);
		}

		int m;
		cin >> m;

		cout << "Scenario " << idx << ":" << '\n';
		for (int i = 0; i < m; i++)
		{
			int a, b;
			cin >> a >> b;
			if (FindParent(pv, a) == FindParent(pv, b))
			{
				cout << 1 << '\n';
			}
			else
			{
				cout << 0 << '\n';
			}
		}

		cout << '\n';

		idx++;
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/b784db87-61b3-48a4-97c0-b361ca5fb3d1)](https://github.com/user-attachments/assets/b784db87-61b3-48a4-97c0-b361ca5fb3d1){: .image-popup}<br>

시간 초과인 부분은<br>
아래의 입력 부분에 대한 처리를 빠트려서 발생하였다<br>

```cpp
ios_base::sync_with_stdio(false);
cin.tie(nullptr);
```