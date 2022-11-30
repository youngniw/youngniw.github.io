---
layout: post
toc: true
title: "[Java] Stream 정리 - 데이터 처리 연산"
categories: 자바
tags: 자바, 자바8, 스트림
---

## 0. 들어가기에 앞서..
본 포스트는 책 "자바 8 인 액션"을 참고하여 작성한 것입니다.

이번 포스트는 스트림에서 제공하는 다양한 연산을 소개하고, 이를 통해 다양한 데이터 처리 질의 방법에 대해 설명합니다.

스트림 연산 소개 순서는 다음과 같습니다.
- [2.1 필터링과 슬라이싱](#21-필터링과-슬라이싱)
- [2.2 매핑](#22-매핑)
- [2.3 검색과 매칭](#23-검색과-매칭)
- [2.4 요소 검색](#24-요소-검색)
- [2.5 리듀싱](#25-리듀싱)
- [2.6 스트림 메서드 정리](#26-스트림-메서드-정리)

<br/>
<hr/>

## 2. 스트림 활용

### 2.1 필터링과 슬라이싱
- 필터링 filter() 
  - 프레디케이트를 인수로 받아, 프레디케이트와 일치하는 모든 요소를 포함하는 스트림 반환
  - 예제 코드
  
    ```java
    List<User> adultUsers = user.stream()
        .filter(User::isAdult)
        .collect(Collectors.toList());
    ```

- 고유 요소 필터링 distinct()
  - 고유 요소로 이루어진 스트림 반환
  - 고유 여부는 스트림에서 만든 객체의 hashCode와 equals 메서드로 결정
  - 예제 코드
  
    ```java
    List<Integer> numbers = Arrays.asList(1, 2, 3, 3, 2, 1).stream()
        .filter(n -> n%2==0)
        .distinct()
        .collect(Collectors.toList());
    ```

- 스트림 축소 limit(n)
  - 주어진 사이즈 이하의 크기를 갖는 새로운 스트림 반환
  - 스트림이 정렬되어 있으면 최대 n 개의 요소 반환 가능, 소스가 정렬되어 있지 않으면 limit의 결과도 정렬되지 않은 상태로 반환
  - 예제 코드
  
    ```java
    List<User> threeUsers = user.stream()
        .limit(3)
        .collect(Collectors.toList());
    ```

- 요소 건너뛰기 skip(n)
  - 처음 n 개 요소를 제외한 스트림을 반환
  - n 개 이하의 요소를 포함하는 스트림에 호출 시, 빈 스트림이 반환됨
  - 예제 코드
  
    ```java
    List<User> skip3Users = user.stream()
        .skip(3)
        .collect(Collectors.toList());
    ```

<br/>
<hr/>

### 2.2 매핑
매핑: 특정 객체에서 특정 데이터로 변환

- 스트림의 각 요소에 함수 적용 map()
  - 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑됨
  - 여러 개의 map 메서드를 연결할 수 있음
  - 예제 코드
  
    ```java
    List<String> usersName = user.stream()
        .map(User::getName)
        .collect(Collectors.toList());
    ```

- 스트림 평면화 flatMap()
  - 스트림의 각 값을 다른 스트림으로 만든 다음, 모든 스트림을 하나의 스트림으로 연결
  - 각 배열을 스트림이 아닌 스트림의 콘텐츠로 매핑 (= 하나의 평면화된 스트림 반환)
  - 예제 코드
  
    ```java
    List<String> uniqueCharacters = user.stream()
        .map(u -> u.getName().split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
    ```

- 예제
1. 숫자 리스트가 주어졌을 때, 각 숫자의 제곱근으로 이루어진 리스트를 반환해 보자.
   
    ```java
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> squares = numbers.stream().map(n -> n*n).collect(Collectors.toList());
    ```

2. 2개의 숫자 리스트가 있을 때 모든 숫자 쌍의 리스트를 반환해 보자.
   
    ```java
    List<Integer> list1 = Arrays.asList(1, 2, 3);
    List<Integer> list2 = Arrays.asList(4, 5);

    List<int[]> pairs = list1.stream().flatmap(
      n1 -> list2.stream().map(n2 -> new int[]{ n1, n2 })
    ).collect(Collectors.toList());
    ```

3. 2번 문제에서 합이 짝수인 쌍만 반환해 보자.
   
    ```java
    List<Integer> list1 = Arrays.asList(1, 2, 3);
    List<Integer> list2 = Arrays.asList(4, 5);

    List<int[]> pairs = list1.stream().flatmap(
      n1 -> list2.stream().filter(n2 -> (n1+n2)%2==0).map(n2 -> new int[]{ n1, n2 })
    ).collect(Collectors.toList());
    ```

<br/>
<hr/>

### 2.3 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부 검색

- 프레디케이트와 적어도 한 요소와 일치하는지 확인 anyMatch()
  - 프레디케이트와 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 이용
  - 불린을 반환하는 최종 연산
  - 예제 코드
  
    ```java
    boolean isAnyAdult = user.stream().anyMatch(User::isAdult);
    ```

- 프레디케이트와 모든 요소와 일치하는지 확인 allMatch()
  - 프레디케이트와 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 이용
  - 불린을 반환하는 최종 연산
  - 예제 코드
  
    ```java
    boolean isAdultList = user.stream().allMatch(User::isAdult);
    ```

- 프레디케이트와 일치하는 요소가 없는지 확인 noneMatch()
  - 프레디케이트와 주어진 스트림에서 일치하는 요소가 없는지 확인할 때 이용
  - 불린을 반환하는 최종 연산
  - 예제 코드
  
    ```java
    boolean isKid = user.stream().noneMatch(User::isAdult);
    ```

<br/>
<hr/>

### 2.4 요소 검색
- 임의의 요소 반환 findAny()
  - 현재 스트림에서 임의의 요소 반환
  * 반환 타입: Optional\<T>
  - 다른 스트림 연산과 연결해서 사용 가능
  - 쇼트서킷을 이용해서 결과를 찾는 즉시 실행 종료
  - 예제 코드
  
    ```java
    Optional<User> oneUser = user.stream()
        .filter(User::isAdult)
        .findAny();
    ```

- 첫 번째 요소 반환 findFirst()
  - 리스트 혹은 정렬된 연속 데이터로부터 생성된 스트림에서 첫 번째 요소를 찾아 반환
  - 예제 코드
  
    ```java
    Optional<User> youngestAdultUser = user.stream()
        .filter(User::isAdult)
        .sorted(Comparator.comparingInt(User::getAge))
        .findFirst();
    ```

<br/>
<hr/>

### 2.5 리듀싱
리듀싱: 모든 스트림 요소를 처리해서 값으로 도출

- reduce(초깃값, 조합 연산)
  - 첫 번째 인수: 초깃값 (생략 가능)
    * 초깃값을 받지 않을 시에는 Optional\<T> 반환
  - 두 번째 인수: 두 요소를 조함해서 새로운 값을 만드는 BinaryOperator\<T>

<br/>

- 요소의 합
  
    ```java
    int ageSum = user.stream().map(User::getAge).reduce(0, Integer::sum);
    ```

- 요소의 최대
  
    ```java
    int maxAge = user.stream().map(User::getAge).reduce(0, Integer::max);
    ```
    
- 요소의 최소
  
    ```java
    int minAge = user.stream().map(User::getAge).reduce(0, Integer::min);
    ```

> 참고로 스트림은 최댓값이나 최솟값을 계산하는 데 사용할 키를 지정하는 Comparator를 인수로 받는 min과 max 메서드를 제공해 위의 reduce 연산을 다음과 같이 대신할 수 있다.
> 
> ```java
> user.stream().min(comparing(User::getAge));
> user.stream().max(comparing(User::getAge));
> ```

<br/>
<hr/>

### 2.6 스트림 메서드 정리
|연산|형식|반환 형식|사용된 함수형 인터페이스 형식|함수 스크립터|
|--|--|--|--|--|
|filter|중간 연산|Stream\<T>|Predicate\<T>|T -> boolean|
|distinct|중간 연산 (상태 있는 언바운드)|Stream\<T>|||
|skip|중간 연산 (상태 있는 바운드)|Stream\<T>|Long||
|limit|중간 연산 (상태 있는 바운드)|Stream\<T>|Long||
|map|중간 연산|Stream\<R>|Function<T, R>|T -> R|
|flatMap|중간 연산|Stream\<R>|Function<T, Stream\<R>>|T -> Stream\<R>|
|sorted|중간 연산 (상태 있는 언바운드)|Stream\<T>|Comparator\<T>|(T, T) -> int|
|min|최종 연산|Optional\<T>|Comparator\<T>|(T, T) -> T|
|max|최종 연산|Optional\<T>|Comparator\<T>|(T, T) -> T|
|anyMatch|최종 연산|boolean|Predicate\<T>|T -> boolean|
|allMatch|최종 연산|boolean|Predicate\<T>|T -> boolean|
|noneMatch|최종 연산|boolean|Predicate\<T>|T -> boolean|
|findAny|최종 연산|Optional\<T>|||
|findFirst|최종 연산|Optional\<T>|||
|forEach|최종 연산|void|Consumer\<T>|T -> void|
|collect|최종 연산|R|Collector<T, A, R>||
|reduce|최종 연산 (상태 있는 바운드)|Optional\<T>|BinaryOperator\<T>|(T, T) -> T|
|count|최종 연산|long|||

<br/>
<hr/>

지금까지 자바의 스트림 API가 지원하는 데이터 처리에 활용할 수 있는 연산에 대해 설명하는 포스트였습니다.

다음 포스트는 자바의 스트림 중 데이터 수집 시에 사용하는 컬렉터를 이용하여 스트림의 최종 결과를 도출하는 다양한 방법에 대해 설명하겠습니다.

감사합니다:)
