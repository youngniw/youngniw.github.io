---
layout: post
toc: true
title: "[자료구조] 10~11. 그래프"
categories: 자료구조
tags: 자료구조, 그래프
---

# &#91;C언어로 쉽게 풀어쓴 자료구조&#93; Chapter 10~11. 그래프

## 0. 들어가기에 앞서..
현재 이 포스트는 생능출판의 "C언어로 쉽게 풀어쓴 자료구조(개정3판)"에 대한 내용을 바탕으로 작성하였습니다. 

자료구조는 대학교 2학년 1학기에 배우고 공부를 하게 되었는데, 3년이 지난 지금 기억이 나지 않는 부분이 있어서 다시 공부하게 되었다. 
특히 이 그래프 부분은 용어들과 알고리즘들을 기억해야 된다고 생각했기에 이 포스트를 작성하게 되었다.

<br/>
<hr/>

## 10.1 그래프란?
<h4>[ 그래프의 소개 ]</h4>

<b>그래프(Graph)</b>: 객체 사이의 연결 관계를 표현할 수 있는 자료 구조

그래프 구조는 인접 행렬이나 인접 리스트로 메모리에 표현되고 처리될 수 있으므로 광범위한 분야의 다양한 문제들을 그래프로 표현하여 컴퓨터 프로그래밍에 의해 해결할 수 있다.

또한 그래프는 아주 일반적인 자료 구조로서 트리도 그래프의 하나의 특수한 종류로 볼 수 있다.
<br/><br/>


<h4>[ 그래프의 역사 ]</h4>

"Konigsberg의 다리" 문제: "임의의 지역에서 출발하여 모든 다리를 단 한번만 건너서 처음 출발했던 지역으로 돌아올 수 있는가"

->오일러는 특정 지역은 정점(node)로, 다리는 간선(edge)로 표현하여 그래프 문제로 변환하여 생각함

<b>오일러 경로(Eulerian tour):</b> 그래프에 존재하는 모든 간선을 한번만 통과하면서 처음 정점으로 되돌아오는 경로<br/>
<b>오일러 정리:</b> 그래프의 모든 정점에 연결된 간선의 개수가 짝수일 때만 오일러 경로가 존재한다.

<br/>
<hr/>

## 10.2 그래프의 정의와 용어
<h4>[ 그래프의 정의 ]</h4>

<b>그래프:</b> 정점<sup>vertex</sup>와 간선<sup>edge</sup>들의 유한 집합

    수학적 기호: G = (V, E)
    이때 V(G): 그래프 G의 정점들의 집합, E(G): 그래프 G의 간선들의 집합

<b>정점:</b> 여러 가지 특성을 가질 수 있는 객체 (=노드<sup>node</sup>)

<b>간선:</b> 정점들 간의 관계 (=링크<sup>link</sup>)
<br/><br/>


<h4>[ 무방향 그래프와 방향 그래프 ]</h4>

그래프는 간선의 종류에 따라 <b>무방향 그래프</b>와 <b>방향 그래프</b>로 구분된다.

<b>무방향 그래프<sup>undirected graph</sup>:</b> 그래프의 간선은 간선<sup>-</sup>을 통해서 양방향으로 갈 수 있음<br/>
-> 정점 A와 정점 B를 연결하는 간선은 (A, B)와 같이 정점의 쌍으로 표현&nbsp;&nbsp;<span style="color:blue">(A,B) = (B,A)</span>

<b>방향 그래프<sup>directed graph</sup>:</b> 간선<sup>-></sup>에 방향성이 존재하는 그래프<br/>
-> 정점 A와 정점 B로만 갈 수 있는 간선은 &lt;A, B&gt;로 표시&nbsp;&nbsp;<span style="color:blue">&lt;A,B&gt; &#8800; &lt;B,A&gt;</span>
<br/><br/>


<h4>[ 네트워크 ]</h4>

<b>가중치 그래프<sup>weighted graph</sup>:</b> 간선에 비용이나 가중치가 할당된 그래프 (= 네트워크<sup>network</sup>)
<br/><br/>


<h4>[ 부분 그래프 ]</h4>

