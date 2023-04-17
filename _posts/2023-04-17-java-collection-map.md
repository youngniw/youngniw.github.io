---
layout: post
title: "[Java] 컬렉션 - Map"
categories: Java
tags: [Java, Collection, 자바, 컬렉션]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 컬렉션 중 `Map`에 대해 설명하고자 합니다.

<br/>
<hr/>

## 1. 컬렉션 인터페이스 - Map\<K, V>

<b>Map\<K, V></b>: Key<sub>키</sub>와 Value<sub>값</sub>의 한 쌍으로 데이터를 저장하는 자료구조

`Map<K, V>` 컬렉션은 `List<E>`와 `Set<E>`와 달리 별도의 인터페이스로 존재하기 때문에, 2개의 컬렉션과 다른 저장 형태와 방식을 이용한다.

> `List<E>`, `Set<E>` : `Collection<E>` 인터페이스 상속

<br/>

**[ 컬렉션 특징 ]**

- `Map<K, V>` 객체는 `Map.Entry` 객체를 포함함
- `Key`와 `Value`를 한 쌍으로 하여 데이터 저장 및 관리
- `Key`를 이용해 데이터를 구분
- `Key`는 중복 저장이 불가능하지만, `Value`는 중복 가능
- `Key`를 이용하여 `Value`에 접근
  - `List<E>`와 달리 인덱스를 사용하여 값을 조회할 수 없음

> <b>엔트리</b><sup>Entry</sup>: 키와 값으로 이루어진 한 쌍의 데이터
> 
> -> Java에서는 엔트리를 `Map.Entry` 타입으로 정의한다.

<br/>

### 1.1 Map\<K, V> 객체 생성

`Map<K, V>`는 인터페이스이기 때문에 객체를 스스로 생성할 수 없기 때문에 상속받는 자식 클래스를 이용해 객체를 생성해야 한다.

`Map<K, V>`의 대표적인 구현 클래스는 다음과 같다.
- `HashMap<K, V>`
- `LinkedHashMap<K, V>`
- `Hashtable<K, V>`
- `TreeMap<K, V>`

> `Map<K, V>`는 제네릭 인터페이스이기 때문에, 객체를 생성할 때 제네릭 변수의 실제 타입<small>(=내부에 Key와 Value로 어떤 데이터 타입을 저장할지)</small>을 지정해야 한다.

<u>`Map<K, V>` 인터페이스 구현 클래스 생성자</u>를 통해 객체를 생성하는 방법은 다음과 같다.

```java
Map<제네릭 타입, 제네릭 타입> 변수 = new HashMap<제네릭 타입, 제네릭 타입>();
Map<제네릭 타입, 제네릭 타입> 변수 = new LinkedHashMap<제네릭 타입, 제네릭 타입>();
Map<제네릭 타입, 제네릭 타입> 변수 = new Hashtable<제네릭 타입, 제네릭 타입>();
Map<제네릭 타입, 제네릭 타입> 변수 = new TreeMap<제네릭 타입, 제네릭 타입>();

HashMap<제네릭 타입, 제네릭 타입> 변수 = new HashMap<제네릭 타입, 제네릭 타입>();
LinkedHashMap<제네릭 타입, 제네릭 타입> 변수 = new LinkedHashMap<제네릭 타입, 제네릭 타입>();
Hashtable<제네릭 타입, 제네릭 타입> 변수 = new Hashtable<제네릭 타입, 제네릭 타입>();
TreeMap<제네릭 타입, 제네릭 타입> 변수 = new TreeMap<제네릭 타입, 제네릭 타입>();

// 예시
Map<String, Integer> hMap1 = new HashMap<String, Integer>();
LinkedHashMap<String, Integer> lMap1 = new LinkedHashMap<String, Integer>();
Hashtable<String, Integer> htMap1 = new Hashtable<String, Integer>(10);
TreeMap<String, Integer> tMap1 = new TreeMap<String, Integer>();
```

다른 방법으로 `Map` 인터페이스의 `of()` 메서드를 이용해 생성할 수 있다.

