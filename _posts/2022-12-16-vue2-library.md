---
layout: post
title: "[Vue] Vue 2 - 라이브러리 (Vue Router, Vuex)"
categories: Vue.js
tags: [Vue.js, Vue2, javascript, 자바스크립트]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 Vue.js 버전 2에서 사용 가능한 Vue Router와 Vuex 라이브러리에 대해 설명하고자 합니다.

이전 Vue 2 정리 포스트를 참고하실 분은 아래의 링크에서 확인해 주세요:)

[Vue 2 정리 포스트]()

<br/>
<hr/>

## 1. Vue Router

뷰 라우터(Vue Router): vue에서 페이지 간 이동을 위해 사용하는 라이브러리

- 페이지 이동 시 url이 변경된다면 변경된 요소의 영역에 컴포넌트 갱신

### 1.1 라이브러리 설치

다음의 명령어로 `vue-router`를 설치한다.

```
npm install vue-router
```

> 만약 설치된 vue의 버전이 높은 경우에는 오류가 발생할 수 있어, Vue 버전에 맞게 Vue Router의 버전을 다음과 같이 명시해야 한다.

```
# vue 2
npm install vue-router@3

# vue 3
npm install vue-router@4
```

<br/>
<hr/>

### 1.2 설정

main.js 파일에 다음과 같이 router를 뷰 인스턴스에 사용하겠다고 추가한다.

```javascript
import Vue from 'vue';
import App from './App.vue';
import router from './router/index.js';

Vue.config.productionTip = false

new Vue({
    router,
    render: h => h(App),
}).$mount('#app')
```

<br/>
<hr/>

### 1.3 router 정보

보통 router는 src 폴더 내에 router 폴더를 생성해 index.js 파일에 필요 설정 정보를 작성한다.

예시 코드는 다음과 같다.

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
import Main from '@/pages/Main.vue';
import SearchCar from '@/pages/SearchCar.vue';
import CarList from '@/pages/CarList.vue';
import CarOperationHistory from '@/pages/CarOperationHistory.vue';

Vue.use(VueRouter);

const router = new VueRouter({
    mode: 'history',
    routes: [
        {
            // 메인 홈페이지
            path: '/',
            name: 'Main',
            component: Main
        },
        {
            // (페이지) 나의 차량 찾기
            path: '/search-car',
            name: 'SearchCar',
            component: SearchCar
        },
        {
            // (페이지) 차량 목록
            path: '/carlist',
            name: 'CarList',
            component: CarList
        },
        {
            // (페이지) 차량 운행 이력
            path: '/operation-history',
            name: 'CarOperationHistory',
            component: CarOperationHistory
        },
    ]
});

export default router;
```

위와 같이 기본적으로 router의 정보를 관리하는 router 객체를 생성한다.

<b>[ mode ]</b>

- mode의 종류: `hash`, `history`
  - `hash`
    - 기본적으로 설정되는 모드로
    - hash 모드로 동작하는 경우에는 URL에 `#`이 추가됨
    - 다른 URL 요청 시에는 전체 리소스에 대한 로딩 없이 SPA 페이지 이동만 발생
  - `history`
    - history 모드로 동작하는 경우에는 URL에 `#`이 추가되지 않음 (요청 URL 그 자체)
    - 다른 URL 요청 시에는 페이지 재로딩 발생
    - 따라서 URL 정의가 없는 곳에 요청한 경우에는 404 페이지 반환
    - 브라우저 히스토리 스택에 기록됨

<b>[ routes ]</b>

routes에서는 경로를 정의하는데, 각 경로는 컴포넌트에 매핑되어 해당 경로 도달 시 지정된 컴포넌트의 화면이 보이게 된다.

각 route 객체의 속성은 다음과 같다.

|속성|설명|
|--|--|
|`path`|라우트의 경로(URL)를 나타낼 문자열|
|`name`|경로에 대한 이름|
|`component`|해당 URL 페이지에 사용할 컴포넌트|
|`redirect`|다시 요청할 페이지 경로(URL)|
|`children`|중첩 라우팅을 위한 배열|

참고로 `path`에서는 Path Variable 또한 사용 가능하다. 

만약 경로에 사용할 변수를 명시해야 하는 경우에는 `:`를 사용하면 된다.

> `path: '/user/:id'`

반면, query를 사용할 경우에는 정보에 추가할 내용 없이 사용하면 된다.

