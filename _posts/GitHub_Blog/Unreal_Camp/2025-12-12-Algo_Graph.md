---
title: "문승현 튜터님 강의 - '알고리즘 : 그래프 탐색과 최단 경로'"
date : "2025-12-12 18:00:00 +0900"
last_modified_at: "2025-12-12T18:00:00"
categories:
  - C++
  - Algorithm
tags:
  - C++
  - Algorithm
  - 그래프
---

# **알고리즘 : 그래프 탐색과 최단 경로 정복하기 (DFS, BFS, 다익스트라)**

지난 강의가 서랍에 데이터를 차곡차곡 정리(정렬)하고 **쏙 뽑아내는(탐색)** 방법이었다면,<br>
오늘은 데이터들이 서로 복잡하게 얽혀있는 세상(그래프)을 다루는 이야기입니다.<br>

친구 관계, 지하철 노선도, 인터넷망, 그리고 게임 속 맵까지.<br>
이렇게 거미줄처럼 연결된 데이터 속에서 우리는 어떻게 길을 잃지 않고 목적지까지 갈 수 있을까요?<br>

### **1. DFS (깊이 우선 탐색, Depth-First Search)**

**1.1 개념 : "일단 끝까지 가본다! 막히면? 되돌아오면 되지."**

DFS는 **호기심 많은 탐험가**의 방식입니다. 미로에 들어갔다고 상상해 보세요. <br>

갈림길이 나왔을 때, 이 탐험가는 고민하지 않고 무조건 한쪽 길을 선택해<br>
**더 이상 갈 수 없을 때까지(바닥을 칠 때까지)** 깊숙이 들어갑니다.<br>

- **행동 패턴:**<br>
    **1.** 갈림길이 나오면 무조건 한쪽(보통 왼쪽이나 순서대로)을 선택해 들어갑니다.<br>
    **2.** 막다른 길(더 이상 갈 곳이 없는 곳)이 나올 때까지 계속 깊게 들어갑니다.<br>
    **3.** 막다른 길을 만나면? 당황하지 않고 **가장 최근의 갈림길로 되돌아옵니다(Backtracking).**<br>
    **4.** 거기서 아직 안 가본 다른 길로 다시 깊게 들어갑니다.<br>

- **핵심 도구:**<br>
    - **스택(Stack) 또는 재귀(Recursion):**<br>
     "되돌아갈 곳"을 기억하기 위해 사용합니다.<br>
     (헨젤과 그레텔의 빵 부스러기처럼, 내가 지나온 경로를 쌓아두는 개념입니다.)<br>

- **시간 복잡도:** O(V + E) (V: 노드 수, E: 간선 수)<br>

### 대표 예제 1 [난이도 하] : 바이러스 (Computers)

**문제 설명:**

어느 날 1번 컴퓨터가 웜 바이러스에 걸렸습니다.<br>
컴퓨터들은 네트워크 상에서 서로 연결되어 있는데, 바이러스는 연결된 선을 타고 전파됩니다.<br>

1번 컴퓨터를 통해 웜 바이러스에 걸리게 되는 컴퓨터의 수는 총 몇 대인지 구하세요.<br>
(단, 1번 컴퓨터 스스로는 카운트에서 제외합니다. 100번 컴퓨터까지 있습니다.)<br>

**입력:**

- 컴퓨터의 수: 7<br>
- 연결 쌍의 수: 6<br>
- 연결 정보: `[[1, 2], [2, 3], [1, 5], [5, 2], [5, 6], [4, 7]]`<br>

**출력:**

- 4 (2번, 3번, 5번, 6번 컴퓨터가 감염됨. 4번과 7번은 1번과 연결이 끊겨 있어 안전함)<br>

**해결 논리:**

1. **그래프 생성:** 연결 정보를 이용해 각 컴퓨터가 누구와 연결되어 있는지 인접 리스트(`adj`)를 만듭니다.<br>
2. **방문 체크:** 이미 감염된 컴퓨터를 또 셀 필요는 없으므로 `visited` 배열을 만듭니다.<br>
3. **DFS 수행:** 1번 컴퓨터부터 시작합니다. 1번과 연결된 컴퓨터를 찾아 들어가고, 거기서 또 연결된 컴퓨터로 **꼬리에 꼬리를 물고** 들어갑니다.<br>
4. 새로운 컴퓨터를 방문할 때마다 카운트를 1씩 늘립니다.<br>

- 정답 코드 예시<br>

```cpp
#include <iostream>
#include <vector>

using namespace std;

// [전역 변수 선언]
// 101의 의미: 컴퓨터 번호가 1번부터 100번까지 들어올 수 있어서,
// 인덱스를 100번까지 쓰기 위해 크기를 101로 넉넉하게 잡은 것입니다.
vector<vector<int>> adj(101); // 컴퓨터 간의 연결 지도 (인접 리스트)
bool visited[101];            // 감염 여부 확인표 (true면 감염됨)
int cnt = 0;                  // 1번 때문에 감염된 컴퓨터 숫자 (정답)

// [DFS 함수 정의]
// cur: 현재 방문해서 검사하고 있는 컴퓨터 번호
void dfs(int cur) {
    // 1. 현재 컴퓨터 감염 처리 (방문 도장 쾅!)
    visited[cur] = true; 
    
    // 2. 현재 컴퓨터(cur)와 연결된 친구들을 하나씩 꺼내봅니다.
    // adj[cur]에는 cur와 연결된 컴퓨터들의 번호가 리스트로 들어있습니다.
    for (int next : adj[cur]) {
        
        // 3. 연결된 컴퓨터(next)가 아직 감염되지 않았다면(!visited)
        if (!visited[next]) { 
            cnt++;     // 감염자 수 1명 추가
            dfs(next); // 그 컴퓨터로 이동해서 또 전파 시작! (재귀 호출)
                        // -> 여기서 함수 안으로 다시 들어가서 깊게 파고듭니다.
        }
    }
}

int main() {
    // 문제에서 주어진 상황 예시
    int n = 7; // 컴퓨터 전체 개수 (1번 ~ 7번)
    int m = 6; // 연결된 선의 개수
    
    // adj 크기를 재설정 (혹시 모르니 넉넉하게 n+1 크기인 8로 설정해도 됨)
    // 여기서는 위에서 이미 101로 잡았으므로 생략 가능하나 명시적으로 적음
    adj.resize(n + 1); 
    
    // [그래프 연결 작업]
    // 1번과 2번이 연결됨 (양방향이므로 서로에게 넣어줌)
    adj[1].push_back(2); adj[2].push_back(1);
    
    // 2번과 3번이 연결됨
    adj[2].push_back(3); adj[3].push_back(2);
    
    // 1번과 5번이 연결됨
    adj[1].push_back(5); adj[5].push_back(1);
    
    // 5번과 2번이 연결됨
    adj[5].push_back(2); adj[2].push_back(5);
    
    // 5번과 6번이 연결됨
    adj[5].push_back(6); adj[6].push_back(5);
    
    // 4번과 7번이 연결됨 (1번 네트워크와 끊겨 있어서 감염 안 됨)
    adj[4].push_back(7); adj[7].push_back(4);

    // [탐색 시작]
    // "1번 컴퓨터가 바이러스에 걸렸다!" -> 1번부터 추적 시작
    dfs(1);

    // 정답 출력
    cout << "감염된 컴퓨터 수: " << cnt << endl; // 결과: 4
    return 0;
}
```