```java
Map<제네릭 타입, 제네릭 타입> map = Map.of(저장 데이터 키1, 저장 데이터 값1, 저장 데이터 키2, 저장 데이터 값2 ...);

// 예시
Map<String, Integer> map = Map.of("Young", 1, "Kim", 2);
```

<br/>

### 1.2 주요 메서드

`Map<K, V>`에는 데이터 추가, **변경**, 삭제, 데이터 추출 등의 추상 메서드가 정의되어 있다.

다음은 `Map<K, V>`의 주요 메서드에 대해 설명하는 표이다.

|메서드명|반환 타입|설명|
|--|:--:|--|
|put(K key, V value)|V|매개변수로 입력된 (키, 값)을 Map 객체에 추가|
|putAll(Map<? Extends K, ? Extends V> m)|void|입력 Map 객체를 모두 추가|
|replace(K key, V value)|V|Key에 해당하는 값을 Value 값으로 변경<br/>(해당 Key가 없을 시 null 반환)|
|replace(K key, V oldValue, V newValue)|boolean|(Key, Value)에 해당하는 엔트리에서 값을 newValue 값으로 변경<br/>(해당 엔트리가 없을 시 false 반환)|
|remove(Object key)|V|매개변수로 입력된 키를 갖는 엔트리 삭제|
|remove(Object key, Object value)|boolean|(Key, Value)에 해당하는 엔트리 삭제|
|clear()|void|Map 객체 내의 모든 데이터 삭제|
|isEmpty()|boolean|Map 객체가 비어있는지 반환|
|get(Object key)|V|매개변수인 Key 값에 해당하는 value 값 반환|
|<mark>containsKey(Object key)</mark>|boolean|매개변수인 Key 값이 Map 객체에 포함되어 있는지 반환|
|<mark>containsValue(Object value)</mark>|boolean|매개변수인 Value 값이 Map 객체에 포함되어 있는지 반환|
|keySet()|Set\<K>|Map 객체에서 키값들만 뽑아 Set 객체로 반환|
|entrySet()|Set\<Entry<K, V>>|Map 객체의 모든 엔트리를 Set 객체에 담아 반환|
|size()|int|Map 객체에 포함된 엔트리 수|

`Map<K, V>`에 있는 메서드는 추상 메서드이기 때문에, `Map<K, V>`의 대표적 구현 클래스인 `HashMap<K, V>`, `LinkedHashMap<K, V>`, `Hashtable<K, V>`, `TreeMap<K, V>`의 내부에는 해당 메서드들이 구현된 상태로 있어 활용이 가능하다.

<br/>
<hr/>

## 2. HashMap\<K, V> 구현 클래스

`HashMap<K, V>`의 특징은 다음과 같다.
- `Map<K, V>` 인터페이스를 구현한 구현 클래스
- Key 값의 중복을 허용하지 않음
  - Key 값의 중복 확인은 `HashSet<E>`와 동일하게 hashCode() 값이 같으며, equals() 메서드가 true를 반환할 때 같은 객체로 인식함
- 저장 데이터를 꺼낼 때는 입력 순서와 다를 수 있음
- 초기 기본 저장 용량 값은 16이며, 16 이상일 때는 자동으로 확장하여 증가

<br/>

### 2.1 데이터 추가 - put()

```java
// put(키, 값)
Map<Integer, String> hMap1 = new HashMap<>();
hMap1.put(1, "일");
hMap1.put(2, "이");
System.out.println(hMap1.toString());   // {1=일, 2=이}

// putAll(Map<? extends K, ? extends V> m)
Map<Integer, String> hMap2 = new HashMap<>();
hMap2.put(3, "삼");
hMap2.putAll(hMap1);
System.out.println(hMap2.toString());   // {1=일, 2=이, 3=삼}
```

<br/>

### 2.2 데이터 변경 - replace()

```java
// replace(키, 값)
hMap2.replace(1, "하나");
System.out.println(hMap2.toString());   // {1=하나, 2=이, 3=삼}

// replace(키, 이전값, 이후값)
hMap2.replace(2, "이", "둘");
System.out.println(hMap2.toString());   // {1=하나, 2=둘, 3=삼}
```

<br/>

### 2.3 데이터 삭제 - remove(), clear()

