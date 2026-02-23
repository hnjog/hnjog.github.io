---
title: "백준 Gold 3 괄호 추가하기"
date : "2026-02-23 12:00:00 +0900"
last_modified_at: "2026-02-23T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 구현
  - 백트래킹
---

## 괄호 추가하기 (백준 Gold 3)
<https://www.acmicpc.net/problem/16637><br>

길이가 N인 수식이 주어졌을때<br>
괄호를 적절히 씌워 가장 큰 수치의 값을 구하기<br>

*문제의 유의할 점*<br>
- 괄호 안엔 '하나'의 연산자만을 허용<br>
- '연산자 우선순위'는 동일하기에 왼쪽부터 실행<br>
- 0~9까지의 수만 들어오며, 연산자는 '+ , - , *' 만 주어짐<br>

## 풀이 방법  

중간 결과를 통해 분기수를 줄일 수 없으므로<br>
'백트래킹' 계열의 문제라 생각하였다<br>

*생각해볼 점들*<br>
- 데이터를 나누는 방식?<br>
  - 먼저 정수 타입/ 연산자로 나눔<br>
  - 정수 타입 개수가 '연산자'보다 1개 많음<br>

- 사용할 자료구조?<br>
  - '추가/제거'가 원할한 자료구조여야 함<br>
  - '정수부'와 '연산부'를 별도의 vector로 나눈 후 관리<br>
  - Oringin 정수 / 연산 , 백트래킹용 정수 / 연산 으로 나누기<br>

- 백트래킹 분기?<br>
  - 그냥 넘기기<br>
    - Origin 정수부의 +1 인덱스의 수치를 현재 백트래킹용 정수부 벡터에 넣기<br>
	- 백트래킹용 연산부에 origin 것 담기<br>
  - 현재 OP로 계산하기<br>
    - 정수부 계산<br>
	  : 백트래킹 정수부의 '뒤'를 보관한 후<br>
	    정수부의 다음 인덱스와 '현재 연산자'를 진행<br>
	- 연산을 이미 완료했으므로 연산자 저장하지 않음<br>
	- 다만 '연속 괄호'연산은 불가능하므로 bool 변수를 확인<br>

## 제출 코드

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<limits.h>

using namespace std;

long maxV = INT_MIN;

long AllCalc(vector<int>& vv, vector<char>& ops)
{
	long ret = vv[0];

	for (int i = 0; i < ops.size(); i++)
	{
		char c = ops[i];

		switch (c)
		{
		case '+':
		{
			ret += vv[i + 1];
		}
			break;
		case '-':
		{
			ret -= vv[i + 1];
		}
		break;
		case '*':
		{
			ret *= vv[i + 1];
		}
		break;
		}
	}

	return ret;
}

long Calc(int v1, int v2, char op)
{
	long ret = v1;
	switch (op)
	{
	case '+':
	{
		ret += v2;
	}
	break;
	case '-':
	{
		ret -= v2;
	}
	break;
	case '*':
	{
		ret *= v2;
	}
	break;
	}

	return ret;
}

void Recur(const vector<int>& vv, const vector<char>& ops, vector<int>& nowvv, vector<char>& nowops, int opIdx,bool bUse)
{
	int oS = ops.size();
	if (opIdx >= oS)
	{
		long tv = AllCalc(nowvv, nowops);
		maxV = max(maxV, tv);
		return;
	}

	{
		nowvv.push_back(vv[opIdx + 1]);
		nowops.push_back(ops[opIdx]);
		Recur(vv, ops, nowvv, nowops, opIdx + 1,false);
		nowops.pop_back();
		nowvv.pop_back();
	}

	if(bUse == false)
	{
		int o = nowvv.back();
		nowvv.pop_back();
		int t = Calc(o, vv[opIdx + 1], ops[opIdx]);
		nowvv.push_back(t);
		Recur(vv, ops, nowvv, nowops, opIdx + 1,true);
		nowvv.pop_back();
		nowvv.push_back(o);
	}
}

int main()
{
	int n;
	cin >> n;

	string v;
	cin >> v;

	vector<int> vv;
	vector<char> ops;

	for (char c : v)
	{
		if (c == '+' ||
			c == '-' ||
			c == '*')
		{
			ops.push_back(c);
		}
		else
		{
			vv.push_back(c - '0');
		}
	}

	vector<int> tvv;
	tvv.reserve(vv.size());
	tvv.push_back(vv[0]);
	vector<char> tops;
	tops.reserve(ops.size());

	Recur(vv, ops, tvv, tops, 0,false);

	cout << maxV;

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/10d83201-2f14-45ff-87cf-4dfb11eefad1)](https://github.com/user-attachments/assets/10d83201-2f14-45ff-87cf-4dfb11eefad1){: .image-popup}<br>

- 구현 접근에 꽤 시간을 쏟았지만 그래도 만족스럽게 풀렸다<br>

