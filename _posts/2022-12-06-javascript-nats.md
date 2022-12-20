---
layout: post
toc: true
title: "[자바스크립트] Javascript로 NATS 통신하기"
categories: 자바스크립트
tags: NATS, 자바스크립트, javascript, Vue.js
---

## 0. 들어가기에 앞서..

이전 포스트에서 NATS에 대해 설명하였다면, 이번 포스트에서는 Javascript 언어로 NATS를 활용하는 코드에 대해 설명하고자 합니다.

> 참고 코드 (Vue.js로 NATS 적용)

[https://github.com/youngniw/demo-nats-vue3](https://github.com/youngniw/demo-nats-vue3)

<br/>

![NATS 이미지](https://user-images.githubusercontent.com/78736070/206066890-fd1bde77-f950-47e9-bc1c-05b2590421dd.png)

NATS 공식 문서는 아래의 링크에서 확인할 수 있습니다.

[https://docs.nats.io/](https://docs.nats.io/)

<br/>
<hr/>

## 1. Javascript로 Nats 활용

### 1.1 라이브러리 설치

본 포스트에서는 NATS 서버 통신을 위해 NATS에서 제공하는 웹소켓 라이브러리를 사용할 것이다.

```bash
npm install nats.ws
```

<br/>
<hr/>

### 1.2 Nats 서버에 연결 및 연결 종료

- Nats 서버 연결을 위해서는 nats.ws의 `connect` 메서드를 사용
- Nats 서버와의 연결을 끊기 위해서는 `close` 메서드 사용

<b>[ connect 메서드 ]</b>

- 서버와 상호 작용하는 데 사용할 수 있는 연결 반환
- 많은 연결 옵션을 지정하여 클라이언트의 동작을 커스터마이즈 할 수 있음
  
> 연결 옵션

|옵션|기본값|설명|
|--|--|--|
|servers|`"localhost:4222"`|서버에 대한 호스트 포트의 문자열 또는 배열|
|port|`4222`|연결할 포트<br/>(서버를 지정하지 않은 경우에만 사용)|
|user||연결을 위한 사용자 이름|
|pass||연결을 위한 사용자의 비밀번호|
|reconnect|`true`|클라이언트가 재연결을 시도할지 여부|
|reconnectTimeWait|`2000`|연결이 끊긴 경우, 클라이언트가 다시 연결을 시도할 때까지 대기할 시간<br/>단위: ms(밀리초)|
|maxReconnectAttempts|`10`|최대 재연결 시도 횟수 설정<br/>`-1` 값: 제한 없음|
|timeout|`20000`|클라이언트가 연결이 설정되기를 기다리는 시간|
|inboxPrefix|`"_INBOX"`|자동으로 작성된 inbox에 대한 접두사 지정 - `createInbox(prefix)`|

- 기본적으로 127.0.0.1:4222에서 연결을 시도
- 연결이 끊어지면 클라이언트가 다시 연결을 시도
- 포트(로컬 연결용) 또는 서버 옵션의 전체 호스트 포트를 지정하여 연결할 서버를 커스터마이즈 가능
  - 서버 옵션: 단일 호스트 포트(문자열) or 호스트 포트 배열

<br/>

<b>[ close 메서드 ]</b>

- 연결 객체의 `close` 메서드를 호출함으로써 연결 종료
- 예기치 않은 오류가 발생할 경우 연결이 종료될 수도 있음
- 기본적으로 클라이언트는 `close` 메서드 호출 이외의 이유로 연결이 닫힌 경우에는 항상 재연결 시도
- 연결이 닫힌 이유를 확인하기 위해서는 `closed` 메서드를 호출하여 반환되는 Promise를 await
  - 반환된 값 중 `NatsError`: 연결이 닫힌 이유

<br/>

<b>[ 예제 코드 ]</b>

```javascript
import { connect } from 'nats.ws';

const connection = await connect({
  servers: "ws://localhost:8000",   // websocket 연결
  user: "client",
  pass: "client_",
});
console.log(`connected to ${connection.getServer()}`);

const done = nc.closed();

// 연결에 대한 작업

await nc.close();       // nats 서버 연결 해제
const err = await done; // 잘 연결이 해제되었는지 확인
if (err) {
  console.log(`error closing:`, err);
}
```

위와 같이 servers의 URL로 웹소켓 URL을 사용하려면 서버에서 설정을 해줘야 한다.

<br/>
<hr/>

### 1.3 게시(Publish) 및 구독(Subscribe)

- `publish` 메서드: 메시지 게시

  ```javascript
  nc.publish(subject: string, data?: Uint8Array, options?: PublishOptions);
  ```

- `subscribe` 메서드: 메시지 구독 (반환값: Subscription)

  ```javascript
  nc.subscribe(subject: string, opts?: SubscriptionOptions);
  ```

- `unsubscribe` 혹은 `drain` 메서드: 구독 취소

  ```javascript
  subscription.unsubscribe();
  subscription.drain();
  ```

클라이언트는 기본적으로 `publish`를 통해 메시지를 송신하고, `subscribe`를 통해 메시지를 수신한다.

- subject에 따라 메시지가 전달됨
- subject에 와일드카드를 사용하여 구독 가능
- subject에 대해 구독(subscribe) 할 시, 해당 subject로의 메시지를 수신할 수 있는 `subscription`이 생성됨
- JavaScript 클라이언트(websocket, Deno 또는 Node)에서 구독(subscribe)은 비동기 이터레이터로 작동
  - 클라이언트는 메시지가 존재할 시, 루프 돌며 메시지 처리


<b>[ 메시지 ]</b>

- 페이로드 타입: Uint8Arrays
- JSONCodec, StringCodec 또는 커스텀 코덱을 사용하여 JSON 또는 문자열로 변환 가능

<br/>

<b>[ 구독 취소 ]</b>

- `unsubscribe()`: 고객에 대한 메시지가 전송 중인지에 관계없이 구독 종료
- `drain()`: 구독을 취소하기 전에 전달 중인 모든 메시지 처리

cf. 연결 또한 `drain()`을 통해 모든 구독을 비우고 모든 메시지를 서버에 보낸 후 연결을 비울 시에 연결이 종료됨.

<br/>

<b>[ 예제 코드 ]</b>

```javascript
import { connect, StringCodec } from 'nats.ws';

const connection = await connect({
  servers: "ws://localhost:8000",
  user: "client",
  pass: "client_",
});

const subscription = connection.subscribe('msg.data');

const sc = StringCodec();

let subj = subscription.getSubject();
console.log(`listening for ${subj}`);

for await (const message of subscription) {   
  // "msg.data"로 publish 된 정보 출력
  console.log(
    `[${subj}] #${subscription.getProcessed()} - ${message.subject} ${message.data ? sc.decode(message.data) : ""}`
  );
}

// "msg.data2"로 메시지 publish
nc.publish("msg.data2", sc.encode("I publish data~"));

await nc.drain();   // 전달 중인 메시지 모두 수신 후 구독 취소
```

<br/>
<hr/>

### 1.4 요청(Request) 및 응답(Reply)

- `request` 메서드: 응답을 바라는 요청 (반환값: Promise\<Msg>)
  
  ```javascript
  nc.request(subject: string, data?: Uint8Array, opts?: RequestOptions);
  ```
  
- `respond` 메서드: 요청에 대한 응답 전달 (반환값: boolean)
  
  ```javascript
  message.respond(data?: Uint8Array, opts?: PublishOptions);
  ```

요청을 하기 위해 클라이언트는 메시지를 게시할 때 회신(`reply`) subject를 명시해야 한다. 
회신 subject는 요청에 대한 응답을 게시할 곳을 말한다.

<br/>

<b>[ 예제 코드 ]</b>

- 요청 코드

```javascript
import { connect, StringCodec } from 'nats.ws';

const connection = await connect({
  servers: "ws://localhost:8000",
  user: "client",
  pass: "client_",
});

const sc = StringCodec();

await connection.request('msg.data.time', 'I want to know the current time.'.toUint8Array(), { timeout: 5000 })
  .then((msg) => {
    console.log(`Response: ${sc.decode(m.data)}`);
  })
  .catch((err) => {
    console.log(`Problem with request: ${err.message}`);
  });

await connection.close();   // 전달 중인 메시지 모두 수신 후 구독 취소
```

- 응답 코드

```javascript
import { connect, StringCodec } from 'nats.ws';

const connection = await connect({
  servers: "ws://localhost:8000",
  user: "client",
  pass: "client_",
});

const sc = StringCodec();
const sub = connection.subscribe('msg.data.time');

for await (const message of sub) {
  // "msg.data.time"으로 들어온 요청에 대한 응답(ex. 현재 날짜) 반환
  if (message.respond(sc.encode(new Date().toISOString()))) {
    console.log(`[Time] handled #${sub.getProcessed()}`);
  } else {
    console.log(`[Time] #${sub.getProcessed()} ignored - no reply subject`);
  }
}

await sub.drain();   // 전달 중인 메시지 모두 수신 후 구독 취소
```

<br/>
<hr/>

지금까지 자바스크립트로 NATS 메시징 시스템을 이용하는 방법에 대해 설명하였습니다.<br/>
다음 포스트에서는 SpringBoot 프로젝트 내에서 NATS 서버를 설정하는 방법에 대해 설명하겠습니다.

감사합니다:)