<b>부분 그래프<sup>subgraph</sup>:</b> 어떤 그래프의 정점의 일부와 간선의 일부로 이루어진 그래프

    그래프 G의 부분 그래프 S일 때,
    
     V(S) ⊆ V(G)
     E(S) ⊆ E(G)
<br/>


<h4>[ 정점의 차수 ]</h4>

그래프에서의 <b>인접 정점<sup>adjacent vertex</sup>:</b> 간선에 의해 직접 연결된 정점

1. <u>무방향 그래프</u>에서 정점의 차수(degree): 그 정점에 인접한 정점의 수<br/>
-> 모든 정점의 차수를 합하면 간선 수의 2배가 됨
2. <u>방향 그래프</u>에서는 외부에서 오는 간선의 개수를 <b>진입 차수<sup>in-degree</sup></b>라고 하고, 외부로 향하는 간선의 개수를 <b>진출 차수<sup>out-degree</sup></b>라고 함
<br/><br/>


<h4>[ 경로 ]</h4>

무방향 그래프에서 정점 s로부터 정점 e까지의 경로는 정점의 나열 s, v<sub>1</sub>, v<sub>2</sub>, ... , v<sub>k</sub>, e로서, 나열된 정점들 간에는 반드시 간선 (s, v<sub>1</sub>), (v<sub>1</sub>, v<sub>2</sub>), ... , (v<sub>k</sub>, e)가 있어야 한다.

방향 그래프라면 나열된 정점들 간에는 반드시 간선 &lt;s, v<sub>1</sub>&gt;, &lt;v<sub>1</sub>, v<sub>2</sub>&gt;, ... , &lt;v<sub>k</sub>, e&gt;가 있어야 한다.

<b>단순 경로<sup>simple path</sup>:</b> 경로 중에서 반복되는 간선이 없을 경우일 때의 경로

<b>사이클<sup>cycle</sup>:</b> 단순 경로의 시작 정점과 종료 정점이 동일할 때의 경로
<br/><br/>


<h4>[ 연결 그래프 ]</h4>

무방향 그래프에서의 <b>연결 그래프<sup>connected graph</sup>:</b> 무방향 그래프 G에 있는 모든 정점쌍에 대해 항상 경로가 존재함 (= G는 연결됨) <br/>
<-> 비연결 그래프

Ex. 트리는 그래프의 특수한 형태로서 사이클을 가지지 않는 연결 그래프

*&nbsp;참고<br/> 
강력 연결 그래프: 모든 쌍의 정점들 사이에 상호방향 경로(양방향 모두)가 존재<br/>
강력 연결 요소: 방향성이 존재하는 유향 그래프에서 모든 정점이 다른 모든 정점들에 대하여 방문할 수 있는 경우 즉, 어떤 두 정점 간의 경로가 존재
<br/><br/>


<h4>[ 완전 그래프 ]</h4>

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/59/Complete_graph_K4.svg/180px-Complete_graph_K4.svg.png" width="120" height="120"/> &nbsp;&nbsp;&nbsp;
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/Complete_graph_K5.svg/180px-Complete_graph_K5.svg.png" width="120" height="120"/>

<b>완전 그래프<sup>complete graph</sup>:</b> 그래프에 속해있는 모든 정점이 서로 연결되어 있는 그래프

무방향 완전 그래프의 정점 수를 n이라고 할 때, 하나의 정점은 n-1개의 다른 정점으로 연결되므로 간선의 수는 n × (n-1) / 2가 된다.

<br/>
<hr/>

## 10.3 그래프의 표현 방법
그래프를 표현하는 방법에는 다음의 2가지 방법이 있다.

- <b>인접 행렬<sup>adjacent matrix</sup>:</b> 2차원 배열을 사용해 그래프를 표현
- <b>인접 리스트<sup>adjacent list</sup>:</b> 연결 리스트를 사용해 그래프를 표현

따라서 문제의 특성에 따라 적합한 표현 방법을 선택해야 한다.

<h4>[ 인접 행렬 ]</h4>

그래프의 정점 수가 n이라면 n × n의 2차원 배열인 인접 행렬 M의 각 원소를 다음의 규칙에 의해 할당함으로써 그래프를 메모리에 표현할 수 있다.
    
    if (간선 (i, j)가 그래프에 존재)
        M[i][j] = 1
    otherwise
        M[i][j] = 0