```java
// remove()
hMap2.remove(2);
hMap2.remove(4);                        // 동작하지 않음
System.out.println(hMap2.toString());   // {1=하나, 3=삼}

hMap2.remove(1, "하나");
System.out.println(hMap2.toString());   // {3=삼}


// clear()
hMap2.clear();
System.out.println(hMap2.toString());   // {}
```

<br/>

### 2.4 데이터 정보 추출 - isEmpty(), contains(), size()

```java
Map<Integer, String> hMap3 = new HashMap<>();
hMap3.put(1, "원");
hMap3.put(2, "투");
hMap3.put(3, "쓰리");

// isEmpty()
System.out.println(hMap3.isEmpty());      // false

// get(), getOrDefault()
System.out.println(hMap3.get(3));         // "쓰리"
System.out.println(hMap3.getOrDefalut(5, 0));   // 0 (키가 없을 시, 지정한 기본값 반환)

// containsKey()
System.out.println(hMap3.containsKey(1)); // true
System.out.println(hMap3.containsKey(4)); // false

// containsValue()
System.out.println(hMap3.containsValue("투"));  // true
System.out.println(hMap3.containsValue("텐"));  // false


// keySet()
Set<Integer> keySet = hMap3.keySet();
System.out.println(keySet.toString());    // [1, 2, 3]

// entrySet()
Set<Map.Entry<Integer, String>> entrySet = hMap3.entrySet();
System.out.println(entrySet.toString());  // [1=원, 2=투, 3=쓰리]

// 순회 for-each
hMap3.forEach((k, v) -> {});


// size()
System.out.println(hMap3.size());         // 3
```

<br/>
<hr/>

## 3. LinkedHashMap\<K, V> 구현 클래스

`LinkedHashMap<K, V>`는 `HashMap<K, V>`의 기본 특성과 함께 <mark>입력 데이터의 순서 정보</mark>를 추가로 갖고 있는 컬렉션

따라서 `HashMap<K, V>`의 메서드 기능 및 사용법 또한 동일하다.

`LinkedHashMap<K, V>`는 입력 데이터의 순서 정보를 갖고 있기 때문에 <u>출력 순서가 항상 입력 순서와 동일</u>한 특징을 갖는다.

> 참고.
>
> `HashMap<K, V>`: 키를 `HashSet<E>`로 관리
> `LinkedHashMap<K, V>`: 키를 `LinkedHashSet<E>`로 관리

<br/>

### 3.1 LinkedHashMap\<K, V> 객체 생성

```java
Map<Integer, String> lMap1 = new LinkedHashMap<Integer, String>();
...
```

<br/>
<hr/>

## 4. Hashtable\<K, V> 구현 클래스

`Hashtable<K, V>`는 `HashMap<K, V>`과 달리 멀티 스레드에 안정성을 가진다.

> `HashMap<K, V>`는 단일 스레드에 적합하다.

따라서 하나의 `Map<K, V>` 객체를 2개의 스레드가 동시에 접근하여도, 모든 내부 주요 메서드가 동기화 메서드로 구현되어 있기 때문에 안전하게 동작한다.

> cf. ArrayList\<E>가 단일 스레드에 적합한 반면, Vector\<E>가 멀티 스레드에 안전한 것과 동일한 관계

이외의 `Hashtable<K, V>`는 멀티 스레드에도 안전한 특징을 제외하고는, `HashMap<K, V>`와 동일한 특징을 갖는다.

따라서 `HashMap<K, V>`의 메서드 기능 및 사용법 또한 동일하다.

<br/>

### 4.1 Hashtable\<K, V> 객체 생성

```java
Map<Integer, String> htMap1 = new Hashtable<Integer, String>();
...
```

<br/>
<hr/>

## 5. TreeMap\<K, V> 구현 클래스

`TreeMap<K, V>`는 마찬가지로 `Map<K, V>`를 상속한 클래스이기 때문에 `Map<K, V>`의 공통적인 특성을 모두 갖고 있으며, <mark>정렬 및 검색 기능</mark>이 추가된 컬렉션이다.

`TreeMap<E>`: 데이터를 입력 순서와 관계없이 <mark>Key 값의 크기 순으로 저장 및 출력</mark>

따라서 Key 객체는 크기 비교의 기준을 갖고 있어야 한다.

