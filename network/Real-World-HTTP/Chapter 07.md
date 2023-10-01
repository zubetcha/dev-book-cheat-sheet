# Chapter 07 - HTTP/2의 신택스: 프로토콜 재정의

## 7.1 HTTP/2

- 데이터 표현이 HTTP/1.1과는 크게 달라짐
  - 스트림(1.1의 파이프라인에 가까움)을 사용해 바이너리 데이터를 다중으로 송수신하는 구조로 변경
  - 스트림 내 우선 순위 설정과 서버 사이드에서ㅜ 데이터 통신을 하는 서버 사이트 푸시 구현
  - 헤더 압축
- HTTP가 제공하는 기본 4개 요소는 변경되지 않음
  - 메서드, 헤더, 스테이터스, 바디
  - 사용하는 쪽(클라이언ㅌ, 서버)을 제외한 통신 레벨에서는 큰 변경이 없음

**프로토콜 개선사항**

- 캐시(max-age): 통신 자체를 취소
- 캐시(ETag, Date): 변경이 없으면 바디를 취소
- Keep-Alive: 액세스마다 연결에 걸리는 시간(1.5TTL)을 줄임
- 압축: 응답 바디 크기 절감
- 청크: 응답 전송 시작을 빠르게 함
- 파이프라이닝: 통신 다중화

### 7.1.1 스트림을 이용한 통신 고속화

- 텍스트 기반 프로토콜에서 바이너리 기반 프로토콜로 변화
- HTTP/1.1까지는 하나의 요청이 TCP 소켓을 독점
  - 하나의 오리지널 서버에 대해 2~6개의 TCP 접속을 해서 병렬화
- HTTP/2에서는 하나의 TCP 접속 안에 스트림이라는 가상의 TCP 소켓을 만들어 통신

**스트림**

- 프레임에 따른 플래그로 간단히 만들고 닫을 수 있는 규칙으로 구성
- 일반 TCP 소켓과 같은 핸드셰이크는 불필요
- 따라서 ID 값과 TCP 통신 용량이 허락하는 한, 손쉽게 몇 만번의 접속이라도 병렬화 가능

<br/>

**공통 헤더**

| 요소              | 크기                 | 의미                                               |
| ----------------- | -------------------- | -------------------------------------------------- |
| Length            | 24                   | 페이로드 크기 (공통 헤더는 제외)                   |
| Type              | 8                    | 프레임 종류                                        |
| Flags             | 8                    |                                                    |
| R                 | 1                    | 예약 영역 (항상 0)                                 |
| Stream Identifier | 31                   | 스트림 식별자. 같은 값이면 같은 스트림 관련 프레임 |
| Frame Payroad     | Length로 지정한 길이 | 프레임의 실제 데이터                               |

Stream Identifier

- 같은 Stream Identifier를 가진 프레임은 수신 시에 그룹화됨
- 또한 같은 스트림에서 나온 데이터로 취급됨
- 0은 예약되어 있기 때문에 사용하면 에러 발생
- 홀수는 클라이언트 > 서버, 짝수는 서버 > 클라이언트로의 통신에 사용됨

<br/>

**프레임 종류**

| 종류          | 데이터                                        | 선택적 데이터                          | 설명                                                                 |
| ------------- | --------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------- |
| HEADERS       | 헤더                                          | 의존하는 스트림과 우선도, 배타 플래그  | 압축된 헤더. 우선도는 최초 헤더만 사용 가능                          |
| DATA          | 데이터                                        |                                        | 바디의 송신에서 사용                                                 |
| PRIORITY      | 의존하는 스트림, 우선도, 배타 플래그          |                                        |                                                                      |
| RST_STREAM    | 오류 코드                                     |                                        | 오류 정보를 반환하고, 스트림을 바로 종료                             |
| SETTINGS      | 식별자(16비트), 설정값(32비트)의 조가 여러 개 |                                        |                                                                      |
| PUSH_PROMISE  | 스트림 ID                                     | 요청 헤더 필드                         | 서버 푸시 시작 예약                                                  |
| PING          | 8바이트 데이터                                |                                        | 응답 속도 측정용 프레임 <br/> PING을 받으면 ACK 플래그를 설정해 반환 |
| GOAWAY        | 최종 스트림 ID, 오류 코드                     | 추가 디버그 정보                       | 커넥션을 종료                                                        |
| WINDOW_UPDATE | 윈도우 크기                                   | 추가로 수신할 수 있는 데이터 크기      |                                                                      |
| CONTINUATION  |                                               | HEADERS/PUSH_PROMISE에 이어지는 데이터 |                                                                      |

