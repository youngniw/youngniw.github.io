---
layout: post
toc: true
title: "[자바스크립트] 자바스크립트 기초 문법 2. 자료형"
categories: 자바스크립트
tags: 자바스크립트, Javascript, 기초 문법
---

## 0. 들어가기에 앞서..
해당 포스트는 Vue.js를 사용하여 프로젝트를 진행하기 위해 스스로 자바스크립트를 공부하게 됨으로써 기억을 위해 작성한 것입니다.

지난 번에는 자바스크립트의 변수를 소개했다면, 해당 변수의 타입인 자료형에 대한 설명입니다.

<br/>
<hr/>

## 2. 자료형

### 2.1 문자형 String

```javascript
const name1 = "young"
const name2 = 'young'
const name3 = `young`

const sentence = `My name is ${name1}.`
console.log(sentence);   // My name is young.
```
문자형 String은 위와 같이 3가지의 방법으로 작성할 수 있다.

1. <b>큰 따옴표 " "</b>
2. <b>작은 따옴표 ' ' </b>
3. <b>백틱 &#96; &#96; </b>

문자열 내용에 " 혹은 '가 있다면 사용되는 문자 이외의 다른 따옴표를 사용하여 문자열을 감쌀 수 있다.<br/>
cf. 같은 따옴표를 사용하고자 한다면 \를 사용하면 특수문자로 인식한다.

또한 백틱은 문자열 내부의 변수로 문자열을 표현할 때 사용하면 편리하다. <br/>
cf. 일반 따옴표를 사용하는 문자열에서 ${ }를 통해 변수로 표현한다면 변수명의 그대로 문자열로 인식됨으로 주의해야 한다.

<br/>
<hr/>

### 2.2 숫자형
숫자는 정수로도 표현이 가능하고, 소수점을 사용해 실수로도 표현이 가능하다.

```javascript
const age = 20;     // 숫자형 number
const PI = 3.14;

console.log(1 + 2);     // 더하기 (3)
console.log(10 - 3);    // 빼기 (7)
console.log(3 * 4);     // 곱하기 (12)
console.log(6 / 2);     // 나누기 (3)
console.log(7 % 3);     // 나머지 연산 (1)
```

숫자형은 위와 같이 사칙연산이 가능하다.

```javascript
const a = 1 / 0;
console.log(a);     // Infinity
```
만약 위의 코드와 같이 0으로 나누는 연산을 한다면 Infinity 즉 무한대를 반환한다.

```javascript
const name = "young";
const b = name / 2;

console.log(b);     // NaN (Not a number)
```
만약 위의 코드와 같이 문자열을 숫자로 나눈다면 NaN을 반환한다.

<br/>
<hr/>

### 2.3 불린형
불린형은 다른 언어들과 마찬가지로 `true`와 `false`를 사용한다.

```javascript
const a = true;     // 참
const b = false;    // 거짓
```

참고로 false는 아래를 의미한다.
- 숫자 0
- 빈 문자열 ''
- null
- undefined
- NaN

<br/>
<hr/>

### 2.4 null과 undefined
<b>null</b> : 변수를 선언하고 빈 값을 할당한 상태(빈 객체)(존재하지 않은 값)

<b>undefined</b> : 변수를 선언하고 값을 할당하지 않은 상태

```javascript
let user = null;

let age;
console.log(age);   // undefined
```

참고로 prompt창에서 취소 버튼을 누르면 해당 변수는 null을 갖게 된다.

<br/>
<hr/>

### 2.5 typeof 연산자
<b>typeof</b> : 변수의 자료형을 알 수 있음

```javascript
const name = "young";

console.log(typeof 3);          // "number"
console.log(typeof name);       // "string"
console.log(typeof true);       // "boolean"
console.log(typeof "hi");       // "string"
console.log(typeof null);       // "object"
console.log(typeof undefined);  // undefined
```

해당 연산자는 다른 개발자가 작성한 코드에서 변수의 타입을 알거나 api 통신을 통해 얻은 데이터를 타입에 따라 다른 방식으로 처리해야 할 때 많이 사용한다.

<br/>
<hr/>

### 2.6 형변환
형변환은 <b>명시적 형변환</b>과 <b>자동 형변환</b>이 존재한다.

명시적 형변환을 이용해 문자형, 숫자형, 불린형으로 변환할 수 있다.

1. 문자형으로 변환: String() <br/>

    ```javascript
    console.log(
    String(5),
    String(true),
    String(false),
    String(null),
    String(undefined)
    );
    // "3" "true" "false" "null" "undefined"
    ```
2. 숫자형으로 변환: Number() <br/>

    ```javascript
    console.log(
    Number("1234"),
    Number("1234abcdef"),
    Number(true),
    Number(false)
    );
    // 1234 NaN 1 0
    ```
    cf. <u>참고로 Number(null)은 0, Number(undefined)는 NaN을 반환한다.</u>
3. 불린형으로 변환: Boolean() <br/>

    ```javascript
    console.log(
    Boolean(1),
    Boolean(123),
    Boolean("hello")
    );
    // true true true
    
    console.log(
    Boolean(0),
    Boolean(''),
    Boolean(NaN),
    Boolean(null),
    Boolean(undefined)
    );
    // false false false false false
    ```

<br/>
예를 들어 prompt 창을 이용해 숫자를 입력받고 입력받은 값 그대로 계산하고자 할 때, 실제로 반환된 값으로 계산을 한다면 이상한 값이 도출된다.<br/>
-> 그 이유는 prompt 창에서는 문자형을 반환하기에 문자형으로 계산이 적용된다. <br/>
-> 따라서 prompt 창에서 입력받은 숫자 값인 문자형을 숫자형으로 형변환을 한 후에 계산을 해야 옳은 결과를 도출할 수 있다.

cf. 자동 형변환: 특정 타입의 값을 기대하는 곳에 다른 타입의 값이 오면, 자동으로 타입을 변환<br/>
-> ex) "300" - "30" = 270 <br/>
-> 위처럼 문자열의 + 연산을 제외한 연산은 자동 형변환이 가능함 <br/>
-> <i>오류의 원인을 찾기 어려움으로 사용을 자제해야 함</i>

<br/>
<hr/>

지금까지 자바스크립트 문법 중 자료형을 설명하는 포스트였습니다. 감사합니다:)
