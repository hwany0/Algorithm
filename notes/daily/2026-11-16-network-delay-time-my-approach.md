# 2026-11-16 Network Delay Time: 내가 푼 방식 정리

## 오늘의 핵심
오늘은 `743. Network Delay Time` 문제를 풀었다.

이 문제는 `Word Ladder`, `Minimum Genetic Mutation`, `Open the Lock`처럼 BFS로 푸는 문제가 아니라,
"가중치가 있는 그래프에서 시작점까지의 최소 비용을 구하는 문제"다.

그래서 정석적으로는 `Dijkstra`가 맞지만,
내가 푼 방식의 핵심 아이디어는 이미 최단거리 비교와 갱신을 하고 있다는 점에서 맞는 방향이었다.

## 1. 문제 이해
문제는 다음과 같다.

- 각 간선은 `(u, v, w)` 형태로 주어진다.
- 신호가 `k`에서 시작한다.
- 각 노드까지의 최소 전달 시간을 구해야 한다.
- 모든 노드에 도달할 수 있으면 그 중 최대값을 반환한다.
- 도달할 수 없는 경우 `-1`을 반환한다.

즉, 이 문제는

```text
시작 노드에서 각 노드까지의 최소 비용을 구하는 문제
```

다.

그래프 관점에서는 다음과 같다.

- 노드 = 컴퓨터/정점
- 간선 = 전송 비용
- 시작점 = `k`
- 정답 = 모든 노드에 도달하는데 걸리는 최대 최소 비용

## 2. 내가 푼 코드의 핵심 아이디어
내가 푼 코드는 다음과 같은 흐름이었다.

```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        int[][] weight = new int[n + 1][n + 1];
        int[] dist = new int[n + 1];
        int[] visit = new int[n + 1];
        int INF = Integer.MAX_VALUE;
        Map<Integer, List<Integer>> map = new HashMap<>();
        for(int row = 0; row < times.length; ++row) {
            int from = times[row][0];
            int to = times[row][1];
            int w = times[row][2];
            map.computeIfAbsent(from, x -> new ArrayList<>()).add(to);
            weight[from][to] = w;
            dist[from] = INF;
            dist[to] = INF;
        }

        if(map.get(k) == null) return -1;

        Queue<Integer> q = new ArrayDeque<>();
        q.offer(k);
        dist[k] = 0;

        while(!q.isEmpty()){
            int node = q.poll();
            visit[node] = 1;

            List<Integer> nexts = map.getOrDefault(node, new ArrayList<>());
            for(int next : nexts) {
                if(dist[node] + weight[node][next] < dist[next]) {
                    dist[next] = dist[node] + weight[node][next];
                    visit[next] = 1;
                    q.offer(next);
                }
            }    
        }

        int max = -1;
        for(int i = 1; i < n + 1; ++i) {
            if(visit[i] == 0) return -1;
        }
        for(int i = 0; i < dist.length; ++i) {
            if(dist[i] == INF) return -1; 
            max = Math.max(max, dist[i]);
        }

        return max;
    }
}
```

## 3. 여기서 맞는 부분
내가 푼 방식에서 맞는 부분은 다음이다.

### 1) 최단거리 비교를 하고 있다
```java
if (dist[node] + weight[node][next] < dist[next]) {
    dist[next] = dist[node] + weight[node][next];
}
```

이 부분은 정확히 최단거리 갱신의 핵심이다.

- 현재 노드까지의 최소 비용이 `dist[node]`다.
- 여기에 간선 비용을 더하면 다음 노드까지의 후보 비용이 된다.
- 후보 비용이 기존 거리보다 작으면 갱신해야 한다.

이 로직은 최단거리 알고리즘의 본질이다.

### 2) `dist`를 이용해 최소 비용을 추적하고 있다
```java
dist[k] = 0;
```

이것은 시작점의 비용을 0으로 두고,
다음으로 갈 수 있는 노드들의 최소 비용을 갱신하는 방식이다.

이건 다익스트라의 핵심 사고다.

## 4. 왜 이 방식이 완전히 정석이 아닌가
핵심 문제는 다음이다.

### 1) `Queue`가 BFS용 큐라서 정렬 보장이 없다
```java
Queue<Integer> q = new ArrayDeque<>();
```

BFS는 큐의 FIFO 특성이 보장되지만,
가중치 그래프에서는 가장 짧은 거리의 노드가 항상 먼저 나오지 않는다.

즉,

- 지금 큐에서 꺼낸 노드가 항상 현재까지 가장 짧은 거리로 도달한 노드라는 보장이 없다.
- 그래서 “현재 노드에서 비용을 갱신하면 끝”이라는 가정을 할 수 없다.

### 2) `visit`를 막는 방식이 문제다
```java
visit[node] = 1;
```

이 부분은 가중치 그래프에서는 위험하다.

왜냐하면,

- 같은 노드라도 더 짧은 경로가 나중에 발견될 수 있기 때문
- 처음 발견한 경로가 최단 경로라고 보장할 수 없기 때문

즉, 가중치 그래프에서는

> “한 번 방문한 노드는 끝”이 아니라
> “더 짧은 거리로 다시 방문되면 갱신”이 필요하다.

## 5. 왜 Dijkstra가 정석인가
이 문제는 간선 비용이 가중치다.

그리고 비용이 모두 양수다.

이 경우에는 `Dijkstra`가 정석이다.

- 우선순위 큐를 사용한다.
- 현재까지의 누적 비용이 가장 작은 노드부터 처리한다.
- 더 짧은 경로를 발견하면 업데이트한다.
- 결국 각 노드까지의 최소 비용이 보장된다.

이러한 이유 때문에 `PriorityQueue` 기반 알고리즘으로 바꾸는 것이 맞다.

## 6. 정석 코드
```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<int[]>[] graph = new ArrayList[n + 1];
        for (int i = 0; i <= n; i++) {
            graph[i] = new ArrayList<>();
        }

        for (int[] time : times) {
            int from = time[0];
            int to = time[1];
            int cost = time[2];
            graph[from].add(new int[]{to, cost});
        }

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, k});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int cost = cur[0];
            int node = cur[1];

            if (cost != dist[node]) continue;

            for (int[] next : graph[node]) {
                int nextNode = next[0];
                int nextCost = next[1];
                int newCost = cost + nextCost;

                if (newCost < dist[nextNode]) {
                    dist[nextNode] = newCost;
                    pq.offer(new int[]{newCost, nextNode});
                }
            }
        }

        int answer = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;
            answer = Math.max(answer, dist[i]);
        }

        return answer;
    }
}
```

## 7. 결론
내가 푼 방식은 핵심적으로는

- 최단거리 비교
- 갱신
- dist 유지

라는 아이디어를 이미 잡고 있었다.

하지만 이 문제는 가중치 그래프이기 때문에,

- FIFO queue로 처리하는 방식은 보장되지 않음
- `visit`를 일단 막는 방식도 위험함
- 그래서 `PriorityQueue` 기반의 Dijkstra가 정석

이 점이 가장 중요하다.

즉,

> 내가 푼 아이디어는 최단거리 개념으로는 맞았지만,
> 처리 순서와 자료구조가 BFS에 가깝고,
> 이 문제는 Dijkstra가 필요하다는 점이 핵심이다.

## 8. 다음 단계
다음에는 이 내용을 기반으로,

- 왜 BFS가 아닌지
- 왜 Dijkstra가 필요한지
- 작은 그래프 예시로 설명하는 방식

을 정리할 예정이다.