- SETTINGS에서 변경 가능한 설정
  - 헤더 테이블 크기
  - 푸시 허가
  - 최대 병렬 스트림 수
  - 초기 윈도우 크기
  - 최대 프레임 크기
  - 최대 헤더 리스트 크기

<br/>

### 7.1.2 HTTP/2의 애플리케이션 계층

- HTTP/1.0: 단순히 데이터를 운반하는 상자 역할
- HTTP/1.1: 텍스트 프로토콜, 하나의 요청 중 다른 요청 처리 불가
- HTTP/2.0: 바이너리화, 서로 독립적으로 분리되어 있는 프레임 구조

### 7.1.3 플로 컨트롤

> 플로 컨트롤  
> 스트림을 효율적으로 흐르도록 하는 통신량 제어 처리로,  
> 서로 통신 속도 차이가 많이 나는 기기 사이에 빠른 쪽이 느린 쪽으로 대량의 패킷을 전송해 처리할 수 없게 되는 사태 방지 목적으로 이용

- HTTP/2는 인터넷(TCP/IP) 4계층 모델 중 어플리케이션 레이어에 해당하지만 내부는 트랜스포트 레이어에 가까움
  - 플로 컨트롤로 TCP 소켓과 거의 같은 기능 구현
- 구체적으로 윈도우 크기를 관리함으로써 제어
  - 패킷을 보내는 쪽은 상대방 윈도우의 최대 버퍼 크기만큼까지 데이터 전송
  - 패킷을 받는 쪽은 버퍼에 여유가 생기면 `WINDOW_UPDATE` 프레임을 이용해 여유 버퍼 크기를 전송한 쪽에 반환
- `SETTINGS` 프레임을 사용하면 초기 윈도우 크키, 최대 병렬 스크림 수, 최대 프레임 크기, 최대 헤더 리스트 크기 등과 같은 속도와 관련된 매개변수 조정 가능

### 7.1.4 서버 푸시

- HTTP/2부터 서버 푸시를 이용해 우선 순위가 높은 콘텐츠를 클라이언트가 요구하기 전 전송할 수 있게 됨
- 웹소켓과 같은 양방향 통신은 아님
- css, 자바스크립트, 이미지 등 웹페이지를 구성하는 파일 다운로드 용도로 이용
- 서버가 푸시한 데이터는 캐시에 들어가며, 이후 클라이언트가 요청 시 즉시 다운로드된 것 처럼 보이도록 함

### 7.1.5 HPACK을 이용한 헤더 압축

- 헤더는 HPACK이라는 방식으로 압축됨
- 대부분의 데이터 압축 알고리즘은 딕셔너리와 딕셔너리 키 배열이라는 두 가지 데이터를 생성
  - 같은 길이의 문장이 많을수록 딕셔너리의 항목은 적어짐
  - 같은 키가 많이 사용될수록 압축률은 올라감
- HPACK은 일반 압축 알고리즘과 달리 사전에 사전을 가지고 있음
  - HTTP/2에서는 정적 테이블(static table)이라는 이름으로 딕셔너리에 빈번하게 출현하는 헤더 이름과 값을 테이블로 가지고 있음
  - 같은 커넥션에서 등장한 HTTP 헤더는 인덱스화되어 동적 테이블에 저장 후, 다시 등장할 때 인덱스 값만으로 표현

### 7.1.6 SPDY와 QUIC

#### SPDY

- SPDY(스피디)는 구글이 개발한 HTTP 대체 프로토콜로, 거의 그대로 HTTP/2가 됨
- 개발 목적은 HTTP가 개선해 온 전송 속도를 더욱 향상시키기 위함

#### QUIC

- HTTP와 같은 층인 TCP 소켓상에 구현됐지만 구글은 속도를 더 빠르게 하기 위해 UDP 소켓상에 QUIC(퀵)이라는 프로토콜을 준비
  - `TCP`는 재전송 처리, 폭주 제어, 순서 정렬, 에러 정정 등의 고급 기능으로 속도가 다소 느림
  - `UDP`는 위와 같은 고급 기능을 제거 해 가볍게 경량화한 프로토콜
