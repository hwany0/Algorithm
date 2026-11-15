# 2026-11-15 BFS: visited vs dist의 구분

## 오늘의 핵심
오늘은 BFS 문제에서

- `visited`만으로도 충분한 경우가 있는지
- 언제 `dist`가 필요한지
- 그리고 언제 목표를 처음 보고도 최단거리라고 보장할 수 있는지

를 정리했다.

핵심은 다음이다.

- `visited`는 "이미 처리했는가?"를 나타낸다.
- `dist`는 "이 상태까지의 최소 거리/비용은 얼마인가?"를 나타낸다.
- BFS는 간선 비용이 모두 1인 unweighted 그래프에서 최단거리 보장을 한다.
- 그래서 이 경우에는 목표를 처음 발견한 순간이 최단거리다.

## 1. visited와 dist의 차이
### visited
```java
visited.add(cur)
```

이건 단순히 이미 처리한 상태인지 여부를 체크하는 용도다.

- 이미 처리했다면 다시 보지 않는다.
- 중복 방문을 막는다.
- 하지만 최단거리의 값 자체를 저장하지는 않는다.

### dist
```java
dist.put(next, dist.get(cur) + 1);
```

이건 상태까지의 최소 거리값을 직접 저장하는 용도다.

- 어떤 상태로 가는 더 짧은 경로를 발견했는지 비교한다.
- 최소 비용이나 최소 단계 수를 기록한다.
- 여러 경로가 같은 상태에 도달할 수 있을 때 중요하다.

## 2. 언제 visited만으로 충분한가
visited만으로 충분한 대표적인 경우는 다음과 같다.

- 입력 그래프가 unweighted이다.
- 한 번 이동할 때 비용이 모두 1이다.
- BFS 레벨 자체가 거리다.
- 처음 목표 상태를 발견하면 그게 최단거리다.

대표 예시:

- `127. Word Ladder`
- `433. Minimum Genetic Mutation`
- `752. Open the Lock`

이 문제들은 모두 간선 비용이 1이기 때문에,
BFS의 레벨이 거의 그대로 거리다.

예시 코드 패턴:

```java
Queue<String> q = new ArrayDeque<>();
Set<String> visited = new HashSet<>();
int level = 0;

q.offer(start);
visited.add(start);

while (!q.isEmpty()) {
    int size = q.size();
    for (int i = 0; i < size; i++) {
        String cur = q.poll();
        if (cur.equals(target)) return level;

        for (String next : getNext(cur)) {
            if (!visited.add(next)) continue;
            q.offer(next);
        }
    }
    level++;
}
```

이 방식이 가능한 이유는,

> BFS는 같은 거리끼리 묶어서 확장하기 때문이다.

그래서 목표를 처음 발견하는 순간이 최단거리다.

## 3. 언제 dist가 필요한가
`dist`가 필요한 경우는 보통 다음이다.

- 간선 비용이 1보다 다를 수 있다.
- 같은 상태에 도달하는 경로가 여러 개다.
- 더 짧은 경로를 계속 갱신해야 한다.
- 최소 비용을 직접 저장해야 한다.

예시:

- weighted graph
- 다익스트라 계열
- 특정 상태까지의 비용을 추적해야 하는 문제

그리고 BFS에서도 다음 조건을 적용할 때 `dist`가 유용하다.

```java
if (dist.get(cur) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
    dist.put(next, dist.get(cur) + 1);
    q.offer(next);
}
```

이 코드는,

- 현재 상태까지의 거리 + 1 이 더 짧다면 갱신하겠다
- 같은 상태가 여러 경로로 도달할 수 있으니 최소거리만 남기겠다

라는 의미다.

## 4. 목표를 처음 발견했을 때 최단인가?
이 질문의 핵심은 다음이다.

> 이 문제의 그래프가 unweighted shortest path 문제인가?

만약 맞다면,

- BFS는 거리 0, 1, 2, ... 순서로 확장한다.
- 따라서 목표를 처음 발견하는 순간이 최단거리다.

그래서 `visited`만으로 충분하다.

반대로,

- 가중치가 있거나
- 비용이 다른 경로가 존재하거나
- 더 짧은 경로가 나중에 발견될 수 있으면

처음 발견한 경로가 최단이라고 보장할 수 없다.

그럴 때는 `dist` 비교가 필요하다.

## 5. 이 문제들에서의 판단
### 127. Word Ladder
- 한 번 변환 비용 = 1
- BFS 레벨 = 거리
- 처음 목표를 발견하면 최단거리
- 따라서 visited + level로도 가능하다.

### 433. Minimum Genetic Mutation
- 한 번 변환 비용 = 1
- 같은 이유로 BFS 레벨 = 거리
- 처음 목표를 발견하면 최단거리
- visited + level도 가능하다.

### 752. Open the Lock
- 한 번 조작할 때 비용 = 1
- BFS 레벨 = 회전 횟수
- 처음 목표를 발견하면 최단 회전 횟수
- visited + level으로도 가능하다.

즉, 이 문제들은 모두 unweighted shortest path의 특성을 가진다.

## 6. 실제 실전 기준
이 문제를 풀 때는 보통 다음 기준으로 생각하면 된다.

- 간선 비용이 모두 1이면 BFS 레벨이 거리다.
- 이 경우 목표를 처음 발견하면 정답이다.
- 그래서 visited + level이 충분하다.

- 하지만 거리 비교가 필요하거나, 비용이 섞여 있으면 dist가 필요하다.

## 7. 정리
핵심은 다음 한 문장이다.

> `visited`는 "이미 본 상태인지"를 체크하고,
> `dist`는 "현재까지의 최소 거리"를 기록한다.

그리고,

> 목표를 처음 발견해도 최단거리라고 보장할 수 있느냐는,
> 이 문제가 unweighted BFS 문제인지 여부에 달려 있다.

이 문제들은 모두 unweighted BFS 문제라서,
처음 목표를 발견하는 순간이 최단거리다.

그래서 visited만으로도 충분히 처리할 수 있다.

## 8. 다음 문제는?
다음 문제는 `752. Open the Lock`이다.

이 문제는 지금까지의 패턴을 그대로 이어받는다.

- 상태 = 현재 비밀번호 조합
- 전이 = 한 자리 숫자 변경
- 최단 횟수 = BFS 레벨
- visited + level로도 충분히 처리 가능

즉, 지금까지 쌓아온 개념이 바로 다음 문제로 이어진다.
