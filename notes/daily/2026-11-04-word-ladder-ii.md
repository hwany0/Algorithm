# 2026-11-04 Word Ladder II

## 오늘의 핵심
오늘은 Word Ladder II 문제를 풀었다.

이 문제는 시작 단어에서 끝 단어까지 한 번에 한 글자씩 바꾸면서 변환하는데,
모든 변환 과정이 단어 목록에 있고, 가장 짧은 경로를 모두 찾아야 하는 문제다.

핵심은 단순히 "한 번에 이동 가능한 단어를 찾는 것"이 아니라,
"최단 경로를 구하고, 그 경로들을 모두 모아야 한다"는 점이다.

예를 들어:

```text
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]
```

정답 경로는 다음처럼 여러 개가 될 수 있다.

```text
["hit","hot","dot","dog","cog"]
["hit","hot","lot","log","cog"]
```

즉, 이 문제는 단순히 최단 거리 하나를 찾는 문제가 아니라,
최단 경로를 모두 모아야 하는 문제다.

## 1. 문제 이해
규칙은 다음과 같다.

- 한 번에 한 글자만 바꿔서 바꾸는 것
- 바뀐 단어가 `wordList` 안에 있어야 함
- 각 단계는 변환 과정에서 하나의 단어를 의미함
- 시작 단어와 끝 단어는 포함됨
- 모든 경로 중 가장 짧은 길이를 찾아야 함

그리고 중요한 조건은,

- "가장 짧은 경로를 모두 찾는 것"
- 단순히 하나의 경로만 찾는 것이 아니라 여러 개가 존재할 수 있음

그래서 일반적인 BFS만으로는 부족하고,
"최단 거리 계산 + 경로 복원"이 필요하다.

## 2. 내가 짠 코드에서 어려운 점
네가 짠 코드의 핵심 아이디어는 꽤 좋았다.

- BFS로 거리 계산
- `map`에 다음 단어를 저장
- DFS로 경로 복원

하지만 이 문제는 문제 자체가 복잡해서,
몇 가지가 꼬이기 쉽다.

### 1) `dict.remove(beginWord)` 처럼 단어를 지우는 방식이 위험하다
단어를 제거하면, 같은 단어를 다시 방문할 수 없기 때문에,
경로를 복원할 때 필요한 정보가 사라질 수 있다.

특히 이 문제는 여러 경로를 찾는 것이 핵심이기 때문에,
단순히 단어를 리스트에서 제거하는 방식은 경로를 보존하는데 불리하다.

### 2) `levels`를 따로 관리하면서 `q`와 `dict`를 함께 바꾸는 로직이 복잡하다
이 부분은 BFS의 레벨 구분을 하려는 의도는 좋지만,
실제로는 경로를 추적하거나 최단 거리 기준을 관리하는 데 약해진다.

특히 이 문제는 최단 경로를 모두 찾는 문제라서,
"현재 층을 처리했다"는 정보와 "최단 거리 계산"이 섞이면 꼬이기 쉽다.

### 3) `map`에 저장된 경로가 재귀 DFS에서 사용되는 구조가 단순하지 않다
이 문제는 여러 경로를 생성할 수 있어서,
각 노드마다 "어디에서 왔는지"를 저장해두는 방식이 중요하다.

그런데 현재 코드처럼 단순하게 `map[word] = listOfNextWords`만 저장하면,
"어떤 경로가 실제로 최단 경로인지"를 정확히 판별하기 어렵다.

## 3. 정석 풀이 아이디어
이 문제는 보통 아래 2단계로 푼다.

### 1단계: BFS로 최단 거리 계산
- 시작 단어에서 각 단어까지 최단 거리를 계산한다.
- 이 때, 한 글자만 다른 단어들만 연결한다.
- `distance[word] = 최소 변환 단계 수`

### 2단계: DFS로 경로 복원
- `beginWord`에서 시작해서
- `endWord`까지 가는 경로를 따라가며
- 현재 단어의 다음 단어가 모두 `distance[next] == distance[current] + 1`인 경우만 탐색한다.
- 이 조건은 "다음 단어가 실제로 최단 경로를 구성하는 경로인지"를 보장한다.

이 방식이 핵심이다.

## 4. 패턴 생성 방식
이 문제에서 가장 많이 쓰는 아이디어는 다음이다.

단어를 한 글자씩 바꾸면서 패턴을 만든다.

예를 들어:

```text
"hit"
```

이 단어는 아래처럼 패턴으로 바꿀 수 있다.

```text
*it, h*t, hi*
```

이 패턴을 사용하면,
"hit"와 "hot" 같은 단어를 빠르게 찾을 수 있다.

예를 들어:

```java
String pattern = "h*t";
```

이런 패턴을 map으로 저장해두면,
같은 패턴을 가진 단어들을 한 번에 찾을 수 있다.

