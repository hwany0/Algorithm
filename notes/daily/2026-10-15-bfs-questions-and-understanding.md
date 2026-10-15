# 2026-10-15 BFS 질문 정리 및 이해 내용

## 오늘의 핵심 결론
BFS는 "시작점에서 가까운 정점부터 넓게 퍼지는 탐색"이다. 대부분의 코딩테스트에서는 그래프를 직접 주는 경우보다, 문제를 그래프로 변환해서 푸는 경우가 많다.

## 1. BFS가 왜 중요한가
- 최단 거리, 최소 이동 횟수, 연결 여부 문제에서 강함
- 간선 비용이 1일 때 가장 자연스럽게 적용됨
- 큐를 사용해서 가까운 노드부터 처리함

## 2. BFS의 기본 동작
1. 시작점을 큐에 넣는다.
2. 큐에서 노드를 꺼낸다.
3. 인접 노드를 확인한다.
4. 아직 방문하지 않은 노드면 방문 처리 후 큐에 넣는다.
5. 큐가 비워질 때까지 반복한다.

핵심:
- 첫 방문한 정점이 보통 최단거리로 도달한 정점이다.
- 따라서 BFS는 최단 경로 문제에 적합하다.

## 3. 그래프 예시와 배열 예시의 차이
### 그래프 예시
```text
0: [1, 2]
1: [0, 3]
2: [0, 4]
3: [1]
4: [2]
```
- 정점이 0, 1, 2, 3, 4
- 각 정점마다 연결된 정점이 직접 적혀 있음
- 이것은 가장 직관적인 그래프 표현

### 배열 / 미로 예시
```text
[1, 1, 1]
[0, 0, 1]
[1, 1, 1]
```
이건 직접 그래프가 아니라 좌표 배열이다.

이걸 그래프로 바꾸면:
- 각 칸이 정점이 된다.
- 상하좌우 한 칸 이동이 간선이 된다.
- 단, 통로인 칸(값이 1)만 연관되며, 벽(값이 0)은 건너뛴다.

즉, 문제를 그래프로 바꾸는 핵심은:
- 정점은 무엇인가?
- 간선은 무엇인가?
- 어떤 조건에서 이동할 수 있는가?

## 4. 그래프를 실전에서 어떻게 적용하나
실제 코테에서는 보통 직접 그래프를 만드는 것이 아니라, 배열/좌표를 그래프로 생각해서 BFS를 적용한다.

예시:
```java
int[][] dirs = {{1,0}, {-1,0}, {0,1}, {0,-1}};
Queue<int[]> q = new ArrayDeque<>();
boolean[][] visited = new boolean[n][m];

q.offer(new int[]{startR, startC});
visited[startR][startC] = true;

while (!q.isEmpty()) {
    int[] cur = q.poll();
    int r = cur[0];
    int c = cur[1];

    for (int[] d : dirs) {
        int nr = r + d[0];
        int nc = c + d[1];

        if (0 <= nr && nr < n && 0 <= nc && nc < m) {
            if (grid[nr][nc] == 1 && !visited[nr][nc]) {
                visited[nr][nc] = true;
                q.offer(new int[]{nr, nc});
            }
        }
    }
}
```

이 코드는 다음을 의미한다:
- 각 좌표는 정점
- 상하좌우 이동은 간선
- 값이 1인 칸만 통로이며, 방문 여부를 관리함
- BFS는 이 구조를 레벨 순서로 탐색한다

## 5. 왜 BFS는 최단거리 보장이 가능한가
BFS는 시작점에서 거리 0, 1, 2, ... 순서로 확장한다.
그래서 처음 발견한 노드가 보통 가장 짧은 거리 경로로 도착한 노드이다.

예:
```text
0 -- 1 -- 3
 \  |
  \ 2 -- 4
```

- 0에서 1, 2는 거리 1
- 0에서 3, 4는 거리 2
- BFS는 1, 2를 먼저 처리하고 그 다음 3, 4를 처리한다.

그래서 BFS는 최소 이동 횟수 문제에 매우 적합하다.

## 6. 핵심 정리
- 그래프를 직접 푸는 문제도 있지만, 대부분은 문제를 그래프로 바꿔서 풀이한다.
- 좌표 문제는 각 칸을 노드로 보고, 이동 가능 조건을 간선으로 생각하면 된다.
- BFS는 "가까운 것부터 넓게 퍼지는 탐색"이다.
- 시작점에서 레벨별로 확장하기 때문에 최단거리 문제에 강하다.
- 문제에서 "최소 이동", "최단거리", "가장 가까운" 표현이 나오면 BFS를 떠올리면 된다.

