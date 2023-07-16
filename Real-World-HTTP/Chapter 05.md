# Chapter 05 - HTTP/1.1의 시맨틱스: 확장되는 HTTP의 용도

- 파일 다운로드 (파일명 지정)
- 다운로드 중단과 재개 (범위 액세스)
- XMLHttpRequest
- 지오로케이션
- X-Powered-By
- 원격 프로시저 호출
- WebDAV
- 웹사이트 간 공통 인증, 허가 플랫폼

<br/>

### 5.1 파일 다운로드 후 로컬에 저장하기

- 브라우저가 파일을 어떻게 처리할 지 결정하는 것
  - 파일 확장자 X
  - 서버가 보낸 MIME 타입
- 서버 응답에 `Content-Disposition` 헤더가 있는 경우
  ```
  Content-Disposition: attachment; filename=filename.xlsx
  ```
  - 브라우저는 다운로드 대화상자를 표시하고 파일 저장
  - `filename`으로 지정된 파일명이 다운로드 대화상자에 기본값으로 표시됨
- 한글 파일명도 가능
  ```
  Content-Disposition: attachment; filename*=utf-8'' 파일명.xlsx; filename=filename.xlsx
  ```
  - UTF-8 인코딩 지정
- `Content-Disposition` 헤더를 이용한 다운로드 기능은 HTTP를 위해 만들어진 것은 아님
  - `이메일의 첨부파일`을 위해 정의된 규격

<br/>

### 5.2 다운로드 중단과 재시작

- 파일 크기가 크면 다운로드에 걸리는 시간도 길어지고, 통신이 불안정해지거나 다운로드가 도중에 실패할 확률 증가
- HTTP는 다운로드가 중단되면 중단 지점부터 다시 시작하는 방법 제공
  - 커다란 파일에서 지정한 범위를 잘라내 다운로드
  - 서버가 범위 지정 다운로드를 지원하는 경우 `Accept-Ranges` 헤더를 응답에 설정함
  ```
  Accept-Ranges: bytes
  ```

`범위 지정 다운로드`

- Accept-Ranges 헤더는 두 가지 값을 가질 수 있음
  - Accept-Ranges: bytes - 범위 지정 다운로드를 지원하며 단위는 바이트이다.
  - Accept-Ranges: none - 범위 지정 다운로드 지원 불가
- 다른 단위도 받을 수는 있으나 IANA에 단위를 등록해야 하며, 현재 등록되어 있는 단위는 `bytes`와 `none`뿐임
- 클라이언트는 요청에 `Range` 헤더를 추가해 원하는 범위 지정 가능
  ```
  Range: bytes=1000-1999
  ```
- 서버는 범위가 지정된 컨텐츠를 반환하며 다음과 같이 응답함
  - Status Code `206`
  ```
  HTTP/1.1 206 Partial Content
  :
  Content-Length: 1000 // 실제로 보낸 바이트 수
  Content-Ranges: 1000-1999/5000 // 범위/전체 바이트 수
  ```
- 클라이언트가 지정한 범위가 무효인 경우
  - Status Code `416`
  ```
  HTTP/1.1 416 Range Not Satisfiable
  :
  Content-Ranges: */5000
  ```

#### 복수 범위 다운로드

- `Ranges` 헤더를 사용해 복수의 범위도 지정 가능
  ```
  Range: bytes=500-999,7000-7999
  ```
- 이 경우 multipart 폼과 비슷한 `mutlipart/byteranges`라는 Content-Type 헤더로 응답

#### 병렬 다운로드

- 영역을 나눠 세션마다 Range 헤더를 이용해 HTTP 접속하여 세션별로 다운로드한 조각을 나중에 결합하는 방식
- 병렬 다운로드는 서버에 부하를 줄 수 있기 때문에 권장하지는 않음

### 5.3 XMLttpRequest
- curl 커맨드 기능을 자바스크립트로 사용할 수 있게 해주는 기능
- 처음에는 마이크로소프트의 IE5 용으로 설계되었지만 현재는 `WHATWG(Web Hypertext Application Working Group)`에서 사양이 정해져, 각종 브라우저에서 사용할 수 있게 됨 
- HTTP처럼 `클라이언트가 서버로` 요청을 보내는 것에는 변함이 없으나 `서버가 클라이언트로` 요청을 보내는 것은 불가능

