---
layout: post
title: "[Java] 컬렉션 - List"
categories: Java
tags: [Java, Collection, 자바, 컬렉션]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 컬렉션을 소개하며, 컬렉션 중 `List`에 대해 설명하고자 합니다.

<br/>
<hr/>

## 1. 컬렉션 프레임워크

<b>컬렉션</b><sup>Collection</sup>: 동일한 타입을 묶어 관리하는 자료구조

> 배열과 컬렉션의 차이는?
> - 배열: 생성 시점에 저장 공간의 크기 확정
> - 컬렉션: 데이터의 저장 용량을 동적으로 관리할 수 있기에, 메모리 공간이 허용하는 한 저장 데이터의 개수에 제약이 없음


<b>컬렉션 프레임워크</b>: 컬렉션과 프레임워크가 조합된 개념으로, 리스트, 스택, 큐 등 자료구조에 정렬, 탐색 등의 알고리즘을 구조화해 놓은 프레임워크

> 프레임워크: 클래스 또는 인터페이스를 생성하는 과정에서 설계의 원칙 또는 구조에 따라 클래스 또는 인터페이스를 설계하고, 이렇게 설계된 클래스와 인터페이스를 묶어 놓은 개념

자바에서 제공하는 컬렉션 프레임워크의 주요 5가지 클래스와 인터페이스는 다음과 같다.
- 컬렉션의 특성에 따라 구분: `List<E>`, `Set<E>`, `Map<K, V>`
- 메모리의 입출력 특성에 따라 구분: 기존의 컬렉션 기능을 확장 또는 조합한 `Stack<E>`, `Queue<E>`

먼저 이번 포스트에서는 `List<E>`에 대해 이어 알아본다.

<br/>
<hr/>

## 2. 컬렉션 인터페이스 - List\<E>

<b>List\<E></b>: 배열과 가장 비슷한 구조를 지니고 있는 자료구조

최초로 리스트를 생성할 때에는 데이터가 없으므로 저장 공간의 크기는 0이다.

> 저장 공간의 크기(size): 실제 저장되어 있는 데이터의 수
> 
> cf. 저장 용량(capacity): 데이터를 저장하기 위해 미리 할당해 놓은 메모리의 크기

<br/>

### 2.1 List\<E> 객체 생성

`List<E>`는 인터페이스이기 때문에 객체를 스스로 생성할 수 없다.

따라서 `List<E>`를 상속받는 자식 클래스를 이용해 객체를 생성해야 한다.

`List<E>` 인터페이스를 구현한 대표적인 클래스는 다음의 3가지가 있다.
- `ArrayList<E>`
- `Vector<E>`
- `LinkedList<E>`

> `List<E>`는 제네릭 인터페이스이기 때문에, 객체를 생성할 때 제네릭 변수의 실제 타입<small>(=내부에 어떤 데이터 타입을 저장할지)</small>을 지정해야 한다.
>
> 기본적으로 객체의 초기 저장 용량(capacity)은 `10`으로 설정된다. 
> 하지만 초기 저장 용량을 매개변수로 포함하는 생성자를 이용해 초기 저장 용량을 지정할 수 있다.
>
> `LinkedList`는 capacity 지정이 불가하다.</u>

<u>`List<E>` 인터페이스 구현 클래스 생성자</u>를 통해 **동적** 컬렉션 객체 생성하는 방법은 다음과 같다.

```java
List<제네릭 타입> 변수 = new ArrayList<제네릭 타입>();
List<제네릭 타입> 변수 = new Vector<제네릭 타입>();
List<제네릭 타입> 변수 = new LinkedList<제네릭 타입>();

ArrayList<제네릭 타입> 변수 = new ArrayList<제네릭 타입>();
Vector<제네릭 타입> 변수 = new Vector<제네릭 타입>();
LinkedList<제네릭 타입> 변수 = new LinkedList<제네릭 타입>();

// 예시
List<Integer> aList1 = new ArrayList<Integer>();    // capacity = 10
Vector<String> vList1 = new Vector<String>(20);     // capacity = 20
LinkedList<Integer> list1 = new LinkedList<Integer>(20);    // 오류
```

또 다른 방법으로는 <u>`Arrays.asList()` 메서드</u>를 이용해 **정적** 컬렉션 객체를 생성할 수 있다.

이 방법은 내부적으로 배열을 먼저 생성한 후에 `List<E>`로 래핑한 것으로, 내부 구조는 배열과 동일하여 컬렉션 객체임에도 저장 공간의 크기를 변경할 수 없다.

```java
List<제네릭 타입> 변수 = Arrays.asList(저장 데이터);

// 예시
List<Integer> aList1 = Arrays.asList(1, 2, 3);
aList1.set(0, 5);   // [5, 2, 3]
aList1.add(8);      // UnsupportedOperationException 오류
aList1.remove(0);   // UnsupportedOperationException 오류
```

