---
layout: post
title: "[자료구조] 그래프 - 최단 경로"
categories: Data-Structure
tags: [DataStructure, Graph, 자료구조, 그래프]
---

#### 최단 경로와 관련 알고리즘(Dijkstra, Floyd)

## 0. 들어가기에 앞서..
현재 이 포스트는 생능출판의 "C언어로 쉽게 풀어쓴 자료구조(개정3판)"책의 Chapter 11.그래프를 바탕으로 작성하였습니다.

이번 포스트는 최단 경로와 최단 경로를 구하는 Dijkstra와 Floyd가 제안한 알고리즘을 설명합니다.

앞의 포스트와 마찬가지로 알고리즘의 이름과 해당 내용을 매치하는 방식으로 포스트를 보면 좋을 것 같습니다:)<br/>
(최소 비용 신장 트리와 비교합시다!!)

<br/>
<hr/>

## 1. 최단 경로
<b>최단 경로<sup>shortest path</sup> 문제:</b> 네트워크에서 정점 i와 정점 j를 연결하는 경로 중에서 간선들의 가중치 합이 최소가 되는 경로를 찾는 문제

최단 경로를 구하는 문제에 대한 알고리즘은 2가지가 있다.
- <b>Dijkstra 알고리즘:</b> 하나의 시작 정점에서 다른 정점까지의 최단 경로를 구함
- <b>Floyd 알고리즘:</b> 모든 정점에서 다른 모든 정점까지의 최단 경로를 구함

이때 가중치는 가중치 인접 행렬에 저장되어 있다고 가정한다. 
또한 만약 정점 u와 정점 v 사이에 간선이 없다면 무한대값이 저장되어 있다고 가정한다.

<br/>
<hr/>

## 2. Dijkstra의 최단 경로 알고리즘
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

<img src="https://user-images.githubusercontent.com/78736070/159287026-a27a613d-7c06-4b47-aa53-7ead8529fe68.png" height="400"/>

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

## 3. Floyd의 최단 경로 알고리즘
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

<img src="https://user-images.githubusercontent.com/78736070/159276909-f12b589e-bbe5-43f9-b525-c63ca2e63231.png" height="300"/>

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

지금까지 그래프 중 최단 경로의 개념과 Dijkstra의 최단 경로 알고리즘, 그리고 Floyd의 최단 경로 알고리즘을 설명하는 포스트였습니다.

다음 포스트는 그래프의 마지막 내용인 위상 정렬을 설명합니다.

감사합니다:)
