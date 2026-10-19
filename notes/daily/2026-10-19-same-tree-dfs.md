# 2026-10-19 LeetCode 100. Same Tree

## 최종 정리
이 문제는 "두 트리가 같은 구조와 값을 가지는가?"를 묻는 문제다.

핵심은 다음 3가지다.
- 둘 다 null이면 같음
- 한쪽만 null이면 다름
- 값이 다르면 다름
- 그다음은 왼쪽/오른쪽 서브트리를 재귀적으로 비교

## 가장 깔끔한 정답: DFS
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

이 방식이 가장 자연스러운 이유는:
- 트리 문제는 보통 재귀로 자식 노드를 내려가며 비교하는 게 가장 직관적이다.
- 현재 노드가 같아야 하고, 자식 구조도 같은지 검사해야 하므로 재귀가 딱 맞다.

## BFS도 가능하지만, 쌍으로 비교해야 함
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

이 방식은:
- 같은 위치의 노드끼리 비교
- null 처리도 같이 보장
- 구조 비교가 정확함

그래서 BFS로도 풀 수는 있지만, 단순히 두 큐를 따로 관리하는 방식은 불안정하다.

## 내가 처음 짠 시도 코드
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
import java.util.ArrayDeque;

class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        return bfs(p,q);
    }

    public boolean bfs(TreeNode p, TreeNode q) {
        boolean result = true;
        Queue<TreeNode> lq = new ArrayDeque<>();
        Queue<TreeNode> rq = new ArrayDeque<>();
        
        if(p != null) {
            lq.offer(p);
        }

        if(q != null) {
            rq.offer(q);
        }

        while(!lq.isEmpty() || !rq.isEmpty()){
            TreeNode lqNode = lq.poll();
            TreeNode rqNode = rq.poll();
            
            if(!(lqNode == null && rqNode == null 
            || lqNode != null && rqNode != null)) {
                result = false;
                break;
            }
            System.out.println(lqNode.val + " " + rqNode.val);
            if(lqNode.left != null) System.out.println(lqNode.left.val);
            if(lqNode.right != null) System.out.println(lqNode.right.val);
            boolean isValDifferent = lqNode.val != rqNode.val;
            boolean isLeftDifferent = 
            (lqNode.left == null && rqNode.left != null)
            || (lqNode.left != null && rqNode.left == null);
            boolean isRightDifferent = 
            (lqNode.right == null && rqNode.right != null)
            || (lqNode.right != null && rqNode.right == null);

            if(isValDifferent || isLeftDifferent || isRightDifferent) {
                result = false;
                break; 
            }
            
            if(lqNode.left != null) lq.offer(lqNode.left);
            if(lqNode.right != null) lq.offer(lqNode.right);
            if(rqNode.left != null) rq.offer(rqNode.left);
            if(rqNode.right != null) rq.offer(rqNode.right);
        } 
        return result;
    }
}
```

## 이 코드에서 배운 점
이 코드는 아이디어가 맞았지만, 아래가 약했다.

- 서로 다른 큐를 따로 관리해서 같은 위치를 비교하지 않음
- null 처리 시점이 불안정함
- 구조 비교가 복잡하고, 로직이 길어짐
- 결국 코드가 "동작하는지"보다 "정확히 어떤 위치를 비교하는지"가 불분명함

그래서 더 안전한 방식은 다음 두 가지다.
- 재귀 DFS
- `Queue<TreeNode[]>` 쌍 비교 BFS

## 문자열 변환 방식이 약한 이유
문자열로 바꿔서 비교하는 방법은 간단해 보이지만, 구조를 정확히 보존하지 못한다.

예시:
```text
A:      1
       / \
      2  null

B:      1
       / \
     null  2
```

이 둘은 구조가 다르지만,
null과 값의 위치를 문자열로 정확히 표현하지 못하면 같은 것처럼 보일 수 있다.

또한 값이 여러 자리 숫자면 경계가 흐려진다.

그래서 이 문제에서는 "문자열 비교"보다는 "구조 자체 비교"가 훨씬 안전하다.

## 문자열 비교 시도 코드
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
import java.util.ArrayDeque;
import java.util.Queue;

class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        String pr = bfs(p);
        String qr = bfs(q);
        return pr.equals(qr);
    }

    public String bfs(TreeNode n) {
        StringBuffer buf = new StringBuffer();
        Queue<TreeNode> q = new ArrayDeque<>();

        if(n != null) {
            q.offer(n);
        }

        while(!q.isEmpty()){
            TreeNode node = q.poll();

            buf.append(node == null ? "v" : node.val);

            if(node.left != null) q.offer(node.left);
            else buf.append("l");
            if(node.right != null) q.offer(node.right);
            else buf.append("r");
        }
        return buf.toString();
    }
}
```

이 코드는 문제를 "트리를 문자열로 직렬화해서 비교"하는 방식으로 접근했다.

핵심 아이디어는:
- 트리 전체를 문자열로 바꾸면 비교가 편하다
- 같은 구조일 경우 같은 문자열이 나온다

하지만 이 방식의 한계는:
- null 위치와 자식 위치를 정확히 표현하지 못한다
- 값의 경계를 정확히 구분하지 못한다
- 구조가 같아도 문자열이 섞여서 비교가 애매해질 수 있다

그래서 이 방법은 아이디어는 좋지만, 구조 비교 문제에서는 안전하지 않다.
정석적인 풀이는 재귀 DFS 또는 같은 위치의 노드 쌍 비교 BFS이다.

## 복잡도
- DFS: O(n), O(h)
- BFS 쌍 비교: O(n), O(w)

## 최종 결론
Same Tree는
- 트리 구조 비교
- null 처리
- 자식 위치 비교
가 핵심이다.

그래서 가장 안전하고 표준적인 풀이는 재귀 DFS다.

내가 처음 시도한 BFS 방식은 아이디어가 있었고, 학습 가치가 있지만,
정확한 비교를 위해서는 같은 위치의 노드 쌍을 묶어서 처리해야 한다.

이 문제는 "같은 위치의 노드 비교"를 정확히 구현하는 것이 핵심이었다.
