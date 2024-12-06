# [Gold III] 벽 부수고 이동하기 - 2206

|시간 제한|메모리 제한|
|-------|--------|
|2 초    | 192 MB |

## 1. 문제
N×M의 행렬로 표현되는 맵이 있다. 맵에서 0은 이동할 수 있는 곳을 나타내고, 1은 이동할 수 없는 벽이 있는 곳을 나타낸다. 당신은 (1, 1)에서 (N, M)의 위치까지 이동하려 하는데, 이때 최단 경로로 이동하려 한다. 최단경로는 맵에서 가장 적은 개수의 칸을 지나는 경로를 말하는데, 이때 시작하는 칸과 끝나는 칸도 포함해서 센다.

만약에 이동하는 도중에 한 개의 벽을 부수고 이동하는 것이 좀 더 경로가 짧아진다면, 벽을 한 개 까지 부수고 이동하여도 된다.

한 칸에서 이동할 수 있는 칸은 상하좌우로 인접한 칸이다.

맵이 주어졌을 때, 최단 경로를 구해 내는 프로그램을 작성하시오.

### 입력
첫째 줄에 N(1 ≤ N ≤ 1,000), M(1 ≤ M ≤ 1,000)이 주어진다. 다음 N개의 줄에 M개의 숫자로 맵이 주어진다. (1, 1)과 (N, M)은 항상 0이라고 가정하자.

### 출력
첫째 줄에 최단 거리를 출력한다. 불가능할 때는 -1을 출력한다.

## 2. 풀이
### 2.1. 첫번째 시도: DFS
처음으로 시도했던 풀이는 DFS를 이용한 풀이였다. 하지만 **DFS는 최단거리 계산을 보장하지 않는 점**을 간과하였고, 과도한 계산량으로 시간초과가 발생했다.

### 2.2. 두번째 시도: BFS로의 단순 변환
이후에는 풀이를 BFS로 변경하였다. 이번에는 제출이 오답으로 처리되었다. 이유는 다음과 같다.  
1. 최단거리를 만들지 못하는 경우가 벽을 부수고 지나가면서 벽이 없는 특정 칸 (row_k, col_k)을 먼저 방문한다. ✅
2. 최단거리를 만드는 경우가 경로 상의 칸들을 지나가다 이미 방문한 칸 (row_k, col_k)을 만나 지나갈 수 없게 된다. ❌

#### map
| |0 |1 |2 |3 |
|-|--|--|--|--|
|0|🏃‍♂️|  |  |  |
|1|🚧|🚧|🚧|  |
|2|🚧|🚧|🚧|  |
|3|  |  |  |  |
|4|  |🚧|🚧|🚧|
|5|  |  |  |🏁|

#### 경로 1: 벽 (1, 2) 부수고 칸 (1, 3) 방문
| |0 |1 |2 |3 |
|-|--|--|--|--|
|0|➡️|➡️|⬇️|  |
|1|🚧|🚧|➡️|✅|
|2|🚧|🚧|🚧|  |
|3|  |  |  |  |
|4|  |🚧|🚧|🚧|
|5|  |  |  |🏁|

#### 경로 2: 칸 (1, 3) 방문 불가
| |0 |1 |2 |3 |
|-|--|--|--|--|
|0|➡️|➡️|➡️|⬇️|
|1|🚧|🚧|🚧|❌|
|2|🚧|🚧|🚧|  |
|3|  |  |  |  |
|4|  |🚧|🚧|🚧|
|5|  |  |  |🏁|

### 2.3. 세번째 시도 : 방문기록 매트릭스의 차원 확장
벽을 이미 부순 경로와 아직 부수지 않은 경로의 방문을 나누어 처리하기 위해서 다음과 같이 방문기록 벡터를 3차원으로 정의하였다.
```cpp
// 벽을 부순 경로와 부수지 않은 경로를 분리하기 위하여 3차원 벡터 사용
vector<vector<vector<int>>> visited(N, vector<vector<int>>(M, vector<int>(2, false)));
```

그리고 방문의 확인과 기록 시에는 벽을 파괴할 기회 `chance_i`를 세번째 차원 인덱스로 사용하여 방문경로를 분리해서 기록했다.
```cpp
int visitable = 0;
            
if (map[next_row][next_col] == 0) {
    visitable = 1;
}
if (map[next_row][next_col] == 1 && chance_i > 0) { // 벽을 부술 기회가 남은 경우
    visitable = 1;
    chance_i--;
}
        
if (visitable && !visited[next_row][next_col][chance_i]) {
    bfs_q.push(BFSArgs{next_row, next_col, dist + 1, chance_i});
    visited[next_row][next_col][chance_i] = 1;
}
```
그렇게 수정한 결과, 문제 풀이에 성공할 수 있었다.

## 3. 전체 코드
```cpp
#include <iostream>
#include <vector>
#include <string>
#include <queue>

using namespace std;

struct BFSArgs {
    int row;
    int col;
    int dist;
    int chance;
};

int BFS(const vector<vector<int>>& map, const int N, const int M) {
    // 벽을 부순 경로와 부수지 않은 경로를 분리하기 위하여 3차원 벡터 사용
    vector<vector<vector<int>>> visited(N, vector<vector<int>>(M, vector<int>(2, false)));
    const vector<pair<int, int>> dirs{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    
    queue<BFSArgs> bfs_q;
    bfs_q.push(BFSArgs{0, 0, 1, 1});
    visited[0][0][1] = 1;
    
    while (!bfs_q.empty()) {
        BFSArgs args = bfs_q.front();
        int row = args.row;
        int col = args.col;
        int dist = args.dist;
        int chance = args.chance;
        bfs_q.pop();
        
        if (row == N - 1 && col == M - 1) {
            return dist;
        }
    
        for (const auto& dir : dirs) {
            int next_row = row + dir.first;
            int next_col = col + dir.second;
            int chance_i = chance;
            
            // map 경계 확인
            if (0 <= next_row && next_row < N && 0 <= next_col && next_col < M) {
                int visitable = 0;
            
                if (map[next_row][next_col] == 0) {
                    visitable = 1;
                }
                if (map[next_row][next_col] == 1 && chance_i > 0) { // 벽을 부술 기회가 남은 경우
                    visitable = 1;
                    chance_i--;
                }
                        
                if (visitable && !visited[next_row][next_col][chance_i]) {
                    bfs_q.push(BFSArgs{next_row, next_col, dist + 1, chance_i});
                    visited[next_row][next_col][chance_i] = 1;
                }
            }
        }
    }
    return -1;
}

int main() {
    int N, M;
    cin >> N >> M;
    
    vector<vector<int>> map(N, vector<int>(M, 0));
    for (size_t i = 0; i < static_cast<size_t>(N); i++) {
        string str_nums;
        cin >> str_nums;
        
        for (size_t j = 0; j < static_cast<size_t>(M); j++) {
            int num = str_nums[j] - '0';
            if (num == 1) map[i][j] = num;
        }
    }
    
    int min_dist = BFS(map, N, M);
    cout << min_dist << endl;
    
    return 0;
}
```

## 4. 고찰
BFS는 절차를 진행하며, 서로 다른 구간에서 동일한 위치를 다음에 방문할 위치로 삼을 수 있다. 그런 경우에는 방문기록 매트릭스의 차원을 확장하여 방문의 확인과 기록을 차원에 따라 나누어 처리할 수 있도록 한다.
