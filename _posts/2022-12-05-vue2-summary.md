---
layout: post
toc: true
title: "[Vue] Vue 2 정리"
categories: Vue.js
tags: Vue.js, 자바스크립트, Vue2
---

## 0. 들어가기에 앞서..

이번 포스트에서는 Vue.js 버전 2 사용 방법을 쉽게 이해할 수 있도록 정리하고자 합니다.

<br/>
<hr/>

## 1. Vue 2

Vue.js는 웹 애플리케이션의 사용자 인터페이스(UI)를 만들기 위해 사용하는 자바스크립트 프레임워크이다.

많은 이전에 진행하고 있는 프로젝트들은 Vue 3이 아닌 Vue 2를 활용하여 프런트엔드를 구현하였다. 

따라서 Vue.js 버전 2의 사용 방법에 대해서 소개하고자 한다.

### 1.1 프로젝트 생성

Vue.js 프로젝트를 위해서는 node.js를 설치해야 하는데, 본 포스트에서는 설치가 이미 되어있음을 가정한다.

#### 1.1.1 vue 설치

다음의 명령어로 `vue`를 설치한다. 

```
npm install -g vue
```

이후, vue가 설치되었는지 확인을 위해 다음의 커맨드를 입력한다.

```
npm vue -v
```

> 결과: `8.19.0`

<hr>

#### 1.1.2 vue-cli 설치

기본 vue 개발 환경을 설정을 쉽게 하기 위해 `vue-cli`를 설치한다. 

```
npm install -g @vue/cli
```

이후, vue-cli가 설치되었는지 확인을 위해 다음의 커맨드를 입력해 버전을 확인한다.

```
vue --version
```

> 결과: `@vue/cli 5.0.8`

<hr>

#### 1.1.3 프로젝트 생성

위의 `vue`와 `vue-cli`를 이미 설치한 적이 있는 경우에는 바로 다음의 명령어를 수행해 프로젝트를 생성할 수 있다.

```
vue create 프로젝트명
```

이후에는 vue의 버전을 선택하는데, 아래의 사진과 같이 Vue 2를 선택하면 된다.

![select-vue-version](https://user-images.githubusercontent.com/78736070/208833553-2bc6c6d2-a999-4331-8efd-57ec91400d16.jpeg)

간단하게 위의 명령어 하나로 Vue.js 프로젝트를 생성할 수 있다.

cf. 프로젝트 생성한 이후에 해당 폴더로 이동하고, 다음의 명령어로 서버를 실행할 수 있다.

```
npm run serve
```

기본적으로 개발 서버를 시작한 후에 애플리케이션은 `localhost:8080`에서 실행된다.

<br/>
<hr/>

### 1.2 데이터와 메서드

#### 1.2.1 데이터 (data)

Vue 인스턴스가 생성되면 객체에서 찾은 모든 속성 data를 Vue의 반응성 시스템에 추가한다.

즉, 데이터를 사용하기 위해서 기본적으로 Vue 인스턴스의 data에 정의해 사용한다.

이때, data는 객체 형태 혹은 함수형으로 선언할 수 있다.

<b>[ 객체 형태의 data ]

Vue는 하나의 인스턴스 안에서 데이터를 작업할 때에는 굳이 함수로 반환하지 않아도 되고 객체 형태로 선언해도 된다.
  
- 예제 코드

```javascript
new Vue({
    el: '#app',
    data: {
        id: 1,
        name : 'youngniw'
    }
});
```

<br/>

<b>[ 함수형 형태의 data ]

하지만 컴포넌트에서의 data 선언은 각각의 컴포넌트마다 개별 데이터를 관리해야 하기 때문에 함수형으로 선언해야 한다.

> 컴포넌트마다 data를 객체로 생성하여 사용하더라도 자바스크립트의 작동 방식에 따라 구성 요소의 모든 단일 인스턴스가 data 속성을 공유하여 값을 참조하기 때문에 data가 컴포넌트 마다의 데이터 관리에서 문제가 발생할 수 있다.
  
- 예제 코드

```vue
<template>
    <div>{{ name }}</div>
</template>
<script>
export default {
    data() {
        return {
            id: 1,
            name: 'youngniw'
        };
    }
};
</script>
```

선언된 데이터를 `<template>`내에서 data 내의 변숫값을 출력하기 위해서는 `{{ 변수명 }}` 을 이용하면 된다.

이때 객체 속성에 접근하고자 할 때는 `.`을 이용해 가능하다.

<hr/>

#### 1.2.3 메서드 (methods)

Vue 인스턴스에서는 메서드 또한 선언할 수 있다.

메서드는 매개변수를 전달받아 메서드 내에서 사용할 수 있다.

이에 대한 예제 코드는 다음과 같다.

```javascript
export default {
    data() {
        return {
            id: 1,
            name: 'youngniw'
        };
    },
    methods: {
        getId() {
            console.log(this.id);
        },
        introduce: function(greeting) {
            return `${greeting}, 저는 ${this.name}입니다.`;
        }
    }
};
```

> Vue 인스턴스 내의 메서드에서는 화살표 함수를 사용하면 안 됨:)