### 대표 예제 2 [난이도 중] : 타겟 넘버 (Target Number)

**문제 설명:**

사용할 수 있는 숫자 담긴 배열 `numbers`가 있습니다.<br>
이 숫자들을 적절히 더하거나 빼서 `target` 넘버를 만들려고 합니다.<br>

예를 들어 `[1, 1, 1, 1, 1]`로 숫자 3을 만들려면,<br>
총 5가지 방법이 있습니다. (예: -1+1+1+1+1 = 3).<br>

주어진 숫자들을 모두 사용해서 타겟 넘버를 만드는 방법의 수를 구하세요.<br>

**입력:**

- `numbers = [4, 1, 2, 1]`<br>
- `target = 4`<br>

**출력:**

- 2 (방법: `+4+1-2+1`, `+4-1+2-1`)<br>

**해결 논리:**

1. **상태 트리 구성:** 각 숫자마다 선택지는 딱 두 가지입니다. **더하거나(+), 빼거나(-)**.<br>
2. **DFS 탐색:** 이진 트리 형태로 모든 경우의 수를 파고듭니다.<br>
3. **종료 조건:** 준비된 숫자를 다 썼을 때(배열 끝까지 갔을 때), 현재까지의 합이 `target`과 같은지 확인합니다. 같으면 정답 카운트를 올립니다.<br>

- **정답 코드 예시**<br>

```cpp
#include <iostream>
#include <vector>

using namespace std;

// [전역 변수]
// 정답 카운트 (타겟 넘버를 만든 횟수)
int answer = 0;

// [DFS 함수 정의]
// numbers: 사용할 숫자들
// target: 우리가 만들어야 하는 목표 숫자
// index: 이번에 결정할 숫자의 순서 (0번째 숫자, 1번째 숫자...)
// sum: 지금까지 계산된 합계
void dfs(vector<int>& numbers, int target, int index, int sum) {
    
    // 1. [종료 조건] 숫자를 끝까지 다 썼을 때
    // numbers.size()가 4라면, index가 0, 1, 2, 3을 지나 4가 되었을 때 끝납니다.
    if (index == numbers.size()) 
    {
        // 지금까지 더하고 뺀 결과(sum)가 목표(target)와 똑같다면?
        if (sum == target) 
            answer++; // 정답 카운트 증가!
        return; // 함수 종료 (뒤로 돌아가기)
    }
    
    // 2. [수행 동작] 두 갈래 길로 뻗어나가기 (재귀 호출)
    // index번째 숫자를 '더하기(+)'로 선택하고 다음 단계(index+1)로 이동
    dfs(numbers, target, index + 1, sum + numbers[index]); 
    
    // index번째 숫자를 '빼기(-)'로 선택하고 다음 단계(index+1)로 이동
    dfs(numbers, target, index + 1, sum - numbers[index]); 
}

int main() {
    // 문제 예시: 숫자 [4, 1, 2, 1]을 이용해 4를 만들어라
    vector<int> numbers = {4, 1, 2, 1};
    int target = 4;
    
    // 탐색 시작!
    // 0번째 숫자부터 볼 것이고, 현재 합계는 0입니다.
    dfs(numbers, target, 0, 0);
    
    cout << "타겟 넘버를 만드는 방법의 수: " << answer << endl; // 결과: 2
    return 0;
}
```
    

### 대표 예제 3 [난이도 상] : 여행 경로 (Travel Route)

**문제 설명:**<br>

주어진 항공권을 **모두** 이용하여 여행 경로를 짜려고 합니다. 항상 "ICN" 공항에서 출발합니다.<br>

만약 가능한 경로가 2개 이상일 경우, 알파벳 순서가 앞서는 경로를 선택해야 합니다.<br>

모든 도시를 방문하는 것이 아니라, **모든 티켓을 사용하는 것**이 목표입니다.<br>

**입력:**

- `tickets = [["ICN", "SFO"], ["ICN", "ATL"], ["SFO", "ATL"], ["ATL", "ICN"], ["ATL", "SFO"]]`<br>

**출력:**

- `["ICN", "ATL", "ICN", "SFO", "ATL", "SFO"]`<br>

**해결 논리:**<br>

1. **정렬:** 알파벳 순서가 앞서는 경로를 찾아야 하므로, 도착지 기준으로 티켓들을 미리 정렬해 둡니다.<br>
2. **DFS & 백트래킹:**<br>
    - 현재 공항에서 갈 수 있는 티켓을 찾습니다.<br>
    - 티켓을 사용(`visited = true`)하고 다음 공항으로 이동합니다.<br>
    - **중요:** 만약 티켓을 다 썼다면 성공입니다. 하지만, 가다가 길이 끊겨서 티켓을 다 못 썼는데 더 갈 곳이 없다면?<br>
    - **백트래킹:** "이 길이 아니네?" 하고 다시 돌아와서 티켓 사용을 취소(`visited = false`)하고 다른 경로를 찾습니다.<br>

- **정답 코드 예시**<br>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm> // sort 함수 사용을 위해

using namespace std;

// [전역 변수]
vector<string> route;   // 우리가 방문한 공항들의 순서를 기록하는 '여권 도장' 같은 배열
bool visited[10001];    // 티켓 사용 여부 (1번 티켓 썼음, 2번 티켓 안 썼음...)
bool finished = false;  // "경로 완성!" 신호. (답을 찾으면 더 이상 헤매지 않게)

