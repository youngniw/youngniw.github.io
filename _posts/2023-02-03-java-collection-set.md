---
layout: post
title: "[Java] 컬렉션 - Set"
categories: Java
tags: [Java, Collection, 자바, 컬렉션]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 컬렉션 중 `Set`에 대해 설명하고자 합니다.

<br/>
<hr/>

## 1. 컬렉션 인터페이스 - Set\<E>

<b>Set\<E></b>: 동일한 타입의 묶음으로, 동일 데이터의 중복 저장을 허용하지 않으며 인덱스 정보를 포함하지 않는 집합의 개념과 같은 자료구조

<br/>

### 1.1 Set\<E> 객체 생성

`Set<E>`는 인터페이스이기 때문에 객체를 스스로 생성할 수 없다.

따라서 `Set<E>`를 상속받는 자식 클래스를 이용해 객체를 생성해야 한다.

`Set<E>` 인터페이스를 구현한 대표적인 클래스는 다음의 3가지가 있다.
- `HashSet<E>`
- `LinkedHashSet<E>`
- `TreeSet<E>`

> `Set<E>`는 제네릭 인터페이스이기 때문에, 객체를 생성할 때 제네릭 변수의 실제 타입<small>(=내부에 어떤 데이터 타입을 저장할지)</small>을 지정해야 한다.

<u>`Set<E>` 인터페이스 구현 클래스 생성자</u>를 통해 **동적** 컬렉션 객체 생성하는 방법은 다음과 같다.

```java
Set<제네릭 타입> 변수 = new HashSet<제네릭 타입>();
Set<제네릭 타입> 변수 = new LinkedHashSet<제네릭 타입>();
Set<제네릭 타입> 변수 = new TreeSet<제네릭 타입>();

HashSet<제네릭 타입> 변수 = new HashSet<제네릭 타입>();
LinkedHashSet<제네릭 타입> 변수 = new LinkedHashSet<제네릭 타입>();
TreeSet<제네릭 타입> 변수 = new TreeSet<제네릭 타입>();

// 예시
Set<Integer> hSet1 = new HashSet<Integer>();
LinkedHashSet<String> lSet1 = new LinkedHashSet<String>();
TreeSet<Integer> tSet1 = new TreeSet<Integer>();
```

다른 방법으로 `Set` 인터페이스의 `of()` 메서드를 이용해 생성할 수 있다.

```java
Set<제네릭 타입> set = Set.of(저장 데이터);

// 예시
Set<Integer> set = Set.of(1, 2, 3);
```

또한 비어있는 `Set`인터페이스 구현 객체를 생성하는 방법 외에도, 비어있지 않은 배열을 이용해 `Set`으로 변환할 수도 있다.

```java
Set<제네릭 타입> 변수 = new HashSet<제네릭 타입>(Arrays.asList(저장 데이터));

// 예시
Set<Integer> aList1 = new HashSet<Integer>(Arrays.asList(1, 2, 3));
```

<br/>

### 1.2 주요 메서드

`Set<E>`에는 데이터 추가, 삭제, 데이터 추출 및 배열 변환 등 추상 메서드가 정의되어 있다.

다음은 `Set<E>`의 주요 메서드에 대해 설명하는 표이다.

|메서드명|반환 타입|설명|
|--|:--:|--|
|add(E element)|boolean|매개변수로 입력된 원소를 집합 리스트에 추가|
|addAll(Collection<? Extends E> c)|boolean|입력 컬렉션 전체를 추가|
|remove(Object o)|boolean|원소 중 매개변수 입력과 동일한 객체 삭제|
|clear()|void|전체 원소 삭제|
|isEmpty()|boolean|Set\<E> 객체가 비어있는지를 반환|
|<mark>contains(Object o)</mark>|boolean|매개변수로 입력된 원소 존재 여부 반환|
|size()|int|`Set` 집합 객체 내에 포함된 원소 수|
|<mark>iterator()</mark>|Iterator\<E>|리스트의 원소가 하나도 없는지 반환<br/>(참고로, for-each 구문을 통해 순회 가능)|
|toArray()|Object[]|집합 리스트를 Object 배열로 변환|
|toArray(T[] t)|T[]|입력 매개변수로 전달한 타입의 배열로 변환<br/>(집합 리스트의 크기와 배열의 길이 중 큰 값을 길이로 하는 배열 생성)|

`Set<E>`에 있는 메서드는 추상 메서드이기 때문에, `Set<E>`의 대표적 구현 클래스인 `HashSet<E>`, `LinkedHashSet<E>`, `TreeSet<E>`의 내부에는 해당 메서드들이 구현된 상태로 있어 활용이 가능하다.