n개의 정점을 가지는 그래프를 인접 행렬로 표현하기 위해서는 간선의 수에 무관하게 항상 n<sup>2</sup>개의 메모리 공간이 필요하다.<br/>
-> 인접 행렬은 밀집 그래프<sup>dense graph</sup>를 표현하는 경우에 적합하지만, 희소 그래프<sup>sparse graph</sup>의 경우에는 메모리 낭비가 크므로 적합하지 않다.

- 두 정점을 연결하는 간선의 존재 여부 소요 시간: O(1)
- 정점의 차수 검색 소요 시간: O(n)<br/>
    (정점 i에 대한 차수 = 인접 배열의 i번째 행에 있는 값을 모두 더한 값)
- 그래프 내의 모든 간선의 수: O(n<sup>2</sup>) -> 인접 행렬 전체를 조사
<br/><br/>


<h4>[ 인접 리스트 ]</h4>

<b>인접 리스트:</b> 그래프를 표현함에 있어 각각의 정점에 인접한 정점들을 연결 리스트로 표시한 것

각 연결리스트들은 헤더 노드를 가지고 있고 이 헤더 노드들은 하나의 배열로 구성되어 있다. 
따라서 정점의 번호만 알면 이 번호를 배열의 인덱스로 하여 각 정점의 연결 리스트에 쉽게 접근할 수 있다.

정점의 개수가 n개이고 간선의 수가 e개인 무방향 그래프를 표시하기 위해서는 n개의 연결 리스트가 필요하고, n개의 헤더 노드와 2e개의 노드가 필요하다.
따라서 인접 리스트 표현은 간선의 개수가 적은 희소 그래프의 표현에 적합하다.

n개 정점과 e개의 간선을 가진 그래프에서 <br/>
전체 간선의 수 조회 소요시간: O(n+e) -> 헤더 노드를 포함하여 모든 인접 리스트를 조사
<br/><br/>


<h4>[ 인접 리스트를 이용한 그래프 추상 데이터 타입의 구현 ]</h4>

다음은 인접 리스트를 이용한 그래프에서의 간선 삽입 연산 코드이다. <br/>
```c
#define MAX_VERTICES 50
typedef struct GraphNode {
    int vertex;
    struct GraphNode* link;
} GraphNode;

typedef struct GraphType {
    int n;  //정점의 개수
    GraphNode* adj_list[MAX_VERTICES];
} GraphType;


//v(정점)를 u(정점)의 인접 리스트에 삽입한다.
void insert_edge(GraphType* g, int u, int v)
{
    GraphNode* node;
    if (u >= g->n || v >= g->n) {
        fprintf(stderr, "그래프: 정점 번호 오류")
    }
 
    node = (GraphNode*)malloc(sizeof(GraphNode));
    node->vertex = v;
    node->link = g->adj_list[u];
    g->adj_list[u] = node;
}
```
정점 u에 간선 (u, v)를 삽입하는 연산은 정점 u의 인접 리스트에 간선을 나타내는 노드를 하나 생성하여 삽입하면 된다. 
이때 위치는 상관이 없으므로 삽입을 쉽게 하기 위해 연결 리스트의 맨 처음에 삽입한다.

<br/>
<hr/>

## 10.4 그래프의 탐색
그래프 탐색: 하나의 정점으로부터 시작하여 차례대로 모든 정점들을 한 번씩 방문

그래프의 탐색 방법은 두 가지가 있다.
- 깊이 우선 탐색(DFS: depth first searching)
- 너비 우선 탐색(BFS: breadth first searching)

<img src="https://ww.namu.la/s/1fe9246903b78fae07577b243a0b22791e02cb39640d5cbaae10d9849343b4ea6f162a9a677a5892fbf7819abd4ef7221ebd3608849cfb66793411fb5e6439513f37d6ea76d434184fa2b423684201bc638b3eec75c11effe1e5ad2ac4a7deb2" width="350" height="200"/>
<span style="color:grey">[참고: 나무위키]</span>