// [DFS 함수]
// cur: 현재 내가 있는 공항 이름
// usedTicketCnt: 지금까지 사용한 티켓의 개수
void dfs(string cur, vector<vector<string>>& tickets, int usedTicketCnt) {
    
    // 1. [조기 종료] 이미 정답을 찾았다면?
    // 다른 경우의 수는 볼 필요 없으니, 묻지도 따지지도 않고 리턴!
    if (finished) return; 
    
    // 2. [기록] 현재 공항을 경로에 추가 (도장 쾅!)
    route.push_back(cur); 
    
    // 3. [성공 조건] 가지고 있는 티켓을 모두 다 썼다면?
    if (usedTicketCnt == tickets.size()) {
        finished = true; // "나 다 찾았어! 이제 멈춰!"라고 깃발 듦
        return;
    }
    
    // 4. [탐색] 내가 가진 티켓들 중 갈 수 있는 곳 찾기
    for (int i = 0; i < tickets.size(); i++) {
        // 조건: 내 위치(cur)에서 출발하는 티켓이고(==), 아직 안 쓴 티켓(!visited)이라면
        if (tickets[i][0] == cur && !visited[i]) {
            
            // (1) 티켓 사용 (찢음)
            visited[i] = true; 
            
            // (2) 다음 공항으로 이동 (재귀 호출 - 더 깊이 들어감)
            // 도착지(tickets[i][1])가 다음번의 출발지(cur)가 됩니다.
            dfs(tickets[i][1], tickets, usedTicketCnt + 1);
            
            // (3) 확인: 갔다 왔는데 정답을 찾았나?
            if (finished) return; // 찾았으면 나도 종료!
            
            // -------------------------------------------------------
            // [중요: 백트래킹 - Backtracking]
            // 여기까지 코드가 내려왔다는 건, 위쪽 길(dfs)로 가봤는데 실패했다는 뜻입니다.
            // (티켓을 다 못 썼는데 더 이상 갈 곳이 없는 막다른 길)
            // -------------------------------------------------------
            
            visited[i] = false; // "아까 그 티켓 쓴 거 취소! 다시 붙여놔." (복구)
            route.pop_back();   // "방명록에 쓴 다음 공항 이름도 지워." (기록 삭제)
        }
    }
}

int main() {
    // 티켓 목록
    vector<vector<string>> tickets = {
        {"ICN", "SFO"}, {"ICN", "ATL"}, {"SFO", "ATL"}, 
        {"ATL", "ICN"}, {"ATL", "SFO"}
    };
    
    // [핵심 포인트: 정렬]
    // 티켓을 도착지 알파벳 순서로 미리 정렬해 둡니다.
    // 이유: DFS는 '먼저 잡힌 길'로 끝까지 가버립니다. 
    // 애초에 알파벳 빠른 티켓부터 집어들면, 
    // 처음 완성된 경로가 자동으로 '알파벳 순서가 가장 빠른 경로'가 됩니다.
    sort(tickets.begin(), tickets.end());
    
    // 항상 인천(ICN)에서 출발
    dfs("ICN", tickets, 0);
    
    // 결과 출력
    cout << "여행 경로: ";
    for(string s : route) cout << s << " ";
    cout << endl;
    // ["ICN", "ATL", "ICN", "SFO", "ATL", "SFO"]
    return 0;
}
```

## 2. BFS (너비 우선 탐색, Breadth-First Search)

### 2.1 개념 : "동심원을 그리며 퍼져나가는 마당발"

BFS는 잔잔한 호수에 돌을 던지면 물결이 퍼지듯,<br>
**시작점에서 가까운 곳부터 차례대로** 방문하는 방식입니다.<br>

- **행동 패턴:**<br>
    1. "나랑 1촌(거리 1)인 사람 다 나와!" -> 모두 방문.<br>
    2. "이제 1촌들의 친구인 2촌(거리 2) 다 나와!" -> 모두 방문.<br>
    3. 이렇게 층(Layer)별로 확산해 나갑니다.<br>
- **핵심 도구:**<br>
    - **큐(Queue):** 줄을 서는 방식(FIFO)입니다. 먼저 발견된 친구들이 줄을 서야, 그 친구들의 친구를 차례대로 만날 수 있기 때문입니다.<br>
- **언제 쓰나요?** **최단 거리** 문제. (모든 간선의 길이가 1일 때, 목적지를 가장 먼저 발견하는 순간이 최단 거리임이 보장됩니다.)<br>

### 대표 예제 1 [난이도 하] : 게임 맵 최단거리

**문제 설명:**<br>

5x5 크기의 맵이 있습니다. 캐릭터는 (0,0)에 있고, 상대방 진영은 (4,4)에 있습니다.<br>

1은 갈 수 있는 길, 0은 벽입니다. 동, 서, 남, 북으로 한 칸씩 이동할 때,<br>
상대방 진영에 도착하기 위해 지나가야 하는 칸의 개수의 **최솟값**을 구하세요. (도착할 수 없으면 -1)<br>

**입력:**<br>

- `maps = [[1,0,1,1,1],[1,0,1,0,1],[1,0,1,1,1],[1,1,1,0,1],[0,0,0,0,1]]`<br>

**출력:**<br>

- 11<br>

**해결 논리:**<br>

1. **큐 준비:** 시작점 `(0,0)`을 큐에 넣습니다.<br>
2. **거리 기록:** 맵 자체(`maps`)를 방문 기록표로 씁니다. 방문한 칸에는 '지금까지 온 거리 + 1'을 적어둡니다.<br>
3. **확산:** 큐에서 하나를 꺼내 상하좌우를 살핍니다. 갈 수 있는 길(값이 1)이면 거리를 갱신하고 큐에 넣습니다.<br>
4. **도착:** 목표 지점의 값이 1보다 크면 그 값이 최단 거리입니다.<br>

- **정답 코드 예시**

```cpp
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

int solution(vector<vector<int>> maps) {
    // 맵의 세로(n)와 가로(m) 크기 구하기
    int n = maps.size();
    int m = maps[0].size();
    
    // [방향키 설정]
    // 상, 하, 좌, 우로 한 칸씩 움직이기 위한 좌표 변화량
    // 예: dx[0]=0, dy[0]=1 -> x는 그대로, y는 +1 (오른쪽 이동)
    int dx[] = {0, 0, 1, -1};
    int dy[] = {1, -1, 0, 0};
    
    // [BFS 필수품: 큐(Queue)]
    // 방문할 곳의 좌표 {x, y}를 순서대로 저장합니다.
    // 이 법칙 덕분에 거리가 1인 칸들이 다 처리되어야만, 거리가 2인 칸들이 큐에서 나올 차례가 됩니다.
    queue<pair<int, int>> q;
    
    // 1. 시작점 (0, 0)을 큐에 넣고 출발!
    q.push({0, 0});
    
    // 큐가 빌 때까지(더 이상 갈 곳이 없을 때까지) 계속 반복
    while (!q.empty()) {
        
        // 2. 큐의 맨 앞에 있는(가장 먼저 발견한) 좌표를 꺼냄
        pair<int, int> cur = q.front(); 
        q.pop();
        
        int x = cur.first;
        int y = cur.second;
        
        // [종료 조건] 현재 위치가 도착점(오른쪽 맨 아래)이라면?
        if (x == n - 1 && y == m - 1) {
            return maps[x][y]; // 그 자리에 적힌 숫자(최단 거리) 반환
        }
        
        // 3. 현재 위치에서 4가지 방향(상하좌우) 확인
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i]; // 다음 x좌표
            int ny = y + dy[i]; // 다음 y좌표
            
            // [유효성 검사]
            // (1) 맵 바깥으로 나가지 않았는지? (인덱스 범위 체크)
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            
            // (2) 벽(0)이라서 못 가는 곳인지?
            if (maps[nx][ny] == 0) continue;
            
            // (3) ★중요★ 처음 방문하는 곳인지? (값이 1이어야 함)
            // 이미 방문했다면 2, 3, 4... 같은 숫자가 적혀있으므로 무시함
            if (maps[nx][ny] == 1) {
                
                // [거리 기록]
                // 다음 칸의 값 = 현재 칸의 값 + 1
                // 예: 내가 3번째 칸이면, 다음 칸은 4번째 칸이 됨
                maps[nx][ny] = maps[x][y] + 1;
                
                // 다음 칸을 방문 예정 리스트(큐)에 등록
                q.push({nx, ny});
            }
        }
    }
    
    // 여기까지 왔는데 리턴을 못 했다면, 도착점에 갈 방법이 없다는 뜻
    return -1;
}

