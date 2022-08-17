---
layout: post
toc: true
title: "[자바스크립트] 자바스크립트 기초 문법 3. 함수"
categories: 자바스크립트
tags: 자바스크립트, Javascript, 기초 문법
---

## 0. 들어가기에 앞서..
해당 포스트는 Vue.js를 사용하여 프로젝트를 진행하기 위해 스스로 자바스크립트를 공부하게 됨으로써 기억을 위해 작성한 것입니다.

지난 번에는 자바스크립트의 자료형을 소개했다면, 이번 포스트는 함수에 대한 설명입니다.

<br/>
<hr/>

## 3. 함수
함수는 하나의 코드를 재사용함으로써 중복된 코드가 줄어들고 유지보수도 쉬워진다.

- 함수는 한 번에 한 작업을 하는 것이 좋음
- 함수명은 읽기 쉽고 어떤 동작을 하는지 알 수 있게 지어야 함

자바스크립트에서는 함수를 선언하고 사용하는 방법은 다양하다.
1. 함수 선언문: function 키워드 사용해 선언 및 호출
2. 함수 표현식: 이름이 없는 함수를 만들고 변수에 값을 할당하는 것처럼 함수가 변수에 할당함

<br/>
<hr/>

### 3.1 함수 기초 (함수 선언문)
함수를 작성할 때는 `funcion`키워드를 통해 함수명과 괄호 안에 매개변수를 작성하고, 함수에서 실행할 코드를 중괄호로 감싸서 작성하면 된다.

```javascript
function sayHello(name) {
    console.log(`Hello~ ${name}.`);
}

sayHello(young);    // Hello~ young.
```
위의 코드는 매개변수로 이름을 전달하고, 콘솔 창에 Hello와 함께 이름을 출력한다.

또한 함수에서 값을 반환하고 싶다면 `return` 키워드를 이용하면 된다.

```javascript
function add(num1, num2) {
    return num1 + num2;
}

const result = add(3, 5);
console.log(result)     // 8
```

참고로 return 문이 없거나 return;만 있는 함수는 항상 undefined를 반환한다.

또한 return 키워드를 통해 함수를 즉시 종료하는 목적으로도 사용된다.

<br/>
<hr/>

### 3.2 지역변수와 전역변수
변수는 유효범위에 따라 지역변수와 전역변수로 구분 가능하다.

- 전역변수: 함수 외부에서 선언된 변수 (프로그램 전체에서 접근할 수 있음)
- 지역변수: 함수 내부에서 선언된 변수 (함수 실행 시 생성되고 함수가 종료되면 소멸됨)

참고로 함수 내부에서 사용하는 지역 변수는 함수 외부에서는 사용하지 못한다.
이에 대한 코드는 다음과 같다.

```javascript
function hello(name) {
    let msg = 'Hello';
    if (name) {
        msg += `~ ${name}`;
    }
    console.log(msg);
}

hello(young);   // Hello~ young

console.log(msg);
// Uncaught ReferenceError: msg is not defined
```
위의 코드에서 오류가 나지 않기 위해서는 msg 변수 선언 및 할당을 함수 밖에서 해야 한다.

<br/>
<u><b>만약 같은 변수명의 전역변수와 지역변수가 있다면 어떤 변수로 인식할까?</b></u><br/>
-> 변수를 사용하는 위치에 따라 달라진다.

```javascript
let msg = 'global';
console.log(msg);           // 출력: global

function hello() {
    let msg = 'local';
    console.log(msg);
}
hello();                    // 출력: local

console.log(msg);           // 출력: global
```
위와 같이 함수 내부에서는 전역변수와 같은 이름의 지역변수가 있다면, 해당 영역에서는 지역변수를 우선으로 인식한다.

참고로 매개변수로 받은 값은 복사 후에 함수의 지역변수가 된다.

<br/>
<hr/>

### 3.3 함수의 매개변수 활용 예시
매개변수는 함수를 호출할 때 넘기는 값을 담고 있다.

```javascript
function hello(name) {
    let newName = name || "javascript";
    let msg = `Hello~ ${newName}`;
    console.log(msg);
}

hello();        // 출력: Hello~ javascript
hello(young);   // 출력: Hello~ young
```
위의 코드와 같이 논리연산자 OR을 사용하여 매개변수에 담긴 값이 없는 undefined 형태일 때는 설정한 기본값으로 출력되게 할 수 있다.

논리연산자 OR을 사용하지 않고 매개변수에 default 값을 지정할 수 있는데, 해당 코드는 다음과 같다.

```javascript
function hello(name = 'javascript') {
    let msg = `Hello~ ${name}`;
    console.log(msg);
}

hello();        // 출력: Hello~ javascript
hello(young);   // 출력: Hello~ young
```
이와 같이 함수의 인자로 전달한 값이 없을 시에는 설정한 기본값으로 할당된다.

<br/>
<hr/>

### 3.4 함수 표현식
함수 표현식은 함수 선언문과 사용 및 실행, 그리고 동작하는 방식도 동일하다. 

<u><b>그렇다면 차이가 무엇일까?</b></u><br/>
-> <span style="color:blue"><b>호출할 수 있는 타이밍 !</b></span>

<b>[ 함수 선언문 ]</b>

-> 어디서든 호출할 수 있음 (함수를 선언한 곳 위에서도 호출 가능)

<b>호이스팅</b>: 인터프리터가 변수와 함수의 메모리 공간을 선언 전에 미리 할당하는 것

따라서 자바스크립트는 호이스팅으로 인해 실행 전 초기화 단계에서 코드의 모든 함수 선언을 찾아서 생성하기 때문에 함수 호출을 어디서든 가능함

<br/>
<b>[ 함수 표현식 ]</b>

-> 코드에 도달하면 생성

```javascript
let hello = function() {
    console.log('Hello');
}

hello();    // Hello
```
함수 표현식은 자바스크립트가 한 줄씩 읽으면서 실행하며, 해당 코드에 도달했을 때 비로소 생성된다.

그렇기 때문에 그 이후에만 사용할 수 있다.

<br/>
<hr/>

### 3.5 화살표 함수
화살표 함수는 위에서 설명한 함수를 더 간결하게 작성할 수 있다.

```javascript
// 기본 함수
let add = function(num1, num2) {
    return num1 + num2;
}

// 화살표 함수1 (위와 동일함)
let add = (num1, num2) => {
    return num1 + num2;
}

// 화살표 함수2 (위와 동일함)
let add = (num1, num2) => (
    num1 + num2;
)

// 화살표 함수3 (위와 동일함)
let add = (num1, num2) => num1 + num2;
```

(화살표 함수1) 위에 코드와 같이 화살표 함수를 사용한다면, `function` 키워드는 사라지고 매개변수 옆에 `=>`가 생긴다.

(화살표 함수2) 해당 함수는 return문이기 때문에 `return` 키워드는 사라지고, 중괄호{} 대신 소괄호()를 사용해 표현할 수 있다.

(화살표 함수3) return문이 한 줄이라면 함수 내용 부분의 소괄호()를 생략할 수 있다.

cf. 함수의 인수가 없다면 함수 내용 정의 괄호를 생략할 수 없음<br/>
cf. return문이 있더라도 return 전에 여러줄의 코드가 있다면 중괄호를 사용해야 함

<br/>
<hr/>

지금까지 자바스크립트 문법 중 함수를 설명하는 포스트였습니다. 감사합니다:)