- TCP의 고급 기능은 QUIC에 자체적으로 구현

<br/>

## 7.2 Fetch API

- XMLHTTPRequest와 마찬가지로 서버에 액세스하는 함수
- 자바스크립트에서 사용

#### 특징

- XMLHTTPRequest보다 오리진 서버 밖으로의 액세스 등 CORS 제어 용이
- 자바스크립트의 모던한 비동기 처리 방식인 Promise를 따름
- 캐시 제어 가능
- 리다이렉트 제어 가능
- referrer 정책 설정 가능
- Service Worker 내에서 이용 가능
- 보안 제한
  - 송수신 시 제한되는 헤더 존재
  - same origin 정책 엄격하게 적용
  - 브라우저에서 ssh로 외부 서버 연결 불가능
  - Git 프로토콜 전송 불가능
  - 웹 서버 개발 불가능

### 7.1.2 Fetch API의 기본

#### fetch 함수 예제

```javascript
fetch('news.json', {
  method: 'GET',
  mode: 'cors',
  credentials: 'include',
  cache: 'default',
  headers: {
    'Content-Type': 'application/json',
  },
})
  .then((response) => {
    return response.json();
  })
  .then((json) => {
    console.log(json);
  });
```

- fetch() 함수 호출
- fetch 함수의 두 번째 인자는 옵션 객체
- 첫 번째 then 함수에 서버로부터 응답이 온 후 호출되는 콜백 함수 전달
  - 응답의 헤더 부근까지 읽기를 마친 시점에 호출됨
  - 바디를 어떤 데이터 형식으로 가져올지 메서드 호출로 결정
- 두 번째 then 함수에는 시간이 걸리는 처리를 작성하고, 그 처리가 Promise를 반환할 경우 다시 then 연결

<br/>

#### Fetch API가 지원하는 데이터 형식

| 메서드        | 형식        | 설명                                                                                            |
| :------------ | :---------- | :---------------------------------------------------------------------------------------------- |
| arrayBuffer() | ArrayBuffer | 고정 길이 바이너리 데이터. Typed Array로 읽고 쓰기 가능                                         |
| blob()        | Blob        | 파일 콘텐츠를 나타내는 MIME 타입 + 바이너리 데이터. FileReader를 경유해 ArrayBuffer로 변환 가능 |
| formData()    | FormData    | HTML 폼과 호환되는 이름과 값의 쌍                                                               |
| json()        | Object      | JSON을 해석해 자바스크립트의 오브젝트, 배열 등으로 구성되는 오브젝트                            |
| text()        | string      | 문자열                                                                                          |

<br/>

#### Fetch API로 사용할 수 있는 메서드

- CORS 안전: GET, HEAD, POST
- 사용 불가: CONNECT, TRACE, TRACK

<br/>

#### Fetch API의 CORS 모드

- cors: 다른 오리진 서버로의 액세스 허용 (XHR 기본값 및 XHR에서는 모드 변경 불가)
- same-origin: 다른 오리진 서버로의 액세스를 오류로 취급
- no-cors: CORS 접속 무시 및 빈 응답으로 돌아옴 (Fetch API 기본값)

<br/>

#### Fetch API의 credentials

XMLHTTPRequest에서는 withCredential 프로퍼티에 true를 설정하면 include를 설정한 것과 동일하다.

- omit: 쿠키를 전송하지 않음 (Fetch API 기본값)
- same-origin: 출처가 같은 경우에만 쿠키 전송 (XHR 기본값)
- include: 쿠키 전송

### 7.2.2 Fetch API만 할 수 있는 것

#### 캐시 제어

- default: 표준 브라우저 동작에 따름 (기본값)
- no-store:
  - 캐시가 없는 것으로 간주하여 HTTP 요청
  - 결과도 캐시하지 않음
- reload:
  - 브라우저 새로고침과 같이 캐시가 없는 간주하여 HTTP 요청
  - Etag 등은 보내지 않음
  - 캐시가 가능하면 결과 캐시
- no-cache:
  - 기한 내의 캐시가 있어도 HTTP 요청 전송
  - 로컬 캐시의 Etag 등도 전송
  - 서버가 304를 반환하면 캐시한 콘텐츠 사용