위의 예시와 같이 `Arrays.asList()` 메서드를 이용하여 객체 생성 시 데이터 추가 및 삭제 연산은 불가능하지만, 저장 공간의 크기를 변경하지 않는 데이터의 변경(`set()`) 연산은 가능하다.

<br/>

### 2.2 주요 메서드

`List<E>`에는 데이터 추가, 변경, 삭제, 리스트 데이터 정보 추출 및 배열 변환 등 추상 메서드가 정의되어 있다.

다음은 `List<E>`의 주요 메서드에 대해 설명하는 표이다.

|메서드명|반환 타입|설명|
|--|:--:|--|
|add(E element)|boolean|매개 변수 element를 리스트 마지막에 추가|
|add(int index, E element)|void|index 위치에 element 원소 추가|
|addAll(Collection<? Extends E> c)|boolean|입력 컬렉션 전체를 마지막에 추가|
|addAll(int index, Collection<? Extends E> c)|boolean|index 위치에 입력 컬렉션 전체 추가|
|set(int index, E element)|E|index 위치의 원소 값을 입력된 원소 element로 변경|
|remove(int index)|E|index 위치의 원소 삭제|
|remove(Object o)|boolean|원소 중 매개변수 입력과 동일한 객체 삭제|
|clear()|void|전체 원소 삭제|
|get(int index)|E|index 위치의 원소 반환|
|size()|int|리스트 객체 내에 포함된 원소 수|
|isEmpty()|boolean|리스트의 원소가 하나도 없는지 여부 반환|
|toArray()|Object[]|리스트를 Object 배열로 변환|
|toArray(T[] t)|T[]|입력매개변수로 전달한 타입의 배열로 변환<br/>리스트의 크기와 배열의 길이 중 큰 값을 길이로 하는 배열 생성|

`List<E>`에 있는 메서드는 추상 메서드이기 때문에, `List<E>`의 대표적 구현 클래스인 `ArrayList<E>`, `Vector<E>`, `LinkedList<E>`의 내부에는 해당 메서드들이 구현된 상태로 있어 활용이 가능하다.

<br/>
<hr/>

## 3. ArrayList\<E> 구현 클래스

`ArrayList<E>`의 특징은 다음과 같다.
- `List<E>` 인터페이스를 구현한 구현 클래스
- 배열처럼 수집한 원소를 인덱스로 관리하며 저장 용량<sub>capacity</sub>을 동적으로 관리
- 생성되는 리스트 객체의 기본 저장 용량 값은 10이며, 10 이상일 때는 자동으로 확대

<br/>

### 3.1 데이터 추가 - add()

```java
// add()
List<Integer> list1 = new ArrayList<>();
list1.add(3);
list1.add(6);
list1.add(12);
System.out.println(list1.toString());   // [3, 6, 12]

list1.add(2, 12);
System.out.println(list1.toString());   // [3, 6, 9, 12]

// addAll()
List<Integer> list2 = new ArrayList<>();
list2.add(0);
list2.addAll(list1);
System.out.println(list2.toString());   // [0, 3, 6, 9, 12]
```

<br/>

### 3.2 데이터 변경 - set()

```java
// add()
List<Integer> list1 = new ArrayList<>();
list1.add(3);
list1.add(4);
list1.add(5);   // [3, 4, 5]

list1.set(1, 6);
// list1.set(4, 10);    // IndexOutOfBoundsException 발생
System.out.println(list1.toString());   // [3, 6, 5]
```

변경할 데이터의 인덱스에 데이터가 없다면 `IndexOutOfBoundsException`이 발생한다.

<br/>

### 3.3 데이터 삭제 - remove(), clear()

```java
// remove()
List<Integer> list1 = new ArrayList<>();
list1.add(3);
list1.add(4);
list1.add(5);     // [3, 4, 5]

list1.remove(1);
System.out.println(list1.toString());   // [3, 5]

list1.remove(new Integer(5));
System.out.println(list1.toString());   // [3]


// clear()
list1.clear();
System.out.println(list1.toString());   // []
```

데이터 삭제 시에는 특정 인덱스 위치의 값을 삭제할 수 있고, 특정 원소를 삭제할 수 있다.

> 참고.
> 
> 모든 컬렉션의 원소에는 객체만 올 수 있다.
>
> 따라서 remove() 메서드 사용 시에 기본 자료형 값으로 1과 같이 숫자를 직접 넣으면 인덱스 값으로 인식한다.

<br/>

### 3.4 데이터 정보 추출 - isEmpty(), size(), get()

