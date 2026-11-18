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

## 우선순위 큐에서 실제로 꺼내는 순서 이해

항공편 목록과 큐에 넣는 상태는 의미가 다르다.

항공편 목록:

```text
[출발 도시, 도착 도시, 비용]
[[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]]
```

이 항공편 목록을 그래프처럼 보면 다음과 같다.

```text
0 -> 1 비용 100
1 -> 2 비용 100
2 -> 0 비용 100
1 -> 3 비용 600
2 -> 3 비용 200
```

반면 우선순위 큐에는 다음 상태를 넣는다.

```text
[현재 도시, 누적 비용, 사용한 항공권 수]
```

예를 들어 `src = 0`, `dst = 3`, `k = 2`라면 시작 상태는:

```text
[0, 0, 0]
```

큐에서 누적 비용이 작은 상태를 먼저 꺼내므로, 처리 과정은 대략 다음과 같다.

```text
큐에서 [0, 0, 0] 꺼냄
-> 1번 항공편으로 [1, 100, 1] 생성

큐에서 [1, 100, 1] 꺼냄
-> [2, 200, 2] 생성
-> [3, 700, 2] 생성

현재 큐
[2, 200, 2]
[3, 700, 2]

누적 비용 기준으로 [2, 200, 2]가 먼저 꺼냄
-> [3, 400, 3] 생성
```

여기서 중요한 점은 다음과 같다.

- 항공편 입력 배열의 숫자는 출발 도시, 도착 도시, 비용이다.
- 큐에 들어가는 숫자는 현재 도시, 누적 비용, 사용 항공권 수다.
- 큐는 항공편 수가 작은 순으로 처리하지 않는다.
- 큐는 누적 비용이 작은 상태를 먼저 꺼낸다.
- 항공권 수가 다른 레벨도 모두 같은 큐에 들어갈 수 있다.

이 예시의 두 목적지 후보는 다음과 같다.

```text
0 -> 1 -> 3
누적 비용: 100 + 600 = 700
항공권 수: 2

0 -> 1 -> 2 -> 3
누적 비용: 100 + 100 + 200 = 400
항공권 수: 3
```

따라서 항공권 수가 2장인 경로보다 항공권 수가 3장인 경로가 더 싼 경우가 있다.
우선순위 큐는 항공권 수를 먼저 비교하지 않고 누적 비용을 먼저 비교한다.

### 레벨별로 넣는다는 표현과 우선순위 큐의 차이

다음 코드는 인접 상태를 큐에 넣는 동작이다.

```java
for (int[] flight : graph.getOrDefault(city, new ArrayList<>())) {
    int nextCity = flight[0];
    int nextCost = cost + flight[1];
    int nextFlights = usedFlights + 1;

    if (nextCost < best[nextCity][nextFlights]) {
        best[nextCity][nextFlights] = nextCost;
        q.offer(new int[]{nextCity, nextCost, nextFlights});
    }
}
```

예를 들어 현재 상태가 다음이라면:

```text
[1, 100, 1]
```

인접 항목을 넣어 생성된 상태는:

```text
[2, 200, 2]
[3, 700, 2]
```

둘 다 같은 레벨이므로 다음에 가장 작은 비용인 상태가 먼저 나올 수 있다.

단, 큐에는 이미 다른 레벨의 상태가 함께 있을 수 있다.
예를 들어 현재 큐에 다음이 있다면:

```text
[2, 200, 2]
[4, 150, 1]
[3, 700, 3]
```

우선순위 큐는 레벨 순서가 아니라 누적 비용 순서로 꺼낸다.

```text
[4, 150, 1]
[2, 200, 2]
[3, 700, 3]
```

즉:

```text
같은 레벨 내부에서는 누적 비용 순서로 선택됨
다른 레벨이 섞여 있어도 누적 비용 순서로 선택됨
레벨 순서가 강제로 고정되는 것이 아니다
```

레벨을 엄격하게 처리하려면 `q.size()`를 저장하고 그 라운드의 상태만 처리하는 방식이 필요하다.
하지만 `787`의 우선순위 큐 구현에서는 상태를 각 레벨로 나누고도,
모든 레벨의 상태를 누적 비용 기준으로 하나의 큐에서 처리한다.

핵심은 다음 한 줄이다.

```java
q.offer(new int[]{nextCity, nextCost, nextFlights});
```

이 코드는 "다음 레벨에서만 처리하자"가 아니다.
이 코드는 "누적 비용이 `nextCost`이고 항공권 수가 `nextFlights`인 새로운 상태를 전체 우선순위 큐에 넣자"라는 의미다.
