# 2026-11-20 Cheapest Flights: 다시 푼 우선순위 큐 풀이

## 오늘 푼 방향

`787. Cheapest Flights Within K Stops`를 다시 풀면서 상태를 다음처럼 잡았다.

```text
(현재 도시, 누적 비용, 사용한 항공권 수)
```

이 문제는 단순히 도시의 최소 비용만 저장하면 안 된다.
같은 도시라도 항공권 수가 다르면 남은 경로 제한이 달라지기 때문이다.

## 큐와 거리 배열의 역할

```java
PriorityQueue<int[]> q =
    new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));

q.offer(new int[]{src, 0, 0});
// [현재 도시, 누적 비용, 사용한 항공권 수]
```

우선순위 큐는 누적 비용이 작은 상태를 먼저 꺼내는 역할을 한다.

```java
int[][] dist = new int[n][k + 2];
```

거리 배열은 다음 의미를 가진다.

```text
dist[도시][사용한 항공권 수]
= 그 상태로 도달한 최소 누적 비용
```

시작 상태는 다음과 같다.

```text
dist[src][0] = 0
```

항공권을 한 장 사용할 때마다 레벨이 1 증가한다.

```text
레벨 0: 항공권 0장 사용
레벨 1: 항공권 1장 사용
레벨 2: 항공권 2장 사용
레벨 k + 1: 항공권 k + 1장 사용
```

## 간선 갱신

현재 도시와 항공권 수에 대해 인접 항공편을 확인한다.

```java
int nextCost = dist[now][level] + nextW;

if (nextCost < dist[next][level + 1]) {
    dist[next][level + 1] = nextCost;
    q.offer(new int[]{next, cost + nextW, level + 1});
}
```

여기서 중요한 것은 큐에 넣는 누적 비용은 거리 배열과 같은 값이어야 한다는 점이다.

가장 자연스러운 구현은 다음과 같다.

```java
int nextCost = cost + nextW;

if (nextCost < dist[next][level + 1]) {
    dist[next][level + 1] = nextCost;
    q.offer(new int[]{next, nextCost, level + 1});
}
```

직접 푼 코드의 다음 형태도 거의 같은 의미다.

```java
q.offer(new int[]{next, cost + nextW, level + 1});
```

다만 두 방식은 큐 상태가 거리 배열과 일치할 때만 안전하다.
따라서 다음과 같이 오래된 큐 항목을 제거하는 코드도 권장된다.

```java
if (cost != dist[now][level]) {
    continue;
}
```

왜냐하면 우선순위 큐에는 같은 상태가 여러 번 들어갈 수 있고,
그중 과거 비용으로 들어온 항목은 최신 최소 비용과 다를 수 있기 때문이다.

## 목적지에서 바로 반환하는 이유

다음 코드는 목적지 상태를 큐에서 꺼내면 즉시 반환한다.

```java
if (dst == now) return cost;
```

우선순위 큐는 누적 비용이 작은 상태를 먼저 꺼낸다.
따라서 목적지가 여러 레벨에서 발견되더라도,
첫 번째로 꺼내는 목적지 상태의 누적 비용이 그때까지 발견된 목적지 후보 중 최소값이다.

예를 들어 목적지 후보가 다음과 같다면:

```text
목적지, 비용 700, 항공권 2장
목적지, 비용 400, 항공권 3장
목적지, 비용 500, 항공권 1장
```

우선순위 큐는 비용순으로 꺼내므로:

```text
목적지, 비용 400
목적지, 비용 500
목적지, 비용 700
```

순서로 처리한다.

## 최대 항공권 수 확인

시작 상태의 항공권 수는 `0`이다.

```text
항공권 수 = k일 때 최대 경유 k번에 도달한 상태
항공권 수 = k + 1일 때 도착한 상태까지 포함한 상태
```

현재 코드는 다음과 같이 상태 확장을 막는다.

```java
if (k + 1 == level) continue;
```

즉 항공권을 `k + 1`장 사용한 상태는 목적지면 먼저 반환하고,
목적지가 아니면 더 이동하지 않는다.

예를 들어 `k = 1`이면 항공권은 최대 2장까지 사용할 수 있다.

```text
레벨 0 -> 시작점
레벨 1 -> 항공권 1장 사용
레벨 2 -> 항공권 2장 사용
```

## 최종 후보 비교

바로 목적지를 반환하지 않고 모든 레벨을 확인하려면 다음과 같이 한다.

```java
int min = INF;

for (int level = 0; level <= k + 1; ++level) {
    min = Math.min(dist[dst][level], min);
}

return min == INF ? -1 : min;
```

이 코드는 목적지의 레벨별 최소 비용을 모두 비교한다.

```text
dist[dst][0]
dist[dst][1]
dist[dst][2]
...
dist[dst][k + 1]
```

## 다시 푼 핵심 코드

```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights,
                                 int src, int dst, int k) {
        Map<Integer, List<int[]>> graph = new HashMap<>();

        for (int[] flight : flights) {
            int from = flight[0];
            int to = flight[1];
            int cost = flight[2];

            graph.computeIfAbsent(from, x -> new ArrayList<>())
                 .add(new int[]{to, cost});
        }

        int[][] dist = new int[n][k + 2];
        int INF = Integer.MAX_VALUE;

        for (int i = 0; i < n; ++i) {
            Arrays.fill(dist[i], INF);
        }

        PriorityQueue<int[]> q =
            new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));

        q.offer(new int[]{src, 0, 0});
        dist[src][0] = 0;

        while (!q.isEmpty()) {
            int[] node = q.poll();
            int now = node[0];
            int cost = node[1];
            int level = node[2];

            if (cost != dist[now][level]) {
                continue;
            }

            if (now == dst) {
                return cost;
            }

            if (level == k + 1) {
                continue;
            }

            for (int[] next : graph.getOrDefault(now, new ArrayList<>())) {
                int nextCity = next[0];
                int flightCost = next[1];
                int nextCost = cost + flightCost;
                int nextLevel = level + 1;

                if (nextCost < dist[nextCity][nextLevel]) {
                    dist[nextCity][nextLevel] = nextCost;
                    q.offer(new int[]{nextCity, nextCost, nextLevel});
                }
            }
        }

        return -1;
    }
}
```

## 오늘 풀이에서 기억할 점

- 큐 원소는 `도시 + 누적 비용 + 사용 항공권 수`다.
- 우선순위 큐는 누적 비용이 작은 상태를 먼저 꺼낸다.
- `dist[도시][항공권 수]`에는 그 상태의 최소 비용을 저장한다.
- 오래된 큐 항목은 `cost != dist[now][level]`로 제거할 수 있다.
- 목적지에 도달한 상태는 우선순위 큐에서 가장 작은 누적 비용으로 먼저 나온다.
- 항공권 수는 최대 `k + 1`장까지 사용할 수 있다.
- 최종적으로 목적지의 모든 레벨 결과 중 최소값을 직접 비교할 수도 있다.
