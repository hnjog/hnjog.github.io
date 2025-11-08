---
title: "백준 Gold 4 민서의 응급 수술"
date : "2025-11-08 10:30:00 +0900"
last_modified_at: "2025-11-08T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
  - 트리
---

## 민서의 응급 수술 (백준 Gold 5)
<https://www.acmicpc.net/problem/20955><br>

N개의 노드와 M개의 간선 정보가 주어졌을때<br>
N개의 노드를 이어 '사이클'이 없는 트리를 만드는 데 필요한<br>
작업 횟수를 구하는 문제<br>

- 간선을 생성하거나 제거하는데에는 1의 횟수로 센다<br>

## 풀이 방법

'트리 구조'를 만들지만<br>
동시에 '분리 집합'을 이용하여 '사이클'을 판별하는 문제이다<br>

그렇기에 다음과 같은 방식으로 풀 수 있다<br>

- 기본적으로 총 필요한 간선은 N - 1 개<br>
  따라서 초기값을 이로 설정<br>
  (작업할 양)<br>

- M 개의 간선이 들어왔을때, 다음과 같이 설정<br>
  - 해당 간선이 '사이클'을 이루는 간선이 아니라면<br>
    작업량이 하나 줄어든 것이므로 -1<br>
  - 해당 간선이 '사이클'을 만드는 간선이라면<br>
    작업량이 하나 늘어난 것이므로 +1<br>

이후 그 결과물을 출력하면 된다!<br>

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
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int n, m;
	cin >> n >> m;
	
	vector<int> pv(n);
	for (int i = 0; i < n; i++)
		pv[i] = i;

	int ans = n - 1;

	for (int i = 0; i < m; i++)
	{
		int t1, t2;
		cin >> t1 >> t2;
		t1--;
		t2--;
		if (Union(pv, t1, t2))
			ans--;
		else
			ans++;
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/eb835817-1cf3-43b7-a94d-86b2f078f625)](https://github.com/user-attachments/assets/eb835817-1cf3-43b7-a94d-86b2f078f625){: .image-popup}<br>

처음에는 단순히 집합 여부만 확인하면 되는 줄 알았으나<br>
'사이클'이 존재해야 하지 않아야 하기에<br>
그에 따른 별도적인 처리가 필요하단 걸 나중에 깨달았다<br>