## 7. 오늘의 이해 포인트
이 질문에서 가장 중요했던 점은 다음이었다:
- 문제를 그래프로 바꾸는 사고가 가장 중요하다

## 8. 다음 공부 방향
다음 단계로는 아래를 정리하면 좋다.
- BFS의 기본 패턴 암기
- 그래프와 좌표 배열의 변환 방식 이해
- 미로/게임맵/네트워크 문제를 BFS로 적용해 보기
- DFS와 BFS 차이 비교

## 9. Queue 자주 쓰는 메서드 (Java)
BFS에서 가장 많이 쓰는 자료구조는 `Queue`다. Java에서 `Queue`는 보통 `ArrayDeque`를 사용한다.

```java
Queue<Integer> q = new ArrayDeque<>();
```

### 자주 쓰는 메서드
```java
q.offer(x);     // 큐에 값 추가 (add와 비슷하지만 offer이 보통 더 안전)
q.poll();       // 큐에서 가장 앞 값 꺼내기 (비어있으면 null)
q.peek();       // 큐의 가장 앞 값 확인 (삭제하지 않음)
q.isEmpty();    // 큐가 비었는지 확인
q.size();       // 큐의 크기 확인
```

### 예시
```java
Queue<Integer> q = new ArrayDeque<>();
q.offer(1);
q.offer(2);
q.offer(3);

System.out.println(q.peek()); // 1
System.out.println(q.poll()); // 1
System.out.println(q.size()); // 2
```

### BFS에서의 의미
- `offer`: 다음에 방문할 노드 추가
- `poll`: 현재 탐색할 노드 꺼내기
- `peek`: 현재 큐의 앞 원소 확인
- `isEmpty`: 큐가 비었는지 확인해서 while문 종료

즉, BFS의 핵심은 다음 구조다.

```java
Queue<Integer> q = new ArrayDeque<>();
q.offer(start);

while (!q.isEmpty()) {
    int cur = q.poll();
    // 현재 노드 처리
}
```

### 실전 팁
- BFS는 보통 `ArrayDeque`를 쓰는 게 가장 편함
- `poll()`을 사용하면 큐에서 가장 앞 원소를 안전하게 꺼낼 수 있음
- `offer()`는 자주 추가할 때 안전하고 빠름
- `peek()`은 다음 노드를 확인만 하고 제거하지 않을 때 사용

### ArrayDeque 정리
`ArrayDeque`는 Java에서 가장 많이 쓰는 큐/덱 구현체다.

- `ArrayDeque`는 실제로 `Deque`를 구현한다.
- 즉, 양쪽에서 넣고 뺄 수 있다.
- BFS에서는 보통 `Queue`처럼 앞에서 넣고 앞에서 꺼내는 방식으로 사용한다.

```java
Queue<Integer> q = new ArrayDeque<>();
```

이렇게 쓰는 이유는:
- 앞에서 추가: `offer()`
- 앞에서 제거: `poll()`
- 앞에 있는 값 확인: `peek()`

그래서 BFS에서는 보통 아래 형태를 쓴다.

```java
Queue<Integer> q = new ArrayDeque<>();
q.offer(start);

while (!q.isEmpty()) {
    int cur = q.poll();
    // 탐색
}
```

즉, `ArrayDeque`는 큐처럼도 쓸 수 있고, 덱처럼도 쓸 수 있다.
BFS에서는 보통 큐처럼 쓰는 게 더 직관적이고 편하다.

### Deque 예시
`ArrayDeque`는 양쪽에서 넣고 뺄 수 있다. 예를 들어:

```java
Deque<Integer> dq = new ArrayDeque<>();

dq.offerLast(1);   // 오른쪽에 1 추가
dq.offerLast(2);   // 오른쪽에 2 추가
dq.offerFirst(0);  // 왼쪽에 0 추가

System.out.println(dq.pollFirst()); // 0
System.out.println(dq.pollLast());  // 2
System.out.println(dq.pollFirst()); // 1
```

실행 결과는 다음과 같다.

```text
0
2
1
```

이 의미는:
- `offerLast`는 오른쪽에 값 추가
- `offerFirst`는 왼쪽에 값 추가
- `pollFirst`는 왼쪽 값 제거
- `pollLast`는 오른쪽 값 제거

즉, `ArrayDeque`는 양쪽에서 넣고 빼는 구조다.
BFS에서는 보통 `Queue`처럼 앞에서 넣고 앞에서 빼는 방식만 사용하지만,
이 덱 특성 자체를 알고 있으면 자료구조 이해가 훨씬 쉬워진다.

### 요약
BFS에서는 큐의 동작을 정확히 이해하는 것이 핵심이다.
- 들어간 순서대로 나온다
- 가까운 정점부터 처리된다
- 최단거리 문제에서 유리하다