<br/>
<hr/>

### 1.3 데이터 바인딩

script 내에서 정의한 인스턴스의 data를 `v-bind:`를 통해 데이터를 바인딩 할 수 있다.

`v-bind`는 생략이 가능하다.

예제 코드는 다음과 같다.

```vue
<template>
    <input 
        v-bind:type="type"
        :value="name"
        required
    >
</template>
<script>
export default {
    data() {
        return {
            id: 1,
            name: 'youngniw',
            type: 'text'
        }
    }
}
</script>
```

Vue에서는 `v-model`을 사용한 데이터 양방향 바인딩도 지원한다.

예제 코드는 다음과 같다.

```vue
<template>
    <input 
        :type="type"
        v-model="name"
        required
    >
</template>
<script>
export default {
    data() {
        return {
            id: 1,
            name: 'youngniw',
            type: 'text'
        }
    }
}
</script>
```

위의 코드에서 input 값을 변경할 경우에는 data의 name 변숫값에 입력한 값이 저장된다.

> 양방향 데이터 바인딩은 props로 내려받은 값은 불가능하기 때문에 바로 v-model에서 사용할 수 없다!


<br/>
<hr/>

### 1.4 이벤트

이벤트: 어떤 사건이 발생했을 때, 어떤 행동을 취할 것이다!를 정의한 것

#### 1.4.1 이벤트 연결 v-on

script 내에서 정의한 인스턴스의 메서드를 `v-on:`를 통해 이벤트로 연결할 수 있다.

`v-on:`은 `@`로 축약해서 사용 가능하다.

버튼 클릭 시에 1씩 덧셈을 수행하는 예제 코드는 다음과 같다.

```vue
<template>
    <div>
        count: {{ count }}
    </div>

    <button 
        @click="addCount"
    >1 더하기</button>
</template>
<script>
export default {
    data() {
        return {
            count: 0
        }
    },
    methods: {
        addCount() {
            this.count++;
        }
    }
}
</script>
```

<b>[ 이벤트 수식어 ]</b>

기본적으로 이벤트 핸들러 내부에서 `event.preventDefault()` 혹은 `event.stopPropagation()`을 호출함으로써 이벤트의 더블링을 막을 수 있다.

Vue 2에서는 `v-on` 이벤트에서 이벤트 수식어를 제공함으로써 쉽게 사용할 수 있다.

- `.stop`: 이벤트가 전파되는 것을 중단 (ex. `@click.stop="함수명"`)
- `.prevent`: 태그의 기본 이벤트의 자동 실행을 중단 (ex. `@submit.prevent="함수명"` -> 데이터를 전달하지만 새로고침은 막음)
- `.capture`: 기존 우선순위를 무시하고 가장 먼저 발동되게 함 (ex. `@click.capture="함수명"`)
- `.self`: 발생 단계에서만 이벤트를 발생시킴 (ex. `@click.self="함수명"` -> 자기 자신 만을 클릭해야 발동됨)
- `.once`: 이벤트를 한 번만 실행 (ex. `@click.once="함수명"`)
- `.passive`: 기본 이벤트를 취소할 수 없게 함 (`event.preventDefault() 실행 안됨`)

<br/>

<b>[ 키 수식어 ]</b>

키 이벤트를 수신할 때 `v-on`에 대한 키 수식어를 추가할 수 있다.

예시는 다음과 같다.
 
```html
<input @keyup.enter="함수명">
```

<hr/>

#### 1.4.2 자식 컴포넌트에 데이터 전달 props

`props`: 부모 컴포넌트에서 자식 컴포넌트로 데이터 전달 시에 사용되는 단방향 데이터 전달 방식

- 부모 컴포넌트에서 자식 컴포넌트 호출 시, 자식 컴포넌트 태그 내 `v-bind`나 `:` 키워드를 통해 데이터를 전달
- 자식 컴포넌트에서 props 객체를 통해 데이터 수신

이에 대한 예제 코드는 다음과 같다.

1. 부모 컴포넌트

```vue
<template>
    <div>
        부모
    </div>

    <ChildComponent :name="name" />
</template>

<script>
import ChildComponent from '@/components/ChildComponent.vue';

export default {
    components: {
        ChildComponent
    },
    data() {
        return {
            name: 'youngniw'
        }
    }
}
</script>

<style>
</style>
```

2. 자식 컴포넌트

```vue
<template>
    <div>
        자식
    </div>

    <div>이름: {{ name }}</div>
</template>

<script>
export default {
    props: {
        name: {
            type: String,
            required: true,
        }
    }
}
</script>

<style>
</style>
```

<hr/>

#### 1.4.3 부모 컴포넌트로 데이터 전달 emit

`emit`: 자식 컴포넌트에서 부모 컴포넌트로 데이터 전달 시에 사용되는 단방향 데이터 전달 방식

- 부모 컴포넌트에서 자식 컴포넌트 호출 시, 자식 컴포넌트 태그 내의 `v-on`이나 `@` 키워드를 통해 메서드 연결
- 자식 컴포넌트에서 `$emit`을 통해 데이터 전달

이에 대한 예제 코드는 다음과 같다.

1. 부모 컴포넌트

