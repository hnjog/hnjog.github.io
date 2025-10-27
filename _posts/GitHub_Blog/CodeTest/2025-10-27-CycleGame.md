---
title: "백준 Gold 4 사이클 게임"
date : "2025-10-27 10:30:00 +0900"
last_modified_at: "2025-10-27T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
  - Disjoint Set
---

## 사이클 게임 (백준 Gold 4)
<https://www.acmicpc.net/problem/20040><br>

평면의 점 n과 진행한 선 그리기 개수 m이 주어질 때<br>
m번 그렸을때 도중 사이클이 발생하는지 확인하고<br>
그 발생 타이밍을 출력하는 문제<br>

- 다만 발생하지 않았다면 0을 출력<br>

## 풀이 방법

`분리 집합`을 이용하여 풀 수 있는 문제<br>
실제로 그래프를 그릴 필요는 없음<br>
(비용에 대한 별도의 처리가 들어가지 않으며<br>
 Edge라 하기에는 다소 작은 개념)<br>

- m번 순회하며<br>
  주어지는 정보를 통하여 Union하며 체크<br>

- 처음 사이클이 발생한 경우에만 저장<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

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
	int n, m;
	cin >> n >> m;

	vector<int> pv(n);
	for (int i = 0; i < n; i++)
		pv[i] = i;

	int ans = 0;

	for (int i = 0; i < m; i++)
	{
		int t1, t2;
		cin >> t1 >> t2;
		if (ans == 0 &&
			Union(pv, t1, t2) == false)
		{
			ans = i + 1;
		}
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/d0abf2e1-3033-4e85-adec-e06fe86ee4d4)](https://github.com/user-attachments/assets/d0abf2e1-3033-4e85-adec-e06fe86ee4d4){: .image-popup}<br>

전형적인 분리 집합에 대한 문제<br>
최근에는 MST와 분리 집합 계열의 문제를 반복적으로 풀며<br>
감을 익히고 있다<br>