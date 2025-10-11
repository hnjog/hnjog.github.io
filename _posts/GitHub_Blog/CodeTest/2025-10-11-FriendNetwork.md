---
title: "백준 Gold 2 친구 네트워크"
date : "2025-10-11 10:30:00 +0900"
last_modified_at: "2025-10-11T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 맵
  - 분리 집합
---

## 친구 네트워크 (백준 Gold 2)
<https://www.acmicpc.net/problem/4195><br>

두 명의 친구 관계가 주어졌을 때<br>
`해당 친구 그룹`의 **총 친구수**를 출력하는 문제<br>

## 풀이 방법

주어지는 관계가 `'같은 친구 관계'`인지를 파악하기 위해<br>
`유니온 파인드 (분리 집합)`을 사용하면 풀 수 있을것 같았다<br>

따라서<br>
다만 String 데이터로 주어지기에 map을 사용하여<br>
<이름, 부모가 되는 친구><br>
로 데이터를 구분하였다<br>

- 다만 `'해당 그룹'의 친구수`를 어떻게 구할까 싶었다<br>
  일반적인 for문 탐색은 지나치게 느릴 것 같았다<br>

- 그렇기에 pair<string,int>를 통하여<br>
  추가적으로 '부모인 친구' 와 그룹 개수 를 같이 관리하도록 하였다<br>

- 처음에는 <자기 자신, 1>을 데이터로 가지면서<br>
  Union을 통해 그룹 개수를 전달해주는 방식을 통해 문제를 풀었다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<unordered_map>

using namespace std;

string FindParent(unordered_map<string, pair<string, int>>& fmap, string x)
{
	if (fmap[x].first == x)
		return x;

	return fmap[x].first = FindParent(fmap, fmap[x].first);
}

bool Union(unordered_map<string, pair<string, int>>& fmap, string a,string b)
{
	a = FindParent(fmap, a);
	b = FindParent(fmap, b);

	if (a == b)
		return false;

	fmap[a].first = b;
	fmap[b].second += fmap[a].second;
	fmap[a].second = 1;
	return true;
}

int main()
{
	
	int t;
	cin >> t;
	while (t > 0)
	{
		int f;
		cin >> f;

		unordered_map<string, pair<string,int>> fmap;

		for (int i = 0; i < f; i++)
		{
			string f1, f2;
			cin >> f1 >> f2;
			if (fmap.find(f1) == fmap.end())
				fmap[f1] = make_pair(f1,1);
			if (fmap.find(f2) == fmap.end())
				fmap[f2] = make_pair(f2, 1);

			Union(fmap, f1, f2);
			cout << fmap[FindParent(fmap, f1)].second << '\n';
		}

		t--;
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/8559243a-e9c2-4594-a912-7f04774245dd)](https://github.com/user-attachments/assets/8559243a-e9c2-4594-a912-7f04774245dd){: .image-popup}<br>

풀이 방법에 대하여 곰곰히 생각하였고<br>
다행히 생각한대로 문제를 풀 수 있었다<br>
