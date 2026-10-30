# 2026-10-30 Binary Tree Level Order Traversal II

## 오늘의 핵심
오늘은 Binary Tree Level Order Traversal II 문제를 풀었다.

이 문제는 트리를 레벨 순서대로 순회하는데, 결과를 아래에서 위로 쌓아야 하는 문제다.

즉,

- 보통은 위에서 아래로 순회한다.
- 이 문제는 아래에서 위로 순회해서 결과를 반환해야 한다.

예를 들어 트리가 아래와 같다면:

```text
    3
   / \
  9  20
    /  \
   15   7
```

일반 레벨 순회 결과는:

```text
[[3], [9, 20], [15, 7]]
```

이 문제에서는 아래에서 위로 출력해야 하므로:

```text
[[15, 7], [9, 20], [3]]
```

이 된다.

## 1. 문제 이해
핵심은 "같은 레벨의 값들을 묶고, 그 묶음을 거꾸로 붙이는 것"이다.

즉, BFS로 각 레벨을 처리하고,
그 결과를 앞쪽에 넣으면 자동으로 아래에서 위로 정렬된다.

## 2. 내가 푼 방식
```java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        return bfs(root);
    }

    public List bfs(TreeNode root) {
        if (root == null) return new ArrayList<>();

        List result = new LinkedList<>();
        Queue<TreeNode> q = new ArrayDeque<>();
        q.offer(root);

        while (!q.isEmpty()) {
            List<Integer> list = new ArrayList<>();
            int size = q.size();

            for (int i = 0; i < size; ++i) {
                TreeNode n = q.poll();
                list.add(n.val);

                if (n.left != null) q.offer(n.left);
                if (n.right != null) q.offer(n.right);
            }

            result.add(0, list);
        }

        return result;
    }
}
```

## 3. 왜 `result.add(0, list)`가 되는가
이 부분이 핵심이다.

`result`는 아래에서 위로 쌓아야 하므로,
매번 새 레벨을 리스트의 가장 앞에 넣는다.

예를 들어 현재 결과가:

```text
[[3]]
```

다음 레벨이 `[9, 20]`이면,

```java
result.add(0, [9, 20]);
```

을 하면 결과는:

```text
[[9, 20], [3]]
```

이 된다.

다음 레벨 `[15, 7]`도 같은 방식으로 앞에 추가하면:

```text
[[15, 7], [9, 20], [3]]
```

최종적으로 아래에서 위로 정렬된다.

## 4. `ArrayList`와 `LinkedList` 차이
오늘 공부했던 핵심은 여기다.

### `ArrayList`
- 내부적으로 배열 기반으로 동작한다.
- 인덱스 접근이 빠르다.
- 중간 삽입/삭제가 상대적으로 느리다.
- 보통 `get(i)`, `set(i, value)`가 빠르다.

```java
List<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);
```

### `LinkedList`
- 노드 기반으로 연결되어 있다.
- 앞/뒤 삽입이 빠르다.
- 중간에 삽입/삭제가 유리하다.
- 인덱스 접근은 상대적으로 느리다.

```java
List<Integer> list = new LinkedList<>();
list.add(10);
list.add(20);
```

## 5. `add(index, value)`의 의미
`List`는 `add(index, value)`를 지원한다.

```java
list.add(0, value);
```

이 코드는 리스트의 맨 앞에 값을 추가한다.

즉,

- `add(0, x)` → 가장 앞에 삽입
- `add(1, x)` → 앞에서 두 번째 위치에 삽입
- `add(list.size(), x)` → 마지막에 추가

이 성질 때문에 이 문제에서 아주 유용하다.

예시:

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(0, 0); // [0, 1, 2]
```

이렇게 특정 위치에 넣을 수 있다는 점이 매우 중요하다.

## 6. `Collections.reverse()`를 쓰는 방식
처음에는 결과를 위에서 아래로 만들고, 마지막에 뒤집는 방식도 가능하다.

```java
Collections.reverse(result);
```

이렇게 하면 같은 결과를 얻을 수 있다.

예시:

```java
List<List<Integer>> result = [[3], [9, 20], [15, 7]];
Collections.reverse(result);
// [[15, 7], [9, 20], [3]]
```

즉,

- 앞에 계속 넣는 방식
- 마지막에 reverse 하는 방식

둘 다 같은 의미이다.

어떤 방식이 더 직관적이냐에 따라 선택하면 된다.

## 7. 코드 정리 버전
보통은 아래처럼 더 깔끔하게 정리한다.

```java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        if (root == null) return new ArrayList<>();

        Queue<TreeNode> queue = new ArrayDeque<>();
        queue.offer(root);

        List<List<Integer>> result = new ArrayList<>();

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);

                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }

            result.add(0, level);
        }

        return result;
    }
}
```

이 방식은 가장 직관적이고,
"앞에 붙이는 것"으로 아래에서 위로 정렬하는 방식이다.

## 8. 시간 복잡도
- 모든 노드를 한 번씩 방문한다.
- 각 레벨에 대해서 리스트를 만들고, 앞에 추가한다.

결과적으로:

- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(n)`

여기서 `n`은 트리의 노드 수이다.

`add(0, list)`는 `LinkedList`에서 더 유리하지만,
보통 트리 문제에서는 `ArrayList` 기반으로 구현하는 것이 더 일반적이다.

## 9. 오늘의 결론
오늘 문제의 핵심은 다음 두 가지였다.

1. BFS로 레벨별로 값을 묶는다.
2. 결과 리스트의 앞에 추가해서 아래에서 위로 정렬한다.

그리고 오늘 배운 것처럼,

- `ArrayList`는 인덱스 접근에 유리
- `LinkedList`는 앞/뒤 삽입에 유리
- `add(0, value)`는 리스트의 맨 앞에 삽입한다.

이 개념을 이해하면 리스트를 다루는 문제에서 훨씬 쉽게 생각할 수 있다.

특히 이 문제는 `add(0, list)`를 활용해서 아주 자연스럽게 아래에서 위로 정렬할 수 있었다.