> 이후에 HTML 내에서 `<router-view></router-view>`를 통해 경로와 일치하는 컴포넌트가 해당 위치에 렌더링 된다.
>
> 또한 스크립트 내에서 `this.$route`를 통해 현재 경로에 대해 접근하여 정보를 조회할 수 있으며, `this.$router`를 통해 라우터에도 접근할 수 있다. 

<br/>
<hr/>

### 1.4 동적 라우팅

라우터를 이용해 동적으로 파라미터를 연결하거나 페이지를 이동해야 하는 경우가 종종 있다. 

이 경우에는 HTML 내에서 `<router-link to="이동경로"></router-link>`를 이용하거나, `this.$router`를 통해 라우터에 접근하여 메서드를 사용해 페이지를 이동할 수 있다.

<b>[ \<router-link> ]</b>

- 라우터에서 페이지 이동을 위한 태그
- HTML의 \<a> 태그로 렌더링 됨

- 속성

|속성|설명|
|--|--|
|`to`|이동할 URL 문자열 혹은 route 정보 명시|
|`replace`|내비게이션은 히스토리를 남기지 않음 (=`router.replace()`)|
|`append`|to로 지정한 상대 경로가 현재 경로에 추가됨|
|`tag`|`<router-link>`를 `<a>`가 아닌 다른 태그로 렌더링|
|`exact`|URL 링크가 완전히 일치하는 경우에만 활성화(active) 되도록 설정|
  
- 예시 코드

```html
<router-link to="/user" exact>User</router-link>
<router-link :to="`/user?id=${id}`" replace>User</router-link>
<router-link :to="{ name: 'user', params: { id: 1 }}">User</router-link>
<router-link :to="{ path: '/user', query: { id: 1 }}">User</router-link>
```

<b>[ router 메서드 ]</b>

|메서드|설명|
|--|--|
|`push()`|URL로 이동하는 메서드로, 히스토리 스택에 추가되기 때문에 뒤로 가기 시 이전 URL로 이동|
|`replace()`|현재 URL을 대체하는 메서드로, 히스토리 스택에 쌓지 않음|
|`go()`|지정한 숫자만큼 뒤로 가기(음수) 또는 앞(양수)으로 가기|

참고로 위와 같이 URL 이동하는 기능 말고도 router는 다양한 기능을 제공한다.

> `beforeEach(to, from , next) => {})`
> - 내비게이션이 트리거 될 때마다 내비게이션 가드가 작성된 순서에 따라서 호출되기 전의 모든 경우에 발생함
> - next()를 호출하지 않는 경우 다음 훅으로 이동되지 않음
>
> `afterEach(to, from) => {})`
> - 내비게이션 작업이 완료된 후에 호출됨

<br/>
<hr/>

## 2. Vuex

Vuex: vue.js에서 컴포넌트의 상태 관리를 위해 사용되는 라이브러리

- Vuex를 사용함으로써 부모-자식과 같은 관계의 컴포넌트에 데이터를 주고받는 불필요하게 중복된 코드를 줄일 수 있음
- Vuex 애플리케이션에는 store가 있는데, store는 기본적으로 애플리케이션 상태를 보관하는 컨테이너

### 2.1 라이브러리 설치

다음의 명령어로 `vuex`를 설치한다.

```
npm install vuex
```

<br/>
<hr/>

### 2.2 설정

main.js 파일에 다음과 같이 store를 뷰 인스턴스에 사용하겠다고 추가한다.

```javascript
import Vue from 'vue';
import App from './App.vue';
import store from './store/index.js';

Vue.config.productionTip = false;

new Vue({
    store,
    render: h => h(App),
}).$mount('#app')
```

<br/>
<hr/>

### 2.3 Store 정보

보통 store는 src 폴더 내에 store 폴더를 생성해 index.js 파일에 상태 관리할 데이터 및 관련 설정 정보를 작성한다.

예시 코드는 다음과 같다.

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

const store = new Vuex.Store({
    modules: {},
    state: {
        count: 0,
    },
    getters: {
        GET_COUNT(state) {
            return state.count;
        }
    },
    mutations: {
        incrementCount(state) {
            state.count++;
        }
    },
    actions: {},
    plugins: [],
})

