---
title: '[Algorithm] 플로이드-와샬 알고리즘'
toc: true
toc-stick: true
search: true
categories:
  - algorithm
tags:
  - algorithm
  - floyd-warshall
  - shortest path problem
excerpt: '그래프 최단거리 탐색인 floyd-warshall algorithm을 알아봅시다'
---

# 플로이드-와샬(Floyd-Warshall) 알고리즘  

다익스트라, 벨만-포드의 경우에는 특정 노드에서 다른 노드까지의 거리를 구한다. 
하지만, 모든 노드 간의 최단 거리를 구할 때는 이 플로이드-와샬 알고리즘을 사용한다.  

정점 u에서 정점 v까지의 최단 거리를 저장하는 ```dist[u][v]```를 유지하며, 
경유점 개념을 통해서 최단 거리를 구할 수 있다.

정점들의 집합을 S, 그 중의 임의의 한 정점을 x라고 정의했을 때 최단 거리는 둘 중 하나의 케이스가 된다.

1. 경로가 x를 경유하지 않는 경우 : S - {x}에 포함된 정점들을 통해 지나간다.
2. 경로가 x를 경유하는 경우 : 시작점 u에서 x, x에서 도착점 v 두 구간으로 지나가는 경로.

$$

D_s(u, v)=min
\begin{cases}
D_{S-{x}}(u, x) + D_{S-{x}}(x, v)\\
D_{S-{x}}(u, v)
\end{cases}
, x \in S

$$

<br/>

이 식을 DP 점화식으로 변경하여 구현을 할 수 있다.  
$ S_k = {0, 1, 2, ... , k} $ 라고하고, $ C_k = D_{S_k} $ 라고 하자. 
그러면 $ C_k(u, v) $ 는 0번 부터 k번 정점까지만을 경유점으로 사용했을 때 최단 경로가 된다.

$$

C_k(i, j)=min
\begin{cases}
C_{k-1}(i, k) + C_{k-1}(k, j)\\
C_{k-1}(i, j)
\end{cases}

$$



## 구현

``` cpp
// 정점의 개수
int V;
// 인접 리스트
int adj[MAX_V][MAX_V];
int C[MAX_V][MAX_V][MAX_V];

void floyd() {
	// C[0] 초기화
	for(int i = 0 ; i < V ; i++){
		for(int j = 0 ; j < V ; j++){
			if(i != j) C[0][i][j] = min(adj[i][j], adj[i][0] + adj[0][j]);
			else C[0][i][j] = 0;
		}
	}

	// C[k-1]로 C[k] 구하기
	for(int k = 1 ; k < V ; k++)
		for(int i = 0 ; i < V ; i++)
			for(int j = 0 ; j < V j ++)
				C[k][i][j] = min(C[k-1][i][j], C[k-1][i][k] + C[k-1][k][j]);
		
	
}
```

위 코드는 C를 $ O(V^3) $ 의 공간을 차지하는데 사실, C[k]를 구하기 위해서는 C[k-1]만 필요하지 그 이전의 값은 필요없다.
따라서, 두 이차원 배열을 저장할 $ O(V^2) $ 짜리 배열만 있으면 된다.  

하지만, 여기서 출발점이나 도착점에 k가 포함될 경우에는 C[k-1][u][k], C[k][u][k]가 같은 값이다. 
k까지 가는 최단 경로나, k를 경유하여 k까지 가는 최단 경로는 같은 의미이기 때문이다. 
따라서 $ O(V^2) $ 짜리 이차원 배열 하나만을 가지고도 해를 구할 수 있다.

``` cpp
// 정점의 개수
int V;
// 인접 리스트
int adj[MAX_V][MAX_V];

void floyd(){
	for(int i = 0 ; i < V ; i++) adj[i][i] = 0;

	for(int k = 0 ; k < V ; k++)
		for(int i = 0 ; i < V ; i++)
			for(int i = 0 ; i < V ; i++)
				adj[i][j] = min(adj[i][j], adj[i][k] + adj[k][i]);

}
```

## 시간 복잡도  

전체 정점을 순회하는 3중 for문으로 $ O(\|V\|^3) $ 이 된다.

<br/>

참고
- 구종만, 프로그래밍 대회에서 배우는 알고리즘 문제 해결 전략