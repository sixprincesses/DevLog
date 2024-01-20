## SockJs

### 개요

WebSocket과 유사한 오브젝트를 제공하는 브라우저 JS 라이브러리
SockJS는 최신 브라우저나 제한적인 기업 프록시 뒤와 같은 WebSocket 프로토콜을 지원하지 않는 환경에서 작동하도록 고안되었다.

### 장점

일관성
크로스 브라우저
JS API(짧은 지연시간)
완전 복제
브라우저-웹서버 사이의 크로스 도메인 소통 채널

### 작동 방식

내부적으로 기본 WebSocket을 먼저 사용하려고 시도한 후, 실패할 경우 다양한 브라우저별 전송 프로토콜을 사용한다.
이는 WebSocket과 유사한 추상화를 통해 이루어진다.

### 연관 라이브러리

SockJS-node : NodeJS 서버 라이브러리
SockJS-client : JS 클라이언트 라이브러리

이외 언어별 라이브러리 : SockJS-erlang, cyclone, tornado, twisted, aiohttp, etc...

## STOMP

STOMP : Simple Text Oriented Messaging Protocol

### 개요

WebSocket 프로토콜은 Text 또는 Binary 두 가지 유형의 메시지 타입은 정의하지만 메시지의 내용에 대해서는 정의하지 않는다. 
WebSocket에서는 메세지의 종류, 형식, 내용 등을 정의하기 위해 WebSocket 위에서 사용할 하위 프로토콜을 구현해야 한다.
STOMP는 이러한 하위 프로토콜 중 일반적으로 사용되는 메시징 패턴의 하위 집합을 처리하도록 설계되어 있다.
STOMP는 TCP, WebSocket과 같은 안정적인 양방향 스트리밍 네트워크 프로토콜을 통해 사용할 수 있다.

### STOMP frame 구조

```
COMMAND
header1: value1
header2: value2

Body^@
```

클라이언트는 `header`의 `destination`에 `subscribe` 할 채널을 설정해서 채널을 구독한 사람들과 메세지를 주고 받을 수 있다.

Spring에서는 Spring WebSocket 애플리케이션을 통해 클라이언트에 대한 STOMP 브로커 역할을 수행할 수 있다.
메세지는 `@Controller`의 메세지 처리 방법을 통해 구독한 사용자에게 메세지를 브로드캐스트 한다.

- Subscribe frame
  ```
  SUBSCRIBE
  id:sub-1
  destination:/topic/price.stock.*
  
  ^@
  ```

클라이언트가 메세지를 `send`했을 경우서버는 `@MessageMapping`을 사용하여 처리할 수 있다.

- Send frame
  ```
  SEND
  destination:/queue/trade
  content-type:application/json
  content-length:44
  
  {"action":"BUY","ticker":"MMM","shares",44}^@
  ```

스톰프 서버는 `MESSAGE` 명렁어를 이용해서 메세지를 모든 구독자에게 브로드캐스트 할 수 있다.
이 때 목적지 경로를 `/topic/..`과 같이 하면 publish-subscribe(one-to-many)를 나타내고,
`/queue/`와 같이 하면 point-to-point(one-to-one)을 나타낸다.

- Message frame
  ```
  MESSAGE
  message-id:nxahklf6-1
  subscription:sub-1
  destination:/topic/price.stock.MMM
  
  {"ticker":"MMM","price":129.45}^@
  ```