int main() {
    // 5x5 미로 맵 예시 (1: 길, 0: 벽)
    // (0,0)에서 시작해 (4,4)까지 가야 함
    vector<vector<int>> maps = {
        {1, 0, 1, 1, 1},
        {1, 0, 1, 0, 1},
        {1, 0, 1, 1, 1},
        {1, 1, 1, 0, 1},
        {0, 0, 0, 0, 1}
    };
    
    // 함수 실행 및 결과 출력
    int answer = solution(maps);
    
    cout << "목적지까지 최단 거리: " << answer << endl; 
    // 정답: 11
    
    return 0;
}
```
    

### 대표 예제 2 [난이도 중] : 숨바꼭질 (Hide and Seek)

**문제 설명:**<br>

수빈이는 위치 `N`에 있고, 동생은 위치 `K`에 있습니다.<br>

수빈이는 1초 후에 `X-1`, `X+1`, `2*X`의 위치로 이동할 수 있습니다.<br>

동생을 찾는 가장 빠른 시간이 몇 초인지 구하세요.<br>

**입력:**<br>

- `N = 5` (수빈 위치)<br>
- `K = 17` (동생 위치)<br>

**출력:**<br>

- 4 (5 -> 10 -> 9 -> 18 -> 17 순서로 이동 시 4초)<br>

**해결 논리:**<br>

1. **1차원 그래프:** 2차원 지도가 아니지만, 현재 숫자에서 갈 수 있는 숫자가 3개인 그래프 문제입니다.<br>
2. **BFS 탐색:** 큐에 `[현재위치]`를 넣고, 다음 단계에 갈 수 있는 `[+1, -1, *2]` 위치들을 큐에 넣습니다.<br>
3. **시간 기록:** 방문 배열 `visited`에 몇 초 만에 도착했는지 기록합니다. 가장 먼저 `K`에 도착하는 순간이 최단 시간입니다.<br>

**정답 코드 & 시뮬레이션:**

```cpp
#include <iostream>
#include <queue>

using namespace std;

// [방문 체크 배열]
// visited[i] = 0 이면: 아직 i번 위치에 안 가봄
// visited[i] = n 이면: i번 위치에 도달하는 데 (n-1)초 걸림
// (시작을 0이 아닌 1로 하는 이유는, '방문 안 함(0)'과 구별하기 위해서입니다.)
int visited[100001]; 

int bfs(int n, int k) {
    queue<int> q;
    
    // 1. 시작점 설정
    q.push(n);
    visited[n] = 1; // 0초지만, 방문 체크를 위해 1로 기록 (나중에 -1 해서 반환)
    
    while (!q.empty()) {
        int cur = q.front(); 
        q.pop();
        
        // 2. [종료 조건] 동생(k)을 찾았는가?
        if (cur == k) {
            return visited[cur] - 1; // 기록된 시간에서 보정값 1을 빼고 반환
        }
        
        // 3. [이동 옵션] 3가지 경우의 수 (걷기 뒤/앞, 순간이동)
        int next_moves[3] = {cur - 1, cur + 1, cur * 2};
        
        for (int next : next_moves) {
            // [유효성 검사]
            // (1) 범위 체크: 0 ~ 100,000 사이여야 함 (문제 조건)
            // (2) 방문 체크: 이미 방문한 곳(0이 아님)은 더 빨리 도착한 기록이 있으므로 패스
            if (next >= 0 && next <= 100000 && visited[next] == 0) {
                
                // 시간 기록: 이전 위치 시간 + 1초
                visited[next] = visited[cur] + 1;
                
                // 큐에 넣고 다음 탐색 예약
                q.push(next);
            }
        }
    }
    return -1; // 큐가 빌 때까지 못 찾음 (이 문제에서는 발생 안 함)
}

int main() {
    int N = 5;  // 수빈 위치
    int K = 17; // 동생 위치
    
    cout << "가장 빠른 시간: " << bfs(N, K) << "초" << endl; 
    // 결과: 4
    return 0;
}
```

### 대표 예제 3 [난이도 상] : 단어 변환 (Word Ladder)

**문제 설명:**<br>

두 개의 단어 `begin`, `target`과 단어의 집합 `words`가 있습니다.<br>

아래 규칙을 지켜 `begin`을 `target`으로 변환하는 가장 짧은 과정을 찾으세요.<br>

1. 한 번에 한 개의 알파벳만 바꿀 수 있습니다.<br>
2. `words`에 있는 단어로만 변환할 수 있습니다.<br>

**입력:**

- `begin = "hit"`, `target = "cog"`<br>
- `words = ["hot", "dot", "dog", "lot", "log", "cog"]`<br>

**출력:**<br>

- 4 (`hit` -> `hot` -> `dot` -> `dog` -> `cog`)<br>

**해결 논리:**

1. **연결의 정의:** 두 단어가 "철자가 딱 한 글자만 다르다"면 연결된(Edge가 있는) 것입니다.<br>
2. **그래프 구성:** 눈에 보이지 않는 연결선을 찾아야 합니다. 현재 단어에서 `words` 목록 중 한 글자만 다른 단어를 모두 찾아 큐에 넣습니다.<br>
3. **최단 거리:** BFS를 사용하여 몇 단계를 거쳐야 `target`이 나오는지 셉니다.<br>

- **정답 코드 예시**

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <queue>
#include <algorithm>

using namespace std;

// [도우미 함수] 두 단어가 연결되어 있는지(한 글자만 다른지) 판별
bool canChange(string a, string b) {
    int diff = 0;
    // 두 단어의 각 글자를 비교해서 다른 글자 개수를 셈
    for(int i=0; i<a.size(); i++) {
        if(a[i] != b[i]) diff++;
    }
    // 딱 한 글자만 달라야 변환 가능 (연결됨)
    return diff == 1;
}

int solution(string begin, string target, vector<string> words) {
    // [BFS 필수품: 큐]
    // {현재 단어, 현재까지 몇 단계 거쳤는지} 쌍으로 저장
    queue<pair<string, int>> q; 
    
    // [방문 체크]
    // words 리스트에 있는 단어들을 이미 사용했는지 체크 (중복 사용 방지)
    vector<bool> visited(words.size(), false);
    
    // 1. 시작 단어 넣기 (아직 변환 안 했으므로 단계는 0)
    q.push({begin, 0});
    
    while(!q.empty()) {
        // 2. 큐에서 하나 꺼내기
        pair<string, int> cur = q.front(); 
        q.pop();
        
        string curWord = cur.first;
        int step = cur.second;
        
        // [종료 조건] 꺼낸 단어가 목표 단어라면?
        if (curWord == target) return step; // 현재까지의 단계 수 반환
        
        // 3. 다음 단계로 갈 수 있는 단어 탐색
        for (int i=0; i<words.size(); i++) {
            
            // 조건 1: 아직 안 써본 단어인가? (!visited)
            // 조건 2: 현재 단어에서 변환 가능한가? (canChange)
            if (!visited[i] && canChange(curWord, words[i])) {
                
                visited[i] = true; // 사용 처리
                
                // 큐에 넣을 때 단계(step)를 1 늘려서 넣음
                q.push({words[i], step + 1});
            }
        }
    }
    
    return 0; // 큐가 빌 때까지 target을 못 만들면 변환 불가능
}

int main() {
    string begin = "hit";
    string target = "cog";
    vector<string> words = {"hot", "dot", "dog", "lot", "log", "cog"};
    
    cout << "최소 변환 단계: " << solution(begin, target, words) << endl; 
    // 결과: 4
    return 0;
}
```
    

