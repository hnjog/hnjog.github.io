---
title: "백준 Gold 3 의리 게임"
date : "2025-12-18 10:30:00 +0900"
last_modified_at: "2025-12-18T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 분리 집합
  - 재귀
---

## 의리 게임 (백준 Gold 3)
<https://www.acmicpc.net/problem/28424><br>

의리 게임을 하려는 사람의 수 N, 실행할 질의 q가 주어진다<br>

- n명의 사람은 각각 주량이 존재하며<br>
  더 이상 술을 마실 수 없는 경우<br>
  다음 사람에게 술을 넘긴다<br>
  (마지막 사람은 넘길 사람이 없기에 술을 버린다)<br>

- q는 1과 2로 주어진다<br>
  - 1일 때,<br>
    i와 x가 주어지며<br>
	i번쨰 사람부터 의리게임을 진행하여<br>
	x 양의 술을 마시기 시작한다<br>
  - 2일때<br>
    i가 주어지며, i 번째 사람이 마신 술의 양을 출력한다<br>

## 풀이 방법  

분리 집합을 응용하면 풀 수 있는 문제이다<br>

**배열 정의**<br>

- pv[i]<br>
 : 현재 사람이 다음 술을 건네줄 사람을 저장한 배열<br>
   (마지막 사람은 자기 자신을 가리킴)<br>  

- nv[i]<br>
  : 현재 사람이 마신 술의 양<br>
    (cv[i]를 넘을 수 없음)

- cv[i]<br>
  : 현재 사람이 마실 수 있는 최대 주량<br>

**함수 정의**<br>

- FindParent<br>
  : 현재 인덱스 + 배열 들을 이용하여<br>
    다음 술 마실 사람의 위치를 반환<br>
  - 원래 '다음'사람 이 마실 수 있다면 그 사람에게 건네줌<br>
  - 아니라면, 다음 위치의 사람을 찾은 후 대입하여 반환<br>
  
- drink<br>
  : 현재 마실 사람 + 마실 양과 배열등을 통해<br>
   마실 양을 점차 줄여나가는 재귀 함수<br>
   위의 FindParent를 통해<br>
   다음 마실 사람을 찾으며<br>
   마지막 사람은 남기더라도 바로 return<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int FindParent(vector<int>& pv, vector<int>& nv, vector<int>& cv, int now)
{
	// 가장 끝
	if (pv[now] == now)
		return now;

	int v = pv[now];

	// 아직 술 마실수 있음
	if (nv[v] < cv[v])
		return v;

	return pv[now] = FindParent(pv, nv, cv, v);
}

void drink(int idx, int lit, vector<int>& pv, vector<int>& nv, vector<int>& cv)
{
	if (idx >= pv.size())
		return;

	int canDrink = cv[idx] - nv[idx];

	// 전부 마실 수 있음
	if (lit <= canDrink)
	{
		nv[idx] += lit;
		return;
	}

	// 전부 마실 수 없는 상태
	nv[idx] = cv[idx];
	int remain = lit - canDrink;

	int n = FindParent(pv, nv, cv, idx);

	if (n == idx)
		return;

	drink(n,remain, pv, nv, cv);
}

int main()
{
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int n, q;
	cin >> n >> q;

	// 각 노드들은 술을 다음으로 넘길 녀석을 인덱스로 지정
	vector<int> pv(n);
	for (int i = 0; i < n - 1; i++)
	{
		pv[i] = i + 1;
	}

	pv[n - 1] = n - 1;

	vector<int> nv(n), cv(n);
	for (int i = 0; i < n; i++)
	{
		cin >> cv[i];
	}

	for (int i = 0; i < q; i++)
	{
		int o;
		cin >> o;
		if (o == 1)
		{
			int id, m;
			cin >> id >> m;
			id--;
			drink(id, m, pv, nv, cv);
		}
		else
		{
			int id;
			cin >> id;
			id--;
			cout << nv[id] << '\n';
		}
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/77ebfd0e-237d-43d0-9f26-772e195827cc)](https://github.com/user-attachments/assets/77ebfd0e-237d-43d0-9f26-772e195827cc){: .image-popup}<br>

처음에는 -1을 부모 배열(pv)에 넣어<br>
'만취' 상태를 표현하려 하였으나<br>
애초에 pv가 '만취한 경우'에 다음 idx를 가리키도록 수정하고 있었음을 깨달았다<br>

- 또한 pv에 -1을 대입하는 구조 자체가<br>
  부모 배열을 망가뜨리는 일환이 될 수 있기에<br>
  제거하는 것이 더 안정적이라 생각하였다<br>