- force-cache:
  - 기한이 지난 캐시라도 존재하면 사용
  - 캐시가 없으면 HTTP 요청 전송
- only-if-cached:

  - 기한이 지난 캐시라도 존재하면 사용
  - 캐시가 없으면 오류 발생

no-store, reload, no-cache 옵션은 캐시 상태와 관계 없이 반드시 HTTP를 요청하며, 반대로 force-cache와 if-only-cached는 적극적으로 캐시 사용한다.

#### 리다이렉트 제어

- follow: 최대 20 리다이렉트까지 리다이렉트를 따라감 (기본값)
- manual: 리다이렉트를 따라가지 않고 리다이렉트가 있다는 사실만 전달
- error: 네트워크 오류로 함

manual 지정 시 리다이렉트가 있으면, 응답 자체가 아니라 응답을 감싸 필터링된 결과를 응답으로서 반환한다. 이 응답은 type 속성에 `opaqueredirect`라는 문자열만 들어 있으며, 보안을 위해 이외의 정보는 필터링되어 얻을 수 없다. 바디는 null이며, 스테이터스는 0, 헤더는 없다.

#### Service Worker 대응

> Service Worker  
> 웹 서비스의 클라이언트와 서버 사이에서 동작하는 중간 레이어로,  
> 웹이 애플리케이션으로서의 기능성을 지닐 수 있도록 라이프사이클과 통신 내용을 제어할 수 있게 해주는 Web API

- 현재 Service Worker 내에서 외부 서비스로 접근할 때는 Fetch API만 사용 가능

<br/>

## 7.3 server-sent events

- server-sent events는 HTML5의 기능 중 하나
- 기술적으로는 HTTP/1.1의 청크 형식을 이용한 통신 기능을 바탕으로 함
  - 코멧의 롱 폴링 + 청크 응답 조합
  - 한 번의 클라이언트 요청에 대해 여러 이벤트 전송
  - 조금씩 전송한다는 특징을 응용해, 서버에서 임의의 시점에 클라이언트로 이벤트 전달
- 청크 방식을 사용하지만 HTTP 위에 `이벤트 스트림`이라고 불리는 별도의 텍스트 프로토콜을 실었음
  - 이벤트 스트림 MIME 타입: text/event-stream
- 자바스크립트에서는 EventSource 클래스를 사용해 server-sent events에 접근

<br/>

#### text/event-stream 예제

- 텍스트 문자 인코딩 방식: UTF-8
- 데이터는 태그 이름 뒤에 기술
- 빈 줄로 데이터 구분
- data 태그가 연속으로 전송되는 곳은 줄바꿈이 포함된 하나의 data로 처리

```text
id: 10
event: ping
data: {"time": 2016-12-26T15:52:01+0000}

id: 11
data: Message From PySpa
data: #eng channel
```

<br/>

#### 이벤트 스트림 태그 종류

- id: 이벤트를 식별하는 ID로, 재전송 처리에서 사용
- event: 이벤트 이름 설정
- data: 이벤트와 함께 보낼 데이터
- retry: 재접속 대기 시간 (단위: ms)

<br/>

#### 자바스크립트 EventSource 예제

```javascript
const eventSource = new EventSource('example.php');

eventSource.onmessage = (e) => {
  const newElement = document.createElement('li');

  newElement.innerHTML = 'message: ' + e.data;
  eventList.appendChilde(newElement);
};

eventSource.addEventListener('pint', (e) => {
  const newElement = document.createElement('li');
  const obj = JSON.parse(e.data);

  newElement.innerHTML = 'pint at ' + obj.time;
  eventList.appendChild(newElement);
});
```

- onmessage 이벤트 핸들러는 이벤트 태그가 없는 data 태그 메시지 수신 시 콜백 함수 호출
- addEventListener는 이벤트 이름을 지정해 콜백 함수 등록하여 특정 이벤트 태그가 붙은 메시지만 다룸
- 클라이언트는 메시지의 ID를 저장하고 재접속 시에는 마지막으로 수신한 ID를 `Last-Event-ID` 헤더에 설정하여 서버로 전송
- 서버는 클라이언트가 Last-Event-ID 헤더에 설정되어 있는 메시지까지 수신한 것으로 판단해 이후 이벤트만을 클라이언트로 전송

<br/>

## 7.4 웹소켓