## 3. 다익스트라 (Dijkstra)

### 3.1 개념 : "비용을 따지는 깐깐한 내비게이션"

BFS는 모든 길의 거리가 1이라고 가정하지만, 현실은 고속도로(빠름), 국도(보통), 비포장도로(느림)가 섞여 있습니다.<br>
다익스트라는 **각 길마다 비용(가중치)이 다를 때** 최단 경로를 찾는 알고리즘입니다.<br>

- **핵심 도구:**<br>
    - **우선순위 큐(Priority Queue):** 일반 큐와 다릅니다. 넣은 순서와 상관없이, **가장 비용이 싼(작은) 데이터가 가장 먼저 튀어나오는** 마법 상자입니다.<br>
    
- **행동 패턴 (Greedy):**<br>
    1. 현재 갈 수 있는 길 중 **가장 싼 곳**을 무조건 먼저 선택합니다.<br>
    2. 그곳을 거쳐서 가는 것이 내가 기존에 알고 있던 길보다 더 싸다면 정보를 갱신(Relaxation)합니다.<br>

### 대표 예제 1 [난이도 하] : 최단 경로 구하기 (Basic Template)

**문제 설명:**

방향 그래프가 주어지면 주어진 시작점에서 다른 모든 정점으로의 최단 경로를 구하는 프로그램을 작성하시오.(경로가 없으면 INF 출력)<br>

**입력:**<br>

- 정점 5개, 간선 6개, 시작점 K=1<br>
- 간선 정보: `1->2(2), 1->3(3), 2->3(4), 2->4(5), 3->4(6), 5->1(1)`<br>

**출력:**<br>

- 1번 정점에서의 거리: `[0, 2, 3, 7, INF]`<br>

**해결 논리:**<br>

1. **초기화:** 거리 테이블 `dist`를 모두 무한대(INF)로 채우고, 시작점만 0으로 설정합니다.<br>
2. **우선순위 큐:** `{비용 0, 시작점 1}`을 넣고 시작합니다.<br>
3. **반복:** 큐에서 가장 비용이 작은 노드를 꺼냅니다. 그 노드를 거쳐서 인접 노드로 가는 비용을 계산합니다.<br>
4. **갱신:** 계산된 비용이 기존 기록보다 작으면 `dist`를 갱신하고 큐에 넣습니다.<br>

**정답 코드 예시**<br>

```cpp
#include <iostream>
#include <vector>
#include <queue>

// 무한대 값 설정 (int 범위 내에서 오버플로우 안 날 정도로 큰 값)
#define INF 1e9 

using namespace std;

// [그래프 정보]
// adj[1] = { {2, 2}, {3, 3} } -> 1번 노드에서 2번으로 비용 2, 3번으로 비용 3
vector<pair<int, int>> adj[20001]; // {연결된 노드 번호, 비용}

// [최단 거리표]
// dist[i] : 시작점에서 i번 노드까지 가는 최단 거리
int dist[20001]; 

void dijkstra(int start) {
    // [우선순위 큐 선언]
    // <자료형, 구현체, 비교함수>
    // greater를 써야 '작은 숫자(비용)'가 먼저 나옵니다. (오름차순 / Min Heap)
    // 안 쓰면 큰 숫자가 먼저 나옵니다. (내림차순 / Max Heap)
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    // 1. 시작점 초기화
    // 큐에 넣을 때는 {비용, 노드번호} 순서로 넣는 것이 국룰입니다.
    // 이유: pair는 첫 번째 요소(비용)를 기준으로 정렬하기 때문입니다.
    pq.push({0, start}); 
    dist[start] = 0;
    
    while (!pq.empty()) {
        // 2. 가장 비용이 작은 경로 꺼내기
        int cost = pq.top().first;   // 지금까지의 비용
        int cur = pq.top().second;   // 현재 위치한 노드
        pq.pop();
        
        // [중요: 최적화]
        // 큐에서 꺼냈는데, 이미 더 짧은 경로로 이곳에 방문한 적이 있다면?
        // (예: 아까 비용 5로 왔는데, 지금 꺼낸 게 비용 10이라면 볼 필요 없음)
        if (dist[cur] < cost) continue;
        
        // 3. 인접 노드 탐색
        // adj에는 {노드번호, 비용} 순으로 저장되어 있음을 주의!
        for (auto next : adj[cur]) {
            int nextNode = next.first;
            int nextCost = cost + next.second; // (현재까지 온 비용 + 거기 가는 비용)
            
            // 4. 더 빠른 길을 발견했다면? (Relaxation)
            if (nextCost < dist[nextNode]) {
                dist[nextNode] = nextCost;      // 거리표 갱신
                pq.push({nextCost, nextNode}); // 우선순위 큐에 등록
            }
        }
    }
}

int main() {
    int n = 5; // 정점 개수
    
    // 거리 테이블을 무한대로 초기화 (아직 아무 곳도 못 감)
    for(int i=1; i<=n; i++) dist[i] = INF;

    // [간선 연결] 방향 그래프
    // 주의: adj에 넣을 땐 {도착지, 비용}
    adj[1].push_back({2, 2}); // 1 -> 2 (비용 2)
    adj[1].push_back({3, 3}); // 1 -> 3 (비용 3)
    adj[2].push_back({3, 4}); // 2 -> 3 (비용 4)
    adj[2].push_back({4, 5}); // 2 -> 4 (비용 5)
    adj[3].push_back({4, 6}); // 3 -> 4 (비용 6)
    adj[5].push_back({1, 1}); // 5 -> 1 (비용 1)

    // 다익스트라 실행 (1번 출발)
    dijkstra(1);

    // 결과 출력
    cout << "1번 정점에서의 거리: ";
    for(int i=1; i<=n; i++) {
        if(dist[i] == INF) cout << "INF ";
        else cout << dist[i] << " ";
    }
    cout << endl;
    
    return 0;
}
```

