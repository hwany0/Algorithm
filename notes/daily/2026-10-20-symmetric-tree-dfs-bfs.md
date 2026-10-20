# 2026-10-20 Symmetric Tree + Same Tree 복습

## 오늘의 핵심
오늘은 트리 비교 문제를 이어서 풀었다.

- Same Tree: 두 트리의 구조와 값이 완전히 같은지 비교
- Symmetric Tree: 트리가 좌우 대칭인지 비교

핵심 아이디어는 같은 위치의 노드를 비교하는 것.

## 1. Same Tree 복습
이 문제는 두 트리의 구조와 값을 비교하는 문제다.

### DFS
```java
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        if (p.val != q.val) return false;

        return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
    }
}
```

### BFS
```java
import java.util.*;

class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        Queue<TreeNode[]> queue = new ArrayDeque<>();
        queue.offer(new TreeNode[]{p, q});

        while (!queue.isEmpty()) {
            TreeNode[] pair = queue.poll();
            TreeNode n1 = pair[0];
            TreeNode n2 = pair[1];

            if (n1 == null && n2 == null) continue;
            if (n1 == null || n2 == null) return false;
            if (n1.val != n2.val) return false;

            queue.offer(new TreeNode[]{n1.left, n2.left});
            queue.offer(new TreeNode[]{n1.right, n2.right});
        }

        return true;
    }
}
```

## 2. Symmetric Tree
Symmetric Tree는 "거울처럼 대칭인지"를 확인하는 문제다.

핵심은:
- 왼쪽 자식과 오른쪽 자식을 서로 비교해야 한다.
- `left`와 `right`를 반대로 비교해야 함

즉,
- `p.left` 와 `q.right` 비교
- `p.right` 와 `q.left` 비교

## DFS 구현
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        // 거꾸로 같은 것 left = right 인것
        return dfs(root.left, root.right);
    }

    public boolean dfs(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        if (p.val != q.val) return false;

        return dfs(p.left, q.right) && dfs(p.right, q.left);
    }
}
```

### DFS의 의미
이 코드는 문제를 다음처럼 이해한다.

- 왼쪽 노드의 왼쪽 자식은 오른쪽 노드의 오른쪽 자식과 같아야 함
- 왼쪽 노드의 오른쪽 자식은 오른쪽 노드의 왼쪽 자식과 같아야 함

즉,
- `p.left` vs `q.right`
- `p.right` vs `q.left`

이 두 쌍을 비교하는 것.

## BFS 구현
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        Queue<TreeNode[]> que = new ArrayDeque<>();
        que.offer(new TreeNode[]{root.left, root.right});

        while (!que.isEmpty()) {
            TreeNode[] pair = que.poll();
            TreeNode n1 = pair[0];
            TreeNode n2 = pair[1];

            if (n1 == null && n2 == null) continue;
            if (n1 == null || n2 == null) return false;
            if (n1.val != n2.val) return false;

            que.offer(new TreeNode[]{n1.left, n2.right});
            que.offer(new TreeNode[]{n1.right, n2.left});
        }
        return true;
    }
}
```

## BFS에서 중요한 포인트
BFS는 같은 위치의 두 노드를 쌍으로 비교한다.

- `n1.left` 와 `n2.right`
- `n1.right` 와 `n2.left`

이 구조를 큐에 넣는 방식이 핵심이다.

즉,
- 왼쪽 자식의 왼쪽은 오른쪽 자식의 오른쪽과 같아야 하고
- 왼쪽 자식의 오른쪽은 오른쪽 자식의 왼쪽과 같아야 한다.

## Same Tree vs Symmetric Tree
### Same Tree
- 두 트리의 구조와 값이 완전히 동일해야 함
- 비교 순서: 좌 → 우, 좌 → 우 그대로

### Symmetric Tree
- 좌우가 거울처럼 대칭이어야 함
- 비교 순서: 좌/우 반대로 연결

즉,
- Same Tree: 같은 위치 비교
- Symmetric Tree: 반대 위치 비교

## 공통적인 패턴
둘 다 공통적으로 중요한 점은 아래다.

- null 체크가 반드시 필요하다
- 같은 위치/반대 위치를 비교해야 한다
- 값이 다르면 바로 false
- 구조가 다르면 바로 false
- BFS든 DFS든 핵심은 문맥에 맞는 비교 방향을 정하는 것이다

## 오늘의 결론
트리 문제는 보통 다음처럼 생각하면 된다.

- 같은 구조 비교: DFS 재귀 또는 쌍 비교 BFS
- 대칭 비교: 왼쪽/오른쪽 반대 방향으로 쌍 비교

오늘은 이 두 가지를 모두 직접 구현해보면서,
"같은 위치 비교"와 "거울 위치 비교"의 차이를 확실히 익혔다.