```vue
<template>
    <div>
        부모
    </div>

    <ChildComponent @update-name="fetchName" />
</template>

<script>
import ChildComponent from '@/components/ChildComponent.vue';

export default {
    components: {
        ChildComponent
    },
    data() {
        return {
            name: ''
        }
    },
    methods: {
        fetchName(name) {
            this.name = name;
        }
    }
}
</script>

<style>
</style>
```

2. 자식 컴포넌트

```vue
<template>
    <div>
        자식
    </div>

    <input 
        :value="name"
        @input="updateName"
        required
    >
</template>

<script>
export default {
    data() {
        return {
            name: ''
        }
    },
    methods: {
        updateName(evt) {
            this.$emit('update-name', evt.target.value);
        }
    }
}
</script>

<style>
</style>
```

<br/>
<hr/>

### 1.5 Vue 인스턴스 속성: Computed, Watch

#### 1.5.1 Computed

`computed`: 연산 결과를 캐싱함으로써, 변경이 일어나지 않을 시에 이미 연산 처리된 결과를 반환함

`methods`와의 차이점은 메서드는 다시 렌더링될 때마다 실행되기 때문에 캐싱의 이득을 얻을 수 없다는 것이다.

예제 코드는 다음과 같다.

```vue
<template>
    <div>
        {{ greeting }}
    </div>
</template>

<script>
export default {
    data() {
        return {
            name: 'youngniw'
        }
    },
    computed: {
        greeting() {
            return `${this.name}님, 반갑습니다.`
        }
    },
}
</script>
```

<hr/>

#### 1.5.2 Watch

`watch`: `state` 나 `props`를 감시하고 해당 값이 변경됐을 때의 행동 정의

이름 변경 시에 대한 풀네임을 구하는 예제 코드는 다음과 같다.

```vue
<script>
export default {
    data() {
        return {
            firstname: '',
            lastname: '',
            fullname: ''
        }
    },
    watch: {
        firstname(value, oldValue) {
            this.fullname = `${value} ${this.lastname}`
        },
        lastname(value, oldValue) {
            this.fullname = `${this.firstname} ${value}`
        }
    },
}
</script>
```

> watch 하고자 하는 값이 객체(Object) 형태이거나 배열인 경우에는 메서드의 `deep` 속성 값을 `true`로 설정함으로써 객체 내의 내용도 감시할 수 있다.
>
> 처음 화면 렌더링 시에 변수를 감지하고 싶다면 `immediate` 속성 값을 `true`로 설정하면 된다.

<br/>
<hr/>

### 1.6 Vue 인스턴스 라이프사이클

|라이프 사이클 훅|설명|
|--|--|
|`beforeCreate()`|인스턴스가 초기화될 때 호출|
|`created()`|인스턴스가 모든 상태 관련 옵션 처리를 완료한 후에 호출 (data, computed 속성, 메서드, watchers 사용 가능)|
|`beforeMount()`|컴포넌트가 마운트 되기 직전에 호출|
|`mounted()`|컴포넌트가 마운트 된 후 호출 (DOM에 접근 가능)|
|`beforeUpdate()`|반응 상태 변경으로 인해 컴포넌트가 DOM 트리를 업데이트하기 직전에 호출|
|`updated()`|반응 상태 변경으로 인해 컴포넌트가 DOM 트리를 업데이트한 후에 호출|
|`beforeDestroy()`|컴포넌트 인스턴스가 마운트 해제되기 직전에 호출|
|`destroyed()`|컴포넌트가 마운트 해제된 후에 호출|

<br/>
<hr/>

### 1.7 조건부 및 루프

#### 1.7.1 조건부 렌더링 v-if

- `v-if`: 블록을 조건부로 렌더링 하는 데 사용
  - 표현식의 값이 true를 반환하는 경우에만 블록이 렌더링 됨
- `v-else-if`: v-if 조건에 해당하지 않으며 v-else-if 조건에 해당하는 경우, 해당 블록을 조건부로 렌더링 하는 데 사용
- `v-else`: 앞의 모든 조건이 성립하지 않은 경우에 해당 블록을 조건부로 렌더링 하는 데 사용

```html
<div v-if="isMe">나는 youngniw입니다.</div>
<div v-else>나는 youngniw가 아닙니다.</div>
```

> `v-show`: `v-if`와 달리 조건이 성립하지 않은 경우에도 태그를 렌더링 하지만 스타일의 `display: none` 처리를 함

<hr/>

#### 1.7.2 리스트 렌더링 v-for

- `v-for`: 배열 혹은 객체를 기반으로 항목 리스트를 렌더링 할 수 있음
  
- 배열의 경우
  - v-for="item in items"
  - v-for="(item, index) in items"

- 객체의 경우
  - v-for="value in object"
  - v-for="(value, key) in object"

> v-for을 사용하는 경우에는 key 값을 명시해 줘야 함

<br/>
<hr/>

[ 참고 사이트 ]

- [Vue.js 2 공식 문서](https://v2.vuejs.org/v2/guide/)

<br/>

지금까지 Vue 2에 대해 정리하는 포스트였습니다.

감사합니다:)