### 대표 예제 2 [난이도 중] : 배달 (Delivery)

**문제 설명:**<br>

N개의 마을이 있는 나라에서 1번 마을에 있는 음식점이 배달을 합니다.<br>

각 마을을 연결하는 도로에는 이동 시간(가중치)이 존재합니다.<br>

음식 배달 시간이 `K` 이하인 마을의 개수를 구하세요.<br>

**입력:**<br>

- `N = 5`, `K = 3`<br>
- `road = [[1,2,1], [2,3,3], [5,2,2], [1,4,2], [5,3,1], [5,4,2]]` (양방향)<br>

**출력:**<br>

- 4 (1번 마을에서 3시간 안에 갈 수 있는 마을은 1, 2, 4, 5번 총 4개)<br>

**해결 논리:**<br>

1. 기본적인 다익스트라 알고리즘을 사용하여 1번 마을에서 출발하는 모든 마을까지의 최단 시간 `dist` 배열을 구합니다.<br>
2. 마지막에 `dist` 배열을 쭉 훑으면서 값이 `K`보다 작거나 같은 것의 개수를 셉니다.<br>

**정답 코드 예시**<br>

```cpp
#include <iostream>
#include <vector>
#include <queue>

// 무한대 값 설정 (충분히 큰 값)
#define INF 2000000000 

using namespace std;

int solution(int N, vector<vector<int>> road, int K) {
    // 1. 그래프 초기화
    // adj[1] = { {2, 1}, {4, 2} } : 1번 마을은 2번(비용1), 4번(비용2)과 연결됨
    vector<pair<int, int>> adj[51];
    
    // 거리 테이블 초기화 (모두 무한대로 설정)
    vector<int> dist(N + 1, INF);

    // [그래프 구성]
    // 도로는 양방향 통행이 가능하므로 양쪽에 정보를 다 넣어줍니다.
    for(auto r : road) {
        adj[r[0]].push_back({r[1], r[2]}); // a -> b (비용 c)
        adj[r[1]].push_back({r[0], r[2]}); // b -> a (비용 c)
    }

    // 2. 다익스트라 준비 (우선순위 큐)
    // {비용, 마을번호} 순서로 저장. 비용이 작은 것이 먼저 나옴(Min Heap).
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    // 시작점(1번 마을) 설정
    dist[1] = 0;
    pq.push({0, 1}); // {비용 0, 1번 마을}

    while(!pq.empty()) {
        int cost = pq.top().first;   // 현재까지의 비용
        int cur = pq.top().second;   // 현재 위치한 마을
        pq.pop();

        // [최적화 & 중복 방지]
        // 큐에서 꺼낸 비용(cost)이 이미 기록된 최단 비용(dist[cur])보다 크다면?
        // -> 이미 더 싸게 도착하는 방법을 찾았다는 뜻이므로, 굳이 이 경로는 볼 필요 없음.
        if(dist[cur] < cost) continue;

        // 인접한 마을 탐색
        for(auto next : adj[cur]) {
            int nextNode = next.first;
            int nextCost = cost + next.second; // (현재 비용 + 이동 비용)
            
            // 더 빨리 갈 수 있는 길을 발견했다면? (Relaxation)
            if(nextCost < dist[nextNode]) {
                dist[nextNode] = nextCost;      // 거리표 갱신
                pq.push({nextCost, nextNode}); // 큐에 등록
            }
        }
    }
    
    // 3. 결과 필터링
    int count = 0;
    for(int i = 1; i <= N; i++) {
        // 최단 거리가 K 이하인 마을만 카운트
        if(dist[i] <= K) count++;
    }
    return count;
}

int main() {
    int N = 5;
    int K = 3;
    // 도로 정보: {마을1, 마을2, 시간}
    vector<vector<int>> road = {
        {1,2,1}, {2,3,3}, {5,2,2}, 
        {1,4,2}, {5,3,1}, {5,4,2}
    };

    cout << "배달 가능한 마을 수: " << solution(N, road, K) << endl; 
    // 결과: 4
    return 0;
}
```


### **대표 예제 3 [난이도 상] : 특정 거리의 도시 찾기**

**문제 설명:**<br>

어떤 나라에는 1번부터 N번까지의 도시와 M개의 단방향 도로가 존재합니다.<br>

특정한 도시 X에서 출발하여 도달할 수 있는 모든 도시 중에서, 최단 거리가 정확히 K인 모든 도시의 번호를 출력하세요.<br>

(도시의 개수 N은 최대 300,000개, 도로 M은 최대 1,000,000개로 매우 큽니다.)<br>

**입력:**<br>

- `N=4`, `M=4`, `K=2`, `X=1` (1번 출발, 거리 2인 곳 찾기)<br>
- 도로: `1->2`, `1->3`, `2->3`, `2->4` (가중치는 모두 1이라고 가정하나, 다익스트라 연습을 위해 가중치 처리 로직 사용)<br>

**출력:**<br>

- 4 (1->2->4 거리가 2임)<br>

**해결 논리:**<br>

1. **대규모 데이터:** 노드와 간선이 매우 많으므로 O(N^2) 알고리즘을 쓰면 시간 초과가 납니다. 반드시 O(E \log V)인 우선순위 큐 다익스트라(또는 가중치가 1이라면 BFS)를 써야 합니다.<br>

2. 다익스트라 수행 후, `dist[node] == K`인 노드들을 찾아 오름차순으로 출력합니다. 하나도 없으면 -1을 출력합니다.<br>

- **정답 코드 예시**

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

#define INF 1e9 // 무한대 (아직 방문 안 함)

using namespace std;

