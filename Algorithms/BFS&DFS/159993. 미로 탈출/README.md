# [level 2] 미로 탈출 - 159993

## 1. 문제
1 x 1 크기의 칸들로 이루어진 직사각형 격자 형태의 미로에서 탈출하려고 합니다. 각 칸은 통로 또는 벽으로 구성되어 있으며, 벽으로 된 칸은 지나갈 수 없고 통로로 된 칸으로만 이동할 수 있습니다. 통로들 중 한 칸에는 미로를 빠져나가는 문이 있는데, 이 문은 레버를 당겨서만 열 수 있습니다. 레버 또한 통로들 중 한 칸에 있습니다. 따라서, 출발 지점에서 먼저 레버가 있는 칸으로 이동하여 레버를 당긴 후 미로를 빠져나가는 문이 있는 칸으로 이동하면 됩니다. 이때 아직 레버를 당기지 않았더라도 출구가 있는 칸을 지나갈 수 있습니다. 미로에서 한 칸을 이동하는데 1초가 걸린다고 할 때, 최대한 빠르게 미로를 빠져나가는데 걸리는 시간을 구하려 합니다.

미로를 나타낸 문자열 배열 maps가 매개변수로 주어질 때, 미로를 탈출하는데 필요한 최소 시간을 return 하는 solution 함수를 완성해주세요. 만약, 탈출할 수 없다면 -1을 return 해주세요.

### 제한사항
- 5 ≤ maps의 길이 ≤ 100  
  - 5 ≤ maps[i]의 길이 ≤ 100  
  - maps[i]는 다음 5개의 문자들로만 이루어져 있습니다.
    - S : 시작 지점
    - E : 출구
    - L : 레버
    - O : 통로
    - X : 벽  
  - 시작 지점과 출구, 레버는 항상 다른 곳에 존재하며 한 개씩만 존재합니다.
  - 출구는 레버가 당겨지지 않아도 지나갈 수 있으며, 모든 통로, 출구, 레버, 시작점은 여러 번 지나갈 수 있습니다.

### 입출력 예
|maps                                     |result|
|-----------------------------------------|------|
|["SOOOL","XXXXO","OOOOO","OXXXX","OOOOE"]|16    |
|["LOOXS","OOOOX","OOOOO","OOOOO","EOOOO"]|-1    |