<br/>
<hr/>

## 2. HashSet\<E> 구현 클래스

`HashSet<E>`의 특징은 다음과 같다.
- `Set<E>` 인터페이스를 구현한 구현 클래스
- 데이터의 중복 저장을 허용하지 않음
- 저장 데이터를 꺼낼 때는 입력 순서와 다를 수 있음
- 생성되는 집합 리스트 객체의 기본 저장 용량 값은 16이며, 16 이상일 때는 자동으로 확장하여 증가

<br/>

### 2.1 데이터 추가 - add()

```java
// add()
Set<Integer> hSet1 = new HashSet<>();
hSet1.add(3);
hSet1.add(6);
hSet1.add(3);
System.out.println(hSet1.toString());   // [3, 6]

// addAll()
Set<Integer> hSet2 = new HashSet<>();
hSet2.add(0);
hSet2.add(6);
hSet2.addAll(hSet1);
System.out.println(hSet2.toString());   // [0, 3, 6]
```

<br/>

### 2.2 데이터 삭제 - remove(), clear()

```java
// remove()
Set<Integer> hSet1 = new HashSet<>();
hSet1.add(3);
hSet1.add(4);
hSet1.add(5);
System.out.println(hSet1.toString());   // [3, 4, 5]

hSet1.remove(4);
System.out.println(hSet1.toString());   // [3, 5]


// clear()
hSet1.clear();
System.out.println(hSet1.toString());   // []
```

<br/>

### 2.3 데이터 정보 추출 - isEmpty(), contains(), size(), iterator()

```java
// isEmpty()
Set<Integer> hSet1 = new HashSet<>();
System.out.println(hSet1.isEmpty());    // true

hSet1.add(1);
hSet1.add(2);
hSet1.add(3);

// contains()
System.out.println(hSet1.contains(3));  // true
System.out.println(hSet1.contains(5));  // false

// size()
System.out.println(hSet1.size());       // 3

// iterator()
Iterator<Integer> iterator = hSet1.iterator();
while (iterator.hasNext()) {
  System.out.print(iterator.next() + " ");  // 1 2 3
}
// for-each문 (iterator와 동일)
for (Integer num: hSet1) {
  System.out.print(num + " ");              // 1 2 3
}
```

- `isEmpty()`: 데이터 개수가 0일 경우에 `true` 반환
- `contains()`: `HashSet<E>` 객체 안에 해당 원소가 있는지 반환
- `size()`: 실제 데이터의 개수 반환
- `iterator()`: 객체 내부의 데이터를 1개씩 꺼내거나 순회할 때 사용

> Iterator\<T>의 대표적인 메서드
>
> |메서드명|반환 타입|설명|
> |--|:--:|--|
> |hasNext()|boolean|다음ㅁ으로 가리킬 원소의 존재 여부 반환|
> |next()|E|다음 원소의 위치로 가서 값 반환|

<br/>

### 2.4 Set -> 배열 변환 - toArray()

```java
Set<Integer> hSet1 = new HashSet<>();
hSet1.add(1);
hSet1.add(2);
hSet1.add(3);

// toArray()
Object[] array1 = hSet1.toArray();
System.out.println(Arrays.toString(array1));  // [1, 2, 3]

// toArray(T[] t)
Integer[] array2 = hSet1.toArray(new Integer[5]);
System.out.println(Arrays.toString(array2));  // [1, 2, 3, null, null]
```

`toArray()` 메서드: 원소의 타입과 관계 없이 모든 데이터를 Object[] 배열에 담아 반환

<br/>
<hr/>

## 3. LinkedHashSet\<E> 구현 클래스

`LinkedHashSet<E>`는 `HashSet<E>`의 자식 클래스로, `HashSet<E>`의 모든 기능과 함께 <mark>데이터 간의 연결 정보</mark>를 추가로 갖고 있는 컬렉션

따라서 `HashSet<E>`의 메서드 기능 및 사용법 또한 동일하다.

`LinkedHashSet<E>`는 데이터 간의 연결 정보를 갖고 있기 때문에 <u>출력 순서가 항상 입력 순서와 동일</u>한 특징을 갖는다.

> 하지만 특정 위치에 저장된 값 조회 혹은 값 추가는 불가능함

<br/>

### 3.1 LinkedHashSet\<E> 객체 생성

```java
Set<Integer> linkedSet1 = new LinkedHashSet<Integer>();
...
```

<br/>
<hr/>

## 4. TreeSet\<E> 구현 클래스

`TreeSet<E>`는 마찬가지로 `Set<E>`를 상속한 클래스이기 때문에 `Set<E>`의 공통적인 특성을 모두 갖고 있다.