int main() {
    // [입력 설정]
    // N: 도시 개수, M: 도로 개수, K: 목표 거리, X: 출발 도시
    int N = 4, M = 4, K = 2, X = 1;
    
    // 그래프 연결 정보 (인접 리스트)
    vector<vector<pair<int, int>>> adj(N + 1);
    // 최단 거리 테이블 (무한대로 초기화)
    vector<int> dist(N + 1, INF);
    
    // [간선 입력] (단방향)
    // 1번 도시에서 갈 수 있는 곳
    adj[1].push_back({2, 1}); // 1->2 (비용 1)
    adj[1].push_back({3, 1}); // 1->3 (비용 1)
    
    // 2번 도시에서 갈 수 있는 곳
    adj[2].push_back({3, 1}); // 2->3 (비용 1)
    adj[2].push_back({4, 1}); // 2->4 (비용 1)
    
    // 3, 4번 도시는 출발하는 간선이 없음 (도착만 가능)
    
    // [다익스트라 알고리즘 수행]
    // 우선순위 큐: {비용, 도시번호} 저장, 비용이 작은 순(오름차순)으로 정렬
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    // 시작점 초기화
    dist[X] = 0;      // 자기 자신까지 거리는 0
    pq.push({0, X});  // 큐에 넣기
    
    while (!pq.empty()) {
        int cost = pq.top().first;   // 현재까지 온 비용
        int cur = pq.top().second;   // 현재 위치
        pq.pop();
        
        // [최적화] 이미 처리된(더 짧은 경로가 있는) 경우 무시
        if (dist[cur] < cost) continue;
        
        // 현재 도시와 연결된 주변 도시 확인
        for (auto next : adj[cur]) {
            int nextNode = next.first;
            int nextCost = cost + next.second; // (지금까지 비용 + 이동 비용)
            
            // 더 짧은 경로를 발견했다면? (Relaxation)
            if (nextCost < dist[nextNode]) {
                dist[nextNode] = nextCost;      // 거리표 갱신
                pq.push({nextCost, nextNode}); // 큐에 추가
            }
        }
    }
    
    // [결과 출력]
    vector<int> result;
    // 모든 도시를 돌면서 거리가 K인 것만 찾기
    for (int i = 1; i <= N; i++) {
        if (dist[i] == K) {
            result.push_back(i);
        }
    }
    
    // 결과가 없으면 -1, 있으면 오름차순 출력
    if (result.empty()) {
        cout << -1 << endl;
    } else {
        sort(result.begin(), result.end());
        cout << "거리가 " << K << "인 도시: ";
        for (int node : result) {
            cout << node << " ";
        }
        cout << endl;
    }
    
    return 0;
}
```



```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

#define INF 1e9 // 무한대 (아직 방문 안 함)

using namespace std;

int main() {
    // [입력 설정]
    // N: 도시 개수, M: 도로 개수, K: 목표 거리, X: 출발 도시
    int N = 4, M = 4, K = 2, X = 1;
    
    // 그래프 연결 정보 (인접 리스트)
    vector<vector<pair<int, int>>> adj(N + 1);
    // 최단 거리 테이블 (무한대로 초기화)
    vector<int> dist(N + 1, INF);
    
    // [간선 입력] (단방향)
    // 1번 도시에서 갈 수 있는 곳
    adj[1].push_back({2, 1}); // 1->2 (비용 1)
    adj[1].push_back({3, 1}); // 1->3 (비용 1)
    
    // 2번 도시에서 갈 수 있는 곳
    adj[2].push_back({3, 1}); // 2->3 (비용 1)
    adj[2].push_back({4, 1}); // 2->4 (비용 1)
    
    // 3, 4번 도시는 출발하는 간선이 없음 (도착만 가능)
    
    // [다익스트라 알고리즘 수행]
    // 우선순위 큐: {비용, 도시번호} 저장, 비용이 작은 순(오름차순)으로 정렬
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    // 시작점 초기화
    dist[X] = 0;      // 자기 자신까지 거리는 0
    pq.push({0, X});  // 큐에 넣기
    
    while (!pq.empty()) {
        int cost = pq.top().first;   // 현재까지 온 비용
        int cur = pq.top().second;   // 현재 위치
        pq.pop();
        
        // [최적화] 이미 처리된(더 짧은 경로가 있는) 경우 무시
        if (dist[cur] < cost) continue;
        
        // 현재 도시와 연결된 주변 도시 확인
        for (auto next : adj[cur]) {
            int nextNode = next.first;
            int nextCost = cost + next.second; // (지금까지 비용 + 이동 비용)
            
            // 더 짧은 경로를 발견했다면? (Relaxation)
            if (nextCost < dist[nextNode]) {
                dist[nextNode] = nextCost;      // 거리표 갱신
                pq.push({nextCost, nextNode}); // 큐에 추가
            }
        }
    }
    
    // [결과 출력]
    vector<int> result;
    // 모든 도시를 돌면서 거리가 K인 것만 찾기
    for (int i = 1; i <= N; i++) {
        if (dist[i] == K) {
            result.push_back(i);
        }
    }
    
    // 결과가 없으면 -1, 있으면 오름차순 출력
    if (result.empty()) {
        cout << -1 << endl;
    } else {
        sort(result.begin(), result.end());
        cout << "거리가 " << K << "인 도시: ";
        for (int node : result) {
            cout << node << " ";
        }
        cout << endl;
    }
    
    return 0;
}
```

## 면접 관련

### std::find와 std::binary_search의 차이를 설명해주세요.

**A. `std::find` (선형 탐색)**<br>

- **동작:** 처음부터 끝까지 하나씩 순차적으로 비교하며 탐색합니다.<br>
- **장점:** 데이터가 정렬되어 있지 않아도 바로 사용할 수 있습니다.<br>
- **단점:** 데이터 양(N)에 비례하여 시간이 걸리므로, 데이터가 많을수록 느려집니다.<br>
- **반환:** 찾는 값이 있으면 해당 위치의 **Iterator**를, 없으면 `end()` Iterator를 반환합니다.<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm> // std::find

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9};

    // 정렬하지 않고 바로 검색 가능
    auto it = std::find(v.begin(), v.end(), 5);

    if (it != v.end()) {
        std::cout << "값 찾음: " << *it << std::endl; // 위치(Iterator)를 활용 가능
    }
}
```

### B. `std::binary_search` (이진 탐색)

- **동작:** 탐색 범위를 절반씩 줄여가며 탐색합니다. (Up/Down 게임과 유사)<br>
- **장점:** 대량의 데이터에서도 압도적으로 빠릅니다. (예: 10억 개의 데이터도 약 30번 비교 안에 찾음)<br>
- **단점:** 데이터가 **반드시 정렬(Sorted)** 되어 있어야만 정상 작동합니다. 정렬되지 않은 상태에서 호출하면 결과는 정의되지 않습니다(오동작).<br>
- **반환:** 값이 존재하면 `true`, 없으면 `false`를 반환합니다. **(위치를 알려주지 않음)**<br>

```cpp
#include <iostream>
#include <vector>
#include <algorithm> // std::binary_search, std::sort

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9};

    // 반드시 정렬 선행
    std::sort(v.begin(), v.end()); 

    // 존재 여부만 확인 (true/false)
    bool found = std::binary_search(v.begin(), v.end(), 5);

    if (found) {
        std::cout << "값이 존재함" << std::endl;
    }
}
```

데이터가 **정렬되어 있지 않고**, 한두 번만 찾을 때 → **`std::find`**<br>

데이터가 **이미 정렬되어 있거나**, 검색 빈도가 매우 높을 때 → **`std::binary_search`** <br>