## 2. 풀이
### 2.1. BFS
이 문제는 이전에 풀이를 하였던 [2206. 벽 부수고 이동하기](https://github.com/AHEAD94/cs-notes/tree/main/Algorithms/BFS%26DFS/2206.%20%EB%B2%BD%20%EB%B6%80%EC%88%98%EA%B3%A0%20%EC%9D%B4%EB%8F%99%ED%95%98%EA%B8%B0/)의 응용 문제다.  
  
이번 문제에서도 서로 다른 상태가 존재하는데, "레버를 당기지 않은 상태"와 "레버를 당긴 상태"로 정의할 수 있다.  
우리는 레버를 당긴 상태에서만 출구를 통해 탈출할 수 있다.  
그에 맞게 2차원 벡터에서 한 차원을 확장해 3차원 벡터로 방문을 관리하였다.
```cpp
// 레버를 당기지 않은 상태 (false)와 당긴 상태 (true)를 나누어 관리
vector<vector<vector<bool>>> visited(mapRows, vector<vector<bool>>(mapCols, vector<bool>(2, false)));
```

이 문제에서 첫번째로 다른 점은 시작 지점과 출구의 위치가 달라질 수 있다는 점이다.  
그렇기 때문에 maps를 탐색하며 시작 위치를 나타내는 문자 'S'를 발견하면, 그 위치에서 BFS를 수행했다.
```cpp
bool startPointFound = false;
for (size_t row = 0; row < mapRows; row++) {
    for (size_t col = 0; col < mapCols; col++) {
        if (maps[row][col] == 'S') {
            answer = BFS(maps, visited, dirs, row, col);
            startPointFound = true;
            break;
        }
    }
    if (startPointFound) break;
}
```

이후에는 "레버를 당기지 않은 상태"에서 BFS를 진행하며 레버인 'L'을 발견한 뒤에 "레버를 당긴 상태"로 방문 기록을 하며 출구 'E'를 찾는다.  
이 과정들을 진행하며 시간 변수 time을 1씩 증가시키고, 출구를 찾으면 반환하도록 하여 문제를 해결하였다.
```cpp
if (0 <= nextRow && nextRow < rowSize && 0 <= nextCol && nextCol < colSize) {
    bool visitable = false;

    if (maps[nextRow][nextCol] != 'X') {
        visitable = true;
    }
    if (maps[nextRow][nextCol] == 'L') {
        leverState = 1;
    }

    if (visitable && !visited[nextRow][nextCol][leverState]) {
        visited[nextRow][nextCol][leverState] = true;
        bfsQueue.push({nextRow, nextCol, time + 1, leverState});
    }
}
```

## 3. 전체 코드
```cpp
#include <string>
#include <vector>
#include <queue>

using namespace std;

int BFS(const vector<string>& maps, vector<vector<vector<bool>>>& visited, const vector<vector<int>>& dirs, const int row, const int col) {
    size_t rowSize = maps.size();
    size_t colSize = maps[0].size();
    
    queue<vector<int>> bfsQueue;

    visited[row][col][0] = true; // 시작 포인트 방문 (레버 당기기 전)
    bfsQueue.push({row, col, 0, 0}); // row, col, time, leverPulled

    while (!bfsQueue.empty()) {
        int curRow = bfsQueue.front()[0];
        int curCol = bfsQueue.front()[1];
        int time = bfsQueue.front()[2];
        int leverPulled = bfsQueue.front()[3];
        bfsQueue.pop();
        
        if (maps[curRow][curCol] == 'E' && leverPulled) {
            return time;
        }

        for (auto dir : dirs) {
            int nextRow = curRow + dir[0];
            int nextCol = curCol + dir[1];
            int leverState = leverPulled;

            if (0 <= nextRow && nextRow < rowSize && 0 <= nextCol && nextCol < colSize) {
                bool visitable = false;

                if (maps[nextRow][nextCol] != 'X') {
                    visitable = true;
                }
                if (maps[nextRow][nextCol] == 'L') {
                    leverState = 1;
                }

                if (visitable && !visited[nextRow][nextCol][leverState]) {
                    visited[nextRow][nextCol][leverState] = true;
                    bfsQueue.push({nextRow, nextCol, time + 1, leverState});
                }
            }
        }
    }

    return -1;
}

int solution(vector<string> maps) {
    int answer = 0;

    size_t mapRows = maps.size();
    size_t mapCols = maps[0].size();

    // 레버를 당기지 않은 상태 (false)와 당긴 상태 (true)를 나누어 관리
    vector<vector<vector<bool>>> visited(mapRows, vector<vector<bool>>(mapCols, vector<bool>(2, false)));
    vector<vector<int>> dirs{{-1, 0}, {0, 1}, {1, 0}, {0, -1}};

    bool startPointFound = false;
    for (size_t row = 0; row < mapRows; row++) {
        for (size_t col = 0; col < mapCols; col++) {
            if (maps[row][col] == 'S') {
                answer = BFS(maps, visited, dirs, row, col);
                startPointFound = true;
                break;
            }
        }
        if (startPointFound) break;
    }

    return answer;
}
```

## 4. 고찰
이 문제는 BFS를 이용하여 두가지 상태에서의 경로를 탐색하며 미로를 탈출하는 문제였다.  
탐색을 하며 상태가 변경되는 문제는 이전에 풀었던 경험을 통해 어렵지 않게 해결할 수 있었다.  
다만, 불특정한 위치에 존재하는 출발점, 시작점, 그리고 레버를 처리하는 과정이 추가되어 고려할 요인이 더 많았던 것 같다.