- 서버/클라이언트 사이에 오버헤드가 적은 양방향 통신 목적으로 사용
- 통신이 확립되면 서버/클라이언트 사이에서 일대일 통신 수행
- 목적지가 정해져 있으므로 바디에 해당하는 2~14 Byte에 해당하는 데이터만 프레임 단위로 송수신

<br/>

### 7.4.1 웹소켓은 스테이트풀

- HTTP 기반 프로토콜과 달리 `스테이트풀`한 통신
- 예를 들어 채팅 같은 경우 대화방 단위로 커넥션을 온 메모리로 관리
  - 이 경우 접속이 끊어졌을 때 재접속하는 경우 이전과 같은 서버로 연결 필요
  - 단순한 HTTP 기반 로드 밸런서는 사용 불가

<br/>

### 7.4.2 자바스크립트의 클라이언트 API

- 웹소켓은 HTTP의 하위 레이어인 TCP 소켓에 가까운 기능을 제공하는 API
- 자바스크립트의 API도 TCP 소켓의 API에 가까운 형태로 되어 있음
- 통신은 서버가 수신을 기다리는 상태에서 클라이언트 쪽에서 먼저 접속
  1. 서버가 특정 IP 주소, 포트 번호로 시작 (Listen)
  2. 클라이언트(브라우저)가 서버에게 통신 시작 선언 (Connect)
  3. 클라이언트가 보낸 접속 요청을 서버가 받아들임 (Accept)
  4. 서버에는 소켓 클래스의 인스턴스가 넘어옴
  5. 서버가 받아서 처리하면 클라이언트의 소켓 인스턴스는 송신 기능, 수신 기능이 활성화됨

<br/>

#### 자바스크립트 예제

- WebSocket 클래스의 생성자로 접속할 URL 지정
- open: 소켓 연결 이벤트 핸들러
- send(): 서버로 데이터 전송
- onmessage: 서버로부터 받는 데이터 수신 이벤트 핸들러
- close(): 소켓 닫기

```javascript
var socket = new WebSocket('ws://game.example.com:12010/updates');

socket.open = () => {
  setIntervel(() => {
    if (socket.bufferedAmount === 0) {
      socket.send(getUpdateData());
    }
  }, 50);
};
```

<br/>

### 7.4.3 접속

- 웹소켓은 프로토콜 업그레이드를 사용
  - 일반 HTTP로 시작한 후 그 안에서 프로토콜을 업그레이드해 웹소켓으로 전환

#### 웹소켓 통신 시작 요청 예제

```shell
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: sfkjnsfnksndkf
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

- Sec-WebSocket-Key: 랜덤하게 생성한 16바이트 값을 base64로 인코딩한 문자열
- Sec-WebSocket-Version: 현 시점에서는 13으로 고정
- Sec-WebSocket-Protocol:
  - 옵션
  - 웹소켓은 단순히 소켓 통신 기능만 제공하기 때문에 그중 어떤 형식을 사용할지 애플리켕션에서 결정
  - 복수의 프로토콜 선택 가능

#### 웹소켓 통신 시작 서버 응답 예제

```shell
HTTP/1.1101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: ksdjnfkjsdnfknskd
Sec-WebSocket-Protocol: chat
```

- Sec-WebSocket-Accept: Sec-WebSocket-Key를 정해진 규칙으로 변환한 문자열로, 이를 통해 클라이언트는 서버와의 통신 확립 검증 가능
- Sec-WebSocket-Protocol:
  - 클라이언트로부터 서브 프로토콜 목록을 받았을 때 하나를 선택해서 반환
  - 클라이언트가 보낸 프로토콜 이외의 다른 프로토콜을 응답을 받으면 클라이언트는 접속을 거부해야 함

<br/>

### 7.4.4 Socket.IO

- 웹소켓을 좀 더 쉽게 사용할 수 있게 해주는 라이브러리

#### Socket.IO의 장점

- 웹소켓을 사용할 수 없을 때는 XMLHttpRequest에 의한 롱 폴링으로 에뮬레이션해, 서버에서의 송신을 실현하는 기능 탑재
- 웹소켓 단절 시 자동으로 재접속 시도
- 클라이언트뿐만 아니라 서버에서 사용할 수 있는 구현도 있어, 클라이언트가 기대하는 절차로 폴백인 XMLHttpRequest 통신 핸들링 가능
- 로비 기능