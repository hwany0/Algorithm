# 2026-11-14 Word Ladder: 리팩토링 정리

## 오늘의 핵심
`127. Word Ladder`는 단순히 "단어를 바꾸는 문제"가 아니라,
"그래프에서 최단 경로를 찾는 문제"로 바꾸면 훨씬 이해가 쉬워진다.

내가 지금 푼 방식은 핵심 아이디어가 맞았다.

- 패턴 기반으로 인접 단어를 빠르게 찾는다.
- BFS로 최단거리를 계산한다.
- `dist`를 이용해서 더 짧은 경로만 유지한다.

다만 구현을 조금 더 정리하면 더 깔끔하고, 불필요한 부분을 줄일 수 있다.

## 1. 문제를 그래프로 바꾸는 관점
이 문제에서 상태는 다음과 같다.

- 상태: 현재 단어
- 전이: 한 글자만 다른 단어로 이동
- 시작 상태: `beginWord`
- 목표 상태: `endWord`

그래프 관점에서 보면,

```text
노드 = 단어
간선 = 한 글자만 다른 단어 사이의 연결
정답 = beginWord에서 endWord까지의 최단거리
```

이 문제는 최단 경로 문제이므로 BFS가 정석이다.

## 2. 왜 패턴 캐시가 중요한가
한 단어에서 다음 단어를 찾는 가장 naive한 방식은,
wordList 전체를 다 훑어보는 것이다.

```java
for (String word : wordList) {
    if (isOneLetterAway(current, word)) {
        ...
    }
}
```

이 방식은 최악의 경우 전체 리스트를 매번 순회하게 되므로 비효율적이다.

그래서 사용하는 것이 패턴 캐시다.

예를 들어:

```text
hit -> *it, h*t, hi*
```

이런 패턴을 키로 해두면,
같은 패턴을 가진 단어들을 바로 찾을 수 있다.

즉,

```text
"*it" -> ["hit", "bit", "sit", ...]
```

이런 식으로 묶어두면,
현재 단어와 한 글자만 다른 단어를 훨씬 빠르게 찾을 수 있다.

## 3. 내가 쓴 코드의 핵심 구조
내가 쓴 코드의 핵심은 다음이었다.

```java
Map<String, Set<String>> cache = new HashMap<>();
```

- `cache`는 패턴별로 연결된 단어를 저장한다.
- `dist`는 현재 단어까지의 최단거리다.
- `Queue`는 BFS 탐색 대기열이다.

이 구조가 맞다.

## 4. 왜 `dist` 업데이트 조건이 중요한가
핵심 조건은 이 부분이다.

```java
if (dist.get(cur) + 1 < dist.getOrDefault(next, Integer.MAX_VALUE)) {
    dist.put(next, dist.get(cur) + 1);
    q.offer(next);
}
```

이 뜻은 다음과 같다.

- 현재 단어 `cur`까지의 거리 = `dist.get(cur)`
- `cur`에서 `next`로 한 칸 이동하면 = `dist.get(cur) + 1`
- 만약 이 값이 기존의 `next`까지의 거리보다 더 작다면
- 더 짧은 경로를 찾은 것이므로 갱신한다.

즉,

> BFS는 최단거리를 관리하는 알고리즘이므로,
> 더 짧은 경로를 발견하면 갱신해야 한다.

이것이 정답을 유지하는 핵심이다.

## 5. 리팩토링한 버전
아래는 좀 더 정리한 버전이다.

```java
import java.util.*;

class Solution {
    private Map<String, Set<String>> patternMap = new HashMap<>();

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        if (!wordList.contains(endWord)) return 0;

        buildPatternMap(wordList);

        Queue<String> q = new ArrayDeque<>();
        Map<String, Integer> dist = new HashMap<>();

        q.offer(beginWord);
        dist.put(beginWord, 1);

        while (!q.isEmpty()) {
            String cur = q.poll();

            for (String next : getNextWords(cur)) {
                if (!dist.containsKey(next)) {
                    dist.put(next, dist.get(cur) + 1);
                    q.offer(next);

                    if (next.equals(endWord)) {
                        return dist.get(next);
                    }
                }
            }
        }

        return 0;
    }

    private void buildPatternMap(List<String> wordList) {
        for (String word : wordList) {
            char[] chars = word.toCharArray();
            for (int i = 0; i < chars.length; i++) {
                char original = chars[i];
                chars[i] = '*';
                String pattern = new String(chars);
                patternMap.computeIfAbsent(pattern, k -> new HashSet<>()).add(word);
                chars[i] = original;
            }
        }
    }

    private Set<String> getNextWords(String word) {
        Set<String> result = new HashSet<>();
        char[] chars = word.toCharArray();

        for (int i = 0; i < chars.length; i++) {
            char original = chars[i];
            chars[i] = '*';
            String pattern = new String(chars);
            result.addAll(patternMap.getOrDefault(pattern, Collections.emptySet()));
            chars[i] = original;
        }

        return result;
    }
}
```

## 6. 이 리팩토링의 장점
### 1) 불필요한 변수 제거
- `graph`를 별도로 만들지 않는다.
- 최단거리 계산만 `dist`로 처리한다.

### 2) BFS 종료 조건을 더 명확하게 만들었다
```java
if (next.equals(endWord)) {
    return dist.get(next);
}
```

다음 단어가 목표 단어면 즉시 종료할 수 있다.

### 3) 코드 흐름이 더 단순하다
- 패턴 생성
- 다음 단어 찾기
- BFS 확장
- 최단거리 계산

이 흐름이 명확하다.

## 7. 정리
핵심은 다음 3개다.

- 패턴 캐시로 다음 단어를 빠르게 찾는다.
- BFS로 최단거리를 계산한다.
- `dist`를 활용해서 더 짧은 경로만 유지한다.

그래프 관점에서 보면,

```text
문제 = 최단 경로 찾기
상태 = 단어
전이 = 한 글자만 다른 단어로 이동
시작점 = beginWord
목표점 = endWord
```

이 구조를 이해해야 이 문제를 제대로 푼다고 볼 수 있다.

## 8. 다음 문제는?
다음 문제는 `433. Minimum Genetic Mutation`이다.

이 문제도 같은 맥락이다.

- 상태 = 현재 gene 문자열
- 전이 = 한 글자만 다른 문자열로 변환
- 최단 거리 = BFS

즉, Word Ladder와 매우 유사한 패턴을 가진다.

이런 문제를 여러 번 풀면,
"문자열 상태 공간에서 BFS로 최단거리 찾기"라는 패턴이 체화된다.
