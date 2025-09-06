---
title: "백준 Gold 4 전화번호 목록"
date : "2025-09-06 11:00:00 +0900"
last_modified_at: "2025-09-06T11:00:00"
categories:
  - 코딩 테스트
tags:
  - 문자열
  - 트라이
---

## 전화번호 목록 (백준 Gold 4)
<https://www.acmicpc.net/problem/5052><br>

전화번호 목록이 주어지고<br>
그 번호들이 일관성이 있는지 없는지를 구하는 문제<br>

- 일관성이 있으려면<br>
  '특정한 번호'가 '다른 번호'의 접두어가 아니여야 함<br>

ex)<br>
911, 976 이라는 번호가 주어졌는데<br>
9114 라는 번호가 주어지면 911까지 누른 순간<br>
실패처리<br>

## 접근 방식

각 문자들에 대한 '트라이'를 구현함으로서<br>
풀 수 있겠다고 느꼈다<br>

[![Image](https://github.com/user-attachments/assets/11affd36-a691-463a-a122-62a4236368c4)](https://github.com/user-attachments/assets/11affd36-a691-463a-a122-62a4236368c4){: .image-popup}<br>

(출처 : 나무위키(트라이))<br>

트라이는<br>
트리 구조의 응용으로서<br>
'각 단어'의 해당하는 부분을 '다음 단어'와 연결시킴으로서<br>
'단어' 자체를 트리 방식으로 저장하는 자료구조이다<br>

따라서<br>
9 -> 1 -> 1(끝)<br>
이런식으로 문자를 저장하고<br>

차후에 들어오는 문자열이<br>
9 -> 1 -> 1 (이미 끝인데? 실패처리!)<br>

하는 것이 이번 문제에 대한 접근방식이다<br>


## 첫 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<map>

using namespace std;

class node
{
public:
	node()
	{
		isEnd = false;
	}

	bool isInsert(string& str,int idx)
	{
		if (isEnd)
			return false;

		char c = str[idx];
		int sSize = str.size();
		
		if (next.find(c) == next.end())
		{
			next[c].now = c;
			if (idx == sSize - 1)
			{
				next[c].isEnd = true;
				return true;
			}
		}

		return next[c].isInsert(str, idx + 1);
	}
private:
	bool isEnd;
	char now;
	map<char, node> next;
};

int main()
{
	int t;
	cin >> t;
	while (t > 0)
	{
		int n;
		cin >> n;
		vector<string> strs(n);
		for (int i = 0; i < n; i++)
			cin >> strs[i];

		node root;
		bool isSuccess = true;
		
		for (string& str : strs)
		{
			if (root.isInsert(str,0) == false)
			{
				isSuccess = false;
				break;
			}
		}

		if (isSuccess)
			cout << "YES" << '\n';
		else
			cout << "NO" << '\n';
		t--;
	}

	return 0;
}
```

## 틀린 이유
자꾸 메모리 초과가 발생하거나<br>
세그먼트 falut 에러가 발생하였다<br>

## 수정 방법

- 메모리 초과는 node를 *로 잡아 new 할당을 해주는 것으로 수정해보았다<br>

- 세그먼트 falut는 처음에 vector<string>(10001)로 잡아두었지만<br>
  반복문을 여전히 (for auto) 방식으로 돌고 있기에 해당 부분을 수정<br>

- 또한 '길이가 긴' 코드가 먼저 들어오는 경우<br>
  현재의 tree insert 방식으로는<br>
  idx가 런타임 에러를 발생할 수 있기에<br>
  sort를 통하여 사전순 정렬을 하며<br>
  동시에 길이가 짧은 문자열이 먼저 들어오도록 수정하였다<br>

## 최종 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
#include<algorithm>

using namespace std;

class node
{
public:
	node()
	{
		isEnd = false;
	}

	~node()
	{
		for (auto& p : next)
		{
			delete p.second;
		}
	}

	bool isInsert(string& str, int idx)
	{
		if (isEnd)
			return false;

		char c = str[idx];
		int sSize = str.size();
		// 정렬 안하면 길이가
		// 높은것이 먼저 들어와 터질 수 있음
		if (next.find(c) == next.end())
		{
			next[c] = new node();
			next[c]->now = c;
			if (idx == sSize - 1)
			{
				
				next[c]->isEnd = true;
				return true;
			}
		}

		return next[c]->isInsert(str, idx + 1);
	}
private:
	bool isEnd;
	char now;
	unordered_map<char, node*> next;
};

int main()
{
	int t;
	cin >> t;
	
	while (t > 0)
	{
		int n;
		cin >> n;
		vector<string> strs(n);
		for (int i = 0; i < n; i++)
			cin >> strs[i];

		// 정렬함으로서, 같은 사전순이라도 길이 짧은게 앞으로 오도록 한다
		sort(strs.begin(), strs.end());

		node root;
		bool isSuccess = true;

		for (int i = 0; i < n;i++)
		{
			if (root.isInsert(strs[i], 0) == false)
			{
				isSuccess = false;
				break;
			}
		}

		if (isSuccess)
			cout << "YES" << '\n';
		else
			cout << "NO" << '\n';

		t--;
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/072e500e-cec4-4424-ac48-7c4bbb5ec2d4)](https://github.com/user-attachments/assets/072e500e-cec4-4424-ac48-7c4bbb5ec2d4){: .image-popup}<br>

트라이를 오랜만에 구현해보았고<br>
길이가 높은것이 먼저들어올 경우를 생각하지 못하여<br>
해당 부분에 대하여 다소 삽질을 하였다<br>