<br/>

### 5.1 TreeMap\<K, V> 객체 생성

```java
// Map<K, V> 메서드만 사용 가능
Map<Integer, String> tMap1 = new TreeMap<Integer, String>();      
...

// Map<K, V> 메서드와 정렬/검색 기능 메서드 또한 사용 가능
TreeMap<Integer, String> tMap2 = new TreeMap<Integer, String>();    // 오름차순
TreeMap<Integer, String> tMap3 = new TreeMap<>(Collections.reverseOrder());  // 내림차순
...
```

<br/>

### 5.2 추가된 주요 메서드

`TreeMap<K, V>`에는 추가적으로 데이터 검색, 데이터 추출, 부분집합 생성, 그리고 데이터 정렬 메서드를 제공한다.

참고로 `TreeSet<E>`의 메서드와 비슷하며, 키와 엔트리에 대한 데이터를 검색 혹은 추출하는 메서드가 포함된다는 차이가 있다.

다음은 `TreeMap<K, V>`의 주요 메서드에 대해 설명하는 표이다. (오름차순 `TreeMap` 기준)

|메서드명|반환 타입|설명|
|--|:--:|--|
|firstKey()|K|Map 원소 중 가장 작은 Key 값 반환|
|firstEntry()|Map.Entry\<K, V>|Map 원소 중 가장 작은 Key 값을 갖는 엔트리 반환|
|lastKey()|K|Map 원소 중 가장 큰 Key 값 반환|
|lastEntry()|Map.Entry\<K, V>|Map 원소 중 가장 큰 Key 값을 갖는 엔트리 반환|
|lowerKey(K key)|K|입력된 매개변수 Key 값보다 작은 Key 값 중 가장 큰 Key 값 반환|
|lowerEntry(K key)|Map.Entry\<K, V>|입력된 매개변수 Key 값보다 작은 Key 값 중 가장 큰 Key 값을 갖는 엔트리 반환|
|higherKey(K key)|K|입력된 매개변수 Key 값보다 큰 Key 값 중 가장 작은 Key 값 반환|
|higherEntry(K key)|Map.Entry\<K, V>|입력된 매개변수 Key 값보다 큰 Key 값 중 가장 작은 Key 값을 갖는 엔트리 반환|
|pollFirstEntry()|Map.Entry\<K, V>|Map 원소 중 가장 작은 Key 값을 갖는 엔트리를 꺼내 반환|
|pollLastEntry()|Map.Entry\<K, V>|Map 원소 중 가장 큰 Key 값을 갖는 엔트리를 꺼내 반환|
|headMap(K tokey)|SortedMap\<K, V>|toKey 미만의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환|
|headMap(K tokey, boolean inclusive)|NavigableMap\<K, V>|toKey 미만/이하의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환<br/>(inclusive 값이 `true` 이면 이하, `false` 이면 미만)|
|tailMap(K tokey)|SortedMap\<K, V>|toKey 이상의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환|
|tailMap(K tokey, boolean inclusive)|NavigableMap\<K, V>|toKey 초과/이상의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환<br/>(inclusive 값이 `true` 이면 이상, `false` 이면 초과)|
|subMap(K fromKey, K tokey)|SortedMap\<K, V>|fromKey 이상 toKey 미만의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환|
|subMap(K fromKey, boolean fInclusive, K tokey, boolean tInclusive)|NavigableMap\<K, V>|fromKey 초과/이상 toKey 미만/이하의 Key 값을 갖는 모든 엔트리를 포함한 Map 객체 반환<br/>(_Inclusive 값이 `true` 이면 포함, `false` 이면 미포함)|
|descendingKeySet()|NavigableSet\<K>|현재 Key 값의 정렬 기준을 반대로 변환한 Set 객체 반환<br/>(cf. 초기 TreeMap 정렬 기준: 오름차순)|
|descendingMap()|NavigableMap\<K, V>|현재 정렬 기준을 반대로 변환한 Map 객체 반환<br/>(cf. 초기 TreeMap 정렬 기준: 오름차순)|

<br/>
<hr/>

지금까지 컬렉션 중 `Map`에 대해 설명하였습니다.

감사합니다:)
