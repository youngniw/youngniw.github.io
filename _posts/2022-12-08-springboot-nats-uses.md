---
layout: post
toc: true
title: "[SpringBoot] NATS 통신하기 - 메시지 게시/구독/요청/회신"
categories: SpringBoot
tags: NATS, SpringBoot, Java
---

## 0. 들어가기에 앞서..

이전 포스트에서는 NATS 서버 관련 설정에 대해 살펴보았다면, 
이번 포스트에서는 Java 언어를 사용한 SpringBoot에서 NATS를 활용하는 코드에 대해 설명하고자 합니다.

> 참고 코드 (Java로 NATS 적용한 SpringBoot 프로젝트)

[https://github.com/youngniw/demo-nats](https://github.com/youngniw/demo-nats)

<br/>

![NATS 이미지](https://user-images.githubusercontent.com/78736070/206066890-fd1bde77-f950-47e9-bc1c-05b2590421dd.png)

NATS 공식 문서는 아래의 링크에서 확인할 수 있습니다.

[https://docs.nats.io/](https://docs.nats.io/)

<br/>
<hr/>

## 1. 라이브러리 설치

프로젝트에서 nats 라이브러리를 적용하기 위해서는 아래의 코드를 `build.gradle`에 추가해야 한다.

```gradle
implementation 'io.nats:jnats:2.16.5'
```

<br/>
<hr/>

## 2. NATS 서버에 연결 및 연결 종료

- 연결: Nats의 `connect` 메서드를 사용해 NATS 서버 연결
- 연결 종료: Connection 객체의 `close` 메서드를 사용해 NATS 서버와의 연결을 끊음

<br/>

### 2.1 NATS 서버 연결: connect()

`connect` 메서드: 서버와 상호 작용하는 데 사용할 수 있는 연결 반환
  
connect의 메서드 시그니처는 다음과 같다.

```java
Nats.connect();
Nats.connect(String url);
Nats.connect(String url, AuthHandler handler);
Nats.connect(Options options);
```
  - connect() : 기본 포트로 로컬 서버 연결
  - connect(String url) : url로 서버 연결
  - connect(String url, AuthHandler handler) : 인증 핸들러 연결
  - connect(Options options) : 연결 옵션을 커스터마이즈 함

> AuthHandler

인증 핸들러를 적용하여 NATS 서버에 연결하는 예제 코드는 다음과 같다.

```java
AuthHandler authHandler = Nats.credentials(System.getenv("NATS_CREDS"));
Connection nc = Nats.connect("nats://localhost:4222", authHandler);
```
  
> 연결 옵션 Options

|빌더 필드 및 메서드|기본값|설명|
|--|--|--|
|server|`"nats://localhost:4222"`|연결할 NATS 서버의 URL|
|userInfo||사용자 이름과 비밀번호를 순서대로 인수로 받음|
|maxReconnects|`60`|최대 재연결 시도 횟수|
|reconnectWait|`2초`|연결이 끊긴 경우, 클라이언트가 다시 연결을 시도할 때까지 대기할 시간|
|connectionTimeout|`2초`|클라이언트가 연결이 설정되기를 기다리는 시간|
|connectionListener||연결 상태(이벤트)에 대한 사용자 정의|

<br/>

### 2.2 NATS 서버 연결 종료: close()

- 연결 객체의 `close` 메서드를 호출함으로써 연결 종료

<br/>

### 2.3 예제 코드

```java
import io.nats.client.Connection;
import io.nats.client.ConnectionListener;
import io.nats.client.Nats;
import io.nats.client.Options;

Options options = new Options.Builder()
  .server(uri)
  .userInfo("admin", "admin_")  // 사용자 연결 계정
  .maxReconnects(10)
  .reconnectWait(Duration.ofSeconds(5))
  .connectionTimeout(Duration.ofSeconds(5))
  .connectionListener((conn, type) -> {   // 연결 이벤트 타입과 관련한 정의
    if (type == ConnectionListener.Events.CONNECTED) {            // 연결
      log.info("Connected to Nats Server");
    } else if (type == ConnectionListener.Events.RECONNECTED) {   // 재연결
      log.info("Reconnected to Nats Server");
    } else if (type == ConnectionListener.Events.DISCONNECTED) {  // 연결 끊김
      log.error("Disconnected to Nats Server, reconnect attempt in seconds");
    } else if (type == ConnectionListener.Events.CLOSED) {        // 연결 종료
      log.info("Closed connection with Nats Server");
    } else {
      log.info("Connection Type: {}", type.toString());
    }
  })
  .build();

Connection nc = Nats.connect(options);

// nc.close();
```

<br/>
<hr/>

## 3. 게시(Publish) 및 구독(Subscribe)

- 게시(Publish): `publish` 메서드로 메시지 게시
- 구독(Subscribe): `subscribe` 메서드로 메시지 구독

클라이언트는 기본적으로 `publish`를 통해 메시지를 송신하고, `subscribe`를 통해 메시지를 수신한다.

- subject에 따라 메시지가 전달됨
- subject에 와일드카드를 사용하여 구독(subscribe) 가능
- subject에 대해 구독(subscribe) 할 시, 해당 subject로의 메시지를 수신할 수 있는 `subscription`이 생성됨
- 이때 subscribe 메서드에 MessageHandler를 정의하여 인수로 전달한다면, 각 subject마다 메시지를 수신할 때 해당 커스터마이즈 한 핸들러가 수행됨

<b>[ 메시지 ]</b>

- 페이로드 타입: byte[]
- String 생성이나 ObjectMapper를 이용해 JSON과 문자열로 변환 가능

<br/>

### 3.1 메시지 게시: publish()

- `publish` 메서드: 메시지를 게시

이때 수신자가 회신할 subject는 필수가 아닌 선택적이다.

Connection 객체의 publish 메서드의 시그니처는 다음과 같다.

```java
nc.publish(String subject, byte[] body);
nc.publish(String subject, String replyTo, byte[] body);
nc.publish(Message message);
```

메서드 시그니처 중 2번째 시그니처에서의 `replyTo`는 게시를 했을 때, 그에 대한 회신 주소(subject)를 의미한다.

<br/>

### 3.2 메시지 구독: subscribe()

- `subscribe` 메서드: 명시된 subject의 메시지를 구독
- 반환 값: `Subscription`
- 메시지 구독 구현 방법: 동기식, 비동기식

1. 동기식

    : 수동으로 메시지를 요청하고 메시지가 도착할 때까지 차단

    동기식으로 구현할 때에는 Connection 객체의 subscribe 메서드를 사용한다.

    다음은 Connection 객체의 subscribe 메서드 시그니처이다.

    ```java
    nc.subscribe(String subject);
    nc.subscribe(String subject, String queueName);
    ```

    아래의 예제 코드는 메시지 구독을 동기식으로 작성한 것이다.

    ```java
    Subscription sub = nc.subscribe("subject");
    Message msg = sub.nextMessage(Duration.ofMillis(500));

    String response = new String(msg.getData(), StandardCharsets.UTF_8);
    System.out.println("Message received: " + response);

    /* 전달받은 데이터 관련 작업 처리 */
    ```

2. 비동기식

    : 백그라운드 스레드에서 어플리케이션의 코드를 호출 (Dispatcher를 이용하여 구독 정의)

    비동기식으로 구현할 때에는 동기식과 달리 Dispatcher 객체의 subscribe 메서드를 사용한다.

    다음은 Dispatcher 객체의 subscribe 메서드 시그니처이다.

    ```java
    dispatcher.subscribe(String subject);
    dispatcher.subscribe(String subject, String queue);
    dispatcher.subscribe(String subject, MessageHandler handler);
    dispatcher.subscribe(String subject, String queue, MessageHandler handler);
    ```

    아래의 예제 코드는 메시지 구독을 비동기식으로 작성한 것이다.

    ```java
    Dispatcher dispatcher = nc.createDispatcher((msg) -> { /* 메시지 관련 작업 처리 구현 or 비워둠 */ });

    Subscription subscription = dispatcher.subscribe("msg.vehicle.telemetry", (msg) -> {
      String response = new String(msg.getData(), StandardCharsets.UTF_8);
      System.out.println("Message received: " + response);

      /* 전달받은 데이터 관련 작업 처리 */
    });
    ```

<br/>

<b>[ 구독 취소 ]</b>

- `unsubscribe()`: 고객에 대한 메시지가 전송 중인지에 관계없이 구독 종료
- `drain(Duration timeout)`: 반환 값은 CompletableFuture\<Boolean>로, 구독을 취소하기 전에 전달 중인 모든 메시지 처리 (새 메시지 수신을 중지)

cf. `Dispatcher`와 `Subscription`, 그리고 `Connection` 모두 `drain()`을 통해 모든 구독을 비우고 모든 메시지를 서버에 보낸 후 연결을 비울 시 연결 종료

<br/>

### 3.3 예제 코드

1.<mark>Publish</mark>

NatsComponenet.java 의 일부 코드

```java
@RequiredArgsConstructor
@Component
public class NatsComponent {
  private final Connection natsConnection;

  // subject로 메시지(message)를 보냄
  public void publish(String subject, String message) {
    natsConnection.publish(subject, message.getBytes(StandardCharsets.UTF_8));
  }

  // subject로 메시지(message)를 보냄 (+ 응답 subject 포함)
  public void publish(String subject, String replyTo, String message) {
    natsConnection.publish(subject, replyTo, message.getBytes(StandardCharsets.UTF_8));
  }
}
```

2.<mark>Subscribe</mark> (비동기식 처리)

NatsAsyncMessageHandler.java 의 일부 코드

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class NatsComponent {
  private final Connection natsConnection;
  private Dispatcher dispatcher;

  @PostConstruct
  private void init() {
    dispatcher = natsConnection.createDispatcher((msg) -> {});

    // 구독 예시
    Subscription subToVehicle = dispatcher.subscribe("msg.vehicle.telemetry", (message) -> {
      String data = new String(message.getData(), StandardCharsets.UTF_8);
      log.info(String.format("Received Message from %s: %s", message.getSubject(), data));

      /* 데이터 가공해서 처리 */
    });
  }

  @PreDestroy
  private void destroy() {
    // 구독 취소
    dispatcher.unsubscribe("msg.vehicle.telemetry");
  }
}
```

<br/>
<hr/>

## 4. 요청(Request) 및 응답(Reply)

- 요청(Request): `request` 메서드로 응답을 바라는 요청 수행
- 응답 및 회신(Reply): 따로 추가 메서드 없이 `publish`를 통해 회신 수행

<br/>

### 4.1 메시지 요청: request()

`request` 메서드: 응답을 바라는 요청 (반환값: CompletableFuture\<Message>)

Connection 객체의 request 메서드의 시그니처는 다음과 같다.
  
```java
nc.request(String subject, byte[] body);
nc.request(String subject, byte[] body, Duration timeout);
nc.request(Message message);
nc.request(Message message, Duration timeout);
```

<br/>

### 4.2 메시지 회신(reply)

subscribe를 통해 요청 정보를 확인 후, 회신 주소(Message 객체의 `getReplyTo()`)가 있으면 그 값을 subject로 하여 publish 한다.

회신 주소의 값은 기본적으로 `_INBOX.`로 시작한다.

```java
nc.publish(message.getReplyTo(), byte[] body);
```

<br/>

### 4.3 예제 코드

1.<mark>Request</mark>

NatsComponenet.java 의 일부 코드

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class NatsComponent {
  private final Connection natsConnection;

  // 요청
  public CompletableFuture<Message> request(String subject, String message) throws ExecutionException, InterruptedException {
    CompletableFuture<Message> incoming = natsConnection.request(subject, message.getBytes(StandardCharsets.UTF_8));
    CompletableFuture<Message> nextFuture = incoming.thenApply(msg -> {
      log.info("Reply message: {}", new String(msg.getData(), StandardCharsets.UTF_8));

      /* 회신받은 데이터를 이용한 작업 처리 */

      return msg;
    });
    return nextFuture;
  }
}
```

2.<mark>Reply</mark> (비동기식 처리)

NatsAsyncMessageHandler.java 의 일부 코드

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class NatsComponent {
  private final Connection natsConnection;
  private final VehicleService vehicleService;
  private Dispatcher dispatcher;

  @PostConstruct
  private void init() {
    dispatcher = natsConnection.createDispatcher((msg) -> {});

    // 차량 관련 요청 통로
    Subscription subToVehicleRequest = dispatcher.subscribe("msg.vehicle.request.*", (message) -> {
      // 차량 요청이 있을 시 (ex. msg.vehicle.request.1234로 "info"라는 데이터를 담아 요청 받을 시, 차량 1234의 정보 반환)
      if (message.getReplyTo() != null) {     
        int serialNumber = Integer.parseInt(message.getSubject().split("\\.")[3]);
        String data = new String(message.getData(), StandardCharsets.UTF_8);

        if (data.equals("info")) {          // 차량 정보 조회
          ObjectMapper mapper = new ObjectMapper();
          try {
            natsConnection.publish(message.getReplyTo(), mapper.writeValueAsBytes(vehicleService.getVehicleInfo(serialNumber)));
          } catch (JsonProcessingException e) {
            natsConnection.publish(message.getReplyTo(), "{}".getBytes());
            throw new RuntimeException(e);
          }
        }
      }
    });
  }

  @PreDestroy
  private void destroy() {
    // 구독 취소
    dispatcher.unsubscribe("msg.vehicle.request.*");
  }
}
```

<br/>
<hr/>

[ 참고 사이트 ]
- [https://www.baeldung.com/nats-java-client](https://www.baeldung.com/nats-java-client)
- [https://github.com/nats-io/nats.java](https://github.com/nats-io/nats.java)

<br/>

이번 포스트에서는 NATS 메시징 시스템을 적용한 자바 코드에 대해 설명하였습니다.

감사합니다:)
