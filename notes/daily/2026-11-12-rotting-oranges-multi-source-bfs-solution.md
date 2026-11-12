# 2026-11-12 Rotting Oranges: 내 풀이 정리

## 오늘의 핵심
오늘은 `994. Rotting Oranges`를 풀었다.

이 문제는 단순한 구현 문제가 아니라,
"여러 시작점에서 동시에 퍼지는 BFS"를 이해하는 문제다.

핵심은 다음과 같다.

- `2`인 칸들이 동시에 시작점이 된다.
- 한 번의 레벨 확장이 1분이다.
- 신선한 토마토를 방문 처리하면서 퍼지는 구조다.
- 최종적으로 방문되지 않은 `1`이 남아 있으면 `-1`이다.

## 1. 문제 이해
그리드에서 각 칸은 다음 값으로 표현된다.

- `0`: 빈 칸
- `1`: 신선한 토마토
- `2`: 썩은 토마토

`2`인 토마토는 상하좌우로 인접한 `1`을 썩게 만든다.

이때 중요한 점은,

- 썩은 토마토들이 동시에 퍼진다.
- 여러 개의 시작점이 존재한다.
- 최소 시간을 구해야 한다.

그래서 이 문제는 일반적인 단일 시작점 BFS가 아니라,
"multi-source BFS"이다.

## 2. 상태와 상태공간
### 상태
이 문제에서 상태는 현재 위치 `(row, col)`다.

```text
현재 좌표: (r, c)
```

이 좌표만 있으면,
상하좌우를 확인해서 다음 상태를 결정할 수 있다.

### 상태공간
상태공간은 grid의 모든 좌표다.

```text
(row, col) ∈ 모든 칸
```

그중 실제로 의미 있는 상태는 다음과 같다.

- `2`: 시작 상태, 썩은 토마토
- `1`: 방문 가능한 상태, 신선한 토마토
- `0`: 빈 칸, 이동 불가

## 3. 왜 BFS인지
이 문제는 "최소 시간"을 구하는 문제다.

- 한 번 퍼질 때마다 1분이 흐른다.
- 각 토마토마다 도달하는 시간이 달라진다.
- 최종적으로 가장 빠른 시간으로 다 퍼지는 것이 필요하다.

BFS는 가장 가까운 거리부터 처리하기 때문에,
한 번의 확장 = 1분으로 처리하는 데 적합하다.

즉,

```text
최소 시간 = BFS의 레벨(거리)
```