<b>깊이 우선 탐색:</b> 트리 탐색 시, 시작 정점에서 한 방향으로 계속 가다가 더 이상 갈 수 없게 되면 다시 가장 가까운 갈림길로 돌아와서 다른 방향으로 다시 탐색을 진행하는 방법과 유사함

<b>너비 우선 탐색:</b> 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법

<br/>
<hr/>

## 10.5 깊이 우선 탐색
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Depthfirst.png/375px-Depthfirst.png" width="180" height="180"/>
<span style="color:grey">[참고: 위키백과]</span>

깊이 우선 탐색 진행 순서
1. 그래프의 시작 정점에서 출발하여 시작 정점 v을 방문하였다고 표시함
2. v에 인접한 정점들 중에서 아직 방문하지 않은 정점 u를 선택<br/>
2-1. 만약 그러한 정점이 없다면 탐색 종료
3. 만약 아직 방문하지 않은 정점 u가 있다면 u를 시작 정점으로 하여 깊이 우선 탐색을 다시 시작
... 2~3을 반복함

따라서 깊이 우선 탐색도 자기 자신을 다시 호출하는 순환 알고리즘의 형태를 가지고 있다.

다음은 깊이 우선 탐색의 알고리즘을 수도코드로 보여준다.
```
depth_first_search(v):

    v를 방문되었다고 표시
    for all u ∈ (v에 인접한 정점) do
        if (u가 아직 방문되지 않았으면)
            then depth_first_search(u)
```

위의 코드에서는 순환을 사용하지만 명시적인 스택을 사용하여 구현도 가능하다.
```
DFS-iterative(G, v):

    스택 S를 생성한다.
    S.push(v)
    while (not is_empty(S)) do
        v = S.pop()
        if (v가 방문되지 않았다면)
            v를 방문되었다고 표시
            for all u ∈ (v에 인접한 정점) do
                if (u가 아직 방문되지 않았으면)
                    S.push(u)
```

<h4>[ 깊이 우선 탐색의 분석 ]</h4>
정점의 수가 n이고 간선의 수가 e인 그래프인 경우, <br/>
- 인접 리스트로 표현 시 시간 복잡도: O(n+e)
- 인접 행렬로 표현 시 시간 복잡도: O(n<sup>2</sup>)

<br/>
<hr/>

## 10.6 너비 우선 탐색
<b>너비 우선 탐색(BFS):</b> 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법

BFS를 위해서는 자료 구조인 큐<sup>queue</sup>가 필요하다.

너비 우선 탐색 진행 순서
1. 큐에서 정점을 꺼내서 정점을 방문한다.
2. 해당 정점의 인접 정점들을 큐에 추가한다.
3. 큐가 소진될 때까지 동일한 코드를 반복한다.

다음은 너비 우선 탐색의 의사 코드이다.
```
breadth_first_search(v):

    v를 방문되었다고 표시;
    큐 Q에 정점 v를 삽입;
    while (Q가 공백이 아니면) do
        Q에서 정점 v를 삭제;
        for all u ∈ (v에 인접한 정점) do
            if (u가 아직 방문되지 않았으면)
                then u를 큐에 삽입;
                    u를 방문되었다고 표시;
```

위의 코드에서는 순환을 사용하지만 명시적인 스택을 사용하여 구현도 가능하다.

<h4>[ 너비 우선 탐색의 분석 ]</h4>
- 인접 리스트로 표현 시 시간 복잡도: O(n+e)
- 인접 행렬로 표현 시 시간 복잡도: O(n<sup>2</sup>)

너비 우선 탐색도 깊이 우선 탐색과 같이 희소 그래프를 사용할 경우 인접 리스트를 사용하는 것이 효율적이다.

<br/>
<hr/>

## 11.1 최소 비용 신장 트리
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

## 11.2 Kruskal의 MST 알고리즘
<b>탐욕적인 방법:</b> 선택할 때마다 그 순간 가장 좋다고 생각되는 것을 선택함으로써 최종적인 해답에 도달하는 방법

