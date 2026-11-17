# 2026-11-17 Cheapest Flights Within K Stops: 오늘 풀이 초안

## 오늘의 핵심
`787. Cheapest Flights Within K Stops`는
`743. Network Delay Time`와 같은 가중치 최단경로 문제다.

다만 이 문제는 추가로,

- 경유 횟수 제한 `k`가 있다.
- 즉, 시작점에서 목적지까지 도달할 때
  최대 `k`번 경유할 수 있다.

이 조건 때문에 그냥 무조건 `Dijkstra`만 쓰는 건 아니고,
"거리 + 경유 횟수"를 상태로 관리해야 한다는 점이 추가된다.

## 1. 문제를 상태로 바꾸는 관점
이 문제는 상태를 다음처럼 정의해야 한다.

```text
상태 = (현재 도시, 경유 횟수)
```

왜냐하면,

- 도시만 봐서는 경유 제한을 체크할 수 없기 때문
- 그래서 현재까지 얼마나 경유했는지도 같이 관리해야 함

그래프적으로는,

```text
노드 = 도시
간선 = 항공권 비용
거리 = 항공권 비용의 누적 합
제한 = 경유 횟수 <= k
정답 = src -> dst까지의 최소 비용
```

이 문제는 가중치 그래프 최단경로 문제다.

## 2. 내가 푼 코드의 핵심 구조
내가 푼 코드의 핵심은 아래와 같다.

```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[] dist = new int[n];
        Map<Integer, List<int[]>> map = new HashMap<>();
        int INF = Integer.MAX_VALUE;
        Arrays.fill(dist, INF);

        for(int i = 0; i < flights.length; ++i) {
            int from = flights[i][0];
            int to = flights[i][1];
            int w = flights[i][2];
            map.computeIfAbsent(from, x -> new ArrayList<>()).add(new int[]{to, w});
        }

        PriorityQueue<int[]> q = new PriorityQueue<>((a,b) -> Integer.compare(b[1], a[1]));
        q.offer(new int[]{src, 0});
        dist[src] = 0;

        int level = 0;
        boolean end = false;
        while(!q.isEmpty() && !end) {
            int size = q.size();
            for(int trial = 0; trial < size; ++trial) {
                int[] node = q.poll();
                int cur = node[0];
                int wei = node[1];
                for(int[] nexts : map.getOrDefault(cur, new ArrayList<>())) {
                    int next = nexts[0];
                    int weight = nexts[1];

                    if(dist[cur] + weight < dist[next]) {
                        dist[next] = dist[cur] + weight;
                        q.offer(new int[]{next, weight});
                    }
                }
            }
            if(level++ == k) end = true;
        }
        return dist[dst] == INF ? -1 : dist[dst];
    }
}
```

## 3. 현재 코드에서 맞는 점
### 1) 최소 비용 갱신 로직 자체는 맞다
```java
if(dist[cur] + weight < dist[next]) {
    dist[next] = dist[cur] + weight;
}
```

이건 최단거리 문제에서 핵심적으로 맞는 패턴이다.

- 현재 도시까지의 비용 `dist[cur]`
- 다음 항공권 비용 `weight`
- 합이 더 작으면 갱신

이것은 Dijkstra의 기본 아이디어와 동일하다.

### 2) `PriorityQueue`를 쓰려는 의도는 맞다
```java
PriorityQueue<int[]> q = new PriorityQueue<>((a,b) -> Integer.compare(b[1], a[1]));
```

이건 cost 기준으로 가장 작은 값부터 처리하려는 의도다.

즉, 비용이 낮은 경로부터 처리하려는 점은 좋다.

## 4. 왜 아직 완전히 맞는 건 아닌가
핵심 문제는 다음과 같다.

### 1) 경유 횟수 `k`를 상태로 관리하지 않았다
이 문제는 단순히 “최소 비용”만 보면 안 된다.

```text
현재 도시 + 지금까지 경유한 횟수
```

이 둘이 상태여야 한다.

왜냐하면,

- 같은 도시라도 경유 횟수가 다르면
- 이후로 더 갈 수 있는 조건이 달라진다.

그래서 단순히 `dist[next]`만 가지고는 부족하다.

### 2) `level` 기준으로 종료하는 방식이 문제다
```java
if(level++ == k) end = true;
```

이건 “k번째 레벨에서 끝”처럼 보이지만,
경유 횟수 제한은 보통 다음과 같이 처리해야 한다.

```text
현재까지 사용한 경로 길이(간선 수) <= k + 1
```

즉, 경유 횟수는 실제로는 경로 길이와 함께 관리해야 한다.

### 3) 큐의 우선순위 기준이 잘못되어 있다
현재 코드에서는

```java
PriorityQueue<int[]> q = new PriorityQueue<>((a, b) -> Integer.compare(b[1], a[1]));
```

이 코드의 정렬 기준은 가장 큰 비용부터 꺼내는 방향이라서,
최소 비용 경로를 먼저 처리하려는 의도와 반대다.

이건 우선순위 큐를 잘못 쓰고 있는 부분이다.

## 5. 핵심 문제의 정답 구조
이 문제는 보통 다음 형태로 푼다.

```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[][] dp = new int[k + 2][n];
        for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE);
        dp[0][src] = 0;

        for (int step = 1; step <= k + 1; step++) {
            for (int[] flight : flights) {
                int from = flight[0];
                int to = flight[1];
                int cost = flight[2];

                if (dp[step - 1][from] != Integer.MAX_VALUE) {
                    dp[step][to] = Math.min(dp[step][to], dp[step - 1][from] + cost);
                }
            }
        }

        int ans = Integer.MAX_VALUE;
        for (int step = 0; step <= k + 1; step++) {
            ans = Math.min(ans, dp[step][dst]);
        }

        return ans == Integer.MAX_VALUE ? -1 : ans;
    }
}
```

이 방식은 상태를

```text
(dp[step][city])
```

로 관리한다.

즉,

- `step` = 지금까지 몇 번 경유했는가
- `city` = 현재 도시
- `dp[step][city]` = 그 상태까지의 최소 비용

이 방식은 경유 횟수 제한을 정확하게 관리한다.

## 6. 정리
지금 내가 푼 초안의 핵심 아이디어는 맞다.

- 비용 비교
- 우선순위 큐를 사용하려는 의도
- 최소 비용 업데이트

이런 부분은 정답 방향이다.

하지만 경유 제한을 상태로 관리하지 않고,
단순히 level로 종료하는 방식은 정확하지 않다.

그래서 다음 단계는,

- 상태를 `(도시, 경유 횟수)`로 잡기
- `dp` 또는 `dist + stops` 같이 관리하기
- 큐 우선순위 기준을 비용 기준으로 정확히 맞추기

이 부분을 정리하는 것이 필요하다.

## 7. 다음 단계
다음에는 이 초안을 기준으로,

- 왜 `k`를 상태로 포함해야 하는지
- 왜 `PriorityQueue` 정렬 기준이 `비용`이어야 하는지
- 왜 `level` 종료 조건이 틀린지

를 실제 예시를 들어서 설명할 예정이다.
