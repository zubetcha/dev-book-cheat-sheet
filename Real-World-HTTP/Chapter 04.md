# Chapter 04 - HTTP/1.1의 신택스: 고속화와 안정성을 추구한 확장

#### HTTP/1.1 주요 변경사항

- 통신 고속화
  - Keep-Alive가 기본적으로 유효
  - 파이프라이닝
- TLS에 의한 암호화 통신 지원
- 새 메서드 추가
  - PUT과 DELETE가 필수 메서드가 됨
  - OPTION, TRACE, CONNECT 메서드 추가
- 프로토콜 업데이트
- 이름을 이용한 가상 호스트를 지원
- 크기를 사전에 알 수 없는 콘텐츠의 청크 전송 인코딩 지원

### 4.1 통신 고속화

- 캐시 또한 HTTP/1.1의 기능
- 캐시는 콘텐츠 리소스의 통신 최적화 기술
- `Keep-Alive`와 `파이프라이닝`은 범용적으로 모든 HTTP 통신을 고속화 하는 기능

#### Keep-Alive

![keep-alive](https://www.haproxy.com/assets/posts/perisitent-connection.png)

| Keep-Alive 유무 | 설명                                        |
| :-------------: | ------------------------------------------- |
|        X        | 하나의 요청마다 통신을 닫음 (복수의 커넥션) |
|        O        | 연석된 요청에는 접속 재이용 (커넥션 지속)   |

- HTTP의 아래층인 TCP/IP 통신을 효율화하는 구조
- 접속 재이용의 장점
  - TCP/IP의 접속까지의 대기 시간 단축
  - 모바일 통신에서는 배터리 낭비 감소
- HTTP/1.1에서는 Keep-Alive 동작이 기본으로 되어 있음
- 연결 유지 기한
  - 타임아웃 or 서버/클라이언트 중 한 쪽에 다음 헤더 부여 시까지
    ```jsx
    Connection: Close;
    ```
- 통신 지속 시간 동안 OS 리소스 소비 → 실제 통신은 없는데 접속을 유지하는 것은 낭비

#### 파이프라이닝

![파이프라이닝](https://blog.kakaocdn.net/dn/bpdV4n/btrub9VkLO4/2UHDW7xWK6x9RIKKorkkQ0/img.png)

- 최초 요청이 완료되기 전에 다음 요청을 보내는 고속화 기술
- 다음 요청까지의 대기 시간 X → 네트워크 가동률 증가, 성능 향상
- 서버는 요청이 들어온 순서대로 응답 반환
- 브라우저, 서버, 프록시 등의 설정에 따라 제대로 동작하지 않을 수 있음
- HTTP/2에서 스트림이라는 구조로 재탄생

> HOL (Head of Line Blocking)

<br/>

### 4.2 전송 계층 보안 (TLS)

- 암호화되어 통신을 엿보거나 변경할 수 없는 양방향 통신

#### 해시 함수

**해시 함수의 특징**

- 같은 알고리즘과 같은 데이터를 입력하면, 생성되는 결과 값은 같다. `h(A) = X`
- 알고리즘이 같으면 해시 값의 길이는 고정된다.
- 해시 값에서 원본 데이터를 유추하기 어렵다. `h(A) = X`의 X에서 A를 찾기 어렵다. (약한 충돌 내성)
- 같은 해시 값을 생성하는 다른 두 개의 데이터를 찾기 어렵다. h(A) = h(B)가 되는 임의의 데이터 A, B를 찾기 어렵다. (강한 충돌 내성)

**해시 값의 용도**

- 다운로드한 파일이 깨지지 않았는지 확인 (체크섬, 핑거프린트)
  - 1바이트라도 데이터가 다르면 해시 값이 바뀌기 때문
- 빠른 비교
  - 데이터 파일 내용을 일일히 비교하지 않고 해시 값으로만 비교 (ex. Git)
- 내용 동일성 판단

**해시 값 확인 CLI**

```
// Mac OS
md5 sample.rst

// Window
fciv.exe sample.rst
```

#### 공통 키 암호와 공개 키 암호 그리고 디지털 서명

- 암호화에서 중요한 것은 변환 알고리즘이 알려져도 안전하게 통신할 수 있게 하는 것
- 일반적으로 사용하는 방식: 알고리즘 공개 + 암호화에 필요한 키 별도로 준비
- TLS에서의 사용 방식 종류
  | 종류 | 특징 |
  | --- | --- |
  |`공통 키` 방식| - `대칭 암호` <br/> - 데이터를 전송하는 쪽과 받는 쪽 모두 **같은 키** 사용 <br/> - 통신하는 측 끼리 키 공유 필요|
  |`공개 키` 방식| - `비대칭 암호` <br /> - **공개 키**와 **비밀 키** 필요 <br /> - 공개 키 → 암호화 하는 키 <br/> - 비밀 키 → 암호를 해독하는 키 <br/> - 비밀 키는 다른 사람에게 알려져선 안 됨|
  - 공통 키, 공개 키 모두 데이터 , 비밀 키, 공개 키가 전부 데이터로 표현됨

**디지털 서명**

- 공개 키 방식을 응용한 예
- 본문 자체를 암호화하는 것이 아닌, 먼저 해시화하고 그 결과를 암호화함

#### 키 교환

- 클라이언트 ↔ 서버 사이에 키를 교환하는 것
- 여러 방법이 있음
  - 클라이언트에서 공통 키 생성 → 서버 인증서의 공개 키로 암호화 해 전송하는 방식
  - 키 교환 전용 알고리즘 사용 방식

**DHE 알고리즘 (일시 디피-헬먼, Diffie-Hellman ephemeral)**

- 키 자체를 교환하는 것 X
- 클라이언트와 서버에서 각자 키 재료를 만들어 교환하고 각자 계산해서 같은 키를 얻는 것
- 생성되는 키의 길이와 안전성 상관 관계
- 현재는 `2048 비트` 이상 길이 권장 (616자리 정도의 소수)
- 프록시는 계산이 불가하기 때문에 오직 클라이언트와 서버만 공통 키를 가질 수 있음

#### 공통 키 방식과 공개 키 방식을 구분해서 사용하는 이유

- 공개 키 방식은 복잡한 만큼 안전성은 높지만 계산에 걸리는 시간이 오래 소요됨
- TLS는 두 방식을 조합해서 사용
  - 통신마다 한 번만 사용되는 공통 키 생성
  - 공개 키 방식으로 상대방에게 키 전달
  - 이후 공통 키로 빠르게 암호화
- RSA와 AES
- 두 방식의 속도 차이

#### TLS 통신 절차

- handshake 프로토콜로 통신을 확립하는 단계
- record 프로토콜로 불리는 통신 단계
- Session Ticket 구조를 이용한 재접속 시의 고속 handshake

1. 서버의 신뢰성 확인

- 서버의 신뢰성을 보증하는 구조는 공개 키를 보증하는 구조이기도 하기 때문에 `공개 키 기반 구조(public key infrastructure)`라고도 불림
- 브라우저는 서버로부터 서버의 SSL 인증서를 가지고 오는 것부터 시작
- 신뢰성 확인의 핵심은 `발행자(인증기관)`

> 💡 루트 인증기관  
> 발행자과 주체자가 동일한 인증서

2. 키 교환과 통신 시작

- 공개 키 암호 사용 방법 or 키 교환 전용 알고리즘 사용 방법이 있음
  | 순서 | 주체 | 설명 |
  | --- | --- | --- |
  |1|Server > Client| SSL 서버 인증서 취득 |
  |2|Client| - 난수를 사용해 통신용 공통 키 생성 <br/> - 서버 인증서에 첨부되어 있는 공개 키로 통신용 공통 키 암호화 |
  |3|Cleind > Server| 암호화한 공통 키를 서버로 전달|
  |4|Server| 공개 키에 대응하는 비밀 키로 데이터 복호화|

3. 통신

- 통신 할 때에도 기밀성과 무결성(조작 방지)를 위해 암호화
- 암호화에는 공통 키 암호 방식 알고리즘 이용
- TLS 1.2 이전 버전
  - 통신 내용의 해시 값 계산 -> 공통 키 암호로 암호화
- TLS 1.3 이후 버전
  - 인증 암호(AEAD)로 제한될 예정

4. 통신의 고속화

- 신규 접속 시 시간이 가장 오래 걸림
  - HTTP로 연결하기 전 TCP/IP 단계: 1.5RTT
  - TLS handshake: 2RTT
  - HTTP 요청: 1RTT
  - 합계: 4RTT (TCP/IP 마지막 0.5RTT + TLS 최초 통신은 함께 이루어짐)

> 💡 `RTT`란?  
> Round Trip Time의 약자로, 요청이 네트워크를 통해 전달되어 응답이 되돌아오는 데까지 걸리는 총 시간을 뜻함  
> RTT는 일반적으로 밀리초 단위로 측정하며, RTT가 낮으면 낮을수록 사용자 경험이 향상됨

**RTT 단축을 위한 TLS, HTTP의 장치**

- Keep-Alive
  - 세션이 계속 열려있으므로, 최초 요청 이후 RTT는 1이 됨
- TLS 1.2
  - 세션 재개 기능
  - 전에 사용하던 세션 ID를 보내면 이후 키 교환 생략 -> 1RTT
- TLS 1.3
  - 사전에 키 고유
  - 최초 요청부터 0RTT

> 💡 QUIC (Quick UDP Internet Connections)  
> TLS 아래 계층을 TCP에서 UDP로 대체하는 방식  
> TCP에서는 handshake가 필요하지만 UDP에서는 재전송과 흐름 제어가 불필요

#### 암호강도

- 공통 키 암호 방식의 비트 수에 따라 레벨이 나뉘며, 비트가 1 증가하면 두 배 강해짐
- 비트 수에 따른 암호 강도
  - 64 비트: 짧은 기간의 공격으로도 해독 가능
  - 80 비트: 단기간의 조직적인 공격으로 해동 가능
  - 96 비트: 10년
  - 112 비트: 20년
  - 128 비트: 30년
  - 256 비트: 양자 컴퓨터 공격에 대한 내성이 있다고 간주

#### 암호화 스위트

> 💡 암호화 스위트  
> 키 교환 방법, 메시지 암호화, 메시지 서명 방식 등 각각의 장면에서 사용하는 알고리즘 조합

**암호화 스위트 조합 확인 CLI**

```
openssl ciphers -v
```

**암호화 스위트 조합**
|값|의미|사용되는 값의 예|
| --- | --- | --- |
|ECDHE-RSA-AES256-GCM-SHA384|암호화 스위트를 식별하는 이름||
|TLSv1.2|암호가 지원된 프로토콜 버전|TLSv1.2 등|
|Kx=ECDH/RSA|교환 키 알고리즘/서명 알고리즘|DH/RSA, ECDH/ECDSA|
|Au=RSA|인증 알고리즘|RSA/ECDSA|
|Enc=AESGCM(256)|레코드 암호 알고리즘|AES-GCM, CHACHA20-POLY1305|
|Mac|메시지 서명|AEAD, SHA386|

- HTTP/2의 RFC에서는 사용하면 위험한 암호화 스위트의 블랙리스트가 정의되어 있음
- 암호화 스위트 중 사용해도 안전한 것, 호환성을 위해 남겨둘 것 등 구체적인 목록은 모질라 사이트에서 확인 가능

#### 프로토콜 선택

**ALPN (Application-Layer Protocol Negotiation)**

- TLS의 최초 handshake 시 (`ClientHello`) 클라이언트에서 서버로 `클라이언트가 이용할 수 있는 프로토콜 목록`을 첨부하여 전송
- 서버는 그에 대한 응답(`ServerHello`)으로 키 교환을 하고 인증서와 함께 `선택한 프로토콜`을 클라이언트로 전송
- 선택할 수 있는 프로토콜 목록은 `IANA`에서 관리하며, 주로 `HTTP` 계열와 `WebRTC` 계열 프로토콜이 있음

**선택 가능한 프로토콜**
| 프로토콜 | 식별자 |
| --- | --- |
| HTTP/1.1 | http/1.1|
| SPDY/1 | spdy/1 |
| SPDY/2 | spdy/2 |
| SPDY/3 | spdy/3 |
| Traversal Using Relays around NAT(TURN) | stun.turn|
| NAT discovery using Session Traversal Utilities for NAT(STUN) | stun.nat-discovery |
| HTTP2 over TLS | h2|
| HTTP2 over TCP | h2c |
| WebRTC 미디어와 데이터 | webrtc|
| Confidential WebRTC 미디어와 데이터 | c-webrtc |
| FTP | ftp |

#### TLS가 지키는 것

- TLS는 통신 경로의 안전을 지키기 위한 구조
  - 클라이언트와 서버 간 통신 경로를 전혀 신뢰할 수 없는 상태에서도 안전하게 통신 가능하도록 설계됨
  - 도청, 조작, 사칭할 수 없는 안전한 통신 제공

**TLS 1.3**

- 인증 암호 알고리즘 이용
  - 조작, 사칭이 불가능하도록 보호
- 공통 키의 안전한 교환이 중요
  - 키를 찾아내기 어렵도록 `DHE`, `ECDHE` 등의 키 교환 알고리즘 이용
  - 인증서와 함께 사용해 조작 위험성 감소

**TLS 1.2**

- 공개 키 암호로 키를 교환하는 방법 제공
- 순방향 비밀성은 약하나, 비밀 키만 지키면 통신 보호 가능
- 인증서의 안전성은 발행자가 보증

**TLS가 보호하지 않는 것들**

- 통신 경로 밖에 있는 데이터
- 서버가 크랙?됐을 때의 정보

### 4.3 PUT 메서드와 DELETE 메서드의 표준화

- HTTP/1.0에서는 옵션이었던 `PUT`과 `DELETE` 메서드가 HTTP/1.1에서는 필수 메서드로 추가됨
- 이로써 DB에서 데이터를 다룰 때 사용하는 CRUD가 갖추어짐
  - `HTTP`는 도큐먼트를 다루는 고수준 API
  - 반면, `CRUD`는 프리미티브한 조작이라는 데 차이가 있음
- `PUT`과 `DELETE`는 `GET`과 `POST`와 달리 HTML form에서는 사용 불가
  - `XMLHttpRequest` 이용해야 함

**데이터를 다루는 기본 메서드**

| HTTP 메서드 | 대응하는 CRUD 조작 |  SQL   |
| :---------: | :----------------: | :----: |
|     GET     |        Read        | select |
|    POST     |       Create       | insert |
|     PUT     |       Update       | update |
|   DELETE    |       Delete       | delete |

> 💡 Axios와 XMLHttpRequest  
> 찾아보고 추가하기

### 4.4 OPTIONS, TRACE, CONNECT 메서드 추가

#### OPTIONS

- 서버가 받아들일 수 있는 메서드 목록 반환
- 응답 중 Allow 헤더에 메서드 목록 존재
- 실제로 대부분의 웹 서버는 OPTIONS 메서드를 허용하고 있지 않으나 브라우저가 다른 서버에 요청을 보낼 때 사전 확인에 사용되는 경우는 있음

```
curl -X OPTIONS -v https://curl.se

.
.
< HTTP/2 200
< server: nginx/1.21.1
< content-type: text/html
< x-frame-options: SAMEORIGIN
< allow: OPTIONS,HEAD,GET,POST
< cache-control: max-age=60
< expires: Sun, 11 Jun 2023 21:53:55 GMT
< x-content-type-options: nosniff
< content-security-policy: default-src 'self' curl.haxx.se www.curl.se curl.se www.fastly-insights.com fastly-insights.com; style-src 'unsafe-inline' 'self' curl.haxx.se www.curl.se curl.se
< strict-transport-security: max-age=31536000
< accept-ranges: bytes
< via: 1.1 varnish, 1.1 varnish
< date: Sun, 11 Jun 2023 21:52:55 GMT
< x-served-by: cache-bma1639-BMA, cache-icn1450058-ICN
< x-cache: MISS, MISS
< x-cache-hits: 0, 0
< x-timer: S1686520374.287675,VS0,VE1087
< vary: Accept-Encoding
< alt-svc: h3=":443";ma=86400,h3-29=":443";ma=86400,h3-27=":443";ma=86400
< content-length: 0
```

#### TRACE(TRACK)

- 서버는 TRACE 메서드를 받으면 `Content-Type`에 `message/http`를 설정하고 `Status Code 200 OK`를 붙여 요청 헤더와 바디를 그대로 반환함
- 크로스 사이트 스크립팅(XSS)과 크로스 사이트 트레이싱(XST)이라는 취약성으로 유명해져 현재는 거의 사용되지 않고 있음
- 현재 브라우저에서도 XMLHttpRequest로 TRACE 메서드를 보내는 것을 허용하고 있지 않음

> 💡 크로스 사이트 스크립팅(XSS, Cross-Site Scripting)  
> 임의의 악의적인 스크립트를 브라우저에 실행

> 💡 크로스 사이트 트레이싱(XST, Cross-Site Tracing)  
> BASIC 인증 사용자와 패스워드 장악

#### CONNECT

- HTTP 프로토콜상에 다른 프로토콜의 패킷을 흘릴 수 있게 해주는 메서드
- 프록시 서버를 거쳐 대상 서버에 접속하는 것을 목적으로 하며, 주로 https 통신을 중계하는 용도로 사용됨
- CONNECT 메서드를 무조건 받는 프록시는 아무 프로토콜이나 통과시키기 때문에 위험한 용도로 사용될 가능성도 있음

### 4.5 프로토콜 업그레이드

- HTTP/1.1부터 HTTP 이외의 프로토콜로 업그레이드 가능
- 업그레이드 종류로는 세 가지가 있음
  - HTTP에서 `TLS`를 사용한 안전한 통신으로 업그레이드 (TLS/1.0, TLS/1.1, TLS/1.2)
  - HTTP에서 `웹소켓`을 사용한 양방향 통신으로 업그레이드 (websocket)
  - HTTP에서 `HTTP/2`로 업그레이드 (h2c)
- HTTP에서 TLS로 업그레이드 하는 건, 업그레이드해도 보안이 지켜지지 않는다는 문제가 있음
- HTTP/2에서는 프로토콜 업그레이드 기능이 삭제되었으며, 현재 프로토콜 업그레이드는 거의 웹소켓용
  - HTTP/2 통신도 TLS를 전제로 하고 있기 때문

#### 클라이언트 쪽에서 업그레이드를 요청

- Upgrade와 Connection 헤더를 포함한 요청을 보냄
- 서버가 업그레이드를 지원하지 않을 수 있기 때문에 우선 OPTIONS 요청으로 업그레이드 가능 여부 확인

```
OPTIONS * HTTP/1.1
Host: example.com
Upgrade: TLS/1.0 // 업그레이드할 프로토콜
Connection: Upgrade
```

- 업그레이드가 가능하면 서버는 아래와 같이 응답

```
HTTP/1.1 101 Switching Protocols
Upgrade: TLS/1.0, HTTP/1.1
Connection: Upgrade
```

- 이후 업그레이드 요청

```
GET http://example.com/123 HTTP/1.1
Host: example.com
Upgrade: TLS/1.0
Connection: Upgrade
```

#### 서버 쪽에서 업그레이드를 요청

- 서버에서 TLS로 갱신을 요청하는 경우 Status Code 426으로 응답
- 이 경우 클라이언트에서 다시 프로토콜 변경을 요청해야 handshake가 이루어짐

```
HTTP/1.1 426 Upgrade Required
Upgrade: TLS/1.0, HTTP/1.1
Connection: Upgrade
```

#### TLS 업그레이드의 문제점

- 프록시가 악의적으로 정보를 훔치거나 다른 의도로 서버에 요청을 보내는 공격에 약하다는 단점이 있음
- `클라이언트-프록시` 사이에서 TLS로 통신하더라도 그 밖의 통신 경로도 암호화되어 있는지 알 수 없음
  - 프록시가 클라이언트의 정보를 읽을 수 있게 됨
- TLS를 사용하려면 클라이언트-최종 서버 간의 전체 통신 경로를 암호화 해야 함
  - TLS 통신으로 업그레이드 (리다이렉트)
  - HTTP Strict Transport Security (HSTS)

### 4.6 가상 호스트 지원

- HTTP/1.0에서는 한 대의 웹 서버로 하나의 도메인만 다루는 것이 전제였음
- HTTP/1.1부터는 하나의 웹 서버로 여러 웹 서비스를 운영할 수 있게 됨

### 4.7 청크

- HTTP/1.1부터 지원하는 새로운 데이터 표현으로 전체를 한번에 전송하지 않고 **작게 나눠서** 전송하는 방식
- 시간이 오래 걸리는 데이터 전송을 앞당길 수 있음
- `스트리밍 다운로드/업로드` 라고 부르기도 함
- 브라우저에서는 청크 전송 불가능
  - 자바스크립트 단에서 파일을 나눠 업로드하는 방법이 있지만 표준은 아님

**전송 방식**

- Transfer-Encoding 헤더에 `chunked`가 설정되어 있으면 Content-Length 헤더를 포함하면 안 됨
- 총 데이터의 크기는 지정된 크기의 합계
- 마지막으로 `0`을 보내면 청크 전송이 끝났다는 신호

#### 메시지 끝에 헤더 추가

- 청크 방식으로 전송하는 경우 메시지 끝에 헤더 추가 가능
```
Trailer: Content-Type
```
- 여기서부터의 헤더는 바디를 보낸 후 전송된다는 것을 알려줌

### 4.8 바디 전송 확인

### 4.9 마치며