```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/json", true);
xhr.onload = function () {
  // 응답이 돌아왔을 때 호출되는 메서드
  if (xhr.status === 200) {
    // JSON 파싱해서 표시
      console.log(JSON.parse(xhr.responseText));
  }
};
xhr.setRequestHeader("MyHeader", "HeaderValue")
xhr.send(JSON.stringify({"message":  "hello world"}));
```

- open() 메서드로 도착지와 메서드를 설정하며, 세 번째 인자를 `true`로 설정하면 `비동기`로 실행됨
- send() 메서드로 실제 전송 시작

#### XMLRequestHTTP와 브라우저의 HTTP(폼) 요청 차이

- 송수신할 때 HTML 화면이 새로고침되지 않음
- GET, POST 이외의 메서드도 전송 가능
- 폼의 경우 `key-value` 쌍의 데이터만 전송할 수 있지만, XMLHttpRequest는 `플레인 텍스트`, `JSON`, `바이너리 텍스트` 등 다양한 형식의 데이터 송수신 가능
- 몇 가지 보안상 제약이 있음

`Ajax`
- asynchronous Javascript + XML의 약자
- 화면을 지우지 않고 웹페이지를 읽어 오거나 시간이나 타이밍에 따라 몇 번이고 갱신할 수 있는 아키텍쳐
- 브라우저가 송수신하면 파일 다운로드를 제외한 나머지 서버 응답을 받을 때 화면이 지워지고 새로운 페이지가 렌더링됨
- XMLHttpRequest는 `자바스크립트`가 송수신을 담당하므로 화면이 지워지지 않음

#### 코멧

- `XMLHttpRequest`를 이용해 거의 실시간 `양방향 통신`할 수 있는 기능
- 단방향 통신을 이용해 양방향 통신을 하기 위한 두 가지 방법
  - `폴링`
  - `롱 폴링`
  
[long polling](pubnub.com/blog/http-long-polling/)

`폴링`
- 클라이언트가 서버 쪽으로 빈번하게 요청하는 방식
- 불필요한 요청과 응답 발생
  - 대역과 CPU 낭비
  - 모바일 환경이라면 소비 전력 낭비

<p align="center">
  <img src="https://images.ctfassets.net/3prze68gbwl1/20KLOkp9uPZUiZtjp5VpWn/e4e6f4e48f9f0129766611ead3da8fa6/HTTP_Polling.png" alt="폴링" />
</p>


`롱 폴링`
- 클라이언트가 서버로 요청을 보내면 서버는 바로 응답하지 않고 응답을 보류한 채 대기
- 서버가 통신을 종료하거나 요청이 타임아웃 될 때까지 응답이 돌아오지 않고 자유로운 타이밍에 서버 응답
- HTTP는 서버에서 클라이언트로 요청을 보내는 전용 API가 아님
- 또한 HTTP는 쿠키를 포함한 대량의 헤더를 송수신에 포함하는 구조
- 서버에서 클라이언트로 연속으로 응답을 해야 하는 케이스에는 약한 편

<p align="center">
  <img src="https://images.ctfassets.net/3prze68gbwl1/2FEB0j6VRCXSe28P5ruiSZ/a149b8fdbbdb77e5f3bb81aab9902dac/HTTP_Long_Polling.png" alt="롱 폴링" />
</p>



#### XMLHttpRequest의 보안

- `접근 가능한 정보 제한`과 `전송 제한`이라는 두 가지 제한으로 구성

`접근 가능한 정보 제한`

- 쿠키
  - 스크립트로 `document.cookie` 를 통해 브라우저의 쿠키 접근 가능
  - 이를 방지하기 위해 쿠키 속성 중 `httpOnly`를 설정하면 스크립트로 쿠키 접근 불가

`전송 제한`
- 도메인
  - `동일 출처 정책(same origin policy)`으로 요청을 보낼 수 있는 도메인 제한 설정
  - `교차 출처 리소스 공유(CORS, cross-origin resource sharing)` 액세스 제한 시스템
