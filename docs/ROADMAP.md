# 알고리즘 학습 로드맵

## 목표
- 자료구조/알고리즘의 기본 개념을 익힌다.
- 코딩 테스트에서 자주 나오는 유형을 반복적으로 푼다.
- 이직 준비를 위한 GitHub 기반 학습 기록을 만든다.
- 시간 제약이 있어도 핵심 패턴 위주로 효율적으로 학습한다.
- 최종 목표: 대기업 코딩테스트 합격 수준에 도달한다.

## 핵심 원칙
- 대기업 코딩테스트는 "모든 알고리즘"을 다 익히는 것보다 핵심 패턴을 정확하게 푸는 것이 중요하다.
- 현재는 BFS/DFS + Tree + Graph shortest path 를 가장 우선적으로 익혀야 한다.
- Dijkstra는 필수는 아니지만, 이후 최단 경로 심화 단계에서 다룬다.
- 매주 1~2개씩 핵심 문제를 반복해서 풀이하고, 오답을 정리한다.

## 대기업 코딩테스트용 4주 계획

### 1주차: BFS/DFS + 그래프 기본 패턴 완성
목표:
- Queue, Stack, visited, level 개념을 익힌다.
- 그래프를 문제로 바꾸는 습관을 만든다.
- 최단거리 문제를 BFS로 접근할 수 있게 한다.

추천 문제:
- 200. Number of Islands
- 994. Rotting Oranges
- 542. 01 Matrix
- 127. Word Ladder
- 433. Minimum Genetic Mutation
- 752. Open the Lock

학습 포인트:
- 노드/간선/방문 여부를 정의하는 법
- 큐에서 레벨 단위로 처리하는 법
- multi-source BFS 이해
- 최단 거리 문제는 BFS로 접근하는 이유

### 2주차: Tree 심화
목표:
- Tree 문제에서 DFS/BFS를 어떻게 적용하는지 익힌다.
- 트리의 구조를 이해하고, 자주 나오는 패턴을 정리한다.

추천 문제:
- 100. Same Tree
- 101. Symmetric Tree
- 102. Binary Tree Level Order Traversal
- 103. Binary Tree Zigzag Level Order Traversal
- 104. Maximum Depth of Binary Tree
- 116. Populating Next Right Pointers in Each Node
- 117. Populating Next Right Pointers in Each Node II

학습 포인트:
- 재귀 DFS vs iterative BFS 구분
- 트리에서 depth, level, null 처리 방법
- 자식/형제 관계를 그래프로 보는 방식

### 3주차: Graph shortest path + 실전 응용
목표:
- BFS 기반 최단 거리 문제를 안정적으로 푼다.
- 그래프 구조를 보고 어떤 알고리즘을 써야 하는지 판단한다.

추천 문제:
- 127. Word Ladder
- 433. Minimum Genetic Mutation
- 752. Open the Lock
- 743. Network Delay Time
- 787. Cheapest Flights Within K Stops

학습 포인트:
- unweighted shortest path = BFS
- weighted shortest path = Dijkstra
- dist 배열, queue, parents/graph 저장의 개념 이해
- 그래프 문제에서 경로를 모두 찾는 방식과 최단거리만 찾는 방식의 차이

### 4주차: Binary Search + Heap + Greedy + DP 입문
목표:
- 정렬된 배열 탐색과 힙 기반 문제를 익힌다.
- 그리디와 DP의 기본 패턴을 이해한다.

추천 문제:
- Binary Search
  - 704, 35, 74, 153, 162
- Heap / Greedy
  - 215, 347, 455, 871, 1046
- DP
  - 70, 198, 300, 322

학습 포인트:
- 이분 탐색은 경계 찾기와 정렬된 구간을 다루는 법
- 힙은 우선순위 처리 문제에 적합
- 그리디는 선택 기준을 정하는 법
- DP는 상태 정의와 점화식

## 우선순위별 추천

### 꼭 해야 하는 핵심 패턴
- BFS/DFS
- Tree traversal
- Graph shortest path
- Level-order processing
- visited + dist 관리

### 다음에 익히면 좋은 것
- Binary Search
  - 704, 35, 74, 153, 162
- Heap / Greedy
  - 215, 347, 455, 871, 1046

### 이후 심화
- Dijkstra
  - 743, 787
- DP/Backtracking 심화
  - 62, 63, 64, 416, 518

## 피드백 루틴
- 매주: 이번 주 문제를 다시 한 번 풀어보고 설명 정리
- 매주: 못 푼 문제를 오답 노트로 정리
- 매주: 다음 주 목표를 조정
- 매주: 핵심 개념을 README/노트에 정리

## 현실적인 결론
시간이 부족하면 전체 문제를 다 푸는 것보다, 핵심 패턴을 정확히 익히는 것이 더 중요하다.
특히 아래 문제들을 우선적으로 풀면 대기업 코딩테스트 대비에 적합하다.

필수 우선순위 문제:
- 200, 994, 542
- 127, 433, 752
- 102, 103, 104, 116, 117
- 46, 198, 300, 322

이 문제들만 익숙해지면, 실전 코딩테스트에서 사용할 수 있는 기본 알고리즘 감각이 크게 올라간다.

## Ace 75 사용 전략
- 전체 75문제를 다 푸는 것보다 핵심 패턴 위주로 푼다.
- 우선순위 문제는 다음과 같다:
  - 200, 994, 542, 127, 433, 752
  - 102, 103, 104, 116, 117
  - 704, 215, 347
  - 743, 787
- 이 문제들이 익숙해지면 Ace 75를 남은 문제 중심으로 보완한다.

## 매주 진행 방식
- 매일 1~2개 문제 풀이
- 주 1회 개념 정리
- 주 1회 회고 작성
- 미해결 문제 재시도
- 매주 1회 이번 주 학습 내용을 README/노트에 정리한다.
