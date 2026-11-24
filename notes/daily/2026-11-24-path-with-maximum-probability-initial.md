# 2026-11-24 Path with Maximum Probability: 초기 풀이

## 오늘 푼 문제

LeetCode `1514. Path with Maximum Probability`

## 오늘 처음 작성한 코드

```java
class Solution {
    public double maxProbability(int n, int[][] edges, double[] succProb,
                                 int start_node, int end_node) {
        Map<Integer, List<double[]>> map = new HashMap<>();
        double[] dist = new double[n];
        Arrays.fill(dist, 1);

        for (int i = 0; i < edges.length; ++i) {
            int a = edges[i][0];
            int b = edges[i][1];
            double w = succProb[i];

            map.computeIfAbsent(a, k -> new ArrayList<>())
               .add(new double[]{b, w});
        }

        PriorityQueue<double[]> q =
            new PriorityQueue<>((a, b) -> Double.compare(a[1], b[1]));

        q.offer(new double[]{start_node, 1.0, 0.0});
        dist[start_node] = 1.0;

        while (!q.isEmpty()) {
            double[] node = q.poll();
            int start = (int) node[0];
            double weight = node[1];

            for (double[] nexts :
                 map.getOrDefault(start, new ArrayList<>())) {
                int next = (int) nexts[0];
                double nextWeight = nexts[1];

                if (dist[start] * nextWeight < dist[next]) {
                    dist[next] = dist[start] * nextWeight;
                    q.offer(new double[]{next, dist[next], 0.0});
                }
            }
        }

        return -1;
    }
}

```

## 초기 코드의 방향

오늘 풀이에서 우선 다음 구조를 잡았다.

- 그래프를 인접 리스트로 만들기
- 시작점의 누적 확률을 `1.0`으로 설정하기
- 우선순위 큐로 최댓값을 탐색하기
- 연결된 두 노드의 확률을 곱해서 다음 확률 만들기

핵심 상태는 다음과 같다.

```text
(현재 노드, 누적 확률)
```

## 초기 구현에서 확인할 부분

### 1. 우선순위 큐 정렬 방향

최대 확률을 먼저 처리하려면 누적 확률이 큰 상태를 먼저 꺼내야 한다.

```java
PriorityQueue<double[]> q =
    new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));
```

현재 초기 코드는 다음과 같이 작은 값부터 꺼내는 방향이다.

```java
(a, b) -> Double.compare(a[1], b[1])
```

최대 확률 문제이므로 큐 정렬 방향은 반대로 적용해야 한다.

### 2. 거리 배열의 초기값

최대 확률을 저장하려면 초기값은 `0.0`이어야 한다.

```java
Arrays.fill(prob, 0.0);
prob[start_node] = 1.0;
```

현재 초기 코드는 다음처럼 초기값을 `1.0`으로 채워 두었다.

```java
Arrays.fill(dist, 1);
```

`1.0`은 시작점만 설정하고, 나머지 노드는 모두 `0.0`으로 두는 것이 자연스럽다.

### 3. 확률 갱신 방향

최대 확률 문제에서는 누적 확률이 커지면 갱신해야 한다.

```java
if (prob[start] * nextWeight > prob[next]) {
    prob[next] = prob[start] * nextWeight;
}
```

현재 초기 코드는 다음과 같이 비교 방향이 반대다.

```java
if (dist[start] * nextWeight < dist[next])
```

최소 거리 문제에서는 `<`이고,
최대 확률 문제에서는 `>`이다.

### 4. 무방향 그래프 처리

항공편과 달리 이 문제의 간선은 양방향이다.

```text
[a, b, probability]
```

이면 다음 두 방향을 모두 저장해야 한다.

```text
a -> b
b -> a
```

### 5. 목적지 도착 확률 반환

목적지의 최대 확률은 다음과 같이 반환한다.

```java
if (node == end_node) {
    return prob[end_node];
}
```

현재 우선순위 큐 구현은 목적지에 도착해도 루프를 계속 돌 가능성이 있으므로,
최종적으로는 `prob[end_node]`를 반환하거나 목적지 노드에서 바로 반환해야 한다.

## 오늘의 핵심 방향

이 문제는 가중치 최단경로의 변형이다.

다만 다음 비교 방향이 다르다.

```text
최단거리: 누적 값이 작아야 갱신
최대확률: 누적 값이 커야 갱신
```

또한 확률은 곱하기로 누적된다.

```text
A까지 확률 × A에서 B로 이동할 확률
= A에서 B까지의 누적 확률
```