- 메서드
  - `CONNECT`, `TRACE`, `TRACK`을 메서드로 지정하면 open() 메서드 호출 시 `SecurityError` 예외 응답
- 헤더
  - 현재 금지되어 있는 것
    - 현재의 프로토콜 규약이나 환경에 영향을 미치는 것
    - 쿠키처럼 보안에 영향을 주는 것
    - 브라우저의 능력을 넘을 수 없을 것
      - 브라우저 자신이 지원하지 않는 압축 형식을 Accept-Encoding으로 지정하는 행위 등
  - 앞으로 사용될 것에 대비해 `Sec-`, `Proxy-`로 시작하는 키 이름도 금지


### 5.4 지오로케이션

#### 클라이언트 자신이 위치를 구하는 방법
- 모던 브라우저는 지오로케이션 API 제공
  - 모바일: 내장된 GPS나 기지국 정보를 활용
  - 컴퓨터: 와이파이 등을 이용해 대략적인 위치 축측


`와이파이에서 위치 정보를 알아내는 방식`
- 클라이언트는 OS의 API를 이용해 현재 자신이 접근 가능한 액세스 포인트의 BSSID 획득
- BSSID로 서버에 문의해 위도 및 경도 조회


> BSSID는 SSID와는 다른 정보로, 와이파이 기기의 식별자의 48비트 수치   
> 기기마다 독특한 수치로 되어 있으며, 맥 주소와 같은 것


#### 서버가 클라이언트 위치를 추측하는 방법
- 지오 IP라고 불리는 IP 주소로 추측하는 방법
- IP 주소는 지역마다 등록 관리 기관이 있어, 기업이나 프로바이더 등에 IP 주소 할당
- 등록 기관이 정확한 장소까지 관리하는 것은 아님

`지오로케이션 API 장단점`
- 장점: 서비스 제공자 입장에서 사용자의 양해를 구하지 않아도 정보를 구할 수 있음
- 단점: 
  - GPS보다 정확도 떨어짐
  - 클라이언트 입장에선 프록시 등을 이용하지 않는 한 숨길 수 없음

### 5.5 X-Powered-By 헤더

- 서버가 브라우저에 응답할 때 부여하는 헤더 중 하나
- 서버에서 시스템 이름을 반환하는 목적으로 사용
- `X-`가 붙어있기 때문에 RFC 규격에는 없지만 사실상 많은 서버에서 사용하고 있어 표준에 가까움

`보안`
- OS 버전 등 서버 이름 이외의 불필요한 정보가 들어가서는 안됨
- 서버 개발자는 헤더의 표시와 숨김을 전환할 수 있게 구현하는 것이 좋음

### 5.6 원격 프로시저 호출

> 프로시저(prodcedure)란?   
> 각 프로그래밍 언어가 제공하는 함수, 클래스 메서드(정적 메서드)와 같은 것을 뜻함

- 원격 프로시저 호출(RPC, Remote Procedure Call)은 다른 컴퓨터에 있는 기능을 마치 자신의 컴퓨터에 있는 것처럼 호출하고, 필요에 따라 반환 값을 받는 구조
- 원격 메서드 호출(RMI, Remote Method Invocation)이라고도 불림

#### 5.6.1 XML-RPC

- 최초로 규격화된 RPC
- HTTP/1.0
- POST 메서드 사용
- 파라미터와 반환값 모두 XML로 표현 -> Content-Type은 항상 text/xml
- Content-Length를 항상 명시해야 함

`XML-RPC 요청 예제`
```xml
POST /RPC2 HTTP/1.0
Host: betty.userland.com
Content-Type: text/xml
Content-length: 181

<?xml version="1.0"?>
<methodCall>
  <methodName>examples.getStateName</methodName>
  <params>
    <param>
      <value>
        <i4>41</i4>
      </value>
    </param>
  </params>
</methodCall>
```

`XML-RPC 응답 예제`
```xml
HTTP/1.1 200 OK
ConnectionL close
Content-Length: 158
Content-Type: text/xml

<?xml version="1.0"?>
<methodResponse>
  <params>
    <param>
      <value>
        <string>
          South Dakota
        </string>
      </value>
    </param>
  </params>
</methodResponse>
```

`XML-RPC 파라미터 데이터 타입`

