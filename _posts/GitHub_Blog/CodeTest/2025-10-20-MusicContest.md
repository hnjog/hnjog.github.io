---
title: "백준 Gold 3 음악프로그램"
date : "2025-10-20 10:30:00 +0900"
last_modified_at: "2025-10-20T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 위상정렬
---

## 음악프로그램 (백준 Gold 3)
<https://www.acmicpc.net/problem/2623><br>

주어지는 '여러 순서'들을 유지하여<br>
최종 순서를 정하여 출력하는 문제<br>

- 모든 순서를 지키지 못하는 경우는 0 출력<br>
- 답이 여러개인 경우는 아무거나 출력<br>

## 풀이 방법

여러 순서들이 주어지고 그 순서를 `유지`한채<br>
출력이 가능한지를 묻고 있음<br>

- 하나의 순서에서 '그 다음'으로 이어지는 경우<br>
  반드시 이전에 그 수가 존재해야 함<br>
  (ex - 1 3 인 경우 1 ... 3 .. 이여야 함)<br>
  
- 그렇기에 처음에는 이전에 수가 없는 수들이 와야 하며<br>
  반대로 '이전'에 수가 존재한다면 그들이 먼저 와야<br>
  현재 수가 위치할 수 있게 됨<br>

어디서 본 풀이 방식인데?<br>
'차수'를 이용한 순서를 정하는 알고리즘?<br>

### 위상정렬

위상정렬 문제의 경우는<br>
입력 차수(Degree)를 통하여 푸는 것이 대표적이다<br>

1. degree를 0으로 초기화하며, 주어지는 Data를 통하여 차수 설정<br>
2. degree가 0인 녀석들을 queue에 넣어 주면서 시작<br>
3. queue에서 하나씩 pop하며, 결과물 vector에 저장<br>
4. 이후, 자신이 가진 '다음' Data의 Degree를 빼주며, 그 값이 0이면 큐에 추가!<br>
5. queue가 빌때까지 3~4과정을 반복
6. 마지막에 결과물 Vector의 Size가 초기에 주어진 사람수와 맞는지를 비교<br>
   - 맞다면 그대로 출력<br>
   - 아니라면 {0}으로 초기화<br>

## 제출 코드

```cpp
#include<iostream>
#include<queue>
#include<vector>

using namespace std;

struct node
{
	int idx;
	int degree;
	vector<int> nextIdx;
};

vector<int> Func(vector<node>& nv)
{
	queue<node> q;
	for (auto& n : nv)
	{
		if (n.degree == 0)
			q.push(n);
	}

	vector<int> ret;

	while (q.empty() == false)
	{
		node& n = q.front();
		ret.push_back(n.idx + 1);

		for (int next : n.nextIdx)
		{
			nv[next].degree--;
			if (nv[next].degree == 0)
				q.push(nv[next]);
		}
		q.pop();
	}

	if (ret.size() < nv.size())
		ret = { 0 };

	return ret;
}

int main()
{
	int n, m;
	cin >> n >> m;

	vector<node> nv(n);
	for (int i = 0; i < n; i++)
	{
		nv[i].idx = i;
		nv[i].degree = 0;
	}

	for (int i = 0; i < m; i++)
	{
		int t;
		cin >> t;

		int prev = -1;
		for (int j = 0; j < t; j++)
		{
			int t2;
			cin >> t2;
			t2--;
			if (prev != -1)
			{
				nv[prev].nextIdx.push_back(t2);
				nv[t2].degree++;
			}
			prev = t2;
		}
	}

	vector<int> ret = Func(nv);

	for (int r : ret)
	{
		cout << r << '\n';
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/9020b4de-f2fc-4785-a951-1057921c892f)](https://github.com/user-attachments/assets/9020b4de-f2fc-4785-a951-1057921c892f){: .image-popup}<br>
