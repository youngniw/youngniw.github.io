---
layout: post
title: "[Java] 컬렉션 - Stack, Queue"
categories: Java
tags: [Java, Collection, 자바, 컬렉션, 스택, 큐]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 컬렉션을 소개하며, 컬렉션 중 `Stack`, `Queue`에 대해 설명하고자 합니다.

<br/>
<hr/>

## 1. 컬렉션 인터페이스 - Stack\<E>

<b>스택</b><sup>Stack</sup>: 데이터를 차곡차곡 쌓아 올린 형태의 자료 구조

> 후입선출<sup>LIFO: Last In First Out</sup>: 나중에 입력된 데이터가 먼저 출력되는 것

`Stack<E>` 컬렉션은 앞에서 설명한 `List`, `Set`, `Map` 컬렉션과 달리 클래스이기 때문에 자체적으로 객체를 생성할 수 있다.

> `Stack<E>` 클래스는 `List<E>` 컬렉션의 구현 클래스인 `Vector<E>` 클래스의 자식 클래스이다.
>
> 따라서 `Stack<E>` 클래스는 `Vector<E>`의 모든 구현 기능을 포함하고 있으며, 추가로 LIFO 구조를 위한 5개의 메서드를 추가로 갖고 있다.

### 1.1 주요 메서드

데이터 추가를 위한 `push` 메서드, 데이터 검색을 위한 `peek`, `search` 메서드, 데이터 반환을 위한 `pop` 메서드, 스택 비임 여부에 대한 `empty` 및 `isEmpty` 메서드가 있다.

주요 5가지 메서드에 대한 내용과 관련한 표는 다음과 같다.

|메서드명|반환 타입|설명|
|--|:--:|--|
|push(E item)|E|매개 변수 item을 `Stack<E>`에 추가|
|peek()|E|가장 상단에 있는 원솟값 반환 (데이터 변화 x)|
|search(Object o)|int|스택 원소의 위치값 반환<br/>(맨 위 값: 1, 아래로 갈수록 1씩 증가, 값이 없을 경우 -1 반환)|
|pop()|E|가장 상단 데이터 반환 (데이터 개수 감소)|
|isEmpty()|boolean|스택 객체가 비어 있는지 여부 반환|

메서드에 대한 활용은 다음과 같다.

```java
// 객체 생성
Stack<Integer> stack = new Stack<Integer>();

// 1. 요소 추가
stack.push(1);
stack.push(3);
stack.push(5);

// 2. 상단 요소 값 확인
System.out.println(stack.peek());     // 5
System.out.println(stack.size());     // 3

// 3. 요소 값 위치 확인
System.out.println(stack.search(1));  // 1
System.out.println(stack.search(5));  // 3
System.out.println(stack.search(9));  // -1

// 4. 요소 삭제
System.out.println(stack.pop());      // 5
System.out.println(stack.pop());      // 3
System.out.println(stack.pop());      // 1

// 5. 스택 비임 여부 확인
System.out.println(stack.isEmpty());  // true
```

<br/>
<hr/>

## 2. 컬렉션 인터페이스 - Queue\<E>

<b>큐</b><sup>Queue</sup>: 스택과 다르게 한쪽 끝에서 삽입 작업이 이루어지고 반대쪽 끝에서 삭제 작업이 이루어지는 선입선출의 자료 구조

> 선입선출<sup>FIFO: First In First Out</sup>: 처음 입력된 데이터가 먼저 출력되는 것

`Queue<E>` 컬렉션은 앞에서 설명한 `List`, `Set`, `Map` 컬렉션과 같이 인터페이스이다. 

> `Queue<E>` 인터페이스는 `Collection<E>`를 상속한 인터페이스로, `LinkedList<E>`가 `Queue<E>` 인터페이스를 구현한다.

### 2.1 주요 메서드

따라서 Queue를 구현하기 위해서는 `LinkedList<E>` 클래스를 통해 객체를 생성하면 된다.

또한 데이터 추가를 위한 `offer` 혹은 `add` 메서드, 데이터 검색을 위한 `peek` 혹은 `element` 메서드, 데이터 삭제를 위한 `poll` 혹은 `remove` 메서드가 있다. 

`Queue<E>` 내에 FIFO 기능을 부여하는 메서드는 두 쌍씩 존재하는데, 그것들의 차이점은 데이터가 없을 시에 예외가 발생거나 기본값으로 null을 대체하는지이다.

아래의 큐의 메서드에 대한 내용과 관련한 표는 예외 처리 기능 미포함 메서드, 포함 메서드 순서이다.

- 예외 처리 기능 **미포함** 메서드

|메서드명|반환 타입|설명|
|--|:--:|--|
|add(E item)|boolean|매개 변수 item을 `Queue<E>`에 추가|
|element()|E|가장 상위에 있는 원소값 반환(데이터 변화 x)<br/>(데이터가 하나도 없을 시 `NoSuchElementException` 발생)|
|remove()|E|가장 상위에 있는 원소값 삭제 및 반환<br/>(꺼낼 데이터가 하나도 없을 시 `NoSuchElementException` 발생)|

- 예외 처리 기능 **포함** 메서드

|메서드명|반환 타입|설명|
|--|:--:|--|
|offer(E item)|boolean|매개 변수 item을 `Queue<E>`에 추가|
|peek()|E|가장 상위에 있는 원소값 반환(데이터 변화 x)<br/>(데이터가 하나도 없을 시 `null` 반환)|
|poll()|E|가장 상위에 있는 원소값 삭제 및 반환<br/>(꺼낼 데이터가 하나도 없을 시 `null` 반환)|

메서드에 대한 활용은 다음과 같다.

```java
// 1. 예외 처리 기능 미포함 메서드
// 객체 생성
Queue<Integer> queue1 = new LinkedList<Integer>();

// System.out.println(queue1.element());  // NoSuchElementException 발생

// 1-1. 요소 추가
queue1.add(2);
queue1.add(4);
queue1.add(6);

// 1-2. 맨 앞 요소 값 확인
System.out.println(queue1.element()); // 2

// 1-3. 요소 삭제
System.out.println(queue1.remove());  // 2
System.out.println(queue1.remove());  // 4
System.out.println(queue1.remove());  // 6


// 2. 예외 처리 기능 미포함 메서드
// 객체 생성
Queue<Integer> queue2 = new LinkedList<Integer>();

System.out.println(queue2.peek());  // null

// 2-1. 요소 추가
queue2.offer(2);
queue2.offer(4);
queue2.offer(6);

// 2-2. 맨 앞 요소 값 확인
System.out.println(queue2.peek());  // 2

// 2-3. 요소 삭제
System.out.println(queue2.poll());  // 2
System.out.println(queue2.poll());  // 4
System.out.println(queue2.poll());  // 6
System.out.println(queue2.poll());  // null
```

<br/>
<hr/>

지금까지 컬렉션 중 `Stack`, `Queue`에 대해 설명하였습니다.

감사합니다:)