| 태그명 | 데이터 타입 |
| :---: | :---: |
|i4, int| 정수 |
|boolean|테이블형|
|string|문자열형|
|double|부동소수정수형|
|dateTime.iso8601|날짜형|
|base64|BASE64 인코딩 바이너리|
|struct|구조체|
|array|배열|

#### 5.6.2 SOAP

- XML-RPC를 확장하여 만들어진 규격
- W3C에서 규격화
- 단순한 RPC였던 XML-RPC보다 복잡한 구조
- 메일 전송 프로토콜(SMTP)를 사용해 SOAP 메시지를 주고 받을 수 있음
- SOAP 메시지 구조
  - 헤더: 요청 메서드, 트랜잭션 정보 등
  - 엔벨로프: 데이터

<img src="https://t1.daumcdn.net/cfile/tistory/1311C6184C8DD6BA5E" alt="SOAP 메시지 구조" />

#### 5.6.3 JSON-RPC
- XML-RPCdml XML 대신 `JSON`을 이용한 원격 프로시져 호출
- 요청 시 필요한 헤더: Content-Type, Content-Length, Accept
- 메서드는 대부분 `POST`를 사용하나 GET도 사용 가능
- json 구조
  - jsonrpc: 버전 지정 목적, 필수
  - id: 요청과 응답을 매칭하기 위한 것으로, 숫자나 문자열 사용 가능
  - method: 메서드 이름
  - params: 파라미터로, 배열과 객체도 가능하며 생략도 가능
- 반환값은 입력에 사용한 id와 반환 값이 result 키에 저장되어 응답
- 요청에서 id를 생략하면 서버에서 응답을 돌려주지 않는 `Notification` 모드가 됨


`JSON-RPC 기본 예제`
```
POST /josnrpc HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 94
Accept: application/json

{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": {
    "subtrahend": 23,
    "minuend": 42
  },
  "id": 3,
}

HTTP/1.1 200OK
Content-Type: application/json
Content-Length: 41

{
  "jsonrpc": "2.0",
  "result": 19,
  "id": 3
}
```

- XML-RPC의 응답 status code는 항상 200이었던 것과 달리 JSON-RPC는 다른 status code도 응답
  - 200 OK: 정상 종료
  - 204 No Response / 202 Accepted: Notification 시의 반환값
  - 307 Temporary Redirect / 308 Permanent Redirect: 리디렉트
  - 405 Method Not Allowed: GET 메서드가 지원되지 않는 안전하지 않은 메서드에 GET 메서드 이용
  - 415 Unsupported Media Type: Content-Type이 application/json이 아님

### 5.7 WebDAV

- HTTP를 확장해 분산 파일 시스템으로 사용할 수 있게 한 기술
  - IE의 네트워크 위치 추가, Mac OS의 파인더, 리눅스 계열 OS의 그놈 등에 쉽게 연결
- 구글 드라이브, 원드라이브 등과 같은 온라인 스토리지 서비스와 다른 점
  - `동기`를 전제로 한다는 것
- Git에서 사용하는 전송용 프로토콜 중 하나인 `HTTPS` 내부에서 WebDAV 사용 
  - 다른 하나인 SSH는 내부에서 오리지널 Git 프로토콜 사용
  - 차이만 전송하므로 WebDAV보다 Git 프로토콜이 속도가 더 빠름


`용어 정리`
- 리소스: 데이터를 저장하는 아토믹 요소, 일반 파일 시스템에서는 `파일`로 부르지만 HTTP 용어 그대로 계승
- 컬렉션: `폴더`와 `디렉터리`에 해당하는 요소
- 프로퍼티: 리소스와 컬렉션이 가질 수 있는 추가 속성으로, 작성일시와 갱신일시, 최종 갱신자와 같은 정보에 해당
- 락: 동시에 같은 파일을 편집하려고 할 때 마지막에 전송된 내용 이외에는 지워져버리는 걸 방지하기 위해 먼저 선언한 사람 이외의 변경을 거절하는 시스템

