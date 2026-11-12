# 2026-11-12 01 Matrix: 타임아웃 시도 정리

## 오늘의 핵심
오늘은 `542. 01 Matrix`를 풀다가 타임아웃이 나서,
기존 아이디어를 정리하고 개선 방향을 찾았다.

핵심은 다음과 같다.

- 각 `1`에서 `0`까지의 최소 거리를 구하려고 했음
- 이 방식은 각 칸마다 BFS를 돌리기 때문에 비용이 커짐
- 결국 `O(m * n * (m + n))` 정도의 비효율이 생김
- 정석은 "모든 0을 시작점으로 한 multi-source BFS"를 사용하는 것

## 1. 문제 이해
`01 Matrix`는 각 칸이 다음 상태를 가진다.

- `0`: 이미 도착 지점
- `1`: 거리 계산이 필요한 칸

문제는 각 `1`에서 가장 가까운 `0`까지의 거리를 구하는 것이다.

즉, 각 칸의 상태는 다음과 같이 정의할 수 있다.

- 상태: 현재 좌표 `(row, col)`
- 전이: 상하좌우 이동
- 목표: 인접한 `0`을 만나는 것
- 거리: 몇 번 이동했는지

## 2. 내가 시도한 방식
내가 한 코드는 각 `1`마다 BFS를 시작하는 방식이었다.

```java
class Solution {
    public int[][] updateMatrix(int[][] mat) {
        int m = mat.length;
        int n = mat[0].length;
        int[][] result = new int[m][n];
        for(int i = 0; i < m; ++i){
            for(int j = 0; j < n; ++j) {
                result[i][j] = -1;
            }
        } 
        int[][] dirs = new int[][]{{-1,0},{1,0},{0,-1},{0,1}};
        for(int row = 0; row < m; ++row) {
            for(int col = 0; col < n; ++col) {
                if(mat[row][col] == 0) result[row][col] = 0;
                if(mat[row][col] == 1) {
                    int min = 0;
                    int cache = Integer.MAX_VALUE;
                    Queue<int[]> q = new ArrayDeque<>();
                    boolean[][] visited = new boolean[m][n];
                    q.offer(new int[]{row, col});
                    visited[row][col] = true;
                    boolean found = false;
                    while(!q.isEmpty() && !found) {
                        int size = q.size();
                        min++;
                        if(min >= cache) {
                            min = cache;
                            found = true;
                        }
                        for(int i = 0; i < size; ++i) {
                            int[] node = q.poll();
                            int r = node[0];
                            int c = node[1];

                            for(int[] dir : dirs) {
                                int rd = r + dir[0];
                                int cd = c + dir[1];
                                if(rd >= 0 && rd < m && cd >= 0 && cd < n) {
                                    if(result[rd][cd] != -1) {
                                        cache = Math.min(mat[row][col] + result[rd][cd], cache);
                                        continue;
                                    }
                                    if(mat[rd][cd] == 0) {
                                        found = true;
                                        break;
                                    }
                                    if(!visited[rd][cd] && mat[rd][cd] == 1) {
                                        q.offer(new int[]{rd, cd});
                                        visited[rd][cd] = true;
                                    }
                                }
                            }
                        }
                    }
                    result[row][col] = min;
                }
            }
        }
        return result;
    }
}
```

## 3. 왜 시간이 오래 걸렸는가
이 방식의 문제는,
"각 `1`마다 BFS를 새로 시작한다"는 점이다.

예를 들어 100x100 격자라면,
대략 10,000 개의 칸에 대해 BFS를 돌리게 된다.

각 BFS도 최악의 경우 격자 전체를 탐색할 수 있으므로,
전체 시간 복잡도는 매우 커진다.

즉,

```text
각 1마다 BFS를 수행
=> O(m * n * (m + n)) 수준
```

이렇게 되면 타임아웃이 나기 쉽다.

## 4. 핵심 원인: 시작점이 너무 많다
이 문제는 단순히 각 `1`에서 BFS를 돌리는 것보다,
"모든 `0`을 시작점으로 한 BFS"를 돌리는 것이 훨씬 효율적이다.

왜냐하면,

- `0`의 개수는 보통 적다
- 모든 `1`이 `0`까지의 거리 정보를 공유할 수 있다
- 한 번의 BFS에서 모든 칸의 최소 거리를 계산할 수 있다

그래서 정석은 아래와 같은 패턴이다.

```java
Queue<int[]> q = new ArrayDeque<>();
for (int r = 0; r < rows; r++) {
    for (int c = 0; c < cols; c++) {
        if (mat[r][c] == 0) {
            q.offer(new int[]{r, c});
            dist[r][c] = 0;
        } else {
            dist[r][c] = Integer.MAX_VALUE;
        }
    }
}

while (!q.isEmpty()) {
    int[] cur = q.poll();
    for (int[] dir : dirs) {
        int nr = cur[0] + dir[0];
        int nc = cur[1] + dir[1];
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
            if (dist[nr][nc] > dist[cur[0]][cur[1]] + 1) {
                dist[nr][nc] = dist[cur[0]][cur[1]] + 1;
                q.offer(new int[]{nr, nc});
            }
        }
    }
}
```

이 방식은 한 번의 BFS로 모든 칸의 최단 거리를 구할 수 있다.

## 5. 정리
내가 시도한 코드는 아이디어 자체가 "거리 계산"쪽으로 맞았지만,
그 구현 방식이 너무 비효율적이었다.

그래프 관점으로 보면,

- 상태 = 현재 좌표
- 시작 상태가 너무 많음
- 각 `1`마다 BFS를 수행하는 구조는 비효율적
- 정석은 여러 시작점이 동시에 출발하는 BFS

즉,

> `01 Matrix`는 "multi-source BFS" 문제다.

이 개념이 가장 중요하다.

## 6. 오늘의 배운 점
- 매 칸마다 BFS를 새로 돌리면 타임아웃 위험이 있다.
- 시작점이 여러 개인 경우, `queue`에 한 번에 넣고 BFS를 돌리는 방식이 훨씬 효율적이다.
- 문제를 그래프로 보되, 알고리즘은 한 번의 BFS로 통합해서 처리하는 연습이 필요하다.

## 7. 다음 단계
다음에는 정석 버전으로 다시 구현해보고,
"왜 이 방식이 더 빠른지"를 다시 코드와 함께 정리할 계획이다.

이 문제는 단순히 구현을 바꾸는 게 아니라,
"탐색의 시작점을 어디로 잡느냐"가 전체 효율을 좌우한다는 점에서 매우 중요한 문제다.