`TreeSet<E>`: 데이터를 입력 순서와 관계없이 <mark>크기순으로 출력</mark>

따라서 `HashSet<E>`와 `LinkedHashSet<E>`와 달리 추가된 정렬 및 검색 메서드를 사용하기 위해 객체 생성 시에 `TreeSet<E>`로 선언해야 한다.

<br/>

### 4.1 TreeSet\<E> 객체 생성

```java
// Set<E> 메서드만 사용 가능
Set<Integer> tSet1 = new TreeSet<Integer>();      
...

// Set<E> 메서드와 정렬/검색 기능 메서드 또한 사용 가능
TreeSet<Integer> tSet2 = new TreeSet<Integer>();    // 오름차순
TreeSet<Integer> tSet3 = new TreeSet<>(Collections.reverseOrder());  // 내림차순
...
```

<br/>

### 4.2 추가된 주요 메서드

`TreeSet<E>`에는 추가적으로 데이터 검색, 데이터 추출, 부분집합 생성, 그리고 데이터 정렬 메서드를 제공한다.

다음은 `TreeSet<E>`의 주요 메서드에 대해 설명하는 표이다. (오름차순 `TreeSet` 기준)

|메서드명|반환 타입|설명|
|--|:--:|--|
|first()|E|Set 원소 중 가장 작은 원소 값 반환|
|last()|E|Set 원소 중 가장 작은 큰 값 반환|
|lower(E element)|E|입력된 매개변수 원소보다 작은 값 중 가장 큰 수|
|higher(E element)()|E|입력된 매개변수 원소보다 큰 값 중 가장 작은 수|
|floor(E element)()|E|입력된 매개변수 원소보다 같거나 작은 값 중 가장 큰 수|
|ceiling(E element)()|E|입력된 매개변수 원소보다 같거나 큰 값 중 가장 작은 수|
|pollFirst()|E|Set 객체 내의 원소 중 가장 작은 원소 값을 꺼내어 반환|
|pollLast()|E|Set 객체 내의 원소 중 가장 큰 원소 값을 꺼내어 반환|
|headSet(E toElement)|SortedSet\<E>|toElement 미만인 모든 원소로 구성된 부분 집합 Set 반환|
|headSet(E toElement, boolean inclusive)|NavigableSet\<E>|toElement 미만/이하인 모든 원소로 구성된 부분 집합 Set 반환<br/>(inclusive 값이 `true` 이면 이하, `false` 이면 미만)|
|tailSet(E toElement)|SortedSet\<E>|toElement 이상인 모든 원소로 구성된 부분 집합 Set 반환|
|tailSet(E toElement, boolean inclusive)|NavigableSet\<E>|toElement 초과/이상인 모든 원소로 구성된 부분 집합 Set 반환<br/>(inclusive 값이 `true` 이면 이상, `false` 이면 초과)|
|subSet(E fromElement, E toElement)|SortedSet\<E>|fromElement이상 toElement 미만인 모든 원소로 구성된 부분 집합 Set 반환|
|subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive)|NavigableSet\<E>|fromElement초과/이상 toElement 미만/이하인 모든 원소로 구성된 부분 집합 Set 반환<br/>(__Inclusive 값이 `true` 이면 포함, `false` 이면 미포함)|
|descendingSet()|NavigableSet\<E>|현재 정렬 기준을 반대로 변환<br/>(cf. 초기 TreeSet 정렬 기준: 오름차순)|

참고로 데이터 크기 비교의 경우, 자바에서 이미 클래스의 크기 비교 기준을 작성해놓았다면 정숫값과 같이 쉽게 비교가 가능하다.

하지만 크기 비교 기준이 없을 시에는 직접 작성해야 한다.

1. `Comparable<T>` 제네릭 인터페이스 구현 (`compareTo() 메서드 오버라이딩`)
   - 자신 객체보다 매개변수 객체 값이 더 작은 경우 음수, 같을 때는 0, 클 때는 양수
2. `TreeSet<E>` 객체를 생성하면서 생성자 매개변수로 `Comparator<T>` 객체 제공
   
    ```java
    TreeSet<TmpClass> treeSet1 = new TreeSet<>(new Comparator<TmpClass>(){
      @Override
      public int compare(TmpClass o1, TmpClass o2) {
        if (ol.data < o2.data)
          return -1;
        else if (ol.data == o2.data)
          return 0;
        else
          return 1;
      }
    });
    ```
   

<br/>
<hr/>

지금까지 컬렉션 중 `Set`에 대해 설명하였습니다.

다음 포스트는 컬렉션 중 `Map`에 대해 설명할 예정입니다.

감사합니다:)