Kruskal의 알고리즘은 탐욕적인 방법으로,<br/>
최소 비용 신장트리가 최소 비용의 간선으로 구성되고 동시에 사이클을 포함하지 않는다는 조건에 근거하여, 각 단계에서 사이클을 이루지 않는 최소 비용 간선을 선택한다.<br/>
-> 네트워크의 모든 정점을 최소 비용으로 연결하는 최적 해답을 구할 수 있음

<b>Kruskal 알고리즘 순서</b>
1. 그래프의 간선들을 가중치의 오름차순으로 정렬
2. 정렬된 간선들의 리스트에서 사이클을 형성하지 않는 간선을 찾아서 현재의 최소 비용 신장 트리의 집합에 추가<br/>
2-1. 만약 사이클을 형성하면 그 간선은 제외된다.

<img src="https://gmlwjd9405.github.io/images/algorithm-mst/kruskal-example2.png" height="400"/>

<span style="color:grey">[참고: gmlwjd9405님의 블로그]</span>

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

## 11.3 Prim의 MST 알고리즘
<b>Prim의 알고리즘:</b> 시작 정점에서부터 출발하여 신장 트리 집합을 단계적으로 확장해나가는 방법

<b>Prim 알고리즘 순서</b>
1. 시작 정점만이 신장 트리 집합에 포함됨
2. 1단계에서 만들어진 신장 트리 집합에, 인접한 정점들 중에서 최저 간선으로 연결된 정점을 선택하여 트리를 확장 <br/>
(트리가 n-1개의 간선을 가질 때까지 반복)

<img src="https://gmlwjd9405.github.io/images/algorithm-mst/prim-example.png" height="400"/>

<span style="color:grey">[참고: gmlwjd9405님의 블로그]</span>

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

## 11.4 최단 경로
<b>최단 경로<sup>shortest path</sup> 문제:</b> 네트워크에서 정점 i와 정점 j를 연결하는 경로 중에서 간선들의 가중치 합이 최소가 되는 경로를 찾는 문제

최단 경로를 구하는 문제에 대한 알고리즘은 2가지가 있다.
- <b>Dijkstra 알고리즘:</b> 하나의 시작 정점에서 다른 정점까지의 최단 경로를 구함
- <b>Floyd 알고리즘:</b> 모든 정점에서 다른 모든 정점까지의 최단 경로를 구함

이때 가중치는 가중치 인접 행렬에 저장되어 있다고 가정한다. 
또한 만약 정점 u와 정점 v 사이에 간선이 없다면 무한대값이 저장되어 있다고 가정한다.

<br/>
<hr/>

## 11.5 Dijkstra의 최단 경로 알고리즘
<b>Dijkstra 최단 경로 알고리즘:</b> 네트워크에서 하나의 시작 정점에서 모든 다른 정점까지의 최단 경로를 찾는 알고리즘

이때 최단 경로는 경로의 길이 순으로 구해진다.<br/>

<b>설정 값</b><br/>
- 집합 S: 시작 정점 v로부터의 최단 경로가 이미 발견된 정점들의 집합
- 1차원 distance 배열: 시작 정점에서 집합 S에 있는 정점만을 거쳐서 다른 정점으로 가는 최단 거리를 기록하는 배열<br/>
-> distance값은 시작정점과 해당 정점간의 가중치값
- 가중치 인접 행렬 weight: 행과 열을 간선의 정점으로 하는 가중치 저장

<b>Dijkstra 알고리즘 순서</b>
1. 시작 정점을 S 집합에 넣음
2. S 안에 있지 않은 정점 중에서 가장 distance값이 작은 정점 u을 S에 추가
3. 새로 추가된 정점 u를 거쳐서 정점까지 가는 거리와 기존의 거리를 비교하여 더 작은 거리로 distance 값을 수정<br/>
```
distance[w] = min(distance[w], distance[u]+weight[u][w])
```

(위의 2와 3단계를 S가 n개의 정점을 가질 때까지 반복)

<img src="https://upload.wikimedia.org/wikipedia/commons/5/57/Dijkstra_Animation.gif" width="250" height="200"/>

<span style="color:grey">[참고: 위키백과]</span>

