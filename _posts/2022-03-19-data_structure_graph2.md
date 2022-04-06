---
layout: post
toc: true
title: "[자료구조] 최소 비용 신장 트리"
categories: 자료구조
tags: 자료구조, 그래프
---

#### 최소 비용 신장 트리와 관련 알고리즘(Kruskal, Prim)

## 0. 들어가기에 앞서..
현재 이 포스트는 생능출판의 "C언어로 쉽게 풀어쓴 자료구조(개정3판)"책의 Chapter 11.그래프를 바탕으로 작성하였습니다.  

이번 포스트는 최소 비용 신장 트리와 최소 비용 신장 트리를 구하는 방법으로 사용되는 Kruskal과 Prim이 제안한 알고리즘을 설명합니다.

알고리즘의 이름과 해당 내용을 매치하는 방식으로 포스트를 보면 좋을 것 같습니다:)

<br/>
<hr/>

## 1. 최소 비용 신장 트리
<h4>[ 신장 트리 ]</h4>

<b>신장 트리<sup>spanning tree</sup>:</b> 그래프내의 모든 정점을 포함하는 트리<br/>
-> 사이클 포함 X

따라서 신장 트리는 그래프에 있는 n개의 정점을 정확히 (n-1)개의 간선으로 연결하게 된다.<br/>
-> 필연적으로 트리가 되고, 이것은 바로 신장 트리가 된다.

신장 트리는 그래프의 최소 연결 부분 그래프가 된다. 
이때 최소의 의미는 간선의 수가 가장 적다는 의미이다. 
<br/><br/>


<h4>[ 최소 비용 신장 트리 ]</h4>

<b>최소 비용 신장 트리<sup>MST: minimum spanning tree</sup>:</b> 신장 트리 중에서 사용된 간선들의 가중치 합이 최소인 신장 트리

최소 비용 신장 트리를 구하는 방법으로는 Kruskal과 Prim이 제안한 알고리즘이 대표적으로 사용되고 있다.

<b>최소 비용 신장 트리 조건</b>
- 최소 비용 신장 트리가 간선의 가중치의 합이 최소
- 반드시 (n-1)개의 간선만 사용
- 사이클이 포함되어서는 안됨

<br/>
<hr/>

## 2. Kruskal의 MST 알고리즘
<b>탐욕적인 방법:</b> 선택할 때마다 그 순간 가장 좋다고 생각되는 것을 선택함으로써 최종적인 해답에 도달하는 방법

Kruskal의 알고리즘은 탐욕적인 방법으로,<br/>
최소 비용 신장트리가 최소 비용의 간선으로 구성되고 동시에 사이클을 포함하지 않는다는 조건에 근거하여, 각 단계에서 사이클을 이루지 않는 최소 비용 간선을 선택한다.<br/>
-> 네트워크의 모든 정점을 최소 비용으로 연결하는 최적 해답을 구할 수 있음

<b>Kruskal 알고리즘 순서</b>
1. 그래프의 간선들을 가중치의 오름차순으로 정렬
2. 정렬된 간선들의 리스트에서 사이클을 형성하지 않는 간선을 찾아서 현재의 최소 비용 신장 트리의 집합에 추가<br/>
2-1. 만약 사이클을 형성하면 그 간선은 제외된다.

<img src="https://user-images.githubusercontent.com/78736070/159276013-f195eef3-ee3d-416c-bad1-5264339d7aaf.png" height="400"/>

다음은 최소 비용 신장 트리를 구하는 Kruskal의 알고리즘을 수도 코드로 표현한 것이다.
```
// 입력: 가중치 그래프 G=(V, E), n은 노드의 개수
// 출력: E(T), 최소비용 신장 트리를 이루는 간선들의 집합

kruskal(G):
    E를 가중치(=w(e))가 오름차순이 되도록 정렬한다.
    E(T) <- Φ; ecounter<-0
    k<-0
    while ecounter<(n-1) do
        k<-k+1
        if E ∪ {e(k)} 이 사이클을 포함하지 않으면
            then E(t) <- E(T) ∪ {e(k)};     ecounter<-ecounter+1
    return E(T)
```

위의 알고리즘에서 다음 간선을 이미 선택된 간선들의 집합에 추가할 때 사이클을 생성하는 지를 체크해야 한다. 
따라서 이때 지금 추가하고자 하는 간선의 양끝 정점이 같은 집합에 속해있는지를 먼저 검사해야 한다.
이 검사를 위한 알고리즘을 union-find 알고리즘이라고 한다.
<br/><br/>


<h4>[ union-find 연산 ]</h4>

- union(x, y) 연산: 원소 x와 y가 속해있는 집합을 입력으로 받아 2개의 집합의 합집합을 만든다. 
- find(x) 연산: 원소 x가 속애있는 집합을 반환함

다양한 방법으로 집합을 구현할 수 있지만 여기서는 트리 형태를 사용하는 것이 가장 효율적이다. <br/>
트리는 배열을 통해 각 정점의 부모 노드의 인덱스를 저장한다. 이때 배열의 값이 -1이면 부모 노드가 없다. 

