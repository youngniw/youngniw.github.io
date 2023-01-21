---
layout: post
title: "[SpringBoot] NATS 통신하기 - NATS 서버 설정"
categories: SpringBoot
tags: [NATS, Spring, SpringBoot, Docker, 스프링, 스프링부트]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 SpringBoot 프로젝트에서 작성해야 하는 NATS 서버 관련 설정과 더불어 Docker에서 NATS 서버를 실행하는 방법에 대해 설명하겠습니다.

> 참고 코드 (Java로 NATS 적용한 SpringBoot 프로젝트)

[https://github.com/youngniw/demo-nats](https://github.com/youngniw/demo-nats)

<br/>

![NATS 이미지](https://user-images.githubusercontent.com/78736070/206066890-fd1bde77-f950-47e9-bc1c-05b2590421dd.png)

NATS 설정 관련 문서는 아래의 링크에서 확인할 수 있습니다.

[https://docs.nats.io/running-a-nats-service/configuration](https://docs.nats.io/running-a-nats-service/configuration)

<br/>
<hr/>

## 1. NATS 서버 관련 설정(Configuration)

NATS 서버 관련 설정 파일을 작성하여 NATS 서버를 커스터마이즈 할 수 있다.
- 파일 확장자: `.conf`

다음은 자주 사용하는 NATS 설정 속성에 대해 소개하고자 한다.

<br/>

### 1.1 연결성

|속성|기본값|설명|
|--|--|--|
|host|`0.0.0.0`|클라이언트 연결을 위한 호스트|
|port|`4222`|클라이언트 연결용 포트|
|websocket||서버에서 웹소켓 지원을 활성화하기 위한 구성|

cf. websocket<br/>
도커에서 실행할 때는 다음의 최소한의 항목으로 구성 파일을 만들어야 한다.

```conf
websocket {
  port: 8000    # 웹소켓 포트 번호를 8000으로 설정함
  no_tls: true
}
```

<br/>

### 1.2 클러스터링

|속성|설명|
|--|--|
|cluster|- 클러스터 모드에서 각 서버를 실행할 수 있도록 지원<br>- 대용량 메시징 시스템과 복원력 및 고가용성을 위해 서버를 함께 클러스터링|

<br/>

### 1.3 제한

|속성|기본값|설명|
|--|--|--|
|max_connections|`64K`|최대 활성 클라이언트 연결 수|
|max_payload|`1MB`|메시지 페이로드의 최대 바이트 수 (최대 64MB)|
|max_subscriptions|`0`, unlimited|클라이언트 및 리프 노드 계정 연결 당 최대 구독 수|

<br/>

### 1.4 인증 (Authentication)

다음은 `authentication` 속성 내의 작성 가능한 속성을 보여준다.

|속성|설명|
|--|--|
|token|서버 인증에 사용할 수 있는 전역 토큰<br/>(사용 시 `user`와 `password` 제외)|
|user|서버에 대한 클라이언트들의 하나의 전역 사용자 이름<br/>(사용 시 `token` 제외)|
|password|서버에 대한 클라이언트들의 하나의 전역 비밀번호<br/>(사용 시 `token` 제외)|
|users|사용자 구성 목록<br/>여러 사용자 이름 및 암호 자격 증명 지정|
|timeout|클라이언트 인증을 기다리는 최대 시간 (단위: s)|

<br/>

아래의 표는 `authentication` 내의 `users` 속성 내의 가능한 속성 값이다.

|속성|설명|
|--|--|
|user|클라이언트 인증을 위한 사용자 이름|
|password|사용자 비밀번호|
|nkey|사용자를 식별하는 공개n키|
|permissions|사용자가 특정 subject에 접근할 수 있는 권한 목록|

<br/>

아래의 표는 `users` 목록 중 하나의 요소 내의 `permissions` 가능한 속성 값이다.

|속성|설명|
|--|--|
|publish|클라이언트가 게시 가능한 subject 혹은 목록, 혹은 권한|
|subscribe|클라이언트가 구독 가능한 subject 혹은 목록, 혹은 권한|

<br/>

예시 코드는 다음과 같다.

```conf
authorization {
  WEBSOCKET_CLIENT = {
    # publish = ["req.a", "req.b"]
    subscribe = ["msg.vehicle.data", "msg.vehicle.data.*", "_INBOX.>"]    # INBOX 는 request 요청을 위함
  }
  WEBSOCKET_VEHICLE = {
    publish = "msg.vehicle.telemetry"
    subscribe = "msg.vehicle.notice"
  }
  users = [
    { user: "admin", password: "admin_" },
    { user: "client", password: "client_", allowed_connection_types: ["WEBSOCKET"], permissions: $WEBSOCKET_CLIENT },
    { user: "vehicle", password: "password", permissions: $WEBSOCKET_VEHICLE },
  ]
}
```
- 사용자 'admin': 권한을 설정하지 않아 어떤 subject에 대해 구독 및 게시 가능
- 사용자 'client': 웹소켓으로만 연결 가능하며, 권한으로는 3가지 지정된 subject 구독만 가능
- 사용자 'vehicle': 권한으로는 `msg.vehicle.notice`로의 게시, `msg.vehicle.notice`로의 구독만 가능

<br/>

### 1.5 모니터링

|속성|설명|
|--|--|
|http_port|서버 모니터링을 위한 HTTP 포트|
|http|서버 모니터링을 하기 위한 `host:port`|
|https_port|서버 모니터링을 위한 HTTPS 포트|
|https|TLS 서버 모니터링을 하기 위한 `host:port`|
|http_base_path|엔드 포인트를 모니터링하기 위한 기본 경로 (기본값: `/`)|

<br/>
<hr/>

## 2. Docker로 NATS 서버 실행

NATS 서버는 도커 데몬을 사용하여 실행할 수 있는 이미지 형태로 제공된다.

<b>[ 사용 방법 ]</b>

먼저 도커 컨테이너 이미지를 사용하려면 도커를 설치 후 공개된 이미지를 가져온다.

```bash
docker pull nats
```

NATS 서버 이미지를 실행한다.

```bash
docker run nats
```

NATS 서버는 기본적으로 4222, 6222, 8222 포트를 노출하고, 
이외의 다른 포트는 `-p` 혹은 `-P` 옵션을 통해 사용자가 정의할 수 있다.
- 4222: 클라이언트 사용 포트
- 6222: 클러스터링을 위한 라우팅 포트
- 8222: 정보 모니터링을 위한 HTTP 관리 포트

위에서 링크로 보여준 참고 예제 프로젝트는 NATS 서버 실행을 위해 다음의 코드를 실행해야 한다.<br/>
(프로젝트 폴더 위치에서 명령어 실행!)

```bash
docker run --name nats \          
  -v $(pwd)/etc/nats:/etc/nats \
  -p 4222:4222 \
  -p 8222:8222 \
  -p 8000:8000 \
  nats --http_port 8222 -c /nats/etc/nats/vehicle.conf
```

참고로 8000 포트는 websocket 통신을 위해 열어둔 것이며, 
nats 서버 관련 설정 파일을 명시하기 위해 -c 옵션을 통해 conf 파일의 위치를 명시하였다.

<br/>
<hr/>

다음 포스트에서는 SpringBoot 프로젝트 내에서 자바 코드로 NATS 메시징 시스템을 적용하는 방법에 대해 설명하였습니다.

감사합니다:)
