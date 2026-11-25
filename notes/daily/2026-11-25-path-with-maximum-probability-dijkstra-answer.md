# 2026-11-25 Path with Maximum Probability: 정답 풀이와 핵심 정리

## 문제

LeetCode `1514. Path with Maximum Probability`

시작 노드에서 목적지 노드까지 이동할 때,
모든 간선을 통과할 확률의 곱이 최대가 되는 경로를 구하는 문제다.

## 핵심 상태

```text
(현재 노드, 시작점부터 현재 노드까지의 최대 확률)
```

이 문제는 누적 비용을 최단거리로 계산하는 문제와 비슷하다.
차이점은 다음과 같다.

```text
최단 거리: 누적 비용이 작아야 한다.
최대 확률: 누적 확률이 커야 한다.
```

## 확률 누적 방법

```text
A까지 도달한 확률 × A에서 B로 이동할 확률
= A를 거쳐서 B까지 도달한 확률
```

예를 들어:

```text
start -> A 확률: 0.5
A -> B 확률: 0.8
```

이면:

```text
start -> A -> B 확률 = 0.5 × 0.8 = 0.4
```

확률은 더해지는 것이 아니라 곱해진다.

## 그래프 구성

이 문제는 양방향 그래프다.

```text
[a, b, probability]
```

간선이 주어졌을 때 양쪽 방향을 모두 저장한다.

```java
map.computeIfAbsent(a, k -> new ArrayList<>())
   .add(new double[]{b, probability});

map.computeIfAbsent(b, k -> new ArrayList<>())
   .add(new double[]{a, probability});
```

## 최대 확률 배열

```java
double[] probability = new double[n];
probability[startNode] = 1.0;
```

나머지 노드의 초기값은 Java의 `double` 기본값인 `0.0`이다.

의미는 다음과 같다.

```text
probability[node]
= 현재까지 찾은 시작점에서 node까지의 최대 확률
```

최대 확률 문제이므로:

```java
if (nextProbability > probability[nextNode])
```

기존 확률보다 커질 때만 갱신한다.

## 우선순위 큐

최대 확률을 먼저 처리하려면 확률이 큰 상태를 먼저 꺼내야 한다.

```java
PriorityQueue<double[]> queue =
    new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));
```

큐 원소는 다음 정보를 가진다.

```text
[현재 노드, 시작점부터 현재 노드까지의 누적 확률]
```

시작 상태:

```java
queue.offer(new double[]{startNode, 1.0});
```

## 오래된 큐 상태 확인

같은 노드가 여러 번 큐에 들어갈 수 있다.
과거 확률을 가진 상태가 큐에 남아 있을 수 있으므로,
현재 큐에서 꺼낸 확률이 최신 배열 값과 일치하는지 확인한다.

```java
if (currentProbability != probability[node]) {
    continue;
}
```

같은 노드가 이미 확정된 최대 확률로 갱신되어 있다면,
이전 경로의 확률은 무시한다.

## 정답 코드

```java
class Solution {
    public double maxProbability(int n, int[][] edges,
                                 double[] succProb,
                                 int start_node, int end_node) {
        Map<Integer, List<double[]>> graph = new HashMap<>();

        for (int i = 0; i < edges.length; ++i) {
            int from = edges[i][0];
            int to = edges[i][1];
            double probability = succProb[i];

            graph.computeIfAbsent(from, k -> new ArrayList<>())
                 .add(new double[]{to, probability});

            graph.computeIfAbsent(to, k -> new ArrayList<>())
                 .add(new double[]{from, probability});
        }

        double[] maxProbability = new double[n];
        maxProbability[start_node] = 1.0;

        PriorityQueue<double[]> queue =
            new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));

        queue.offer(new double[]{start_node, 1.0});

        while (!queue.isEmpty()) {
            double[] current = queue.poll();
            int node = (int) current[0];
            double currentProbability = current[1];

            if (currentProbability != maxProbability[node]) {
                continue;
            }

            if (node == end_node) {
                return currentProbability;
            }

            for (double[] next :
                 graph.getOrDefault(node, new ArrayList<>())) {
                int nextNode = (int) next[0];
                double edgeProbability = next[1];
                double nextProbability =
                    currentProbability * edgeProbability;

                if (nextProbability > maxProbability[nextNode]) {
                    maxProbability[nextNode] = nextProbability;
                    queue.offer(new double[]{
                        nextNode, nextProbability
                    });
                }
            }
        }

        return 0.0;
    }
}
```

## 기존 풀이에서 잘못된 부분

### 1. 확률 배열 초기값

기존 코드는 다음처럼 나머지 노드도 `1.0`으로 초기화했다.

```java
Arrays.fill(dist, 1);
```

최대 확률에서는 나머지 노드를 `0.0`으로 둬야 한다.

```java
double[] maxProbability = new double[n];
maxProbability[startNode] = 1.0;
```

### 2. 우선순위 큐 방향

기존 큐는 확률이 작은 값부터 꺼낸다.

```java
(a, b) -> Double.compare(a[1], b[1])
```

최대 확률에서는 반대로 해야 한다.

```java
(a, b) -> Double.compare(b[1], a[1])
```

### 3. 갱신 비교 방향

최단 거리 문제:

```java
nextCost < dist[next]
```

최대 확률 문제:

```java
nextProbability > maxProbability[next]
```

### 4. 확률 곱하기

확률은 누적해서 곱한다.

```java
double nextProbability =
    currentProbability * edgeProbability;
```

### 5. 무방향 그래프

간선을 양쪽 방향 모두 저장해야 한다.

```text
a -> b
b -> a
```

### 6. `visited` 배열 사용 금지

이 문제에서는 일반적인 `visited`만으로 방문을 막으면 안 된다.

같은 노드를 더 큰 확률로 다시 방문할 수 있기 때문이다.

```text
같은 노드에 확률 0.6으로 먼저 도달
나중에 확률 0.8로 다시 도달 가능
```

대신 확률 배열로 갱신한다.

```java
if (nextProbability > maxProbability[nextNode])
```

## 복잡도

간선 수를 `E`, 노드 수를 `V`라고 하면:

```text
시간 복잡도: O(E log V)
공간 복잡도: O(V + E)
```

이 복잡도는 우선순위 큐를 사용한 최단경로 변형 풀이의 일반적인 복잡도다.

## 꼭 기억할 것

- 최대 확률은 확률을 곱해서 누적한다.
- 확률 배열의 나머지 초기값은 `0.0`이다.
- 큐는 확률이 큰 상태를 먼저 꺼낸다.
- 갱신 조건은 `nextProbability > maxProbability[next]`이다.
- 간선은 양방향으로 저장한다.
- 단순 `visited`보다 확률 배열 갱신이 중요하다.