다음은 최단 거리 알고리즘을 구하는 다익스트라의 알고리즘을 의사 코드로 정리한 것이다.
```
// 입력: 가중치 그래프 G, 가중치는 음수가 아님
// 출력: distance 배열, distance[u]는 v에서 u까지의 최단 거리임

shortest_path(G, v):
    for 긱 정점 w∈G do
        distance[w]<-weight[v][w];
    
    while 모든 정점이 S에 포함되지 않으면 do
        u<-집합 S에 속하지 않는 정점 중에서 최소 distance 정점;
        S<-S ∪ {u}
        for u에 인접하고 S에 있는 각 정점 z do
            if distance[u]+weight[u][z] < distance[z]
                then distance[z]<-distance[u]+weight[u][z];
```

다음은 위의 의사 코드를 c언어로 구현한 코드의 일부이다.
```c
int distance[MAX_VERTICES]; /* 시작정점으로부터의 최단 경로 거리 */
int found[MAX_VERTICES];    /* 방문한 정점 표시 */

//가장 distance가 작은 정점 반환
int choose(int distance[], int n, int found[]) {
    int i, min, minpos;
    min = INT_MAX;
    minpos = -1;
    for (i=0; i<n; i++) {
        if (distance[i]<min && !found[i]) {
            min = distance[i];
            minpos = i;
        }
    }
    return minpos;
}

//최단 경로 알고리즘
void shortest_path(GraphType* g, int start) {
    int i, u, w;
    for (i=0; i<g->n; i++) {
        distance[i] = g->weight[start][i];
        found[i] = FALSE;
    }
    found[start] = TRUE;
    distance[start] = 0;
    
    for (u=0; u<g->n-1; u++) {
        u = choose(distance, g->n, found);
        found[u] = TRUE;
        
        for (w=0; w<g->n; w++)
            if (!found[w])
                if (distance[u]+g->weight[u][w] < distance[w])
                    distance[w] = distance[u]+g->weight[u][w];
    }
}
```
<br/>


<h4>[ 시간 복잡도 분석 ]</h4>

다익스트라 알고리즘의 시간 복잡도: O(n<sup>2</sup>)<br/>
-> 네트워크에 정점이 n개 있을 때, 주반복문을 n번 반복하고 내부 반복문을 2n번 반복

<br/>
<hr/>

## 11.6 Floyd의 최단 경로 알고리즘
<b>Floyd 최단 경로 알고리즘:</b> 그래프에 존재하는 모든 정점 사이의 최단 경로를 한 번에 모두 찾아주는 알고리즘

이 알고리즘은 2차원 배열 A를 이용하여 3중 반복을 하는 루프로 구성되어 있다.

<b>설정 값</b><br/>
- 가중치 인접 행렬 weight: 행과 열을 간선의 정점으로 하는 가중치 저장
    - i==j일 때, weight[i][j] == 0
    - 간선이 존재하지 않을 때, weight[i][j] = ∞
    - 간선이 존재하면, weight[i][j] = 간선(i,j)의 가중치

<b>Floyd 알고리즘 설명</b>
- A<sup>k</sup>[i][j]: 0부터 k까지의 정점만을 이용한 정점 i에서 j까지의 최단 경로
- 이 알고리즘은 A<sup>-1</sup>->A<sup>0</sup>->A<sup>1</sup>->A<sup>2</sup>-> ... ->A<sup>n-1</sup> 순으로 최단 거리를 구하는 것
- A<sup>k-1</sup>까지는 완벽한 최단 거리가 구해져 있다고 가정했을 때, 아래의 2가지 경우 중 적은 값이 A<sup>k</sup>[i][j]가 됨
    - 정점 k를 거쳐서 가지 않는 경우: 최단 거리는 A<sup>k-1</sup>[i][j]
    - 정점 k를 거쳐서 지나가는 경우: 최단 거리는 A<sup>k-1</sup>[i][k] + A<sup>k-1</sup>[k][j]

<b>연습 문제 및 풀이</b>

<img src="https://user-images.githubusercontent.com/78736070/159106029-45561553-74ff-4fba-bce6-feffd2d5e320.png" width="600" height="400"/>

<span style="color:grey">[참고: reakwon님 블로그]</span>

