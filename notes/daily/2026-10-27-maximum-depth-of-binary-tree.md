# 2026-10-27 Maximum Depth of Binary Tree

## 오늘의 핵심
오늘은 Maximum Depth of Binary Tree 문제를 풀었다.

이 문제는 트리의 가장 깊은 노드까지의 깊이를 구하는 문제다.

예를 들어 아래 트리의 깊이는 3이다.

```text
    3
   / \
  9  20
    /  \
   15   7
```

결과는:

```text
3
```

핵심은 "루트에서 가장 깊은 리프 노드까지의 거리"를 구하는 것이다.

## 1. 문제 이해
트리의 깊이는 보통 다음처럼 생각한다.

- 루트 노드의 깊이는 1 또는 0 둘 다 정의할 수 있다.
- 보통 LeetCode에서는 루트 깊이를 1로 취급한다.
- 이 문제는 보통 `root`가 있는 경우 깊이를 1부터 세게 된다.

즉, 아래처럼 개념적으로 보면 된다.

- 루트: 깊이 1
- 자식: 깊이 2
- 손자: 깊이 3

## 2. DFS로 푼 방식
내가 먼저 생각한 방법은 재귀 기반 DFS였다.

```java
class Solution {
    public int maxDepth(TreeNode root) {
        return dfs(root, 0);
    }

    public int dfs(TreeNode n, int depth) {
        if (n == null) return depth;
        return Math.max(dfs(n.left, depth + 1), dfs(n.right, depth + 1));
    }
}
```

### DFS 의 핵심 아이디어
재귀적으로 트리를 내려가면서 깊이를 증가시킨다.

- `n == null`이면 더 이상 내려갈 수 없으므로 현재 깊이를 반환한다.
- 왼쪽 자식과 오른쪽 자식 각각의 깊이를 구한다.
- 두 값 중 더 큰 값을 선택한다.

### 왜 이게 맞는가
어떤 노드에서 최종 깊이는 다음과 같이 결정된다.

- 왼쪽 서브트리의 최대 깊이
- 오른쪽 서브트리의 최대 깊이
- 둘 중 큰 값 + 1

즉,

```text
maxDepth(node) = 1 + max(maxDepth(left), maxDepth(right))
```

이 공식이 핵심이다.

## 3. BFS로 푼 방식
다음으로는 BFS로도 풀 수 있다.

```java
class Solution {
    public int maxDepth(TreeNode root) {
        Queue<TreeNode> q = new ArrayDeque<>();
        if (root == null) return 0;

        q.offer(root);
        int depth = 0;

        while (!q.isEmpty()) {
            int size = q.size();

            for (int i = 0; i < size; i++) {
                TreeNode node = q.poll();

                if (node.left != null) q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }

            depth++;
        }

        return depth;
    }
}
```

### BFS 의 핵심 아이디어
BFS는 레벨 단위로 트리를 탐색한다.

- 루트가 있는 첫 레벨: 깊이 1
- 자식 레벨: 깊이 2
- 그 다음: 깊이 3

큐에서 한 레벨이 끝날 때마다 `depth`를 증가시키면,
마지막으로 계산된 깊이가 트리의 최대 깊이가 된다.

즉,

- 한 층씩 처리한다.
- 한 층을 다 처리하면 깊이를 1 증가시킨다.
- 마지막 층까지 다 처리한 뒤의 `depth`가 정답이다.

## 4. DFS vs BFS
### DFS
- 재귀 구조가 직관적이다.
- 문제의 정의상 "왼쪽/오른쪽 서브트리의 최대값"을 비교하는 방식과 잘 맞는다.
- 구현이 간단하고, 트리 문제에서 많이 사용된다.

### BFS
- 층별 탐색이기 때문에, 각 레벨을 한 번씩 처리하는 구조와 잘 맞는다.
- 같은 레벨을 한 번에 처리할 수 있어서 레벨 문제에 유리하다.
- 이 문제에서는 최대 깊이를 층 수로 측정할 수 있어서 자연스럽다.

## 5. 시간 복잡도
### DFS
- 각 노드를 한 번씩 방문한다.
- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(h)`
  - `h`는 트리 높이
  - 재귀 스택 때문에 사용됨

### BFS
- 각 노드를 한 번씩 방문한다.
- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(w)`
  - `w`는 가장 넓은 레벨의 노드 수

즉, 두 방식 모두 최종적으로는 `O(n)`이다.

## 6. 내 코드에서의 포인트
내가 짠 코드에는 다음 포인트가 중요하다.

### DFS 코드
```java
return Math.max(dfs(n.left, ++depth), dfs(n.right, depth));
```

이 부분은 값이 깊이로 누적되면서 재귀적으로 내려가고,
왼쪽과 오른쪽 중 더 큰 값을 선택하는 구조다.

여기서 `++depth`를 쓸 때는 조심해야 하며,
가독성을 위해 보통 아래처럼 쓰는 편이 더 명확하다.

```java
return Math.max(dfs(n.left, depth + 1), dfs(n.right, depth + 1));
```

이 방식이 더 직관적이고 실수도 적다.

### BFS 코드
```java
int size = q.size();
for (int i = 0; i < size; ++i) {
    TreeNode n1 = q.poll();
    if (n1.left != null) q.offer(n1.left);
    if (n1.right != null) q.offer(n1.right);
}
depth++;
```

이 구조는 한 레벨을 다 처리한 뒤에 깊이를 증가시키는 방식이다.

즉,

- 이 레벨 끝 → depth 증가
- 다음 레벨 처리

이 규칙으로 마지막까지 진행하면 최대 깊이가 나온다.

## 7. 오늘의 결론
Maximum Depth of Binary Tree는 트리의 최대 깊이를 구하는 대표 문제다.

내가 두 방식 모두 풀어본 것처럼,

- DFS: 재귀로 가장 깊은 경로를 비교
- BFS: 레벨 단위로 카운트

둘 다 정답을 만들 수 있다.

이 문제에서는 보통 DFS가 더 직관적이고 간단하게 풀리며,
BFS는 레벨 접근을 연습하기 좋다.

중요한 점은,

- 재귀는 각 노드에서 왼쪽/오른쪽 서브트리의 최대값을 비교한다.
- BFS는 각 레벨을 처리하면서 depth를 증가시킨다.

이 두 흐름을 이해하면 트리 문제를 훨씬 쉽게 접근할 수 있다.
