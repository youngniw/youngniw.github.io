---
layout: post
title: "[HTTP] 웹 브라우저 요청 흐름"
categories: HTTP
tags: [Web, HTTP]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 URL을 브라우저에서 입력 시에 일어나게 되는 과정인 웹 브라우저 요청 흐름에 대해 설명하겠습니다.

<br/>
<hr/>

## URL 입력 시, 웹 브라우저 요청 흐름은?

웹 브라우저에 URL을 입력 시에는 많은 과정을 거쳐 웹 사이트의 HTML이 렌더링 되어 보이게 된다.

만약 아래의 URL을 웹 브라우저에 입력하다면 어떻게 될까?

> "https://www.google.com:443/search?q=hello&hl=ko"

### 1. `www.google.com`인 서버를 찾기 위해 DNS 서버에 조회해 IP 주소를 획득

브라우저는 먼저 캐시를 확인하여 요청 URL 관련 DNS 기록을 찾는다.

만약 요청 URL이 캐시에 존재하지 않는다면, DNS 서버가 DNS 쿼리로 `www.google.com` 사이트를 호스팅하는 웹 서버의 IP 주소를 찾는다.

예를 들어 `www.google.com`의 IP 주소는 `142.250.196.110`인데, URL을 입력하면 DNS 서버로부터 해당 IP 주소를 획득하게 되는 것이다.

<br/>

### 2. 브라우저가 해당 서버와 TCP 연결 시작

브라우저가 올바른 IP 주소를 수신하면 IP 주소와 일치하는 서버와 연결해 정보를 전송한다.

브라우저는 인터넷 프로토콜(IP)을 사용하여 연결을 구축한다. 일반적으로 HTTP 요청에서는 TCP 프로토콜을 사용한다.

<br/>

### 3. 웹 브라우저에서 웹 서버로 HTTP 요청 메시지 전달

#### 3.1 웹 브라우저에서 HTTP 요청 메시지 생성

HTTP 요청 메시지의 예시는 다음과 같다.

```
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

[ HTTP 요청 메시지 ]

- HTTP 메서드. path, HTTP 버전 정보
- 호스트 정보
- 기타 정보

#### 3.2 Socket 라이브러리를 통해 전달

- TCP/IP 연결(IP, Port 정보)
- 데이터 전달
  
#### 3.3 HTTP 메시지를 포함하는 TCP/IP 패킷을 생성하여 인터넷망에 전달함으로써 목적지 서버에 전달

<br/>

### 4. 요청 서버에서 HTTP 메시지를 추출하여 요청에 대한 응답을 반환 (HTTP 응답 메시지)
   
HTTP 응답 메시지 예시는 다음과 같다.

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423

<html>
   <body>...</body>
</html>
```

[ HTTP 응답 메시지 ]

- HTTP 버전 정보, HTTP 상태
- 응답 데이터 상세 정보
- 응답 데이터
- 기타 정보

<br/>

### 5. 응답 데이터인 HTML을 웹 브라우저가 렌더링 하여 화면이 보임

<br/>
<hr/>

[ 참고 사이트 ]

- 인프런 강의 중 김영한 님의 "모든 개발자를 위한 HTTP 웹 기본 지식"

<br/>

지금까지 웹 브라우저 요청 흐름에 대해 간략하게 설명하였습니다.

감사합니다:)
