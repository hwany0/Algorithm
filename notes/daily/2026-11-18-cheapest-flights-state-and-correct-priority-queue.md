# 2026-11-18 Cheapest Flights: 상태와 우선순위 큐 이해

## 오늘 풀이 방향

오늘은 `787. Cheapest Flights Within K Stops`를 풀면서 사용한 방식과,
현재 풀이에서 부족했던 부분을 정리한다.

이 문제의 핵심은 다음과 같다.

- 가중치가 있는 최단경로 문제다.
- 최대 경유 횟수 `k`가 제한된다.
- 같은 도시라도 지금까지 사용한 항공권 수가 다르면 다른 상태다.

따라서 상태는 다음처럼 관리해야 한다.

```text
(현재 도시, 누적 비용, 사용한 항공권 수)
```

## 도시 하나만 보는 것은 부족한 이유

이 문제에서는 같은 도시를 여러 비용과 항공권 수로 도달할 수 있다.

```text
A -> B 비용 100, 항공권 1장
A -> C -> B 비용 50, 항공권 2장
```

같은 `B`지만 경로가 다르다.

- 비용이 작은 경로가 더 유리할 수 있다.
- 반대로 항공권 수가 더 적은 경로가 `k` 제한 안에서 유리할 수 있다.

그래서 `dist[도시]` 하나만 관리하면 경유 횟수 제한을 처리할 수 없다.

## 현재 풀이의 문제점

현재 시도에서는 `dist[n][k+2]`를 사용했다.

```java
int[][] dist = new int[n][k + 2];
```

여기서 배열의 두 번째 축은 항공권 수 또는 레벨로 사용할 수 있다.

하지만 큐에 들어간 값은 누적 비용이 아니라 방금 이동한 간선 비용이다.

```java
q.offer(new int[]{next, weight});
```

여기서 `weight`는 다음 값을 의미한다.

- 누적 비용이 아니다.
- `next`까지 이동한 전체 비용이 아니다.

따라서 큐에서 꺼낸 `node[1]`은 누적 비용으로 사용할 수 없다.

## 우선순위 큐에 들어가야 하는 상태

큐에는 다음 값을 함께 넣어야 한다.

```java
PriorityQueue<int[]> q;
// [도시, 누적 비용, 사용한 항공권 수]
```

예를 들어 시작 상태는 다음과 같다.

```java
q.offer(new int[]{src, 0, 0});
// city = src
// cost = 0
// usedFlights = 0
```

항공권을 한 번 사용할 때마다 사용한 항공권 수가 증가한다.

```text
A -> B
도시: B
누적 비용: 항공권 비용
사용한 항공권 수: 1
```

## 우선순위 큐 정렬 기준

누적 비용이 가장 작은 상태를 먼저 꺼내야 한다.

```java
PriorityQueue<int[]> q =
    new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
```

비교자 인덱스 의미는 다음과 같다.

```text
0: 도시
1: 누적 비용
2: 사용한 항공권 수
```

따라서 `a[1]`과 `b[1]`을 비교하는 것이 맞다.

기존의 다음 비교자는 반대로 정렬된다.

```java
(a, b) -> Integer.compare(b[1], a[1])
```

이 코드는 비용이 큰 상태를 먼저 꺼내므로 최소 비용 경로 탐색에 맞지 않는다.

## 최소 비용 갱신 배열

다음 배열을 사용한다.

```text
best[도시][사용한 항공권 수] = 해당 상태까지의 최소 누적 비용
```

예를 들어:

```java
best[nextCity][nextFlights]
```

의 의미는 다음과 같다.

- `nextCity`에 도달했다.
- 항공권을 `nextFlights`장 사용했다.
- 그 상태까지 도달한 최소 누적 비용을 저장한다.

인접 도시를 확장할 때는 다음처럼 갱신한다.

```java
int nextCost = cost + flightCost;
int nextFlights = usedFlights + 1;

if (nextCost < best[nextCity][nextFlights]) {
    best[nextCity][nextFlights] = nextCost;
    q.offer(new int[]{nextCity, nextCost, nextFlights});
}
```

## 경유 횟수와 항공권 수의 관계

문제의 `k`는 경유 횟수다.

항공권을 한 장 사용할 때마다 한 번 이동한다.

```text
항공권 수 = 이동 횟수 = 경유 횟수 + 1
최대 항공권 수 = k + 1
```

예를 들어 `k = 1`이라면 다음 경로까지 허용된다.

```text
A -> B
항공권 수: 1
경유 횟수: 0

A -> B -> C
항공권 수: 2
경유 횟수: 1
```

따라서 상태 확장 전에는 다음 조건을 확인한다.

```java
if (usedFlights == k + 1) continue;
```

이 의미는 항공권을 이미 최대 횟수만큼 사용했으므로 더 이동할 수 없다는 뜻이다.

## 최종 코드

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

        int[][] best = new int[n][k + 2];
        for (int i = 0; i < n; ++i) {
            Arrays.fill(best[i], Integer.MAX_VALUE);
        }

        PriorityQueue<int[]> q =
            new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));

        q.offer(new int[]{src, 0, 0}); // city, cost, usedFlights
        best[src][0] = 0;

        while (!q.isEmpty()) {
            int[] cur = q.poll();
            int city = cur[0];
            int cost = cur[1];
            int usedFlights = cur[2];

            if (city == dst) {
                return cost;
            }

            if (usedFlights == k + 1) {
                continue;
            }

            if (cost != best[city][usedFlights]) {
                continue;
            }

            for (int[] flight :
                 graph.getOrDefault(city, new ArrayList<>())) {
                int nextCity = flight[0];
                int flightCost = flight[1];
                int nextCost = cost + flightCost;
                int nextFlights = usedFlights + 1;

                if (nextCost < best[nextCity][nextFlights]) {
                    best[nextCity][nextFlights] = nextCost;
                    q.offer(new int[]{nextCity, nextCost, nextFlights});
                }
            }
        }

        return -1;
    }
}
```

## 꼭 기억할 부분

- 큐의 원소는 누적 비용을 포함한 상태다.
- 우선순위 큐는 누적 비용이 작은 상태를 먼저 꺼내야 한다.
- 상태는 `도시 + 사용한 항공권 수`다.
- 같은 도시라도 항공권 수가 다르면 별도 상태다.
- 최대 항공권 수는 `k + 1`이다.
- 목적지에 도달한 상태를 큐에서 꺼냈을 때 누적 비용이 가장 작으면 바로 반환할 수 있다.
