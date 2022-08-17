---
layout: post
toc: true
title: "[자바스크립트] 자바스크립트 기초 문법 #1"
categories: 자바스크립트
tags: 자바스크립트, Javascript, 기초 문법
---

# &#91;Javascript&#93; 자바스크립트 기초 문법 1. 변수

## 0. 들어가기에 앞서..
해당 포스트는 Vue.js를 사용하여 프로젝트를 진행하기 위해 스스로 자바스크립트를 공부하게 됨으로써 기억을 위해 작성한 것입니다.

<br/>
<hr/>

## 1. 변수
기본적으로 Javascript에서는 ES6 이전에 변수 선언 방식으로 `var`을 사용했다.

```javascript
var name = 'young'
console.log(name);      // young

var name = 'ni'
console.log(name);      // ni
```
하지만 var을 사용해서 똑같은 이름의 변수를 선언해도 오류가 발생하지 않고 각각 다른 값을 출력한다.

해당 코드는 혼자서 작업을 할 때에는 문제가 발생하지 않을 수 있지만, 여러 사람과의 프로젝트에서는 어디에서 변수를 어떻게 사용할 지 모르며 값이 바뀔 수 있는 문제가 발생할 수 있다.

따라서 ES6 이후에는 var을 보완하는 `let`과 `const`로 변수를 선언할 수 있다.

<br/>
<hr/>

### 1.1 let과 const
```javascript
let name = 'young'
console.log(name);      // young

let name = 'ni'
console.log(name); 
// Uncaught SyntaxError: Identifier 'name' has already been declared.
```
먼저 `let`을 사용하여 변수를 선언하는 코드는 위와 같다.

해당 코드는 실행하면 `name`이 이미 선언되었다는 Uncaught SyntaxError 오류가 발생한다. 

마찬가지로 let 대신에 `const`를 사용하여 변수를 선언해도 같은 오류가 발생한다.

<br/>
<u><b>그렇다면 let과 const의 차이는 무엇일까?</b></u>
- <b>let</b>은 변수 재선언을 불가능하지만, <b>변수에 값 재할당이 가능</b>하다. 
    ```javascript
    let name = 'young'
    console.log(name);      // young

    name = 'ni'
    console.log(name);      // ni
    ```
- <b>const</b>는 변수 재선언 뿐만 아니라, <b>변수에 값 재할당이 불가능</b>하다.
    <br/>따라서 const는 절대로 바뀌지 않는 상수를 입력할 때 사용됨 (수정 X)
    <br/>보통 대문자로 선언을 하는 게 좋음.
    
    ```javascript
    const name = 'young'
    console.log(name);      // young

    name = 'ni'
    console.log(name);
    // Uncaught TypeError: Assignment to constant variable.
    ```

<br/>
따라서 javascript에서 변수를 선언할 때, <br/>
변하지 않는 값은 <span style="color:skyblue"><b>const</b></span> , <br/>
변할 수 있는 값은 <span style="color:orange"><b>let</b></span>으로 선언한다.

<br/>
<hr/>

### 1.2 참고 자료
<b>[ 변수 사용 시 주의사항 ]</b>
1. 변수는 문자와 숫자, $와 _만 사용이 가능함
2. 첫 글자는 숫자가 될 수 없음
3. 예약어는 변수명으로 사용할 수 없음
4. 가급적 상수는 대문자로 선언하는 것이 좋음
5. 변수명은 일기 쉽고 이해할 수 있게 선언하는 것이 좋음

<br/>
<hr/>

지금까지 자바스크립트 문법 중 변수를 설명하는 포스트였습니다. 감사합니다:)
