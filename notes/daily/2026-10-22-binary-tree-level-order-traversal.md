# 2026-10-22 Binary Tree Level Order Traversal

## 오늘의 핵심
오늘은 Binary Tree Level Order Traversal 문제를 풀었다.

이 문제는 트리를 레벨 순서대로 탐색해서 각 층의 값을 리스트로 묶어 반환하는 문제다.

핵심 아이디어는 다음과 같다.

- 트리의 넓이 우선 탐색(BFS)을 사용한다.
- 큐에 노드를 넣고, 현재 큐의 크기만큼 순회한다.
- 같은 레벨의 노드들을 한 번에 처리해서 `List<Integer>`로 묶는다.
- 각 레벨이 끝나면 결과 리스트에 추가한다.

## 1. 문제 이해
문제는 트리를 각 층별로 순서대로 출력하는 것이다.

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
[[3], [9,20], [15,7]]
```

즉, 루트부터 한 층씩 보면서 각 층의 값들을 리스트로 담아야 한다.

## 2. 왜 BFS인가
이 문제는 깊이 우선 탐색(DFS)보다 넓이 우선 탐색(BFS)가 더 자연스럽다.

이유는 다음과 같다.

- 트리의 각 층을 함께 처리해야 한다.
- 루트 → 자식 → 손자 순서로 퍼지기 때문에, 큐로 한 레벨씩 관리하는 것이 편하다.
- `queue.size()`를 사용하면 현재 레벨의 노드 수를 정확히 알 수 있다.

## 3. BFS 구현
```java
import java.util.*;

class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new ArrayDeque<>();
        List<List<Integer>> result = new ArrayList<>();

        if (root == null) {
            return result;
        }

        queue.offer(root);

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();

                level.add(node.val);

                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }

            result.add(level);
        }

        return result;
    }
}
```

## 4. 핵심 포인트
### 1) 큐의 크기 기준으로 레벨 구분
```java
int size = queue.size();
```

이 코드는 "현재 레벨에 들어있는 노드 개수"를 의미한다.

그래서 다음 코드에서 `for (int i = 0; i < size; i++)`로 현재 레벨만 처리한다.

### 2) 각 레벨의 값들을 `level` 리스트에 담음
```java
List<Integer> level = new ArrayList<>();
```

같은 층에 있는 값들을 묶어서 `result`에 넣는다.

### 3) 자식 노드를 큐에 추가
```java
if (node.left != null) queue.offer(node.left);
if (node.right != null) queue.offer(node.right);
```

현재 노드를 처리한 뒤, 그 자식들을 다음 레벨로 넘긴다.

## 5. 시간 복잡도
- 각 노드를 한 번씩 방문한다.
- 모든 노드에 대해 `queue.offer`, `queue.poll`이 수행된다.

결과적으로:

- 시간 복잡도: `O(n)`
- 공간 복잡도: `O(n)`

여기서 `n`은 트리의 노드 수다.

## 6. DFS 참고 코드
DFS로도 풀 수 있지만, 이 문제에서는 `depth`를 같이 관리해야 한다.

```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        dfs(root, 0, result);
        return result;
    }

    private void dfs(TreeNode node, int depth, List<List<Integer>> result) {
        if (node == null) return;

        if (depth == result.size()) {
            result.add(new ArrayList<>());
        }

        result.get(depth).add(node.val);

        dfs(node.left, depth + 1, result);
        dfs(node.right, depth + 1, result);
    }
}
```

이 방식은 재귀 호출 시 깊이 정보를 함께 넘기고,
현재 깊이가 결과 리스트의 크기와 같으면 새 레벨을 만들고 값을 넣는다.

즉,

- `depth`가 현재 레벨을 의미한다.
- `result.get(depth)`를 통해 같은 레벨의 값을 묶는다.

이 방식은 BFS보다 구현이 조금 더 복잡하지만,
"깊이 정보를 이용해 레벨을 구분한다"는 점을 이해하는 데 도움이 된다.

## 7. 이 문제에서 중요한 사고 방식
이 문제의 핵심은 단순히 "트리를 순회하는 것"이 아니라,

"한 층씩 묶어서 순회한다"는 점이다.

즉,

- 노드를 하나씩 꺼내는 것이 아니라
- 현재 레벨의 노드들을 모두 꺼내서
- 하나의 리스트로 묶는다는 사고가 중요하다.

## 8. 오늘의 결론
Level Order Traversal은 트리의 너비 우선 탐색을 통해 아주 깔끔하게 해결할 수 있다.

핵심은 다음이다.

- 큐를 사용한다.
- 큐의 현재 크기를 기준으로 한 레벨씩 처리한다.
- 각 레벨의 값을 별도 리스트로 모은다.

이 패턴은 이후의 트리 문제에서도 자주 나온다.

특히 다음과 같은 문제에서 응용이 가능하다.

- Binary Tree Zigzag Level Order Traversal
- Binary Tree Right Side View
- Average of Levels in Binary Tree

오늘은 이 문제를 통해, 트리 문제에서 "레벨 단위의 처리"가 얼마나 중요한지 체감했다.