아래의 알고리즘은 집합의 연산을 구현한 것이다.
```
union(a, b):
    root1 = find(a);
    root2 = find(b);
    if root1 ≠ root2
        parent[root1] = root2;


find(curr):
    if (parent[curr] == -1)
        return curr;    //루트 반환
    while (parent[curr] != -1)
        curr = parent[curr];
    return curr;
```

위의 알고리즘을 C언어로 구현한 것은 다음과 같다.
```c
void kruskal(GraphType* g) {
    int edge_accepted = 0;  //현재까지 선택된 간선의 수
    int uset, vset;         //정점 u와 v의 집합 번호
    struct Edge e;
    
    set_init(g->n);
    qsort(g->edges, g->n, sizeof(struct Edge), compare);
    
    int i = 0;
    while (edge_accepted < g->n-1) {
        e = g->edges[i];
        uset = set_find(e.start);
        vset = set_find(e.end);
    
        if (uset != vset) { //서로 속한 집합이 다름
            edge_accepted++;
            set_union(uset, vset);
        }
        i++;
    }
}
```
<br/>


<h4>[ 시간 복잡도 분석 ]</h4>

위의 union-find 알고리즘을 이용하면 Kruskal의 알고리즘의 시간 복잡도는 간선들을 정렬하는 시간에 좌우된다.

효율적인 정렬 알고리즘 사용 시,<br/>
Kruskal 알고리즘의 시간 복잡도: (|e|log<sub>2</sub>|e|)

<br/>
<hr/>

## 3. Prim의 MST 알고리즘
<b>Prim의 알고리즘:</b> 시작 정점에서부터 출발하여 신장 트리 집합을 단계적으로 확장해나가는 방법

<b>Prim 알고리즘 순서</b>
1. 시작 정점만이 신장 트리 집합에 포함됨
2. 1단계에서 만들어진 신장 트리 집합에, 인접한 정점들 중에서 최저 간선으로 연결된 정점을 선택하여 트리를 확장 <br/>
(트리가 n-1개의 간선을 가질 때까지 반복)

<img src="https://user-images.githubusercontent.com/78736070/159276309-bf333669-2a88-41b9-88bf-c86889c09f43.png" height="300"/>

<b>Kruskal 알고리즘과의 비교</b>
- Kruskal의 알고리즘은 간선을 기반으로 하는 알고리즘, 이전 단계와 상관없이 무조건 최저 간선만을 선택
- Prim의 알고리즘은 정접을 기반으로 하는 알고리즘, 이전 단계에서 만들어진 신장 트리를 확장

다음은 최소 비용 신장 트리를 구하는 Prim의 알고리즘을 수도 코드로 표현한 것이다.
```
// 입력: 네트워크 G=(V, E), s는 시작 정점
// 출력: 최소 비용 신장 트리를 이루는 정점들의 집합

prim(G, s):
    for each u∈V do
        distance[u]<-∞
    distance[s]<-0
    
    우선 순위 큐 Q에 모든 정점을 삽입(우선순위는 dist[])
    for (i<-0 to n-1) do
        u<-delete_min(Q)
        화면에 u를 출력
        for each v∈(U의 인접 정점) do
            if (v∈Q and weight[u][v] < dist[v])
                then dist[v]<-weight[u][v]
```

다음은 위의 의사 코드를 배열만을 이용해 구현한 것이다.
```c
void prim(GraphType* g, int s) {
    int i, u, v;
    
    for (u=0; u<g->n; u++)
        distance[u] = INF;
    distance[s] = 0;
    for (i=0; i<g->n; i++) {
        u = get_min_vertex(g->n);
        selected[u] = TRUE;
        if (distance[u] == INF)     return;
        
        for (v=0; v<g->n; v++)
            if (g->weight[u][v] != INF)
                if (!selected[v] && g->weight[u][v] < distance[v])
                    distance[v] = g->weight[u][v];
    }
}
```
<br/>


<h4>[ 시간 복잡도 분석 ]</h4>

Prim 알고리즘의 시간 복잡도: O(n<sup>2</sup>)<br/>
->주 반복문이 정점의 수 n만큼 반복하고, 내부 반복문이 n번 반복하기 때문

- 희소 그래프를 대상으로 최소 비용 신장 트리를 구하는 경우에는 Kruskal 알고리즘이 적합함
- 밀집 그래프인 경우, Prim 알고리즘이 적합함

<br/>
<hr/>

지금까지 그래프 중 최소 비용 신장 트리 개념과 Kruskal의 MST 알고리즘, 그리고 Prim의 MST 알고리즘을 설명하는 포스트였습니다.

다음 포스트는 최단 경로의 개념과 최단 경로를 구하는 방법으로 사용되는 Dijkstra와 Floyd가 제안한 알고리즘을 설명합니다.

감사합니다:)
