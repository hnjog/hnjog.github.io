---
title: "백준 Gold 3 텀 프로젝트"
date : "2025-10-09 10:30:00 +0900"
last_modified_at: "2025-10-09T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DFS
  - 큐
---

## 텀 프로젝트 (백준 Gold 3)
<https://www.acmicpc.net/problem/9466><br>

조별과제에서 원하는 사람들끼리만<br>
조를 짜게 한 후, 조를 짜지 못한 사람들을 구하는 문제<br>

- 자기 자신만을 원하는 1명 조도 가능<br>
- s1 -> s2 ... sn -> s1 으로 끝나야 함<br>

## 풀이 방법

dfs를 통해 자기 자신이 나올때까지<br>
탐색하면 풀 수 있는 문제라고 생각하였다<br>

- 그렇기에 flag와 자기 자신을 미리 기록한 후<br>
  dfs + 백트래킹으로 검사하는 방식을 취하였다<br>

## 첫 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

bool dfs(vector<int>& v, int now, int flag, int start)
{
	if (v[now] == flag)
	{
		if(now == start)
			return true;

		return false;
	}

	if (v[now] < 0)
		return false;

	int origin = v[now];
	v[now] = flag;

	if (dfs(v, origin, flag,start))
		return true;

	v[now] = origin;
	return false;
}

int main()
{
	cin.tie(nullptr);
	ios::sync_with_stdio(false);

	int t;
	cin >> t;
	while (t > 0)
	{
		int n;
		cin >> n;
		vector<int> wants(n);
		int flag = -1;
		for (int i = 0; i < n; i++)
		{
			cin >> wants[i];
			wants[i]--;
			if (wants[i] == i)
				wants[i] = flag;
		}

		flag--;

		for (int i = 0; i < n; i++)
		{
			if (wants[i] >= 0)
			{
				if (dfs(wants, i, flag,i))
					flag--;
			}
		}

		int ans = 0;
		for (int i = 0; i < n; i++)
			if (wants[i] >= 0)
				ans++;

		cout << ans << '\n';
		t--;
	}

	return 0;
}
```

## 틀린 이유 - 시간 초과

문제는 바로<br>
다음과 같은 경우에 중복 탐색이 지나치다는 것이다<br>

- s1 -> s2 -> ... sn -> s2<br>
  : s1에서 시작하였지만 s2에서 조가 완성되는 경우<br>
    다시 s2에서 시작하여야 함<br>

따라서 문제를 더 자세히 보고<br>
몇 가지 힌트를 더 얻을 수 있었다<br>

- 이미 `'완성된 조'`를 가리키는 녀석은 **조를 짤 수 없음**<br>
- 따라서 한번 방문한 순간 `조 완성 가능/ 불가능`이 정해지기에 재방문 여부 x<br>

그렇기에 백트래킹은 적용할 필요가 없다<br>

또한, 위의 경우를 막기 위하여<br>
Queue를 사용하기로 하였다<br>

- Queue로 방문한 노드의 index를 집어넣어 보관<br>
- 이미 방문한 노드를 재방문 한 경우<br>
  Queue를 돌며 Front와 현재를 비교<br>
  (맞을때까지 pop 하며 반복)<br>
- 마지막으로 정답 개수에서 Queue의 사이즈를 빼줌<br>

## 최종 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>

using namespace std;

void dfs(vector<int>& v, int now, vector<bool>& visit, int& ans, queue<int>& visitQ)
{
	if (visit[now] == true)
	{
		while (visitQ.empty() == false &&
			visitQ.front() != now)
		{
			visitQ.pop();
		}
		ans -= visitQ.size();
		return;
	}

	visit[now] = true;
	visitQ.push(now);
	dfs(v, v[now], visit, ans, visitQ);

	return;
}

int main()
{
	cin.tie(nullptr);
	ios::sync_with_stdio(false);

	int t;
	cin >> t;
	while (t > 0)
	{
		int n;
		cin >> n;
		vector<int> wants(n);
		vector<bool> visit(n);

		int ans = n;

		for (int i = 0; i < n; i++)
		{
			cin >> wants[i];
			wants[i]--;
			if (wants[i] == i)
			{
				visit[i] = true;
				ans--;
			}
		}

		for (int i = 0; i < n; i++)
		{
			if (visit[i] == false)
			{
				queue<int> q;
				dfs(wants, i, visit, ans, q);
			}
		}

		cout << ans << '\n';
		t--;
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/e2714519-9fb7-4c69-a0f1-bfab7fbd986b)](https://github.com/user-attachments/assets/e2714519-9fb7-4c69-a0f1-bfab7fbd986b){: .image-popup}<br>

처음에는 간단한 문제라 생각하였지만<br>
시간 초과에 부딪히며 생각할 거리를 주는 좋은 문제라 느꼈다<br>