다음은 최단 경로 알고리즘을 구하는 다익스트라의 알고리즘을 의사 코드로 정리한 것이다.
```
floyd(G):
    for k<-0 to n-1
        for i<-0 to n-1
            for j<-0 to n-1
                A[i][j] = min(A[i][j], A[i][k]+A[k][j])
```

다음은 위의 의사 코드를 c언어로 구현한 코드의 일부이다.
```c
void floyd(GraphType* g) {
    int i, j, k;
    for (i=0; i<g->n; i++)
        for (j=0; j<g->n; j++)
            A[i][j] = g->weight[i][j];
    
    for (k=0; k<g->n; k++)
        for (i=0; i<g->n; i++)
            for (j=0; j<g->n; j++)
                A[i][j] = min(A[i][j], A[i][k]+A[k][j])
}
```
<br/>


<h4>[ 시간 복잡도 분석 ]</h4>

Floyd 알고리즘의 시간 복잡도: O(n<sup>3</sup>)

플로이드 알고리즘은 매우 간결한 반복 구문을 사용하므로 다익스트라의 알고리즘보다 상당히 빨리 모든 정점 간의 최단 경로를 찾을 수 있다.

<br/>
<hr/>

## 11.7 위상 정렬
방향 그래프의 <b>위상 정렬<sup>topological sort</sup>:</b> 방향 그래프에 존재하는 각 정점들의 선행 순서를 위배하지 않으면서 모든 정점을 나열하는 것

<b>위상 정렬 알고리즘 순서</b>
1. 진입 차수가 0인 정점을 선택<br/>
-> 해당 정점이 여러 개 존재할 경우에 어느 정점을 선택해도 무방합<br/>
-> <b>하나의 그래프에는 복수의 위상순서가 있을 수 있음</b>
2. 선택된 정점과 해당 정점과 부착된 모든 간선을 삭제
3. 진입 차수가 0인 정점의 선택과 삭제 과정을 반복해서 모든 정점이 선택/삭제되면 알고리즘 종료

<img src="https://gmlwjd9405.github.io/images/algorithm-topological-sort/topological-sort-example.png" width="600"/>

<span style="color:grey">[참고: gmlwjd9405님 블로그]</span>

다음은 위상 정렬 알고리즘을 의사 코드로 정리한 것이다.
```
// 입력: 그래프 G=(V,E)
// 출력: 위상 정렬 순서

topo_sort(G):
    for i<-0 to n-1 do
        if (모든 정점이 선행 정점을 가지면)
            then 사이클이 존재하고 위상 정렬 불가;
        
        선행 정점을 가지지 않는 정점 v 선택;
        v를 출력;
        v와 v에서 나온 모든 간선들을 그래프에서 삭제;
```

다음은 위의 의사 코드를 c언어로 구현한 코드의 일부이다. <br/>
참고로 in_degree 1차원 배열은 각 정점의 진입 차수를 기록한다. 
또한 후보 정점들을 스택에 저장한다. 
```c
int topo_sort(GraphType* g) {
    int i;
    StackType s;
    GraphNode* node;
    
    //모든 정점의 진입 차수를 계산
    int* in_degree = (int*) malloc(g->n * sizeof(int));
    for (i=0; i<g->n; i++)
        in_degree[i] = 0;
    for (i=0; i<g->n; i++){
        GraphNode* node = g->adj__list[i];
        while (node != NULL) {
            in_degree[nnode->vertex]++;
            node = node->link;
        }
    }
    
    //진입 차수가 0인 정점을 스택에 삽입
    init(&s);
    for (i=0; i<g->n; i++)
        if (in_degree[i] == 0)
            push(&s, i);
    
    //위상 순서를 생성
    while (!is_empty(&s)) {
        int w;
        w = pop(&s);
        printf("정점 %d -> ", w);    //정점 출력
        node = g->adj_list[w];      //각 정점의 진입 차수를 변경
        while (node != NULL) {
            int u = node->vertex;
            in_degree[u]--;         //진입 차수 감소
            if (in_degree[u] == 0)
                push(&s, u);
            node = node->link;      //다음 정점
        }
    }
    free(in_degree);
    
    return (i == g->n);
}
```

<br/>
<hr/>

지금까지 자료구조 중 그래프를 설명하는 포스트였습니다. 감사합니다:)