그래서 최적화가 가능하다.

## 5. 정답 코드
아래는 표준적인 BFS + DFS 조합 방식이다.

```java
import java.util.*;

class Solution {
    public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
        HashSet<String> dict = new HashSet<>(wordList);
        if (!dict.contains(endWord)) {
            return new ArrayList<>();
        }

        Map<String, Integer> distance = new HashMap<>();
        Map<String, List<String>> graph = new HashMap<>();

        Queue<String> queue = new ArrayDeque<>();
        queue.offer(beginWord);
        distance.put(beginWord, 0);

        while (!queue.isEmpty()) {
            String word = queue.poll();
            List<String> neighbors = getNeighbors(word, dict);

            for (String next : neighbors) {
                if (!distance.containsKey(next)) {
                    distance.put(next, distance.get(word) + 1);
                    queue.offer(next);
                }

                if (distance.get(word) + 1 == distance.getOrDefault(next, Integer.MAX_VALUE)) {
                    graph.computeIfAbsent(word, k -> new ArrayList<>()).add(next);
                }
            }
        }

        List<List<String>> result = new ArrayList<>();
        List<String> path = new ArrayList<>();
        path.add(beginWord);
        dfs(beginWord, endWord, distance, graph, path, result);

        return result;
    }

    private void dfs(String word, String endWord,
                     Map<String, Integer> distance,
                     Map<String, List<String>> graph,
                     List<String> path,
                     List<List<String>> result) {
        if (word.equals(endWord)) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (String next : graph.getOrDefault(word, new ArrayList<>())) {
            if (distance.getOrDefault(next, Integer.MAX_VALUE) == distance.get(word) + 1) {
                path.add(next);
                dfs(next, endWord, distance, graph, path, result);
                path.remove(path.size() - 1);
            }
        }
    }

    private List<String> getNeighbors(String word, Set<String> dict) {
        List<String> result = new ArrayList<>();
        char[] chars = word.toCharArray();

        for (int i = 0; i < chars.length; i++) {
            char original = chars[i];
            for (char c = 'a'; c <= 'z'; c++) {
                if (c == original) continue;

                chars[i] = c;
                String next = new String(chars);
                if (dict.contains(next)) {
                    result.add(next);
                }
            }
            chars[i] = original;
        }

        return result;
    }
}
```

## 6. 이 코드가 좋은 이유
이 코드는 다음을 모두 잘 지킨다.

- BFS로 최단 거리 계산
- 워딩 패턴 생성으로 빠른 탐색
- graph를 통해 각 단어의 다음 경로 저장
- DFS로 최단 경로만 따라가며 경로 수집
- 최단 경로 조건을 보장하면서 여러 경로 생성

즉, 이 문제는 단순히 한 번 BFS로 끝나는 문제가 아니라,
"최단 거리와 경로 복원"을 같이 고려해야 한다는 점이 핵심이다.

## 7. 왜 이 문제는 어렵나
이 문제는 어려운 이유가 크게 두 가지다.

### 1) 최단 경로를 찾는 것과 모든 경로를 찾는 것이 동시에 필요하다
단순히 BFS에서 가장 짧은 거리만 찾는 것으로는 부족하다.

모든 최단 경로를 찾기 위해서는,
각 노드에 대해 실제로 어떤 노드들이 최단 경로를 구성하는지 추적해야 한다.

### 2) 단어 탐색을 단순하게 하면 시간 초과가 날 수 있다
단어 수가 크면, 모든 단어를 매번 순회하는 방식은 느리다.

그래서 패턴 기반 접근이 중요하다.

## 8. 내가 배운 포인트
이 문제를 푸는 과정에서 가장 중요했던 건 다음이다.

- BFS는 거리 계산용
- DFS는 경로 복원용
- "최단 경로 조건"을 DFS에서 반드시 체크해야 함
- 직접 단어를 지우거나 비효율적으로 순회하면 경로 복원이 꼬일 수 있음

즉,

- 단어를 찾는 과정은 BFS로 최단 거리 계산
- 경로를 모으는 과정은 DFS로 최단 경로 복원

이 두 단계의 구분이 핵심이었다.

## 9. 오늘의 결론
Word Ladder II는 단순한 BFS 문제처럼 보이지만,
실제로는 최단 경로를 계산하고, 그 경로들을 모두 복원하는 문제다.

그래서 핵심은 다음이다.

- BFS로 거리 계산
- 다음 경로를 연결하는 그래프 구성
- DFS에서 `distance[next] == distance[current] + 1` 조건을 체크
- 최단 경로만 탐색해서 결과 수집

이 문제는 "거리 계산 + 경로 복원"이라는 두 가지를 동시에 이해해야 해서 어렵다.

하지만 패턴을 익히면, 골치 아픈 문제가 아니라
"정석 BFS + backtracking" 패턴으로 해결할 수 있는 문제다.
