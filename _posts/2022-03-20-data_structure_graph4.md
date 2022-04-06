---
layout: post
toc: true
title: "[자료구조] 그래프 - 위상 정렬"
categories: 자료구조
tags: 자료구조, 그래프
---

#### 위상 정렬

## 0. 들어가기에 앞서..
현재 이 포스트는 생능출판의 "C언어로 쉽게 풀어쓴 자료구조(개정3판)"책의 Chapter 11.그래프를 바탕으로 작성하였습니다.

이번 포스트는 위상 정렬에 대해 설명합니다.

내용은 간단하니 그림으로 먼저 위상정렬을 이해하고 코드를 보면 좋을 것 같습니다:)

<br/>
<hr/>

## 1. 위상 정렬
방향 그래프의 <b>위상 정렬<sup>topological sort</sup>:</b> 방향 그래프에 존재하는 각 정점들의 선행 순서를 위배하지 않으면서 모든 정점을 나열하는 것

<b>위상 정렬 알고리즘 순서</b>
1. 진입 차수가 0인 정점을 선택<br/>
-> 해당 정점이 여러 개 존재할 경우에 어느 정점을 선택해도 무방합<br/>
-> <b>하나의 그래프에는 복수의 위상순서가 있을 수 있음</b>
2. 선택된 정점과 해당 정점과 부착된 모든 간선을 삭제
3. 진입 차수가 0인 정점의 선택과 삭제 과정을 반복해서 모든 정점이 선택/삭제되면 알고리즘 종료

<img src="https://user-images.githubusercontent.com/78736070/159277254-88a278d3-d8e5-4892-bbb3-71fb6c42537a.png" height="300"/>

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

지금까지 그래프 중 위상 정렬을 설명하는 포스트였습니다.

감사합니다:)