`메서드`
- HTTP/1.1의 POST, GET, PUT, DELETE 메서드 외에 몇 가지 메서드가 추가됨
  - COPY / MOVE: 대용량의 컨텐츠 처리 목적
  - MKCOL: 컬렉션을 작성하는 메서드 (리소스 작성은 POST)
  - PROPFIND: 컬렉션 내의 요소 목록 획득
  - LOCK / UNLOCK: 파일 잠금 여부 제어

### 5.8 웹사이트 간 공통 인증 및 허가 플랫폼

| 용어 | 설명 |
| :---:  | :---:  |
| 인증<br/>(authentication) | - 로그인하려는 사용자가 누군지 확인<br/>- 브라우저를 조작하는 사람이 서비스에 등록된 어느 사용자 ID의 소유자인지 확인 |
| 권한 부여<br/>(authorication) | - 인증된 사용자가 누군지 확인 후 그 사용자에게 부여할 권한 범위 결정 |

#### 5.8.1 싱글 사인온 (SSO, single sign-on)
- 시스템 간 계정 관리를 따로 하지 않고, 한 번의 로그인을 전체 시스템에 유효하게 하는 기술
- 프로토콜이나 정해진 규칙이 아닌, 이런 용도로 사용되는 시스템을 가르키는 명칭
- 사용 예
  - 사용자 ID를 일원화해 관리
  - 서비스 앞단에 HTTP 프록시 서버를 두고 인증을 대행
  - 각 서비스에 인증을 대행하는 에이전트를 넣고 로그인 시 중앙 서버에 액세스 해 로그인됐는지 확인
  
#### 5.8.2 커베로스 인증

`LDAP(lightweight directory access protocol)`
- 사용지 관리 구조를 하나로 정리해 모든 시스템에서 이용하는 방법
  - ex) OpenLDAP, AD(active directory)
- 이용자, 조직, 서버 등 기업 내 정보를 일원화해 관리하는 데이터베이스로, v3에서 추가된 `SASL(simple authentication security layer)`이라는 인증 기능과 세트로 주로 기업 내 마스터 인증 시스템으로 사용
  - `커베로스` 인증이 널리 사용됨

`커베로스 인증`
- 커베로스 인증시 티켓 보증 티켓(TGT, ticket granting ticket)과 세션 키 획득
- 서비스와 시스템 사용시
  - TGT와 세션 키를 티켓 보증 서버로 전송
  - 클라이언트가 서버로 액세스 하기 위해 필요한 티켓과 세션 키 획득
  - 클라이언트가 서비스를 사용할 때 티켓과 세션 키를 함께 전송함으로써 싱글 사인온 실현

#### 5.8.3 SAML(security assertion markup language)
- 웹 계통 기술(HTTP/SOAP)을 전제로 한 싱글 사인온 구조
- XML 기반 표준을 많이 다루는 OASIS에서 책정된 규격
- 도메인을 넘어선 서비스 간 통합 인증 가능
- 서비스 간 정보 교환 메타데이터도 공통화 되어 있음

`용어`
- 사용자: 브라우저를 조작하는 사람
- 인증 프로바이더(IdP): ID를 관리하는 서비스
- 서비스 프로바이더(SP): 로그인이 필요한 서비스

`구현 방법`
- SAML SOAP 바인딩
- 리소스 SOAP(PAOS) 바인딩
- HTTP 리디렉트 바인딩
- HTTP POST 바인딩
- HTTP 아티팩트 바인딩
- SAML URI 바인딩

<br/>
- 메타데이터로 불리는 XML 준비
  - 서비스ID (인증 프로바이더가 서비스르 식별하기 위함)
  - 인증 프로바이더가 HTTP-POST할 엔드포인트 URL
  - 바인딩
  - 경우에 따라서는 X.509 형식의 공개 키
- 인증 프로바이더에 서비스 정보 등록
- 서비스 프로바이더에 인증 프로바이더 정보 등록
  - 등록할 정보는 인증 프로바이더가 XML 파일로 제공
  - 통신에 사용할 엔드포인트 URL 목록과 인증서 포함


#### 5.8.4 오픈아이디
- 중앙 집중형 ID 관리를 하지 않고, 이미 등록된 웹 서비스의 사용자 정보로 다른 서비스에 로그인할 수 있는 시스템