### 레드블랙 트리란?

자가 균형 이진 탐색 트리(Self-Balancing Binary Search Tree)의 일종으로 <br>

데이터가 어떻게 들어오든 **트리의 높이를 항상 log N 수준으로 유지**하여,<br>
**최악의 경우에도 삽입, 삭제, 검색 모두 O(log N)을 보장하는** 자료구조 입니다.<br>

일반적인 `이진 탐색 트리`(BST)는 치명적인 약점이 있습니다.<br>
데이터가 순서대로 들어오면 트리가 한쪽으로 치우친 *경사 트리*(Skewed Tree)가 됩니다. <br>

예를 들어 이진 탐색 트리에 데이터를 넣을 때, 숫자가 무작위가 아니라<br>
**`1, 2, 3, 4, 5`** 처럼 이미 정렬된 순서대로 들어온다고 가정해 봅시다.<br>

**이진 탐색 트리의 삽입 규칙:**<br>

- 새로운 값 < 현재 노드 →  왼쪽으로<br>
- 새로운 값 > 현재 노드 ← 오른쪽으로<br>

이 규칙대로 `1, 2, 3, 4, 5`를 넣으면 다음과 같은 모양이 됩니다.<br>

1. **1** 삽입: 루트(Root)가 됩니다.<br>
2. **2** 삽입: 1보다 크므로 1의 **오른쪽** 자식이 됩니다.<br>
3. **3** 삽입: 1보다 크고, 2보다 크므로 2의 **오른쪽** 자식이 됩니다.<br>
4. **4** 삽입: ... 3의 **오른쪽** 자식이 됩니다.<br>
5. **5** 삽입: ... 4의 **오른쪽** 자식이 됩니다.<br>

이 경우 탐색 시간이 트리의 높이에 비례하게 되어,<br>
최악의 경우 배열과 다를 바 없는 O(N)의 시간이 걸립니다.<br>

레드-블랙 트리는 노드에 **색깔(Red/Black)** 속성을 부여하고, 다음 5가지 규칙을 통해 균형을 맞춥니다. <br>

이 규칙들이 깨지면 트리는 스스로 회전하거나 색을 바꿔서 규칙을 다시 만족시킵니다.<br>

- **노드의 색상:** 모든 노드는 **Red** 혹은 **Black** 중 하나의 색을 가진다.<br>
- **루트 노드 조건:** 트리의 시작점인 루트 노드(Root Node)는 무조건 **Black**이어야 한다.<br>
- **리프 노드(NIL) 조건:** 모든 리프 노드(NIL)는 **Black**이어야 한다.<br>
    - *> 참고: 여기서 리프 노드는 실제 데이터를 담고 있는 노드가 아니라, 트리의 끝을 알리는 더미 노드(Sentinel Node)를 의미합니다.*<br>
- **Red 노드 제약 (No Double Red):** **Red** 노드의 자식은 무조건 **Black**이어야 한다.<br>
    - *> 즉, **Red** 색상의 노드가 연속으로 두 번 나타날 수 없습니다.*<br>
    - *> 반면, **Black** 노드는 연속으로 나타나도 상관없습니다.*<br>
- **블랙 높이(Black Height) 일관성:** 어떤 노드에서 그 노드의 자손인 리프 노드(NIL)까지 가는 **모든 경로**에는 **동일한 개수의 Black 노드**가 존재해야 한다.<br>
    - *> 트리의 좌우 균형을 맞추는 가장 강력하고 중요한 규칙입니다.*<br>

위 규칙들(특히 4번과 5번) 덕분에, 루트에서 리프까지의 가장 긴 경로(Red-Black 반복)는<br>
가장 짧은 경로(Black만 존재)보다 **2배 이상 길어질 수 없습니다.**<br>

결과적으로 트리는 한쪽으로 크게 치우치지 않게 되며,<br>
최악의 경우에도 탐색, 삽입, 삭제 연산에서 O(log N)의 시간 복잡도를 보장합니다.<br>

데이터를 삽입하면 트리의 모양이 바뀌므로 규칙이 깨질 수 있습니다.<br>
이때 트리는 스스로를 수정하여 다시 규칙을 만족시킵니다.<br>

**기본 삽입 전략**

- **새로운 노드는 항상 'Red'로 삽입한다.**<br>
    - 규칙 '5번(블랙 높이)을 위반하지 않기 위해'서입니다.<br>
    - Black 노드를 추가하면 해당 경로의 Black 개수만 늘어나서 전체 경로를 다 수정해야 하는 큰 문제가 생깁니다.<br>
    - Red를 추가하면 규칙 5번은 안전하고, **규칙 4번(No Double Red)** 위반 여부만 확인하면 됩니다.<br>

```cpp
			[20](B)  <--- 할아버지 (Grandparent)
      /     \
   [10](R)  [30](?) <--- 삼촌 (Uncle): 이 색깔이 중요함!
   /
 [5](R) <--- 새로 들어온 나 (New, Red)
```

새로 삽입한 노드가 **Red**인데, 그 부모 노드도 **Red**라면<br>
'규칙 4번(Red는 연속될 수 없다)'이 깨집니다. 이를 **Double Red** 현상이라고 합니다.<br>

Double Red가 발생하면, 부모의 형제인<br>
삼촌 노드의 색깔을 확인하여 해결 방법을 결정합니다.<br>

| **상황 구분** | **삼촌 노드(U)의 색깔** | **해결 전략** | **동작 방식** |
| --- | --- | --- | --- |
| **Case 1** | **Red** | **재색칠 (Recoloring)** | 부모와 삼촌을 Black으로 바꾸고, 할아버지를 Red로 바꿈. 할아버지가 Red가 되면서, 할아버지의 부모와 또다시 Double Red 문제가 발생할 수 있음. |
| **Case 2** | **Black** (또는 NIL) | **회전 (Rotation)** | 트리를 **회전**시켜 구조를 재배치(중간 값을 가진 노드를 위로 올림). 회전된 중심 노드를 **Black**으로, 아래로 내려간 노드를 **Red**로 칠함. |

**재색칠**<br>

```cpp
			< 변경 전 >                     < 변경 후 >

        [20](B)                         [20](R)  <-- Red로 변신 (위로 문제 전파 가능)
        /     \                         /     \
     [10](R)  [30](R)   ====>        [10](B)  [30](B) <-- Black으로 진정시킴
     /        (삼촌 Red)             /
   [5](R)                          [5](R)
```

**회전**<br>

```cpp
				< 변경 전 >                     < 회전 및 색변환 후 >

        [20](B)                                [10](B) <--- 부모가 중심(루트)으로 승격
        /     \                                /     \
     [10](R)  [NIL](B)      ====>           [5](R)  [20](R) <--- 할아버지가 자식으로 내려옴
     /        (삼촌 Black)
   [5](R)
```