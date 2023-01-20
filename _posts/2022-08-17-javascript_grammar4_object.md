---
layout: post
toc: true
title: "[자바스크립트] 자바스크립트 기초 문법 4. 객체 Object"
categories: Javascript
tags: 자바스크립트, Javascript, 문법
---

## 0. 들어가기에 앞서..
해당 포스트는 Vue.js를 사용하여 프로젝트를 진행하기 위해 스스로 자바스크립트를 공부하게 됨으로써 기억을 위해 작성한 것입니다.

지난 번에는 자바스크립트의 함수를 소개했다면, 이번 포스트는 객체에 대한 설명입니다.

<br/>
<hr/>

## 4. 객체(Object)
자바스크립트의 기본 타입은 객체이다.

객체란 중괄호를 사용해 키(key)와 값(value)으로 구성된 프로퍼티(property)의 정렬되지 않은 집합이다.

각 프로퍼티는 쉼표(,)로 구분한다.

프로퍼티의 값으로 함수가 올 수도 있는데, 이러한 프로퍼티를 메소드(method)라고 한다.

<br/>
<hr/>

### 4.1 객체 프로퍼티 접근, 추가, 삭제

```javascript
const person = {
    name: 'young',
    age: '24',
}
```

<b>[ 객체 접근 ]</b>

객체에 접근을 하는 경우에는 `.`을 사용하거나 `[]` 대괄호를 사용한다.
```javascript
person.name     // 'young'
person['age']   // 24
```

참고로 객체의 프로퍼티가 존재하지 않는다면 undefined를 반환한다.

cf. `in` 연산자를 통해 프로퍼티가 있는지 확인할 수 있다.
```javascript
'birthday' in person;   // false
'age' in person;        // true
```

또한 for ... in 반복문을 통해 객체를 순회하며 프로퍼티 값을 얻을 수 있다.
```javascript
for (let key in person) {
    console.log(key);           // key 값 출력
    console.log(person[key]);   // value 값 출력
}
```

<br/>
<b>[ 객체의 프로퍼티 추가 ]</b>

객체의 프로퍼티 내용을 추가하는 경우에는 `.`을 사용하거나 `[]` 대괄호를 사용한다.
```javascript
person.gender = 'female';
person['nationality'] = 'South Korea';
```

<br/>
<b>[ 객체의 프로퍼티 삭제 ]</b>

객체의 프로퍼티를 삭제하고 싶은 경우에는 `delete` 키워드를 통해 삭제한다.
```javascript
delete person.nationality;
```

<br/>
<hr/>

### 4.2 단축 프로퍼티

```javascript
const name = 'young';
const age = 24;

// 기본 형태
const person = {
    name: name,
    age: age,
    gender = 'female',
}

// 단축 프로퍼티 (위의 객체와 동일함)
const person = {
    name,
    age,
    gender = 'female',
}
```

위의 코드와 같이 key와 value 쌍을 묶어주는 것이 아닌 단축해서 객체를 생성할 수 있다.

<br/>
<hr/>

### 4.3 객체 활용 예시

함수에 값을 전달하여 객체를 반환하는 예시는 다음과 같다.

```javascript
function makeObject(name, age) {
    return {
        name: name,
        age: age,
    }
}

const Young = makeObject("Young", 24);
console.log(Young);
```
위의 코드에 대한 출력은 다음과 같다.

```
Object {
    age: 24,
    name: "Young"
}
```

<br/>
<hr/>

### 4.4 메소드(method)
메소드: 객체 프로퍼티로 할당된 함수 

```javascript
const person = {
    name: 'young',
    age: '24',
    introduce: function() {
        console.log('안녕하세요');
    }
}

person.fly();       // 안녕하세요
```
위의 introduce 함수를 메소드라고 할 수 있다.

위의 함수를 동일하게 단축 구문으로도 작성할 수 있다. 해당 코드는 다음과 같다.

```javascript
const person = {
    name: 'young',
    age: '24',
    introduce() {
        console.log('안녕하세요');
    }
}
```
단축 구문은 function 키워드를 생략하면 된다.

<br/>
<hr/>

### 4.5 this 키워드
`this`는 자신이 속한 객체 혹은 자신이 생성할 인스턴스를 가리키는 자기 참조 변수이다.

이 키워드는 크게 전역에서 사용할 때와 함수 내부에서 사용할 때로 나눌 수 있다.

함수 내부에서 this 키워드는 현재 함수를 실행하고 있는 그 객체를 참조한다.

```javascript
const person = {
    name: 'young',
    age: '24',
    introduce() {
        console.log(`안녕하세요. 저는 ${this.name}입니다.`);
    }
}
```
따라서 위의 코드에서 `${person.name}`을 사용한다면 문제가 발생할 수 있기 때문에, 자기 참조 변수인 this를 사용하는 것이 좋다.


this는 실행하는 시점인 런타임에 결정된다.

```javascript
let boy = {
    name: 'Male',
    introduce : function() {
        console.log(`안녕하세요, 저는 ${this.name}입니다.`);
    },
}

let girl = {
    name: 'Female',
    introduce : function() {
        console.log(`안녕하세요, 저는 ${this.name}입니다.`);
    },
}
```
만약 위의 introduce함수를 화살표 함수로 선언하면 동작이 달라진다.

> 화살표 함수는 일반 함수와는 달리 자신만의 this를 갖지 않는다.

> 화살표 힘수 내부에서 this를 사용한다면, 그 this는 외부에서 값을 가져온다.

따라서 화살표 함수에서 this를 사용하면 전역 객체를 가리키게 된다.<br/>
(브라우저 환경: window)

<br/>
<hr/>

지금까지 자바스크립트 문법 중 객체를 설명하는 포스트였습니다. 감사합니다:)