`용어`
- 오픈아이디 프로바이더(OpenID Provider): 사용자 정보를 가진 웹서비스로, 사용자는 이미 이 서비스의 ID가 있다.
- 릴레일 파티(Relying Party): 사용자가 새로 이용하고 싶은 웹 서비스
- 사용자 입력 식별자: 사용자가 입력하라 URL 형식으로 된 문자열로, 오픈아이디 프로바이더가 제공하며 오픈아이디 프로바이더의 사용자 프로필 화면 등에 표시됨


#### 5.8.5 오픈소셜

- 플랫폼을 지향하며 다양한 기능 지원
  - Person&Friend API: 회원 정보나 친구 관계 가져옴
  - Activities API: 액티비티 작성
  - Persistence API: 정보를 젖아하거나 공유
  - requestSendMessage: 다른 멤버에세 메시지 전송
- 소셜 네트워크 서비스 제공자에서 인증과 권한 관리

#### 5.8.6 OAuth

- 인증이 아닌 권한을 부여하는 시스템으로서 개발됨


`용어`
- 권한 부여 서버: 
  - 오픈아이디에서 말하는 오픈아이디 프로바이더
  - 사용자는 이 권한 부여 서버에 계정이 있음
- 리소스 서버:
  - 사용자가 허가한 권한으로 자유롭게 접근할 수 있는 대상
- 클라이언트:
  - 오픈아이디에서 말하는 릴레잉 파티
  - 사용자가 사용할 서비스나 애플리케이션
  - 오픈아이디와 달리 권한 부여 서버에 애플리케이션 정보를 등록하고 전용 ID(credential)를 획득해야 함

`OAuth 2.0 플로`

- Authorization Code:
  - 웹 서비스의 서버 내에 client_secret을 감출 수 있고, 외부에서 볼 가능성이 없는 경우에 사용
- Implicit Grant
  - client_secret 없이 액세스할 수 있는 패턴
  - 클라이언트 신원을 보증하지 않음
  - client_secret을 안전하게 유지할 수 없는 자바스크립트, 사용자 단말에 애플리케이션 코드를 다운로드하는 스마트폰 앱 등을 위한 방법
- Resource Owner Password Credentials Grant
  - 클라이언트 자신이 사용자의 ID와 패스워드에 접근
  - 허가 서버가 신뢰하는 클라이언트에서 사용
- Client Credentials Grant
  - 사용자 동의 없이 client_id와 client_secret만으로 액세스하는 방법

#### 5.8.7 오픈아이디 커넥트

- `OAuth 2.0`을 기반으로 한 `권한` 부여뿐만 아니라 `인증`으로 사용해도 문제가 없도록 확장한 규격
- `OAuth 2.0`과 달리 사용자 프로필에 액세스 하는 방법을 규격화
- 일반적인 액세스 토큰과 별개로 ID 토큰 발행
- 토큰 획득을 위해 2개의 엔드포인트와 2개의 플로 정의

`엔드포인트`
- 권한 부여 엔드포인트:
  - 클라이언트가 권한 부여 요청을 보낼 서비스 창구
  - 클라이언트 인증 플로에서는 토큰 엔드포인트에 액세스하기 위한 키(권한 부여 코드)를 반환
  - 인증하지 않는 플로에서는 `액세스 토큰`과 `ID 토큰`은 이 엔드포인트가 반환
- 토큰 엔드포인트:
  - `액세스 토큰`과 `ID 토큰`을 반환하는 창구
  - 클라이언트를 인증해 강한 권한을 가진 토큰 반환

`플로`
- Authorization Code Flow:
  - client_secret을 은닉할 수 있는 서버 환경용
  - `권한 부여 앤드포인트`에 접근해서 권한 부여 코드를 가져온 후, `토큰 엔드포인트`에 접근해 액세스 토큰 획득
- Implicit Flow:
  - HTML 상의 자바스크립트 등 client_secret을 은닉할 수 없는 클라이언트 환경용
  - `권한 부여 엔드포인트`에 접근해 코드와 토큰을 한 번에 가져옴
- Hybrid Flow:
  - `권한 부여 엔드포인트`에서 통신에 필요한 토큰과 추가 정보를 얻기 위한 권함 부여 코드 획득
  - 권한 부여 코드를 사용해 토큰 앤드포인트에 접근

