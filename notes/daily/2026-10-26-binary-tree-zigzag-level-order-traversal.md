# 2026-10-26 Binary Tree Zigzag Level Order Traversal

## 오늘의 핵심
오늘은 Binary Tree Zigzag Level Order Traversal 문제를 풀었다.

이 문제는 트리를 레벨 순서대로 방문하되, 각 층마다 방문 방향이 번갈아 바뀌는 문제다.

- 1번째 레벨: 왼쪽에서 오른쪽
- 2번째 레벨: 오른쪽에서 왼쪽
- 3번째 레벨: 왼쪽에서 오른쪽
- ...

즉, 같은 depth의 노드들을 리스트로 묶고, 레벨마다 반대 방향으로 정렬해주면 된다.

## 1. 문제 이해
예를 들어 트리가 아래와 같다면:

```text
    3
   / \
  9  20
     /  \
    15   7
```

결과는:

```text
[[3], [20, 9], [15, 7]]
```

처음에는 루트만 한 번 출력하고,
다음 레벨은 오른쪽부터 왼쪽 순서로 출력하는 것이 핵심이다.

## 2. 왜 BFS인가
이 문제는 각 층을 묶어서 처리해야 하므로 BFS가 가장 자연스럽다.

- 큐를 사용해 레벨 단위로 탐색한다.
- `queue.size()`로 현재 층의 노드 수를 확인한다.
- 각 층마다 값을 리스트에 담는다.
- 레벨이 짝수/홀수인지에 따라 뒤집기 여부를 결정한다.

## 3. 구현 코드
```java
import java.util.*;

class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>();

        Queue<TreeNode> queue = new ArrayDeque<>();
        List<List<Integer>> result = new ArrayList<>();
        queue.offer(root);

        boolean leftToRight = true;

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);

                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }

            if (!leftToRight) {
                Collections.reverse(level);
            }

            result.add(level);
            leftToRight = !leftToRight;
        }

        return result;
    }
}
```

## 4. 핵심 포인트
### 1) 레벨 단위 처리
```java
int size = queue.size();
```

현재 레벨에 있는 노드 수만큼만 처리한다.

### 2) 방향 전환
```java
boolean leftToRight = true;
...
leftToRight = !leftToRight;
```

이 값이 true면 왼쪽에서 오른쪽, false면 오른쪽에서 왼쪽으로 처리한다.

### 3) 한 레벨이 끝나면 뒤집기
```java
if (!leftToRight) {
    Collections.reverse(level);
}
```

현재 레벨의 리스트는 방향에 맞게 뒤집어 준다.

## 5. 시간 복잡도
- 모든 노드를 한 번씩 방문한다.
- 각 노드는 큐에 한 번 들어가고 한 번 빠져나온다.

결과적으로:

- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(n)`

## 6. 내가 푼 코드와의 차이
내가 처음 푼 방식도 핵심 아이디어는 같았지만,
방향 전환을 더 직관적으로 정리하면 아래처럼 깔끔하게 쓸 수 있다.

```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>();

        Queue<TreeNode> queue = new ArrayDeque<>();
        List<List<Integer>> result = new ArrayList<>();
        queue.offer(root);

        boolean leftToRight = true;

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);

                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }

            if (!leftToRight) {
                Collections.reverse(level);
            }

            result.add(level);
            leftToRight = !leftToRight;
        }

        return result;
    }
}
```

이 방식이 가장 보편적이고 이해하기 쉽다.

## 7. 오늘의 결론
이 문제는 "BFS + 레벨별 방향 전환"이 핵심이다.

- 큐로 한 레벨씩 탐색
- 같은 레벨을 리스트로 묶기
- 레벨마다 방향을 뒤집기

이 패턴을 익히면 비슷한 트리 레벨 문제들도 쉽게 풀 수 있다.

## 8. 왜 `Collections.reverse()`가 필요한가
이 문제에서 가장 헷갈렸던 부분은 "반대 방향으로 순회하려면 어떻게 해야 하나"였는데,
핵심은 "노드를 하나씩 반대로 꺼내는 것"이 아니라 "한 레벨의 리스트를 뒤집는 것"이다.

예를 들어 같은 레벨의 값이:

```text
[9, 20]
```

이고 현재 레벨이 오른쪽에서 왼쪽으로 출력되어야 하면,

```java
Collections.reverse(level);
```

을 적용해서:

```text
[20, 9]
```

로 바꾸면 된다.

즉,

- 왼쪽 → 오른쪽: 그대로 유지
- 오른쪽 → 왼쪽: 해당 레벨 리스트만 뒤집기

이게 가장 단순하고 안전한 방법이다.

## 9. `Collections` API 핵심 정리
문제 풀이에서 자주 쓰는 메서드를 정리하면 아래와 같다.

### 1) `Collections.reverse(List<?> list)`
리스트의 순서를 뒤집는다.

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
Collections.reverse(list); // [3, 2, 1]
```

### 2) `Collections.sort(List<T> list)`
오름차순 정렬

```java
Collections.sort(list);
```

### 3) `Collections.sort(List<T> list, Comparator<? super T> c)`
커스텀 정렬

```java
Collections.sort(list, (a, b) -> b - a); // 내림차순
```

### 4) `Collections.swap(List<?> list, int i, int j)`
두 원소 위치 교환

```java
Collections.swap(list, 0, 2);
```

### 5) `Collections.max(Collection<? extends T>)`
최대값 찾기

```java
int max = Collections.max(list);
```

### 6) `Collections.min(Collection<? extends T>)`
최소값 찾기

```java
int min = Collections.min(list);
```

### 7) `Collections.binarySearch(List<? extends Comparable<? super T>> list, T key)`
정렬된 리스트에서 이분 탐색

```java
Collections.sort(list);
int idx = Collections.binarySearch(list, 3);
```

### 8) `Collections.frequency(Collection<?> c, Object o)`
특정 값의 개수 계산

```java
int count = Collections.frequency(list, 2);
```

이 문제에서는 가장 핵심적으로 필요한 메서드는 `Collections.reverse()`이고,
그 다음으로는 `Collections.sort()` 정도를 이해하면 충분하다.

## 10. 오늘의 결론
이번 문제는 단순히 "거꾸로 탐색"이 아니라,

"한 레벨의 결과 리스트를 뒤집어 반대 방향 출력"이라는 사고가 핵심이었다.

즉,

- 레벨 단위로 묶기
- 방향 전환
- 해당 레벨만 `Collections.reverse()` 적용

이 패턴을 기억하면 비슷한 레벨 순회 문제를 훨씬 쉽게 풀 수 있다.
