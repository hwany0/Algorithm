# 2026-11-14 Minimum Genetic Mutation: 리팩토링 정리

## 오늘의 핵심
`433. Minimum Genetic Mutation`은 
`127. Word Ladder`와 거의 같은 형태의 문제다.

- 상태 = 현재 gene 문자열
- 전이 = 한 글자만 다른 문자열로 변환
- 최단 변환 횟수 = BFS 레벨
- 목표 = `endGene`까지의 최소 변환 횟수

즉, 이 문제도 "문자열 상태 공간에서 최단거리 찾기" 문제다.

## 1. 문제를 그래프로 바꾸는 관점
문자열 하나하나가 상태다.

```text
노드 = gene 문자열
간선 = 한 글자만 다른 gene 문자열 사이의 연결
정답 = startGene에서 endGene까지의 최단 거리
```

그래서 BFS가 정석이다.

- 시작 상태 = `startGene`
- 목표 상태 = `endGene`
- 한 번의 변환 = 간선 비용 1
- 최단 변환 횟수 = BFS 거리

## 2. 내가 푼 코드의 핵심 아이디어
내가 푼 방식은 다음과 같다.

```java
Map<String, Integer> dist = new HashMap<>();
Queue<String> q = new ArrayDeque<>();

q.offer(startGene);
dist.put(startGene, 0);

while (!q.isEmpty()) {
    String cur = q.poll();
    for (String next : _bank) {
        boolean isNext = getNext(cur, next);
        if (isNext) {
            if (dist.get(cur) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
                dist.put(next, dist.get(cur) + 1);
                q.offer(next);
            }
        }
    }
}
```

이 구조는 핵심적으로 맞다.

- `dist`는 문자열까지의 최소 변환 횟수
- `Queue`는 탐색 대상
- `getNext()`는 한 글자만 다른지 판별

즉, 문제를 최단거리 그래프 문제로 바라본 것이 맞다.

## 3. 한 글자만 다른지 판별하는 방식
내가 쓰는 `getNext()`는 다음과 같다.

```java
public boolean getNext(String a, String b) {
    int size = a.length();
    for(int i = 0; i < size; ++i) {
        if(a.charAt(i) != b.charAt(i)) {
            return a.substring(i+1).equals(b.substring(i+1));
        }
    }
    return true;
}
```

이 로직의 의미는,

- 맨 처음 다른 위치를 찾는다.
- 그 이후의 문자열이 모두 같은지 확인한다.
- 모두 같으면 정확히 한 글자만 다르다는 뜻이다.

즉,

```text
A = ACGT
B = AGGT
```

여기서 첫 문자가 다르지만, 뒤의 문자열이 같은 경우,
정확히 한 글자만 다르다.

그래서 이 로직은 맞다.

다만 더 직관적인 버전은 아래처럼 쓰는 게 좋다.

```java
public boolean isOneMutation(String a, String b) {
    int diff = 0;
    for (int i = 0; i < a.length(); i++) {
        if (a.charAt(i) != b.charAt(i)) diff++;
        if (diff > 1) return false;
    }
    return diff == 1;
}
```

이 버전은 한 글자 차이 판별을 더 쉽게 이해할 수 있다.

## 4. 왜 BFS인지
이 문제는 최소 변환 횟수를 구하는 문제다.

- 한 번 변환 = 비용 1
- 현재 상태에서 여러 다음 상태가 있을 수 있음
- 목표까지의 최소 횟수 = 최단 거리

따라서 BFS가 가장 적합하다.

정리하면:

```text
한 번 변환 = 간선 비용 1
최단 변환 횟수 = BFS 레벨
```

## 5. 왜 dist 갱신 조건이 중요한가
핵심 조건은 다음이다.

```java
if (dist.get(cur) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
    dist.put(next, dist.get(cur) + 1);
    q.offer(next);
}
```

이 뜻은,

- 현재 문자열 `cur`까지의 비용이 `dist.get(cur)`다.
- 여기서 한 번 변환하면 `+1`이 된다.
- 새 경로가 기존보다 더 짧다면 업데이트한다.

즉,

> 더 짧은 경로를 발견하면 갱신하고,
> 그렇지 않으면 무시한다.

이게 최단거리 유지의 핵심이다.

## 6. 리팩토링 버전
아래는 조금 더 정리한 버전이다.

```java
import java.util.*;

class Solution {
    public int minMutation(String startGene, String endGene, String[] bank) {
        Set<String> bankSet = new HashSet<>(Arrays.asList(bank));
        if (bankSet.isEmpty() || !bankSet.contains(endGene)) return -1;

        Queue<String> q = new ArrayDeque<>();
        Map<String, Integer> dist = new HashMap<>();

        q.offer(startGene);
        dist.put(startGene, 0);

        while (!q.isEmpty()) {
            String cur = q.poll();

            for (String next : bankSet) {
                if (!isOneMutation(cur, next)) continue;

                if (!dist.containsKey(next)) {
                    dist.put(next, dist.get(cur) + 1);
                    q.offer(next);
                }
            }
        }

        return dist.getOrDefault(endGene, -1);
    }

    private boolean isOneMutation(String a, String b) {
        int diff = 0;
        for (int i = 0; i < a.length(); i++) {
            if (a.charAt(i) != b.charAt(i)) diff++;
            if (diff > 1) return false;
        }
        return diff == 1;
    }
}
```

## 7. 이 리팩토링의 장점
### 1) `getNext()`가 더 직관적이다
- 한 글자만 다른지 판별이 훨씬 읽기 쉬움
- 로직이 추상적이지 않고 명확함

### 2) `dist.containsKey(next)`로 체크하는 게 더 단순하다
- 이미 방문한 상태면 큐에 넣지 않는다.
- 이미 더 짧은 경로가 있다면 또 처리할 필요가 없음

### 3) BFS 의도 자체가 더 선명하다
- 큐에서 꺼낸 문자열을 기준으로 인접 상태를 모두 찾는다.
- 최단 변환 횟수를 계산한다.

## 8. 정리
내가 푼 코드의 핵심 아이디어는 맞았다.

- 문자열을 상태로 보고 그래프 문제로 접근
- BFS로 최단 변환 횟수를 계산
- `dist`로 더 짧은 변환 경로를 유지
- `isOneMutation()`으로 한 글자 차이를 판별

이 구조는 `127. Word Ladder`와 같은 패턴이다.

즉, 이 계열 문제는

> 문자열 상태 공간에서 BFS로 최단거리를 찾는 문제

라고 보면 된다.

## 9. 다음 문제는?
다음 문제는 `752. Open the Lock`이다.

이 문제도 거의 같은 패턴이다.

- 상태 = 현재 비밀번호 조합
- 전이 = 한 자리만 움직여서 다음 조합 생성
- 최단 횟수 = BFS

즉,

- 문자열 상태 공간 문제
- BFS 최단거리 문제
- 변환 가능 상태를 그래프로 보는 문제

이 패턴이 계속 이어진다.
