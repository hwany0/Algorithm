# 2026-11-11 Rotting Oranges: BFS 요약

## 오늘의 핵심
`994. Rotting Oranges`는 단순히 토마토를 “썩게 하는 문제”가 아니라,
"여러 시작점에서 동시에 퍼지는 BFS"를 이해하는 문제다.

핵심은 다음과 같다.

- 시작점이 하나가 아니라 여러 개다.
- 모든 썩은 토마토가 동시에 퍼진다.
- 시간은 각 칸에 도달하는 최소 경로 길이로 계산한다.
- 이 문제는 `BFS`의 대표적인 응용 문제다.

## 1. 문제 이해
그리드 안에서 각 칸은 다음 상태를 가진다.

- `0`: 빈 칸
- `1`: 신선한 토마토
- `2`: 썩은 토마토

규칙:
- 썩은 토마토는 상하좌우로 인접한 신선한 토마토를 1분마다 썩게 한다.
- 동시에 여러 썩은 토마토가 퍼진다.
- 모든 토마토가 썩는 데 걸리는 최소 시간을 구한다.
- 만약 다 썩지 못하면 `-1`을 반환한다.

## 2. 상태와 상태공간
### 상태
이 문제에서 상태는 다음과 같이 정의할 수 있다.

```text
현재 좌표 (row, col)
```

그리고 각 상태는 "지금 이 칸이 썩었는지 / 아직 안 썩었는지"에 따라 전이 조건이 달라진다.

### 상태공간
상태공간은 모든 좌표의 집합이다.

```text
(row, col) ∈ grid의 모든 위치
```

그중 실제로 의미 있는 상태는 다음과 같다.

- 썩은 토마토가 시작 상태
- 신선한 토마토가 방문 가능한 상태
- 빈 칸은 지나갈 수 없는 상태

## 3. 왜 BFS인지
이 문제는 최단 시간을 구하는 문제다.

- 한 번 퍼질 때마다 1분이 증가한다.
- 시작점이 여러 개면, 동시에 퍼져야 한다.
- 모든 칸에 도달하는 최소 시간 = 최단 거리

이런 경우에 BFS가 적합하다.

BFS의 특징:
- 시작점에서 가까운 거리부터 탐색한다.
- 같은 시간 단위로 확장한다.
- 최소 시간 문제에서 자연스럽게 맞아 떨어진다.

## 4. 여러 시작점 처리: multi-source BFS
이 문제의 핵심 포인트는 시작점이 여러 개인다는 점이다.

기존의 단일 시작점 BFS와 달리,
이 문제는 모든 썩은 토마토를 한 번에 queue에 넣어야 한다.

예시:

```java
Queue<int[]> q = new ArrayDeque<>();
for (int row = 0; row < rows; row++) {
    for (int col = 0; col < cols; col++) {
        if (grid[row][col] == 2) {
            q.offer(new int[]{row, col});
            visited[row][col] = true;
        }
    }
}
```

이렇게 한 번에 넣고 시작하면,
모든 썩은 토마토가 동시에 퍼지게 된다.

## 5. 시간 관리 방법
보통은 `dist`나 `minute`을 같이 관리한다.

```java
int minutes = 0;
while (!q.isEmpty()) {
    int size = q.size();
    for (int i = 0; i < size; i++) {
        int[] cur = q.poll();
        // 인접한 1 칸을 확인
    }
    minutes++;
}
```

이 방식은 다음을 의미한다.

- 같은 레벨의 토마토를 한 번에 처리한다.
- 한 번의 레벨 확장이 1분 흐름이다.
- 마지막에 남은 신선한 토마토가 있으면 `-1`이다.

## 6. 전이 규칙
상태 전이는 다음과 같다.

- 현재 칸이 썩은 토마토면,
- 상하좌우 인접 칸을 확인한다.
- 그 칸이 `1`이면 썩게 한다.
- 방문 처리 후 queue에 넣는다.

즉,

```text
상태: 현재 좌표 (r, c)
전이: 상하좌우 이동
조건: 다음 칸이 신선한 토마토(1)여야 함
```

## 7. 왜 visited를 쓰는가
같은 칸을 여러 번 처리하면 중복 계산이 생긴다.

예를 들어:

- 두 개의 썩은 토마토가 같은 신선한 토마토를 동시에 보게 되면
- 같은 칸이 여러 번 queue에 들어갈 수 있다.

그래서 `visited`를 써서 이미 처리한 칸은 다시 처리하지 않는다.

## 8. 정석 풀이 패턴
```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        Queue<int[]> q = new ArrayDeque<>();
        int fresh = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 2) {
                    q.offer(new int[]{r, c});
                } else if (grid[r][c] == 1) {
                    fresh++;
                }
            }
        }

        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        int minutes = 0;

        while (!q.isEmpty() && fresh > 0) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                int[] cur = q.poll();
                int r = cur[0];
                int c = cur[1];

                for (int[] dir : dirs) {
                    int nr = r + dir[0];
                    int nc = c + dir[1];

                    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                        if (grid[nr][nc] == 1) {
                            grid[nr][nc] = 2;
                            q.offer(new int[]{nr, nc});
                            fresh--;
                        }
                    }
                }
            }
            minutes++;
        }

        return fresh == 0 ? minutes : -1;
    }
}
```

## 9. 핵심 정리
`994. Rotting Oranges`는 다음 관점으로 보면 쉽다.

- 상태 = 현재 좌표
- 시작 상태 = 모든 썩은 토마토 위치
- 전이 = 상하좌우로 신선한 토마토를 썩게 만들기
- 목표 = 모든 신선한 토마토가 썩는 최소 시간
- 해결 방식 = Multi-source BFS

즉,

> 여러 시작점에서 동시에 퍼지는 최단 거리 문제다.

이 개념을 익히면 다음에 나오는 `542. 01 Matrix`, `127. Word Ladder`, `752. Open the Lock` 같은 문제도 훨씬 쉽게 느껴진다.

## 10. 오늘의 학습 포인트
- `Queue`에 시작점을 여러 개 넣는 패턴
- `while (!q.isEmpty())` 안에서 `size`로 레벨 처리하는 방식
- `visited` 또는 `grid` 값을 상태값으로 재활용하는 방식
- "최소 시간 = BFS 레벨"이라는 인식

다음 단계에서는 이 패턴을 `542. 01 Matrix`와 `127. Word Ladder`에서 공통적으로 연결해 보면 더 이해가 잘 된다.