## 4. 내 코드의 아이디어
내가 한 코드는 다음 흐름이다.

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        boolean[][] visited = new boolean[m][n];
        Set<int[]> keys = new HashSet<>();
        int[][] dirs = new int[][]{{-1,0},{1,0},{0,-1},{0,1}};
        int count = -1;
        Queue<int[]> q = new ArrayDeque<>();
        for(int row = 0; row < m; row++) {
            for(int col = 0; col < n; col++) {
                if(grid[row][col] == 1) {
                    keys.add(new int[]{row, col});
                }
                if(grid[row][col] == 2) {
                    q.offer(new int[]{row, col});
                    visited[row][col] = true;
                }
            }
        }

        if(keys.isEmpty()) return 0;

        while(!q.isEmpty()){
            int size = q.size();
            count++;
            for(int i = 0; i < size; ++i) {
                int[] node = q.poll();
                int r = node[0];
                int c = node[1];
                for(int[] dir : dirs) {
                    int rd = r + dir[0];
                    int cd = c + dir[1];
                    if(rd >= 0 && rd < m && cd >= 0 && cd < n) {
                        if(grid[rd][cd] == 1 && !visited[rd][cd]) {
                            visited[rd][cd] = true;
                            q.offer(new int[]{rd, cd});
                        }
                    }
                }
            }
        }

        for(int[] key : keys) {
            if(!visited[key[0]][key[1]]) return -1;
        }

        return count;
    }
}
```

## 5. 코드의 핵심 포인트
### 1) 시작점이 여러 개다
```java
if (grid[row][col] == 2) {
    q.offer(new int[]{row, col});
    visited[row][col] = true;
}
```

모든 썩은 토마토를 큐에 넣어 동시에 퍼지게 한다.

### 2) 레벨별로 처리한다
```java
int size = q.size();
count++;
for (int i = 0; i < size; i++) {
    ...
}
```

이게 중요한 이유는,
같은 레벨의 토마토들이 같은 시간 동안 퍼지는 구조를 표현하기 위해서다.

### 3) 신선한 토마토를 방문 처리한다
```java
if(grid[rd][cd] == 1 && !visited[rd][cd]) {
    visited[rd][cd] = true;
    q.offer(new int[]{rd, cd});
}
```

이 부분이 곧 "썩은 토마토가 인접한 신선한 토마토를 썩게 하는 로직"이다.

### 4) 마지막 확인
```java
for(int[] key : keys) {
    if(!visited[key[0]][key[1]]) return -1;
}
```

이 부분은,
"아직 다 안 썩은 신선한 토마토가 남아 있으면 불가능한 상태"를 확인하는 로직이다.

## 6. 장점
이 방식의 장점은,

- “여러 시작점 BFS”를 자연스럽게 구현했다는 점
- `count`를 이용해 시간 단계를 표현했다는 점
- `visited`를 이용해 중복 방문을 막았다는 점

즉, BFS의 핵심 구조를 잘 이해하고 있다.

## 7. 보완할 점
다만 여기서 몇 가지를 더 정교하게 정리하면 더 깔끔하다.

### 1) `Set<int[]>`는 배열 키로는 부적절하다
```java
Set<int[]> keys = new HashSet<>();
```

배열은 `equals()`와 `hashCode()`를 오버라이드하지 않기 때문에,
배열의 참조값 기준으로 비교된다.

즉, `new int[]{1,2}`와 `new int[]{1,2}`는 같은 값처럼 보이지만,
HashSet에서는 서로 다른 키로 취급될 수 있다.

그래서 실무적으로는 아래처럼 하는 편이 더 안전하다.

- `freshCount`를 사용해서 개수를 세기
- 좌표를 문자열 또는 정수 조합으로 변환해서 키로 사용하기

### 2) `visited`와 `grid`를 함께 쓰는 구조는 조금 혼란스럽다
이 문제는 `grid` 값 자체를 썩은 상태로 바꾸는 방식이 더 단순하다.

그러면 별도 `visited` 배열 없이도 상태를 관리할 수 있다.

예:

```java
if (grid[nr][nc] == 1) {
    grid[nr][nc] = 2;
    q.offer(new int[]{nr, nc});
}
```

이 방식이 더 깔끔한 정석 풀이다.

## 8. 정석으로 바꾸면 더 깔끔한 형태
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

        int[][] dirs = {{1,0}, {-1,0}, {0,1}, {0,-1}};
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

이 버전은,

- `Set<int[]>` 같은 불필요한 구조를 없앰
- `freshCount`로 남은 신선한 토마토를 관리함
- `grid`를 상태 관리 도구로 사용함
- 로직이 훨씬 간결하고 빠름

## 9. 최종 정리
내가 오늘 구현한 아이디어는 핵심적으로 맞았다.

- 여러 시작점 BFS
- 레벨별 처리
- 신선한 토마토 방문 처리
- 마지막에 남은 신선한 토마토 확인

이런 요소는 모두 정석적인 문제 풀이 패턴이다.

다만 정리하면,

- `Set<int[]>` 대신 `freshCount` 방식이 더 안전하고 깔끔하다.
- `grid` 자체를 상태 관리에 활용하면 코드가 더 단순해진다.

즉,

> 이번 문제는 “multi-source BFS + 레벨 처리”가 핵심이고,
> 그 사고 방식은 맞았지만 구현을 조금 더 정석적으로 다듬으면 더 좋다.

## 10. 오늘의 이해 포인트
오늘 가장 중요했던 것은 다음이었다.

- 하나의 시작점이 아니라 여러 시작점에서 동시에 확장한다는 점
- BFS에서 레벨을 시간으로 표현한다는 점
- 상태는 현재 좌표이며, 그 좌표를 기준으로 전이를 계산한다는 점

이런 개념이 잡히면,
다음 문제인 `542. 01 Matrix`와 `127. Word Ladder`도 비슷한 패턴으로 이해하기 쉬워진다.
