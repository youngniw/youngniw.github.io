---
layout: post
toc: true
title: "[자바스크립트] 자바스크립트 기초 문법 5. 배열"
categories: Javascript
tags: 자바스크립트, Javascript, 문법
---

## 0. 들어가기에 앞서..
해당 포스트는 Vue.js를 사용하여 프로젝트를 진행하기 위해 스스로 자바스크립트를 공부하게 됨으로써 기억을 위해 작성한 것입니다.

지난 번에는 자바스크립트의 객체(Object)를 소개했다면, 이번 포스트는 배열에 대한 설명입니다.

<br/>
<hr/>

## 5. 배열(Array)
배열: 순서가 있는 리스트

객체는 중괄호{}를 이용해 생성한다면, 배열은 대괄호[]를 이용해 생성할 수 있다.

<br/>
<hr/>

### 5.1 배열의 특징
배열은 문자 뿐만 아니라 숫자, 객체, 함수 등도 포함할 수 있다.

```javascript
let array = ['young', 
             24, 
             false, 
             { 
                gender: 'female', 
                nationality: 'South Korea'
             }, 
             function() {
                console.log('Hello') 
             }
            ];
```

<br/>
<hr/>

### 5.2 배열의 길이
`length`를 통해 배열의 길이를 확인할 수 있다.

배열의 길이는 해당 배열이 갖고 있는 요소의 개수를 말한다.

```javascript
array.length    // 5
```

<br/>
<hr/>

### 5.3 배열의 메소드

<b>[ push() ]</b>

push 함수는 배열 끝에 추가할 때 호출한다.

```javascript
let days = ['월', '화', '수', '목', '금', '토'];

days.push('일');

console.log(days);  // ['월', '화', '수', '목', '금', '토', '일']
```

<br/>
<b>[ pop() ]</b>

pop 함수는 배열 끝 요소를 삭제할 때 호출한다.

```javascript
let days = ['월', '화', '수', '목', '금', '토'];

days.pop();

console.log(days);  // ['월', '화', '수', '목', '금']
```

<br/>
<b>[ shift()와 unshift() ]</b>

shift 함수는 배열 앞 요소를 삭제할 때 호출한다.

```javascript
let days = ['월', '화', '수'];

days.shift();

console.log(days);  // ['화', '수']
```

unshift 함수는 배열 앞에 요소를 추가할 때 호출한다.

```javascript
let days = ['월', '화', '수'];

days.unshift('일');

console.log(days);  // ['일', 월', '화', '수']
```

참고로 push()와 unshift()는 여러 요소를 한번에 추가할 수 있다.

<br/>
<hr/>

### 5.4 배열 반복문
배열을 쓰는 가장 큰 이유는 반복을 위해서이다. 

특히 배열의 길이 length를 통해 for문을 반복함으로써 순회가 가능하다.

또한 for문을 돌 때 index를 활용하여 0부터 length-1까지 순회하면 배열의 모든 요소에 접근할 수 있다.

```javascript
let days = ['월', '화', '수'];

for (let index=0; index<days.length; index++) {
    console.log(days[index]);
}
```

위와 같이 index를 통해 순회할 수 있지만 for ... of를 통해 배열을 순회할 수도 있다.
```javascript
let days = ['월', '화', '수'];

for (let day of days) {
    console.log(day);
}
```

위의 2가지 방식은 인덱스의 사용 유무에 따라 선택적으로 활용하면 된다.

<br/>
<hr/>

지금까지 자바스크립트 문법 중 배열을 설명하는 포스트였습니다. 감사합니다:)