```java
// isEmpty()
List<Integer> list1 = new ArrayList<>();
System.out.println(list1.isEmpty());    // true

// size()
list1.add(1);
list1.add(2);
list1.add(3);
System.out.println(list1.toString());   // [1, 2, 3]
System.out.println(list1.size());       // 3

// clear()
System.out.println(list1.get(1));       // 2
```

- `isEmpty()`: 데이터 개수가 0일 경우에 `true` 반환
- `size()`: 실제 데이터의 개수 반환
- `get(int index)`: 특정 위치의 값 반환
  - 만약 리스트의 크기보다 큰 인덱스를 인자로 전달하면 `IndexOutOfBoundsException` 예외 발생

<br/>

### 3.5 리스트 -> 배열 변환 - toArray()

```java
List<Integer> list1 = new ArrayList<>();
list1.add(1);
list1.add(2);
list1.add(3);
System.out.println(list1.toString());         // [1, 2, 3]

// toArray()
Object[] object = list1.toArray();
System.out.println(Arrays.toString(object));  // [1, 2, 3]

// toArray(T[] t)
Integer[] integer = list1.toArray(new Integer[5]);
System.out.println(Arrays.toString(integer)); // [1, 2, 3, null, null]
```

`toArray()` 메서드: 원소의 타입과 관계없이 모든 데이터를 Object[] 배열에 담아 반환

cf. 특정 원소 타입으로 다운 캐스팅하기 위해 매개변수로 특정 타입의 배열 객체를 만들어 전달하면 됨
  - 매개변수로 전달되는 배열의 크기가 리스트 크기보다 큰 경우에는 나머지 값을 null로 채움
  - 보통 크기가 `0`인 배열을 넣어주는 방법을 가장 많이 사용

<br/>
<hr/>

## 4. Vector\<E> 구현 클래스

`Vector<E>`는 `List<E>`를 상속한 클래스이기 때문에 `List<E>`의 공통적인 특성<sub>(동일 타입 저장, 메모리 동적 할당, 데이터 추가/변경/삭제 메서드 포함)</sub>을 모두 갖고 있다.

또한 `ArrayList<E>`의 메서드 기능 및 사용법 또한 동일하다.

> `Vector<E>`와 `ArrayList<E>`의 차이점은?
>
> `Vector<E>`의 주요 메서드는 <b>동기화 메서드</b><sup>synchronized method</sup>로 구현되어 있어 멀티 스레드에 적합하다.

> 동기화 메서드란?
>
> : 하나의 공유 객체를 2개의 스레드가 동시에 사용할 수 없도록 만든 메서드

정리.

`Vector<E>`는 `ArrayList<E>`와 동일한 기능을 수행하지만, <u>멀티 스레드에서 사용할 수 있도록 기능이 추가</u>된 것이다.

따라서 싱글 스레드에서는 `ArrayList<E>`를, 멀티 스레드에서는 `Vector<E>`를 사용하는 것이 효율적이다.

<br/>

### 4.1 Vector\<E> 객체 생성

```java
List<Integer> vector1 = new Vector<Integer>();
...
```

<br/>
<hr/>

## 5. LinkedList\<E> 구현 클래스

`LinkedList<E>`는 마찬가지로 `List<E>`를 상속한 클래스이기 때문에 `List<E>`의 공통적인 특성을 모두 갖고 있다.

또한 `ArrayList<E>`와 같이 메서드를 동기화하지 않기 때문에 싱글 스레드에서 사용된다. 

> `LinkedList<E>`와 `ArrayList<E>`의 차이점은?
>
> 1. `LinkedList<E>`는 객체 생성 시에 저장 용량<sup>capacity</sup>을 지정할 수 없다.
>
> 2. `LinkedList<E>`의 내부적 데이터 저장 방식은 인덱스가 아닌 앞뒤 객체의 정보를 저장해 모든 데이터가 서로 연결된 형태로 관리된다.<br/>
> cf. `ArrayList<E>`: 모든 데이터를 인덱스와 값으로 저장

> `ArrayList<E>`와 `LinkedList<E>`의 성능 비교
>
> |구분|`ArrayList<E>`|`LinkedList<E>`|
> |:--:|:--:|:--:|
> |추가, 삭제|속도 느림|<mark>속도 빠름</mark>|
> |검색|<mark>속도 빠름</mark>|속도 느림|

<br/>

### 5.1 LinkedList\<E> 객체 생성

```java
List<Integer> linkedList1 = new LinkedList<Integer>();  // 저장 용량 지정 못함
...
```

<br/>
<hr/>

지금까지 컬렉션 중 `List`에 대해 설명하였습니다.

다음 포스트는 컬렉션 중, 데이터의 중복이 가능하지 않도록 하는 `Set`에 대해 설명할 예정입니다.

감사합니다:)
