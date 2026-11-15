# 2026-11-15 Open the Lock: 오늘 풀이 정리

## 오늘의 핵심
오늘은 `752. Open the Lock`를 풀었다.

이 문제는 지금까지 한 `Word Ladder`와 `Minimum Genetic Mutation`과 같은 계열이다.

- 상태 = 현재 비밀번호 조합
- 전이 = 한 자리만 위/아래로 바꾸는 조합 이동
- 비용 = 한 번 이동할 때마다 1
- 목표 = target 비밀번호까지 도달하는 최소 횟수

즉, 이 문제도 최단거리 문제이고, BFS가 정석이다.

## 1. 상태와 상태공간
### 상태
상태는 현재 조합이다.

```text
현재 상태: "0000"
다음 상태: "0001", "0010", "0100", "1000" 등
```

한 자리만 바꾸면 다른 조합으로 이동할 수 있다.

그래서 상태는:

```text
현재 비밀번호 문자열
```

이다.

### 상태공간
상태공간은 가능한 모든 비밀번호 조합들의 집합이다.

- 4자리 숫자 조합
- 각 자리는 0~9
- 전체 경우의 수는 10^4

그래프 관점으로 보면,

- 노드 = 조합
- 간선 = 한 자리만 바뀌는 연결
- 정답 = 시작 상태에서 target까지의 최단 경로 길이

## 2. 왜 BFS인지
이 문제는 최단 횟수를 구하는 문제다.

- 한 번의 이동 비용 = 1
- 각 조합은 같은 비용으로 연결된다.
- 따라서 레벨별로 확장하는 BFS가 가장 자연스럽다.

즉,

```text
BFS 레벨 = 현재까지의 이동 횟수
```

이 된다.

## 3. 내가 푼 방식의 핵심
내가 푼 방식은 다음과 같은 아이디어였다.

```java
class Solution {
    public int openLock(String[] deadends, String target) {
        Queue<String> q = new ArrayDeque<>();
        List<String> dead = new ArrayList<>(Arrays.asList(deadends));
        if(dead.contains(target)) return -1;

        Map<String, Integer> dist = new HashMap<>();
        q.offer(target);
        dist.put(target, 0);

        while(!q.isEmpty()) {
            String node = q.poll();
            for(String next : getNext(node)){
                if(dist.containsKey(next)) continue;
                if(dead.contains(next)) continue;
                if(dist.get(node) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
                    dist.put(next, dist.get(node) + 1);
                    q.offer(next);
                }
            }
        }
        return dist.getOrDefault("0000", -1);
    }

    public Set<String> getNext(String node) {
        Set<String> result = new HashSet<>();
        char[] n = node.toCharArray();
        for(int i = 0; i < n.length; ++i) {
            char digit = n[i];
            int up = plus(digit - '0');
            int down = minus(digit - '0');
            n[i] = (char) (up + '0');
            result.add(new String(n));

            n[i] = (char) (down + '0');
            result.add(new String(n));
            
            n[i] = digit;
        }
        return result;
    }

    public int plus(int i) {
        return (i + 1) % 10;
    }
    public int minus(int i) {
        return i == 0 ? 9 : i - 1;
    }
}
```

핵심 포인트는 다음과 같다.

- `target`을 시작점으로 잡았다.
- 비밀번호는 한 자리씩 위/아래로 움직일 수 있다.
- `getNext()`는 현재 조합에서 가능한 다음 상태를 모두 만든다.
- `dist`는 지금까지의 최소 이동 횟수를 저장한다.
- deadend는 건너뛴다.

## 4. 왜 target을 시작점으로 잡아도 되는가
이게 헷갈리는 부분인데,
핵심은 그래프가 양방향이라는 점이다.

- 한 조합에서 다음 조합으로 가는 이동이 가능하면,
- 그 반대 방향도 가능하다.
- 즉, 조합 그래프는 양방향 그래프다.

그래서:

```text
0000 -> target 까지의 최단거리
=
target -> 0000 까지의 최단거리
```

이 성립한다.

즉, `target`에서 시작해도,
최종적으로 `0000`까지의 거리값을 구하면, 문제의 정답과 동일하다.

## 5. 왜 `dist` 비교가 필요한가
이 문제에서도 같은 상태가 여러 경로를 통해 도달할 수 있다.

예를 들어:

```text
0000 -> 0001 -> 0011 -> 0012
0000 -> 0010 -> 0011 -> 0012
```

같은 조합 `0012`를 여러 경로로 도달할 수 있다.

그래서 최단거리를 보장하려면:

```java
if (dist.get(node) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
    dist.put(next, dist.get(node) + 1);
    q.offer(next);
}
```

이 조건이 필요하다.

즉,

- 더 짧은 경로를 찾았으면 갱신
- 그 상태를 다시 탐색해 더 짧은 경로를 찾는 작업을 계속함

이게 최단거리 BFS의 핵심이다.

## 6. 왜 중복 방문을 막아야 하는가
`visited` 또는 `dist`를 사용해서 어느 상태를 이미 처리했는지 관리해야 한다.

그 이유는,

- 같은 조합은 여러 경로로 다시 도달할 수 있음
- 반복 탐색하면 아주 비효율적이기 때문

그래서 보통은 다음처럼 처리한다.

```java
if (dist.containsKey(next)) continue;
```

또는

```java
if (!visited.add(next)) continue;
```

이 구조가 중요하다.

## 7. 더 전형적인 방식과의 차이
보통은 시작점을 `"0000"`으로 잡고 BFS를 돌리는 방식이 더 직관적이다.

```java
Queue<String> q = new ArrayDeque<>();
Set<String> visited = new HashSet<>();
q.offer("0000");
visited.add("0000");
int steps = 0;
```

이 방식은 `Open the Lock`을 더 직관적으로 설명한다.

다만, 그래프가 양방향이므로,
`target`에서 시작해도 동일한 최단거리를 계산할 수 있다.

즉, 둘 다 맞는 방식이다.

## 8. 정리
오늘 푼 방식의 핵심은 다음이다.

- 상태는 조합 문자열
- 한 자리만 바꾸는 조합을 전이로 본다.
- BFS를 통해 최소 거리 계산
- `deadends`는 건너뛴다.
- 그래프가 양방향이라서 `target`에서 시작해도 같다.
- `dist`를 통해 더 짧은 경로만 유지한다.

즉,

> `Open the Lock`은 상태공간 그래프에서 최단거리 찾기 문제다.

이 문제를 이해하면,
앞서 배운 `Word Ladder`, `Minimum Genetic Mutation`과 같은 패턴이 이어진다.

## 9. 다음 문제
다음에는 추가적인 BFS 응용 문제를 이어서 다루면 된다.

하지만 이 시점에서 중요한 건,

- 상태를 그래프로 보는 사고
- BFS에서 거리와 방문을 어떻게 다루는지
- 목표를 처음 발견했을 때 최단거리인지 보장하는 이유

를 이해하는 것이다.

이 3가지는 다른 BFS 문제에서도 동일하게 적용된다.