export default store;
```

store는 위와 같이 `modules`, `state`, `getters`, `mutations`, `actions` 등으로 구성된다.

아래에서 각각에 대한 설명을 참고할 수 있다.

cf. 애플리케이션의 모든 상태는 하나의 큰 객체 안에 포함되는데, modules를 이용함으로써 store를 나눌 수 있다.
  - `namespaced`를 이용하여 여러 개의 스토어를 구분

<br/>
<hr/>

### 2.4 state

저장소에서 저장하는 데이터를 의미하며, Vue 인스턴스의 `data`와 동일한 기능을 한다.

`state`는 객체의 형태로 객체 내부의 값으로 데이터들을 저장할 수 있다.

```javascript
state: {
    count: 0,
},
```

`state` 내의 값을 조회할 경우에는 `store.state.변수명`으로 가능하다.

cf. `mapState`
- 컴포넌트에서 여러 저장소의 state 혹은 getter를 사용해야 하는 경우 이러한 모든 computed 속성을 선언하면 반복적이고 장황해질 수 있음
- 이에 따라 `mapState`를 통해 간단하게 연결할 수 있음

```javascript
computed: {
    ...mapState(['count'])
}
```

<br/>
<hr/>

### 2.5 getters

저장소에 저장된 state를 기반으로 조회하거나 계산해야 하는 경우에는 getters에 정의하면 된다.

`getters`는 Vue 인스턴스의 `computed`와 동일한 기능을 한다. 
따라서 getter의 결과는 종속성을 기반으로 캐시 되며 일부 종속성이 변경된 경우에만 재평가된다.

`getters`는 첫 번째 인수로 state를 받는다.

```javascript
state: {
    users: [
        { id: 1, name: 'young' },
        { id: 2, name: 'eun' },
    ],
},
getters: {
    nameLengthOverThree(state) {
        return state.users.filter(user => user.name.length > 3);
    }
}
```

`getters` 내의 함수를 조회할 경우에는 `store.getters.함수명`으로 가능하다.

cf. `mapGetters`
- 저장소 내의 getters를 로컬 computed 속성에 매핑함
- 만약 다른 이름으로 매핑할 경우에는 객체를 사용해 가능함

```javascript
computed: {
    ...mapGetters(['nameLengthOverThree'])
}
```

<br/>
<hr/>

### 2.6 mutations

Vuex의 Store에서 실제로 상태(state)를 변경하는 유일한 방법은 `mutation`을 커밋(`commit`) 하는 것이다.

`mutations`는 첫 번째 인수로 state를 받는다. 그리고 선택적으로 두 번째 인수로 페이로드를 받을 수 있다.

```javascript
state: {
    count: 0,
},
mutations: {
    incrementCount(state) {
        state.count++;
    }
}
```

`mutations` 내의 함수를 호출할 경우에는 `store.commit('함수명')`으로 가능하다.

<br/>
<hr/>

### 2.7 actions

`actions`는 보통 비동기 작업을 수행하며 `mutations`를 커밋 한다.

`actions`는 첫 번째 인수로 context를 받으며, 선택적으로 두 번째 인수로 페이로드를 받을 수 있다.

즉, context 객체를 수신하기 때문에 컨텍스트의 `commit`을 하거나, `state` 혹은 `getters`에 접근할 수 있다.

```javascript
state: {
    count: 0,
},
mutations: {
    incrementCount(state) {
        state.count++;
    }
},
actions: {
    incrementCount(context) {   // 구조 분해 가능 context -> { commit, state }
        context.commit('incrementCount');
    }
}
```

`actions` 내의 함수를 호출할 경우에는 `store.dispatch('함수명')`로 가능하다.

> 그렇다면 `actions`과 `mutations`과의 차이점은 무엇일까?
>
> `mutations`는 동기식 트랜잭션이다.
> 반면 `actions`는 비동기 작업이 포함될 수 있으며, 상태를 변경하는 대신 `mutations`를 커밋(commit) 한다.

<br/>
<hr/>

[ 참고 사이트 ]

- [Vue Router 버전 3 공식 가이드 문서](https://v3.router.vuejs.org/kr/guide/)
- [Vuex 버전 3 공식 가이드 문서](https://v3.vuex.vuejs.org/guide/)

<br/>

지금까지 Vue 2에서 페이지 이동을 위해 사용하는 Vue Router와 상태 관리를 위해 사용하는 Vuex에 대해 정리하는 포스트였습니다.

감사합니다:)
